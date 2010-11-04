!SLIDE smbullets incremental

# Why no ORM? #

## Performance Problems ##

* Most-heard complaint when we do query-tuning for a client: "Some queries are too slow, but we don't know where they're coming from."

* Mapping SQL queries back to ORM calls takes some work, especially on production systems.

* In essence, you have to do the job of the ORM backwards, in heels.

!SLIDE smbullets incremental

## Performance problems ##

* Second-most-heard complaint:  "Some of our queries were too slow, so we
  rewrote them in raw SQL."

* So why not write them in SQL in the first place?

!SLIDE smbullets incremental
  
## How ORMs degrade performance ##

* The worst fetch an entire resultset at once, rather than lazy-loading.

* (Think *$sth->fetchall_arrayref()* versus *$sth->fetchrow_arrayref()* .)

* Some do really dumb things, like JOINs or FK enforcement on the client side…

* … often because they want to support MyISAM or SQLite.

* ORMs encourage you to fetch whole rows, not just the columns you need.

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

!SLIDE smaller code

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

!SLIDE smaller code

## ORMs can make JSON a pain. ##

### Naive approach: ###

    @@@ perl
    # Note use of aliased
    use aliased 'My::DB::Object::User::Manager', 'UserM';

    sub get_users_json :Local {
        my ($self, $c) = @_;
    	
        my $users = UserM->get_users(…);
    	    
        $c->stash(
            # Doesn't work!  The JSON serializer barfs.
            users => $users,
        },
    	
        $c->forward('View::JSON');
    }

!SLIDE

## (Aside:  note the use of the aliased pragma—very handy for saving typing.) ##

!SLIDE smaller code

### Working JSON approach: ###

    @@@ perl
    use aliased 'My::DB::Object::User::Manager', 'UserM';

    sub get_users_json :Local {
        my ($self, $c) = @_;
    
        my $users = UserM->get_users(…);

        # Unpack the user object manually.  What a pain.
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
                # So easy.
                users => $users_ref,
            },
    );

!SLIDE smbullets incremental

## Is there hope in Moose? ##

* In theory, you could unpack using MOP introspection.

* This  doesn't work for all ORMs, though, as not all are written in Moose.

!SLIDE transition=fade

# Why no ORM? #

## Some types of queries are very hard to translate into ORM-ese. ##

!SLIDE smaller code

## Using aggregates is impossible in most ORMs. ##

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

!SLIDE smbullets incremental

## Sound esoteric?  Aggregates are common. ##

* Does your ORM support…
* max?
* min?
* count?
* count distinct?

!SLIDE smbullets incremental

## Other queries are possible in ORMs, but really hard. ##

* E.g. this [query to list your most bloated tables](http://quinnweaver.com/heretical-perl/table-bloat-query.sql): http://quinnweaver.com/heretical-perl/table-bloat-query.sql

!SLIDE smbullets incremental 

## Again, it is possible. ##

* But it would require a lot of contortions.
* Which raises the question:  why not SQL?

!SLIDE smbullets incremental

# SQL is a great DSL. #

* Compact
* Expressive
* Ideal for making queries against relational databases
* (Imagine that!)
* "Makes the easy things easy and the hard things possible."

!SLIDE smbullets incremental

# SQL and the Web #

* Raw SQL gets a bad rap…
* … because of the CGI.pm–era practice of putting SQL in your HTML.
* SQL + HTML + Perl ( + JS ) = a mess, to be sure.
* But MVCs give you a clean separation of concerns.
* Let's take a look…

!SLIDE transition=fade

# our $method; #
