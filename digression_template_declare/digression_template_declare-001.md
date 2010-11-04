!SLIDE transition=fade smbullets

## Digression:  Template::Declare features ##

!SLIDE smaller code

## Helper methods ##

    @@@ perl
    sub dot {
        return span { class is 'dot'; '.'; };
    };

    # Now you can call dot anywhere in your template
    # to insert such a span.

!SLIDE smaller code

## Wrapper methods ##

    @@@ perl
    # Define a wrapper.
    BEGIN {
        create_wrapper wrap => sub {
            my ($code, $c, %p) = @_;

            … header stuff …
            div { id is 'cnt';
                $code->();
            };
          … footer stuff …
        };
    }

    # Use the wrapper.
    template foo => sub {
        my ($self, $c) = @_;
        wrap { h1 {'Hello, world!'}; } $c;
    };

!SLIDE incremental smbullets

## Catalyst::View::TD advantages ##

* Good for programmers:
* Less typing.
* Bad XHTML tags = bad Perl method calls.

!SLIDE small code

## Catalyst::View::TD advantages ##

### No worries about closing tags improperly: ###

    @@@ html
    <div>blah<div> <!-- Oops! -->

!SLIDE incremental smbullets

## Catalyst::View::TD disadvantages ##

* Not good if you have dedicated XHTML/CSS designers.
* They won't appreciate the Perl syntax.
