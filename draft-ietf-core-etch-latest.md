---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-etch-latest
date: 2016-10-09
cat: std
pi:
  toc: yes
  tocompact: yes
  tocdepth: 3
  tocindent: yes
  symrefs: yes
  sortrefs: no
  comments: yes
  inline: yes
  compact: yes
  subcompact: no
  symrefs: yes
title: Patch and Fetch Methods for Constrained Application Protocol (CoAP)
abbrev: CoAP Fetch Patch
area: Applications
wg: core
kw: CoAP
author:
- ins: P. van der Stok
  name: Peter van der Stok
  org: Consultant
  email: consultancy@vanderstok.org
- ins: C. Bormann
  name: Carsten Bormann
  org: Universitaet Bremen TZI
  street: Postfach 330440
  city: Bremen
  code: D-28359
  country: Germany
  phone: "+49-421-218-63921"
  email: cabo@tzi.org
- ins: A. Sehgal
  name: Anuj Sehgal
  org: Consultant
  email: anuj@iurs.org
normative:
  RFC2119:
  RFC5789: http-patch
  RFC7231: http-semantics
  RFC7252: coap
  RFC7641: observe
  RFC7959: block
informative:
  RFC5323:
  RFC6902:
  RFC7159: json
  RFC7396:
  I-D.vanderstok-core-comi:
  I-D.hartke-core-apps:
  I-D.snell-search-method: search

--- abstract


The methods defined in RFC 7252 for the Constrained Application Protocol (CoAP)
only allow access to a complete resource, not to parts of a resource. In case of resources
with larger or complex
data, or in situations where a resource continuity is
required, replacing or requesting the whole resource is undesirable. Several
applications using CoAP will need to perform partial resource accesses.

This specification defines the new CoAP methods, FETCH, PATCH and
iPATCH, which are used to access and update parts of a resource.

--- middle

# Introduction {#intro}

Similar to HTTP, the GET method defined in
{{-coap}} for the Constrained Application Protocol (CoAP) only allows
the specification of a URI and request
parameters in CoAP options, not the transfer of a request payload
detailing the request.  This leads to some applications to using POST
where actually a cacheable, idempotent, safe request is desired.

Again similar to the original specification of HTTP,
the PUT method defined in {{-coap}} only allows to replace a complete resource. This also leads
applications to use POST where actually a cacheable, possibly idempotent
request is desired.

The present specification adds new CoAP methods: FETCH, to perform the
equivalent of a GET with a request body; and the twin methods PATCH and iPATCH, to
modify parts of a CoAP resource.

## FETCH {#intro-fetch}

The CoAP GET method {{-coap}} is used to obtain the representation of
a resource, where the resource is specified by a URI and additional
request parameters can additionally shape the representation.  This
has been modelled after the HTTP GET operation and the REST model in
general.

In HTTP, a resource is often used to search for information, and
existing systems varyingly use the HTTP GET and POST methods to
perform a search.  Often a POST method is used for the sole reason
that a larger set of parameters to the search can be supplied in the
request body than can
comfortably be transferred in the URI with a GET request.
The draft {{I-D.snell-search-method}} proposes a SEARCH method that is similar to
GET in most properties but enables sending a request body as with
POST.
The FETCH method defined in the present specification is inspired by
{{I-D.snell-search-method}}, which
updates the definition and semantics of the HTTP SEARCH request
method previously defined by {{RFC5323}}.  However, there is no intention
to limit FETCH to search-type operations, and the resulting
properties may not be the same as those of HTTP SEARCH.

A major problem with GET is that the information that controls the
request needs to be bundled up in some unspecified way into the URI.
Using the request body for this information has a number of
advantages:

* The client can specify a media type (and a content encoding),
  enabling the server to unambiguously interpret the request
  parameters in the context of that media type.  Also, the request
  body is not limited by the character set limitations of URIs,
  enabling a more natural (and more efficient) representation of
  certain domain-specific parameters.

* The request parameters are not limited by the maximum size of the
  URI.  In HTTP, that is a problem as the practical limit for this
  size varies.  In CoAP, another problem is that the block-wise
  transfer is not available for transferring large URI options in
  multiple rounds.

As an alternative to using GET, many implementations make use of the
POST method to perform extended requests, even if they are
semantically idempotent, safe, and even cacheable, to be able to pass
along the input parameters within the request payload as opposed to
using the request URI.

The FETCH method provides a solution that spans the gap between the
use of GET and POST.  As with POST, the input to the FETCH operation
is passed along within the payload of the request rather than as part
of the request URI.  Unlike POST, however the semantics of the FETCH
method are more specifically defined.


## PATCH and iPATCH {#intro-patch}

PATCH is also specified for HTTP in {{-http-patch}}. Most of the motivation for PATCH described
in {{-http-patch}} also applies here. iPATCH is the idempotent version of PATCH.

The PUT method exists to overwrite a resource with completely
new contents, and cannot be used to perform partial changes.
When using PUT for partial changes, proxies and caches, and
even clients and servers, may get confused as to the result of
the operation.
PATCH was not adopted in an early design stage of CoAP,
however, it has become necessary with the arrival of
applications that require partial updates to resources
(e.g. {{I-D.vanderstok-core-comi}}). Using PATCH
avoids transferring all data associated with a resource in
case of modifications, thereby not burdening the constrained
communication medium.

This document relies on knowledge of the PATCH specification
for HTTP {{-http-patch}}. This document provides
extracts from {{-http-patch}} to make independent
reading possible.


## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL",
"SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY",
and "OPTIONAL" in this document are to be interpreted as
described in {{RFC2119}}.


## Terminology and Acronyms

This document uses terminology defined in {{-http-patch}} and {{-coap}}.

Specifically, it uses the terms "safe" and "idempotent" as defined in Section 5.1 of {{-coap}}.
(Further discussion of safe and idempotent methods can now be found in Section
4.2.1 and 4.2.2 of {{-http-semantics}}, respectively; the implications of idempotency of
methods on server implementations are also discussed in Section 4.5 of
{{-coap}}.)

# FETCH Method {#fetch}

The CoAP FETCH method is used to obtain a representation of a
resource, giving a number of request parameters.  Unlike the CoAP GET
method, which requests that a server return a representation of the
resource identified by the effective request URI (as defined by {{-coap}}), the FETCH method is used by a client to ask the server to
produce a representation as described by the request parameters (including
the request options and the payload) based on the resource specified
by the effective request URI.  The payload returned in response to a
FETCH cannot be assumed to be a complete representation of the
resource identified by the effective request URI, i.e., it cannot be
used by a cache as a payload to be returned by a GET request.

Together with the request options, the body of the request (which may
be constructed from multiple payloads using the block protocol
{{-block}}) defines the request parameters.
With the
FETCH method, implementations may submit a request body of any media type that is
defined with the semantics of selecting information from
a resource in such a FETCH request; it is outside the scope of this document how
information about media types admissible for the specific resource is obtained by the client
(although we can hint that form relations ({{I-D.hartke-core-apps}})
might be a preferred way).

FETCH requests are both safe and idempotent with regards to the
resource identified by the request URI.  That is, the performance of
a fetch is not intended to alter the state of the targeted resource.
(However, while processing a fetch request, a server can be expected
to allocate computing and memory resources or even create additional
server resources through which the response to the search can be
retrieved.)

A successful response to a FETCH request is expected to provide some
indication as to the final disposition of the requested operation.
If a successful response includes a body payload, the payload is expected to
describe the results of the FETCH operation.

Depending on the response code as defined by {{-coap}}, the response
to a FETCH request is cacheable; the request body is part of the
cache key.  Specifically, 2.05 "Content" response codes, the
responses for which are cacheable, are a usual way to respond to a
FETCH request.  (Note that this aspect differs markedly from
{{I-D.snell-search-method}}.)  (Note also that caches that cannot use
the request payload as part of the cache key will not be able to
cache responses to FETCH requests at all.)  The Max-Age option in the
response has equivalent semantics to its use in a GET.

The semantics of the FETCH method change to a "conditional FETCH" if
the request message includes an If-Match, or If-None-Match option
({{-coap}}).  A conditional FETCH requests that the query be
performed only under the circumstances described by the conditional
option(s).  It is important to note, however, that such conditions
are evaluated against the state of the target resource itself as
opposed to the results of the FETCH operation.


## Response Codes {#fetch-response}

FETCH for CoAP adopts the response codes as specified in
sections 5.9 and 12.1.2 of {{-coap}} as well as additional
response codes mentioned in {{fetch-errors}}.

## Error Handling {#fetch-errors}

A FETCH request may fail under certain known conditions.
Beyond the conditions already defined in {{-coap}} for GET, noteworthy ones are:

Malformed FETCH payload:
: If a server
  determines that the payload provided with a FETCH request is
  not properly formatted, it can return a 4.00 (Bad Request)
  CoAP error. The definition of a malformed payload depends
  upon the CoAP Content-Format specified with the request.

Unsupported FETCH payload:
: In case a client
  sends a payload that is inappropriate for the resource
  identified by the Request-URI, the server can return a 4.15
  (Unsupported Content-Format) CoAP error. The server can
  determine if the payload is supported by checking the CoAP
  Content-Format specified with the request.

Unprocessable request:
: This situation occurs
  when the payload of a FETCH request is determined as valid,
  i.e. well-formed and supported, however, the server is
  unable to or incapable of processing the request. The server
  can return a 4.22 (Unprocessable Entity) CoAP error. In
  situations when the server has
  insufficient computing resources to
  complete the request successfully, it can return a 4.13 (Request Entity
  Too Large) CoAP error (see also below).

  In case there are more specific errors that provide more
  insight into the problem, then those should be used.

Request too large:
: If the payload of the FETCH
  request is larger than a CoAP server can process, then it
  can return the 4.13 (Request Entity Too Large) CoAP
  error.

It is possible that other error situations, not mentioned
here, are encountered by a CoAP server while processing the
FETCH request. In these situations other appropriate CoAP
status codes can also be returned.

## Option Numbers {#fetch-option}

FETCH for CoAP adopts the option numbers as specified in
sections 5.10 and 12.2 of {{-coap}}.

Generally, options defined for GET act in an analogous way for FETCH.
Two specific cases are called out in the rest of this section.

### The Content-Format Option {#fetch-format}

A FETCH request MUST include a Content-Format option (see Section
5.10.3 of {{-coap}}) to specify the
media type and content encoding of the request body.
(Typically, the media type will specifically have been designed to
specify details for a selection or a search on a resource.)

### The ETag Option {#fetch-etag}

The ETag Option on a FETCH result has the same semantics as defined in
Section 5.10.6 of {{-coap}}.  In particular, its use as a response
option describes the "tagged representation", which for FETCH is the
same as the "selected representation".  The FETCH payload is input to
that selection process and therefore needs to be part of the cache
key.  Similarly, the use of ETag as a request option can elicit a 2.03 Valid
response if the representation associated with the ETag would still be
selected by the FETCH request (including its payload).

## Working with Observe {#fetch-observe}

The Observe option {{-observe}} can be used with a FETCH request as it
can be used with a GET request.


## Working with Block {#fetch-block}

The Block1 option {{-block}} can be used with a FETCH
request as it would be used with a POST request; the Block2 option
can then be used as with GET or POST.


## FETCH discussion {#fetch-discuss}

One property of FETCH that may be non-obvious is that a FETCH request
cannot be generated from a link alone, but also needs a way to
generate the request payload.  Again, form relations
({{I-D.hartke-core-apps}}) may be able to fill parts of this gap.


## A Simple Example for FETCH {#fetch-example}

The FETCH method needs a media type for its payload (as expressed by
the Content-Format request option) that specifies the search query in
a similar detail as is shown for the patch payload in the PATCH
example in {{example}}. ({{I-D.snell-search-method}} invents a
`text/query` format based on some hypothetical SQL dialect for its
examples.)

The example below illustrates retrieval of a subset of a JSON {{-json}} object
(the same object as used in {{example}}).  Using a hypothetical media type
`application/example-map-keys+json` (with a Content-Format ID of
NNN -- not defined as this is just an example), the client specifies
the items in
the object that it wants: it supplies a JSON array giving the map keys
for these items.  A resource located at `coap://www.example.com/object`
can be represented by a JSON document that we will consider as the
target of the FETCH. The client wants to learn the contents of the
single map key `foo` within this target:

~~~
{
  "x-coord": 256,
  "y-coord": 45,
  "foo": ["bar","baz"]
}
~~~
{: title="FETCH example: JSON document returned by GET"}

The example FETCH request specifies a single top-level member desired
by giving its map key as the sole element of the `example-map-keys`
payload:

~~~
FETCH CoAP://www.example.com/object
Content-Format: NNN (application/example-map-keys+json)
Accept: application/json
[
  "foo"
]
~~~
{: title="FETCH example: Request"}

The server returns a subset document with just the selected member:

~~~
2.05 Content
Content-Format: 50 (application/json)
{
  "foo": ["bar","baz"]
}
~~~
{: title="FETCH example: Response with subset JSON document"}

By the logic of this example, the requester could have entered more
than one map key into the request payload array and would have
received a more complete subset of the top-level JSON object that is
representing the resource.


# PATCH and iPATCH Methods {#patch}

The PATCH and iPATCH methods request that a set of changes described in
the request payload is applied to the target resource of the
request.  The set of changes is represented in a format
identified by a media type.  If the Request-URI does not point
to an existing resource, the server MAY create a new resource
with that URI, depending on the patch document type (whether
it can logically modify a null resource) and permissions, as well as other
conditions (see also {{errors}}).
Creation of a new resource would result in a 2.01 (Created)
Response Code dependent on the patch document type.

Restrictions to a PATCH or iPATCH request can be made by including the If-Match
or If-None-Match options in the request (see Section 5.10.8.1
and 5.10.8.2 of {{-coap}}).  If the resource
could not be created or modified, then an appropriate Error
Response Code SHOULD be sent.

The difference between the PUT and PATCH requests is documented in {{-http-patch}}.

The PATCH method is not safe and not idempotent, as with the HTTP
PATCH method
specified in {{-http-patch}}.

The iPATCH method is not safe but idempotent, as with the CoAP PUT method
specified in {{-coap}}, Section 5.8.3.

A client can mark a request as idempotent by using the iPATCH method
instead of the PATCH method.  This is the only difference between the
two.  The indication of idempotence may enable the server to keep less
state about the interaction; some constrained servers may only
implement the iPATCH variant for this reason.

<!-- An iPATCH request is idempotent to prevent bad outcomes from -->
<!-- collisions between two iPATCH requests on the same resource in -->
<!-- a similar time frame. These collisions can be detected with -->
<!-- the MessageId and the source end-point provided by the CoAP -->
<!-- protocol (see section 4.5 of {{-coap}}. -->

PATCH and iPATCH are both atomic.
The server MUST apply the entire set of changes atomically and
never provide a partially modified representation to a
concurrently executed GET request. Given the constrained
nature of the servers, most servers will only execute CoAP
requests consecutively, thus preventing a concurrent partial
overlapping of request modifications. Resuming,
modifications MUST NOT be applied to the server state when an
error occurs or only a partial execution is possible on the resources present
in the server.
<!-- When the PATCH request is over-specified (i.e. Request specifies -->
<!-- modifications to attributes which do not exist in the server), the server -->
<!-- MAY execute all modifications to existing attributes and return a response -->
<!-- code 2.02 Accepted. -->

The atomicity applies to a single server. When a PATCH or iPATCH request is
multicast to a set of servers, each server can either execute all required
modifications or not. It is not required that all servers execute all modifications
or none. An Atomic Commit protocol that provides multiple server atomicity
is out of scope.

A PATCH or iPATCH response can invalidate a cache as with the
PUT response. Caching behaviour as function of the successful (2.xx)
response codes for PATCH or iPATCH are:

* A 2.01 (Created) response invalidates any cache entry for
  the resource indicated by the Location-\* Options; the
  payload is a representation of the action result.

* A 2.04 (Changed) response invalidates any cache entry
  for the target resource; the payload is a representation of
  the action result.

There is no guarantee that a resource can be modified with
PATCH or iPATCH.
<!-- Servers are required to support a subset of the content -->
<!-- formats as specified in sections 12.3 and 5.10.3 of {{-coap}}. -->
Servers MUST ensure that a received PATCH
body is appropriate for the type of resource identified by
the target resource of the request.

<!-- a bit redundant -->
When a request is intended to effect a partial update of a given resource, clients
cannot use PUT while supplying just the update, but are free to use PATCH or iPATCH.

<!-- PATCH or iPATCH MUST not be used to restore default values to resource attributes -->
<!-- which are not specified in the payload. PATCH or iPATCH specifically guarantees -->
<!-- that unspecified resource attributes are not changed. -->

## Simple Examples for PATCH and iPATCH {#example}

The example is taken over from {{RFC6902}},
which specifies a JSON notation for PATCH operations. A
resource located at coap://www.example.com/object contains a target
JSON document.


~~~~
JSON document original state:
    {
      "x-coord": 256,
      "y-coord": 45,
      "foo": ["bar","baz"]
    }

REQ: iPATCH CoAP://www.example.com/object
Content-Format: 51 (application/json-patch+json)
    [
      { "op":"replace", "path":"x-coord", "value":45}
    ]

RET: CoAP 2.04 Changed

JSON document final state:
    {
      "x-coord": 45,
      "y-coord": 45,
      "foo": ["bar","baz"]
    }
~~~~
{: artwork-align="left"}

This example illustrates use of an idempotent modification to the
x-coord member of the existing resource "object".
The 2.04 (Changed) response code is conform with the CoAP
PUT method.

The same example using the Content-Format application/merge-patch+json from {{RFC7396}} looks like:


~~~~
JSON document original state:
    {
      "x-coord": 256,
      "y-coord": 45,
      "foo": ["bar","baz"]
    }

REQ: iPATCH CoAP://www.example.com/object
Content-Format: 52 (application/merge-patch+json)
     { "x-coord":45}

RET: CoAP 2.04 Changed

JSON document final state:
    {
      "x-coord": 45,
      "y-coord": 45,
      "foo": ["bar","baz"]
    }
~~~~
{: artwork-align="left"}

The examples show the use of the iPATCH method, but the use of the PATCH
method would have led to the same result. Below a non-idempotent modification
is shown. Because the action is non-idempotent, iPATCH returns an error,
while PATCH executes the action.


~~~~
JSON document original state:
    {
      "x-coord": 256,
      "y-coord": 45,
      "foo": ["bar","baz"]
    }

REQ: iPATCH CoAP://www.example.com/object
Content-Format: 51 (application/json-patch+json)
    [
      { "op":"add","path":"foo/1","value":"bar"}
    ]
RET: CoAP 4.00 Bad Request
Diagnostic payload: Patch format not idempotent

JSON document final state is unchanged

REQ: PATCH CoAP://www.example.com/object
Content-Format: 51 (application/json-patch+json)
    [
      { "op":"add","path":"foo/1","value":"bar"}
    ]
RET: CoAP 2.04 Changed

JSON document final state:
    {
      "x-coord": 45,
      "y-coord": 45,
      "foo": ["bar","bar","baz"]
    }
~~~~
{: artwork-align="left"}


## Response Codes {#response}

PATCH and iPATCH for CoAP adopt the response codes as specified in
sections 5.9 and 12.1.2 of {{-coap}} and add 4.09 "Conflict" and 4.22
"Unprocessable Entity" with the semantics specified in {{errors}} of
the present specification.

## Option Numbers {#option}

PATCH and iPATCH for CoAP adopt the option numbers as specified in
sections 5.10 and 12.2 of {{-coap}}.


## Error Handling {#errors}

A PATCH or iPATCH request may fail under certain known conditions. These
situations should be dealt with as expressed below.



Malformed PATCH or iPATCH payload:
: If a server
  determines that the payload provided with a PATCH or iPATCH request is
  not properly formatted, it can return a 4.00 (Bad Request)
  CoAP error. The definition of a malformed payload depends
  upon the CoAP Content-Format specified with the request.


Unsupported PATCH or iPATCH payload:
: In case a client
  sends a payload that is inappropriate for the resource
  identified by the Request-URI, the server can return a 4.15
  (Unsupported Content-Format) CoAP error. The server can
  determine if the payload is supported by checking the CoAP
  Content-Format specified with the request.


Unprocessable request:
: This situation occurs
  when the payload of a PATCH request is determined as valid,
  i.e. well-formed and supported, however, the server is
  unable to or incapable of processing the request. The server
  can return a 4.22 (Unprocessable Entity) CoAP error. More
  specific scenarios might include situations when:

  * the server has insufficient computing resources to
    complete the request successfully --- 4.13 (Request Entity
    Too Large) CoAP Response Code (see below),

  * the resource specified in the request becomes invalid
    by applying the payload --- 4.09 (Conflict) CoAP
    Response Code (see below)).

  In case there are more specific errors that provide more
  insight into the problem, then those should be used.


Resource not found:
: The 4.04 (Not Found) error
  should be returned in case the payload of a PATCH request
  cannot be applied to a non-existent resource.


Failed precondition:
: In case the client uses
  the conditional If-Match or If-None-Match option to define a
  precondition for the PATCH request, and that precondition
  fails, then the server can return the 4.12 (Precondition
  Failed) CoAP error.


Request too large:
: If the payload of the PATCH
  request is larger than a CoAP server can process, then it
  can return the 4.13 (Request Entity Too Large) CoAP
  error.


Conflicting state:
: If the modification specified by a PATCH or iPATCH request causes
  the resource to enter an inconsistent state that the server cannot resolve,
  the server can return
  the 4.09 (Conflict) CoAP response.   The server
  SHOULD generate a payload that includes enough information for a user
  to recognize the source of the conflict.
  The server MAY return the actual resource state to provide the client with
  the means to create a new consistent resource state. Such a situation might
  be
  encountered when a structural modification is applied to a
  configuration data-store, but the structures being modified do not
  exist.


Concurrent modification:
: Resource constrained
  devices might need to process requests in the order they are
  received. In case requests are received concurrently to
  modify the same resource but they cannot be queued, the
  server can return a 5.03 (Service unavailable) CoAP response code.


Conflict handling failure:
: If the modification implies the reservation of resources or the waiting on
  conditions to become true,
  leading to a too long request execution time, the server can return 5.03
  (service unavailable) response code.

It is possible that other error situations, not mentioned
here, are encountered by a CoAP server while processing the
PATCH request. In these situations other appropriate CoAP
status codes can also be returned.

# The New Set of CoAP Methods

Adding three new methods to CoAP's existing four may seem like a major
change.  However, both FETCH and the two PATCH variants fit well into
the REST paradigm and have been anticipated on the HTTP side.
Adding both a non-idempotent and an idempotent PATCH variant allows to
keep interoperability with HTTP's PATCH method as well as the use/indication of
an idempotent PATCH if that is possible, saving significant effort on
the server side.

Interestingly, the three new methods fit into the old table of methods
with a surprising similarity in the idempotence and safety attributes:

| Code | Name   | Code | Name   | safe | idempotent |
|------|--------|------|--------|------|------------|
| 0.01 | GET    | 0.05 | FETCH  | yes  | yes        |
| 0.02 | POST   | 0.06 | PATCH  | no   | no         |
| 0.03 | PUT    | 0.07 | iPATCH | no   | yes        |
| 0.04 | DELETE |      |        | no   | yes        |

# Security Considerations

This section analyses the possible threats to the CoAP FETCH and PATCH or iPATCH
methods.  It is meant to inform protocol and application
developers about the security limitations of CoAP FETCH and PATCH or iPATCH
as
described in this document.

The FETCH method is subject to the same general security
considerations as all CoAP methods as described in {{-coap}}.
The payload of a FETCH request may reveal more detailed information
about the specific portions of a resource of interest to the
requester than a GET request for the entire resource would; this may
mean that confidentiality protection of the request by DTLS or other
means is needed for FETCH where it wouldn't be needed for GET.

The security consideration of Section 11 of {{-coap}} as well as those
of Section 5 of {{-http-patch}} also apply.

The security considerations for PATCH or iPATCH are nearly identical to
the security considerations for PUT ({{-coap}}).  The mechanisms used for PUT can be used
for PATCH or iPATCH as well.

PATCH or iPATCH are secured following the CoAP recommendations as
specified in section 9 of {{-coap}}. When additional security
techniques are standardized for CoAP,
PATCH or iPATCH can also be (and need to be) secured by those new techniques.


# IANA Considerations


IANA is requested to add the following entries to the sub-registry "CoAP Method
Codes":

| Code | Name   | Reference |
|------|--------|-----------|
| 0.05 | FETCH  | [RFCthis] |
| 0.06 | PATCH  | [RFCthis] |
| 0.07 | iPATCH | [RFCthis] |

The FETCH method is idempotent and safe, and it returns the same
response codes that GET can return, plus 4.13 (Request Entity Too
Large), 4.15 (Unsupported Content-Format), and 4.22 (Unprocessable
Entity) with the semantics specified in {{fetch-errors}}.

The PATCH method is neither idempotent nor safe.  It returns the same
response codes that POST can return, plus 4.09 (Conflict) and 4.22
(Unprocessable Entity) with the semantics specified in {{errors}}.

The iPATCH method is identical to the PATCH method, except that it is
idempotent.

IANA is requested to add the following code to the sub-registry "CoAP
response codes":

| Code | Name                 | Reference |
|------|----------------------|-----------|
| 4.09 | Conflict             | [RFCthis] |
| 4.22 | Unprocessable Entity | [RFCthis] |


IANA is requested to add entries to the sub-registry "CoAP
Content-Formats", within the "CoRE Parameters" registry:

| Media Type                   | Encoding | ID | Reference   |
|------------------------------|----------|----|-------------|
| application/json-patch+json  |          | 51 | {{RFC6902}} |
| application/merge-patch+json |          | 52 | {{RFC7396}} |


# Change log

When published as an RFC, this section needs to be removed.

Version 00 is a composition from draft-vanderstok-core-patch-03 and
draft-bormann-core-coap-fetch-00 and replaces these two drafts.

Version 01 added an example for FETCH and is more explicit about
some response codes and options.

Version 02 addresses the WGLC comments.

Version 03 addresses the IETF last-call comments.

--- back

# Acknowledgements
{: numbered="no"}

Klaus Hartke has pointed out some essential differences between CoAP and
HTTP concerning PATCH, and found a number of problems in an earlier version
of {{fetch}}. We are grateful for discussions with Christian Amsuss,
Andy Bierman, Timothy Carey, Paul
Duffy, Matthias Kovatsch, Michel Veillette, Michael Verschoor, Thomas Watteyne,
and Gengyu Wei.  Christian Groves provided detailed
comments during the Working-Group Last Call, and Christer Holmberg's
Gen-ART review provided some further editorial improvement.
Further last-call reviews were provided by Sheng Jiang and Phillip
Hallam-Baker.
As usual, the IESG had some very good reviews, we would like to
specifically call out those by Alexey Melnikov (responsible AD) and
Alissa Cooper.

<!--  LocalWords:  atomicity iPATCH idempotence cacheable varyingly
 -->
<!--  LocalWords:  idempotency Unprocessable
 -->
