!SLIDE transition=fade

# Testing #

!SLIDE incremental smbullets

## pgTAP ##

* A TAP-emiting test harness written as PL/pgSQL functions
* Another gem from David Wheeler
* He gave [a talk about it](https://www.postgresqlconference.org/content/test-driven-database-development) Tuesday
* Just the quickest taste, in case you missed it:

!SLIDE incremental smbullets

## pgTAP usage ##

* SELECT plan( 49 );
* SELECT has_function( 'public', 'major_versions', '{}'::name[] );
* SELECT results_eq( 'have', 'want', 'major_versions() should return expected query values' );
* … and much more

!SLIDE incremental smbullets

## Test::XPath ##

* Test whether XPath predicates are true of your XHTML
* Good for picking out subparts of the doc
* Somewhat awkward for validating overall structure of the doc

!SLIDE smaller code

## Test::XPath usage ##

    @@@ perl
    $tx->is( 'count(/html/body)', 1, 'Should have 1 body element' );

    # Test the body.
    $tx->is('count(/html/body/*)', 2, 'Should have two elements below body' );

    $tx->is(
        '/html/head/title',
        "PostgreSQL Experts’ PGVersionCompare (TEST): $title",
        'Title should be correct'
    );

!SLIDE incremental smbullets

## Test::XPath usage ##

* Can use anon subs to operate on subsections of the document…
* i.e. grab the matching section first, then operate on it.

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

!SLIDE incremental smbullets

## Test::XPath complement? ##

* I'd like to write a Test::RNC to test against a schema
* More concise, declarative
* Maybe next Drakecon…

!SLIDE transition=fade incremental smbullets
        
# Testing:  Module::Build::DB #

* Another Wheeler module
* Facilitates testing
* Idea:  your database is a fixture, i.e. part of your test suite
* Different DB contents for different environments

!SLIDE small code

## Module::Build::DB usage ##

    @@@ yaml
    --- conf/dev.yml 
    name: PostgreSQL Experts’ PGVersionCompare (DEV)
      dbi:
      dsn: dbi:Pg:dbname=version_compare_dev
      user: postgres
      pass: ''

!SLIDE small code
      
@# Module::Build::DB usage ##

    @@@ sh
    $ perl Build.pm
    $ ./Build --context test # or dev, or your_id
    $ ./Build db # sets up fixture; populates from migrations in sql/
    $ ./Build test # runs tests against it
    $ ./Build $MODULE_BUILD_COMMAND
