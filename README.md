# Tests for HTTP Caches

This is a test suite for the behaviours of [HTTP caches](http://httpwg.org/specs/rfc7234.html),
including browsers, proxy caches and CDNs. Its public results are available at
[cache-tests.fyi](https://cache-tests.fyi).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Goals](#goals)
- [Installation](#installation)
  - [Installing from NPM](#installing-from-npm)
- [Running the Test Server](#running-the-test-server)
- [Testing Reverse Proxies and CDNs](#testing-reverse-proxies-and-cdns)
  - [Testing from the Command Line](#testing-from-the-command-line)
- [Testing Browser Caches](#testing-browser-caches)
- [Interpreting the Results](#interpreting-the-results)
  - [Test Results FAQ](#test-results-faq)
- [Getting your results onto cache-tests.fyi](#getting-your-results-onto-cache-testsfyi)
- [Test Format](#test-format)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Goals

Overall, the goal of these tests is to identify variances in the behaviour, both from the normative specifications and between implementations. This in turn can help avoid situations where they act in surprising ways.

The underlying aim is to provide a basis for discussion about how HTTP caches -- especially in CDNs and reverse proxies -- should behave, so that over time we can adapt the tests and align implementations to behave more consistently.

In other words, **passing all of the tests currently means nothing** -- this is not a conformance test suite, it's just the start of a conversation, and a **tool to assess how a cache behaves**.

Therefore, if you believe a test should change (based upon common behaviour or your interpretation of the specifications), or have additional tests, please [contribute](CONTRIBUTING.md).


## Installation

The tests require a recent version of [NodeJS](https://nodejs.org/) (10.8.0 or greater), which includes the `npm` package manager.

To install the most recent source from GitHub (*recommended; things are moving fast*):

> git clone https://github.com/http-tests/cache-tests.git

and then install dependencies:

> cd cache-tests; npm i

### Installing from NPM

Alternatively, for the most recent release:

> npm i --legacy-bundling http-cache-tests


## Running the Test Server

First, start the server-side by running:

> npm run server

inside the directory (the repository's directory if you cloned from git, or `node_modules/http-cache-tests` if you installed from npm).

By default, the server runs on port 8000; to choose a different port, use the `--port` argument; e.g.,

> npm run server --port=8080

If you want to run an HTTPS origin, you'll need to specify the `protocol`, `keyfile` and `certfile`:

> npm run server --protocol=https --keyfile=/path/to/key.pem --certfile=/path-to-cert.pem

Note that the default port for HTTPS is still 8000.

Make sure that the browser is not configured to use a proxy cache, and that the network being tested upon does not use an intercepting proxy cache.


## Testing Reverse Proxies and CDNs

To test an reverse proxy or CDN, configure it to use the server as the origin and point a browser to `https://{hostname}/test-cdn.html`.

Note that they only work reliably on Chrome for the time being; see [this bug](https://github.com/whatwg/fetch/issues/722).


### Testing from the Command Line

To test a reverse proxy or CDN from the command line::

> npm run --silent cli --base=http://server-url.example.org:8000/

... using the URL of the server you want to test. This will output the test results in JSON to STDOUT, suitable for inclusion in the `results` directory. See `summary.mjs` for details of how to interpret that.

To run a single test, use:

> npm run cli --base=http://server-url.example.org:8000/ --id=test-id

... where `test-id` is the identifier for the test. This will output the request and response headers as seen by the client and server, along with the results. This is useful for debugging a particular failure.


## Testing Browser Caches

To test a browser, just point it at `https://{hostname:port}/test-browser.html` after setting up the server.


## Interpreting the Results

HTTP caching by its nature is an optimisation; implementations aren't required to cache everything. However, when they do cache, their behaviour is constrained by [the specification](https://httpwg.org/specs/rfc7234.html).

As a result, there are a few different kinds of test results:

* ✅ - The test was successful.
* ⛔️ - The test failed, and likely indicates a specification conformance problem.
* ⚠️ - The cache didn't behave in an optimal fashion (usually, it didn't use a stored response when it could have), but this is not a conformance problem.
* ● / ○ - These are tests to see how deployed caches behave; we use them to gather information for future specification work. "yes" and "no" respectively.

Some additional results might pop up from time to time:

* ⁉️ - The test harness failed; this is an internal error, please [file a bug if one doesn't exist](https://github.com/http-tests/cache-tests/issues/).
* 🔹 - The test failed during setup; something interfered with the harness's communication between the client and server. See below.
* ⚪️ - Another test that this test depends on has failed; we use dependencies to help assure that we're actually testing the behaviour in question.
* `-` - Not tested; usually because the test isn't applicable to this cache.

When you're testing with a browser, each test has a `uuid` that identifies that specific test run; this can be used to find its requests in the browser developer tools or proxy logs. Click ⚙︎ to copy it to the clipboard.


### Test Results FAQ

If you see a lot of failures, it might be one of a few different issues:

* If you see lots of grey circles at the top (dependency failures), it's probably because the cache will store and reuse a response without explicit freshness or a validator. While this is technically legal in HTTP, it interferes with the tests. Disabling "default caching" or similar usually fixes this.

* If you see lots of blue diamonds (setup failures), it's likely that the cache is refusing `PUT` requests. Enable them to clear this; the tests use PUT to synchronise state between the client and the server.


## Getting your results onto cache-tests.fyi

[cache-tests.fyi](https://cache-tests.fyi) collects results from caches in browsers, reverse proxies, and CDNs. Its purpose is to gather information about how HTTP caching works "in the wild", to help the [HTTP Working Group](https://httpwg.org) make decisions about how to evolve the specification.

If your implementation isn't listed and you want it to be, please file an issue, or contact [Mark Nottingham](mailto:mnot@mnot.net). Both open source and proprietary implementations are welcome; if there are commercial concerns about disclosing your results, your identity can be anonymised (e.g., "CDN A"), and will not be disclosed to anyone.

Right now, all of the reverse proxy and CDN implementations are run by a script on a server, using the command-line client; to keep results up-to-date as the tests evolve, it's most helpful if you can provide an endpoint to test (for reverse proxies and CDNs).


## Test Format

Each test run gets its own URL, randomized content, and operates independently.

Tests are kept in JavaScript files in `tests/`, each file representing a suite.

A suite is an object with a `name` member, `id` member, a `tests` member, and an optional `description` member that can contain Markdown; e.g.,

```javascript
export default {
  name: 'Example Tests',
  id: 'example',
  description: 'These are the `Foo` tests!'
  tests: [ ... ]
}
```

The `tests` member is an array of objects, with the following members:

- `name` - A concise description of the test. Can contain Markdown. Required.
- `id` - A short, stable identifier for the test. Required.
- `description` - Longer details of the test. Optional.
- `kind` - One of:
  - `required` - This is a conformance test for a requirement in the standard. Default.
  - `optimal` - This test is to see if the cache behaves optimally.
  - `check` - This test is gathering information about cache behaviour.
- `requests` - a list of request objects (see below).
- `browser_only` - if `true`, will not run on non-browser caches. Default `false`.
- `browser_skip` - if `true, will not run on browser caches. Default `false`.
- `depends_on` - a list of test IDs that, when one fails, indicates that this test's results are not useful. Currently limited to test IDs in the same suite. Optional.

Possible members of a request object:

- `template` - A template object for the request, by name -- see `templates.js`.
- `request_method` - A string containing the HTTP method to be used.
- `request_headers` - An array of `[header_name_string, header_value_string]` arrays to
                    emit in the request.
- `request_body` - A string to use as the request body.
- `query_arg` - query arguments to add.
- `filename` - filename to use.
- `mode` - The mode string to pass to `fetch()`.
- `credentials` - The credentials string to pass to `fetch()`.
- `cache` - The cache string to pass to `fetch()`.
- `pause_after` - Boolean controlling a 3-second pause after the request completes.
- `magic_locations` - Boolean; if `true`, the `Location` and `Content-Location` headers will be rewritten to full URLs.
- `response_status` - A `[number, string]` array containing the HTTP status code
                    and phrase to return from the origin.
- `response_headers` - An array of `[header_name_string, header_value_string]` arrays to
                     emit in the origin response. These values will also be checked like
                     expected_response_headers, unless there is a third value that is
                     `false`.
- `response_body` - String to send as the response body from the origin. If not set, it will contain
                  the test identifier.
- `expected_type` - One of:
  - `cached`: The response is served from cache
  - `not_cached`: The response is not served from cache; it comes from the origin
  - `lm_validate`: The response comes from cache, but was validated on the origin with Last-Modified
  - `etag_validate`: The response comes from cache, but was validated on the origin with an ETag
- `expected_status` - A numeric HTTP status code; checked on the client.
                    If not set, the value of `response_status[0]` will be used; if that
                    is not set, 200 will be used.
- `expected_request_headers` - An array of `[header_name_string, header_value_string]` representing
                              headers to check the request for on the server, or an array of
                              strings representing header names to check for presence in the
                              request.
- `expected_response_headers` - An array of `[header_name_string, header_value_string]` representing
                              headers to check the response for on the client, or an array of
                              strings representing header names to check for presence in the
                              response. See also response_headers.
- `expected_response_headers_missing` - An array of `header_name_string` representing headers to
                                      check that the response on the client does not include.
- `expected_response_text` - A string to check the response body against on the client.
- `setup` - Boolean to indicate whether this is a setup request; failures don't mean the actual test failed.
- `setup_tests` - Array of values that indicate whether the specified check is part of setup;
  failures don't mean the actual test failed. One of: `["expected_type", "expected_status",
  "expected_response_headers", "expected_response_text", "expected_type",
  "expected_request_headers"]`

`server.js` stashes an entry containing observed headers for each request it receives. When the
test fetches have run, this state is retrieved and the expected_* lists are checked, including
their length.

