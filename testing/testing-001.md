!SLIDE transition=fade

# Testing #

!SLIDE incremental bullets

# pgTAP #

* A tap-emiting test harness written as SQL stored procs
* Another gem from David Wheeler
* He's giving a talk about it at PostgreSQL West

!SLIDE incremental smbullets

# pgTAP usage #

* SELECT plan( 49 );
* SELECT has_function( 'public', 'major_versions', '{}'::name[] );
* SELECT results_eq( 'have', 'want', 'major_versions() should return expected query values' );
* PREPARE want AS VALUES ('8.0'), ('8.1'), ('8.2'), ('8.3'), ('8.4'), ('8.5'), ('9.0');
* SELECT results_eq( 'have', 'want', 'major_versions() should return actual values' );
* … and much more

!SLIDE incremental smbullets

# Test::XPath #

* Test whether XPath predicates are true of your XHTML
* Good for picking out subparts of the doc
* Not so good for validating overall structure of the doc
* E.g. not good for saying "this element has 1 to n children"

    $tx->is( 'count(/html/body)', 1, 'Should have 1 body element' );

!SLIDE smaller code

# Test::XPath usage #

    @@@ perl
    $tx->is( 'count(/html/body)', 1, 'Should have 1 body element' );

    # Test the body.
    $tx->is('count(/html/body/*)', 2, 'Should have two elements below body' );

    $tx->is(
        '/html/head/title',
        "PostgreSQL Experts’ PGVersionCompare (TEST): $title",
        'Title should be correct'
    );

!SLIDE incremental bullets

# Test::XPath usage #

* Can use anon subs to operate on subsections of doc
* = grab matching section first, then search in it

!SLIDE smaller code

    @@@ perl
    $tx->ok( '/html/body/div[@id="ccn"]', sub {
        shift->ok('./div[@id="con"]', sub {
            $_->is(
              'count(./*)',
              2,
              q(Should have 2 elements below "con"),
            );
            …
        }
    }

!SLIDE incremental bullets

# Test::XPath complement? #

* I'd like to write a Test::RNC to test schemas
* More concise, declarative
* No tuits right now…

!SLIDE transition=fade incremental bullets
        
# Module::Build::DB #

* Yet another awesome David Wheeler module
* Idea:  your database is a fixture, i.e. part of your test suite
* Different DB contents for different environments

!SLIDE small code

# Module::Build::DB usage #

    @@@ yaml
    --- conf/dev.yml 
    name: PostgreSQL Experts’ PGVersionCompare (DEV)
      dbi:
      dsn: dbi:Pg:dbname=version_compare_dev
      user: postgres
      pass: ''

!SLIDE small code
      
# Module::Build::DB usage #

    @@@ sh
    $ perl Build.pm
    $ ./Build --context test # or dev, or your_id
    $ ./Build db # sets up fixture
    $ ./Build test
    $ ./Build $MODULE_BUILD_COMMAND
