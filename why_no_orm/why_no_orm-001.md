!SLIDE smbullets incremental

# Why no ORM? #

### Performance Problems ###

* Most-heard complaint when we do query-tuning for a client: "Some queries are too slow, but we don't know where they're coming from."

* Mapping SQL queries back to ORM calls takes some work, especially on production systems.

* In essence, you have to do the job of the ORM backwards, in heels.

* Second-most-heard complaint:  "Some of our queries were too slow, so we
  rewrote them in raw SQL."

* So why not write them in SQL in the first place?

!SLIDE smbullets incremental
  
# How ORMs degrade performance #

* The worst fetch an entire resultset at once, rather than lazy-loading

* (think $sth->fetchall_arrayref() versus $sth->fetchrow_arrayref() )

* Some do really dumb things, like JOINs or FK enforcement on the client side

* … often because they want to support MyISAM or SQLite

* ORMs encourage you to fetch the entire object, not just the columns you need

!SLIDE smaller code

## ORMs encourage bad transaction management ##

    @@@ perl
    my $address = My::DB::Object::Address->new({
        street_address => '1112 E Broad St',
        city  => 'Westfield',
        state => 'NJ',
        zip   => '07090',
    });
    $address->save();
    
    my $order = My::DB::Object::Order->new({
        customer_name => 'Gomez Addams',
        shipping_address => $address
    });
    order->save();

!SLIDE code

## This is equivalent to ##

    @@@ sql
    BEGIN;
    INSERT INTO Address VALUES (…);
    COMMIT;
    
    BEGIN;
    INSERT INTO Order VALUES (…);
    COMMIT;

!SLIDE

## … even though there's a foreign-key relationship.  Ack! ##

!SLIDE small code

## ORMs can make JSON a pain. ##

### Naive approach: ###

    @@@ perl
    # Note use of aliased
    use aliased 'My::DB::Object::User::Manager' => 'UserM';

    sub get_users_json :Local {
        my ($self, $c) = @_;
    	
        my $users = UserM->get_users(…);
    	    
        $c->stash(
            # Doesn't work!
            # The JSON serializer barfs.
            users => $users,
        },
    	
        $c->forward('View::JSON');
    }

!SLIDE

## (Aside:  note use of aliased.  Very handy for saving typing.) ##

!SLIDE smaller code

## Working JSON approach: ##

    @@@ perl
    use aliased 'My::DB::Object::User::Manager' => 'UserM';

    sub get_users_json :Local {
        my ($self, $c) = @_;
    
        my $users = UserM->get_users(…);

        # Unpack the user object manually.
        my @user_hashes;
        for my $user (@{ $users }) {
            my $birth_string = $user->birthdate->mdy('/');
            my %user_hash = (
                birthdate  => $birth_string,
                last_name  => $user->last_name(),
                first_name => $user->first_name(),
            );
            push @user_hashes, \%user_hash;
        }
    
        $c->stash(
            json => {
                users => \@user_hashes,
            },
        );
    	
        $c->forward('View::JSON');
    }

!SLIDE smaller code

### Compare the DBI approach: ###

    @@@ perl
    …
    my $users_ref = $dbh->selectall_hashref(
        'SELECT birthdate, last_name, first_name'
        . ' FROM users'
        . ' WHERE …'
    );
    $c->stash(
            json => {
                users => $users_ref
            },
    );

!SLIDE smbullets incremental

## Is there hope in Moose? ##

* In theory, you could unpack using Moose introspection.

* This  doesn't work for all ORMs, though, as not all are written in Moose.

!SLIDE transition=fade

# Why no ORM? #

## Some queries are very hard to translate into ORM-ese. ##

!SLIDE smaller code

## Queries that use aggregates are impossible in most ORMs. ##

    @@@ sql
    -- Find duplicate indexes in a database
    CREATE AGGREGATE array_accum (anyelement)
    (
        sfunc = array_append,
        stype = anyarray,
        initcond = '{}'
    );

    SELECT indrelid::regclass,
         , array_accum(indexrelid::regclass)
    FROM pg_index
    GROUP BY indrelid
           , indkey
    HAVING COUNT(*) > 1;

!SLIDE smaller code

## Custom aggregates are impossible in *all* ORMs. ##

    @@@ sql
    -- From Bricolage
    CREATE   FUNCTION append_id(TEXT, INTEGER)
    RETURNS  TEXT AS '
       SELECT CASE WHEN $2 = 0 THEN
                   $1
              ELSE
                   $1 || '' '' || CAST($2 AS TEXT)
              END;'
    LANGUAGE 'sql'
    WITH     (ISCACHABLE, ISSTRICT);
    
    CREATE AGGREGATE group_concat (
       SFUNC    = append_id,
       BASETYPE = INTEGER,
       STYPE    = TEXT,
       INITCOND = ''
    );
    
    …

!SLIDE smaller code

## Want to try this in an ORM? ##

    @@@sql
    -- Find bloated tables, from largest to smallest.
    SELECT
      schemaname, tablename, 
      ROUND(CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,
      CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,
      CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,
      CASE WHEN relpages < otta THEN pg_size_pretty(0) ELSE pg_size_pretty((bs*(relpages-otta))::bigint) END AS wastedsize,
      iname, ituples::bigint, ipages::bigint, iotta,
      ROUND(CASE WHEN iotta=0 OR ipages=0 THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,
      CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,
      CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,
      CASE WHEN ipages < iotta THEN pg_size_pretty(0) ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize
    FROM (
      SELECT
        schemaname, tablename, cc.reltuples, cc.relpages, bs,
        CEIL((cc.reltuples*((datahdr+ma-
          (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) AS otta,
        COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,
        COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta -- very rough approximation, assumes all cols
      FROM (
        SELECT
          ma,bs,schemaname,tablename,
          (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,
          (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2
        FROM (
          SELECT
            schemaname, tablename, hdr, ma, bs,
            SUM((1-null_frac)*avg_width) AS datawidth,
            MAX(null_frac) AS maxfracsum,
            hdr+(
              SELECT 1+count(*)/8
              FROM pg_stats s2
              WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename
            ) AS nullhdr
          FROM pg_stats s, (
            SELECT
              (SELECT current_setting('block_size')::numeric) AS bs,
              CASE WHEN substring(v,12,3) IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,
              CASE WHEN v ~ 'mingw32' THEN 8 ELSE 4 END AS ma
            FROM (SELECT version() AS v) AS foo
          ) AS constants
          GROUP BY 1,2,3,4,5
        ) AS foo
      ) AS rs
      JOIN pg_class cc ON cc.relname = rs.tablename
      JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname AND nn.nspname <> 'information_schema'
      LEFT JOIN pg_index i ON indrelid = cc.oid
      LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid
    ) AS sml
    WHERE sml.relpages - otta > 0 OR ipages - iotta > 10
    ORDER BY wastedbytes DESC;
