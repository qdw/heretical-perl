!SLIDE

# our $method; #

!SLIDE transition=fade smaller code

## The controller ##

    @@@ perl
    sub compare :Path('/compare') {
      my ($self, $c, $v1, $v2) = @_;
      my $q = $c->req->params->{'q'} // '';
      my $conn = $c->conn; # a DBIx::Connector object
    
      …

      # The sth is what the view sees
      $c->stash->{fixes_sth} = $conn->run(
        fixup => sub {
          my $sth = shift->prepare(q{
                 SELECT * FROM get_fixes(?, ?, ?, ?)
          });
          $sth->execute($major_1, $minor_1, $minor_2, $q);
          return $sth;
        }
      );
    }

!SLIDE incremental bullets

# Controller discussion #

* So just return an sth ready for fetchrow_*whatever* calls.
* The view will unpack results by calling $sth->fetchrow_arrayref();
* In effect, the sth(s) *are* the interface between view and controller.

!SLIDE incremental bullets

# Model #

* “The database *is* the model!” (Wheeler)
* Actually, SQL is the model

!SLIDE transition=fade incremental smbullets

# Model = SQL #

* Specifically, we access the DB via stored procs only
* Separates interface and implementation.
* Keeps the interface small and simple.
* Uses the full power of SQL in the implementation.
* The implementation remains flexible (e.g. you can change table structure if needed).
* Yet the interface remains constant (keeps its contract with the user).

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

## In the controller:  #

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
