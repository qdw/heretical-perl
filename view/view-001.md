!SLIDE transition=fade incremental bullets

# The view #

* We happen to be using Template::Declare and Catalyst::View::TD;
* It's like HAML for Perl.
* It's not for everyone, but we like it.

!SLIDE small code

# Template::Declare example #

    @@@ perl
    form {
        id is 'query';
        action is '/handle_form';
        method is 'post';
        
        p { 'Fixes from' };
        select {
            …
        };
        …
    };

!SLIDE incremental bullets

# Catalyst::View::TD paradigm #

* Tags are methods
* Attrs are properties:  attr is $value;
* You can define wrapper methods…

!SLIDE smaller code

## Concise wrapper and helper methods ##

    @@@ perl
    # Helper
    sub dot {
        return span { class is 'dot'; '.'; };
    };

    # Wrapper
    wrap {
        if (exists $c->{stash}->{error}) {
            p {
                class is 'error';
                $c->{stash}->{error};
            };
        };
        …
        $code->();
        …
    } $c;

!SLIDE incremental bullets

# Catalyst::View::TD advantages #

* Good for programmers:
* Less typing
* Bad XHTML tags = bad Perl method calls

!SLIDE small code

## No worries about closing tags properly ##

    @@@ html
    <div>blah<div> <!-- Oops! -->

!SLIDE incremental bullets

# Catalyst::View::TD disadvantages#

* Not good if you have dedicated XHTML/CSS designers
* They won't appreciate the Perl syntax

!SLIDE smaller code

## How the view interacts with the controller ##

    @@@ perl
    table {
        class is 'upgrade_warnings';
        row {
          th {'Upgrade warning'}; th {'For Installing Version'};
        };
        while (my ($version, $warning)
         = $upgrade_warnings_sth->fetchrow_array()) {
          row { cell {$warning}; cell {$version}; };
        }
    };

!SLIDE incremental bullets

### Want a fake OO approach? ###

* *$sth->fetchrow_hashref()* would work fine too

!SLIDE transition=fade bullets

## Goodies ##

* [The web app this talk is based on](http://pgexperts.com/)
* [Its source code](http://github.com/qdw/pg-version-compare)
* [Slides for this talk](http://heroku.com/)
* [showoff, the app I used to write this talk](http://github.com/schacon/showoff)

!SLIDE transition=fade incremental bullets

## Thanks! ##

* Feedback

* Questions?
