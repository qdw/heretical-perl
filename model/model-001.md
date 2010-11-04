!SLIDE incremental smbullets

# The model #

* “The database *is* the model!” —David E. Wheeler
* Actually, SQL is the model.
* Specifically, we access the DB via stored procs/functions only.

!SLIDE transition=fade incremental smbullets

## Advantages of a functions-based model ##

* It separates interface and implementation.
* It keeps the interface small and simple.
* It uses the full power of SQL in the implementation.
* The implementation remains flexible.
* … e.g. you can change table structure if needed.
* Yet the interface remains constant
* … i.e. keeps its contract with the user.

!SLIDE smaller code

## Model code example ##

    @@@ sql
    -- Return all minor version numbers
    -- (8.1.*1*, 8.1.*2* et cetera)
    CREATE FUNCTION minor_versions(
        major text
    ) RETURNS SETOF integer LANGUAGE sql
      STABLE STRICT AS $$
        SELECT minor
        FROM   versions
        WHERE  super
          = substring($1 from $x$(\d+)\.\d+$x$)::INT
        AND    major
          = substring($1 from $x$\d+\.(\d+)$x$)::INT
        ORDER BY minor;
    $$;

!SLIDE smaller code

## Controller–model interaction ##

    @@@ perl
    $c->stash->{minor_sth} = $conn->run(
        fixup => sub {
          my $sth = shift->prepare(q{
              SELECT * FROM minor_versions(?, ?);
          });
          $sth->execute($super, $major);
          return $sth;
        }
    );
