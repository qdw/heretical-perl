!SLIDE

# Digression:  DBIx::Connector #

!SLIDE incremental smbullets

## DBIx::Connector ##

* That's the $c->conn object from the last code sample.
* $_—or $_[0], or shift()—is the underlying dbh.
* Defined in MyApp.pm…

!SLIDE smaller code

    @@@perl
    has conn => (is => 'ro', default => sub {
        DBIx::Connector->new(
            @{ shift->config->{dbi} }{qw(dsn user pass)},
            {
              PrintError     => 0,
              RaiseError     => 0,
              HandleError    => Exception::Class::DBI->handler,
              AutoCommit     => 1,
              pg_enable_utf8 => 1,
            },
        );
    });

!SLIDE incremental smbullets

# Features #

* Does connection-pooling, in a non-Apache-dependent way
* Doesn't muck with DBI semantics like Apache::DBI
* Safe with fork() and threads
* Sophisticated transaction and ping support

!SLIDE incremental smbullets

# ping versus fixup #

* Run with or without pinging the DB to see if it's up first
* Normally it's up, so running in no ping mode saves time
* Or you can run in fixup mode…
* … in which case it tries, then retries on failure
* (Make sure your query is idempotent!)

!SLIDE smaller code

# Examples #

    @@@perl
    my $sth = $conn->run(fixup => sub {
        my $sth = $_->prepare('SELECT isbn, title, rating FROM books');
        $sth->execute;
        $sth;
    });

    my $sth2 = $conn->run(ping => sub {
        …
    });

!SLIDE code

# Transaction support #

    @@@ perl
    $conn->txn(sub {
        # Make a bunch of normal DBI calls
    });

!SLIDE smbullets

# Transaction support #

* If the transaction fails, DBIx::Connector rolls it all back.

!SLIDE incremental smbullets

# Transaction support #

* Also supports savepoints
* And fine-grained OO exception-handling
* For transaction or other failures
* … more than I have time for.  See DBIx::Connector on CPAN.
