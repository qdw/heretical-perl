!SLIDE transition=fade smaller code

# The controller #

    @@@ perl
    sub compare :Path('/compare') {
      my ($self, $c, $v1, $v2) = @_;
      my $q = $c->req->params->{'q'} // '';
      my $conn = $c->conn; # a DBIx::Connector object
      ##### […]
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

!SLIDE incremental smbullets

## Controller discussion ##

* So just return an sth ready for fetchrow_*whatever* calls.
* E.g. our view will unpack results by calling $sth->fetchrow_arrayref();
* In effect, the sth(s) *are* the interface between view and controller.
