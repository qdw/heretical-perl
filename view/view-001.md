!SLIDE transition=fade incremental smbullets

# The view #

* Unpacks results from the sth.
* We happen to be using Template::Declare and Catalyst::View::TD.
* It's like HAML for Perl.
* It's not for everyone, but it works for us:

!SLIDE smaller code

## View–controller interaction ##

    @@@ perl
    …
    table {
      class is 'upgrade_warnings';
      row {
        th {'Upgrade warning'}; th {'For Installing Version'};
      };
      while (my ($version, $warning)
          # The sth(s) in the stash are the interface between
          # the controller and the view.
          = $upgrade_warnings_sth->fetchrow_array()) {
        row { cell {$warning}; cell {$version}; };
      }
    };
    …

!SLIDE incremental smbullets

## Want an OO approach? ##

* $sth->fetchrow_hashref() would work fine too.
* Hashes = poor person's OO.

!SLIDE smaller code

## Controller–view interaction (the other direction) ##

    @@@ perl
    # Just as in any other Catalyst app:
    sub compare :Path('/compare') {

      # $v1 and $v2 are args from the URL (e.g. http://…/8.1.1/8.1.5)
      my ($self, $c, $v1, $v2) = @_;

      # $q is an arg from POST (<input name="q" … />)
      my $q = $c->req->params->{'q'} // '';
      …
    }

!SLIDE incremental smbullets

## Note Template::Declare paradigm ##

* Tags are methods.
* Attrs are properties:  *attr is $value;*
* Other features (will cover if time allows)…
