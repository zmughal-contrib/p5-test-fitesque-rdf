# NAME

Test::FITesque::RDF - Formulate Test::FITesque fixture tables in RDF

# SYNOPSIS

    my $suite = Test::FITesque::RDF->new(source => $file)->suite;
    $suite->run_tests;

See `t/integration-basic.t` for a full test script example.

# DESCRIPTION

This module enables the use of Resource Description Framework to
describe fixture tables. It will take the filename of an RDF file and
return a [Test::FITesque::Suite](https://metacpan.org/pod/Test::FITesque::Suite) object that can be used to run
tests.

The RDF serves to identify the implementation of certain fixtures, and
can also supply parameters that can be used by the tests, e.g. input
parameters or expectations. See [Test::FITesque](https://metacpan.org/pod/Test::FITesque) for more on how the
fixtures are implemented.

## ATTRIBUTES AND METHODS

This module implements the following attributes and methods:

- `source`

    Required attribute to the constructor. Takes a [Path::Tiny](https://metacpan.org/pod/Path::Tiny) object
    pointing to the RDF file containing the fixture tables. The value will
    be converted into an appropriate object, so a string can also be
    supplied.

- `suite`

    Will return a [Test::FITesque::Suite](https://metacpan.org/pod/Test::FITesque::Suite) object, based on the RDF data supplied to the constructor.

- `transform_rdf`

    Will return an arrayref containing tests in the structure used by
    [Test::FITesque::Test](https://metacpan.org/pod/Test::FITesque::Test). Most users will rather call the `suite`
    method than to call this method directly.

- `base_uri`

    A [IRI](https://metacpan.org/pod/IRI) to use in parsing the RDF fixture tables to resolve any relative URIs.

## RDF EXAMPLE

The below example starts with prefix declarations. Since this is a
pre-release, some of the prefixes are preliminary examples. Then, the
tests in the fixture table are listed explicitly. Only tests mentioned
using the `test:fixtures` predicate will be used. Tests may be an RDF
List, in which case, the tests will run in the specified sequence, if
not, no sequence may be assumed.

Then, two test fixtures are declared. The `test:handler` predicate is
used to identify the class containing implementations, while
`dc:identifier` is used to name the function within that class.

The `test:params` predicate is used to link the parameters that will
be sent as a hashref into the function.

There are two different mechanisms for passing parameters to the test
scripts, one is simply to pass arbitrary key-value pairs, the other is
to pass lists of HTTP request-response objects.

### Key-value parameters

The key of the hashref passed as arguments will be the local part of
the predicate used in the description (i.e. the part after the colon
in e.g. `my:all`). It is up to the test writer to mint the URIs of
the parameters, and the `param_base` is used to set indicate the
namespace, so that the local part can be resolved, if wanted. The
resolution itself happens in [URI::NamespaceMap](https://metacpan.org/pod/URI::NamespaceMap).

    @prefix test: <http://example.org/test-fixtures#> .
    @prefix deps: <http://ontologi.es/doap-deps#>.
    @prefix dc:   <http://purl.org/dc/terms/> .
    @prefix my:   <http://example.org/my-parameters#> .


    <#test-list> a test:FixtureTable ;
      test:fixtures <#test1>, <#test2> .

    <#test1> a test:Test ;
      test:handler "Internal::Fixture::Simple"^^deps:CpanId ;
      dc:identifier "string_found" ;
      test:param_base <http://example.org/my-parameters#> ;
      test:params [ my:all "counter-clockwise dahut" ] .

    <#test2> a test:Test ;
      test:handler "Internal::Fixture::Multi"^^deps:CpanId ;
      dc:identifier "multiplication" ;
      test:param_base <http://example.org/my-parameters#> ;
      test:params [
          my:factor1 6 ;
          my:factor2 7 ;
          my:product 42
      ] .

### HTTP request-response lists

To allow testing HTTP-based interfaces, this module also allows the
construction of two ordered lists, one with HTTP requests, the other
with HTTP responses. With those, the framework will construct
[HTTP::Request](https://metacpan.org/pod/HTTP::Request) and [HTTP::Response](https://metacpan.org/pod/HTTP::Response) objects respectively. In tests
scripts, the request objects will typically be passed to the
[LWP::UserAgent](https://metacpan.org/pod/LWP::UserAgent) as input, and then the response from the remote
server will be compared with the expected [HTTP::Response](https://metacpan.org/pod/HTTP::Response)s made by
the test fixture.

This gets more complex, please see the test data file
`t/data/http-list.ttl` file for example.

# TODO

Separate the implementation-specific details (such as `test:handler`)
from the actual fixture tables.

# BUGS

Please report any bugs to
[https://github.com/kjetilk/p5-test-fitesque-rdf/issues](https://github.com/kjetilk/p5-test-fitesque-rdf/issues).

# SEE ALSO

# AUTHOR

Kjetil Kjernsmo <kjetilk@cpan.org>.

# COPYRIGHT AND LICENCE

This software is Copyright (c) 2019 by Inrupt Inc.

This is free software, licensed under:

    The MIT (X11) License

# DISCLAIMER OF WARRANTIES

THIS PACKAGE IS PROVIDED "AS IS" AND WITHOUT ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF
MERCHANTIBILITY AND FITNESS FOR A PARTICULAR PURPOSE.
