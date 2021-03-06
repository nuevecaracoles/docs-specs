..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Rework API Reference Information
================================

The blueprint we'll use to track this work supersedes prior blueprints that are
listed in the References section.

https://blueprints.launchpad.net/openstack-manuals/+spec/api-site

The API Reference site needs an update and better methods for maintaining and
providing accurate information for application developers using different cloud
provider's cloud resources.

Problem description
===================

With this blueprint we want to solve the following problems:

* API reference information has been historically difficult to maintain using
  community or company resources due to the specialized nature of both the
  tools and the REST API knowledge.
* API reference information needs to be sourced where API writers, API
  developers, contributor developers, and the API working group members can
  review together.

This blueprint should provide a major rework of the way we provide upstream API
reference information to cloud users. The intended audience targets application
developers and SDK developers who need precise and accurate information about
each service's REST APIs. This is not API information for internal APIs such as
the RPC API used by the Compute project (nova) for scheduling compute hosts,
for example. These references are used to build a correct API call with the
correct request parameters and double-check your cloud provider's results
against the results you see on screen when you make a call.

There are currently 915 GET/PUT/POST/DELETE/PATCH calls documented on the
OpenStack API Complete Reference site at
http://developer.openstack.org/api-ref.html.

Currently, the API reference is in a separate repo, api-site. In the kilo
release (Nov14-Apr15) we moved all "narrative" information to the repo of the
project's choosing. For most projects, that location was the <project>-spec
repo. For Compute and Object Storage, it was their project's doc/source repo.
The API reference information remained in api-site repo.

With the growth of teams with REST APIs to more than 20, we need to strictly
enforce where API reference information is maintained and built from.

Ideally we will enable teams to review while bringing all the sources together
into one consumable, readable guide to the various services's REST APIs. These
should be handy Developer Guides that provide a unified view of
separately-sourced information.

Now that we've described the problem and a brief discussion of the vision,
let's talk about solutions.

Scope for Liberty release
-------------------------

For the first set of developer guides sourced across multiple repos with
automation for reference information, the scope will be limited to
infrastructure services used most based on the most recent User Guide survey.

* Identity v2.0
* Compute v2
* Block Storage v2
* Image service v2
* Networking v2.0

However, while we are working on this proof-of-concept, the WADL needs to be
maintained so that we have a benchmark comparison point for quality testing
purposes.

Proposed change
===============

We want to ensure that we use the current project's source code to create
the most accurate and up-to-date API reference documentation. We also want to
ensure reviews are in the repo where the best reviewers are keeping vigil.

So, as a proof of concept, write a Python library that uses the Python routes
library to inspect the source code and output a standard such as Swagger
version 2 that can be used for doc output or mock testing of the requests and
responses so that we can eventually build a continuous test layer that ensures
backwards compatibility for examples even when the code changes.

Here is an overview of the envisioned process:
#. Do a git clone command to get a copy of the project's repo.
#. Run the automation script with some config info in the api-paste.ini file
to create Swagger.
#. Repeat and test for each project.

Here is a list of what Web Server Gateway Interface (wsgi) frameworks are in
use for OpenStack projects:

  - Ironic: pecan, wsme
  - Nova: routes, JSONSchema
  - Cinder: routes
  - Glance: routes, JSONSchema
  - Neutron: routes
  - Trove: routes
  - Heat: routes
  - Saraha: flask
  - Swift: None
  - Keystone: routes
  - Ceilometer: pecan
  - Barbican: pecan

For reference, these projects are already documented in the `openstack/api-site
repository <https://github.com/openstack/api-site/>`_:

* ceilometer: Telemetry or Telemetry module
* cinder: OpenStack Block Storage (two versions)
* glance: OpenStack Image service (two versions)
* heat: Orchestration or Orchestration module
* keystone: OpenStack Identity (two versions)
* neutron: OpenStack Networking
* nova: OpenStack Compute (two versions)
* sahara: Data processing service for OpenStack
* swift: OpenStack Object Storage
* trove: Database service for OpenStack

API docs elsewhere or unknown state:

* barbican: Key management service
* ironic: Bare metal service
* tripleo: Deployment
* zaqar: Message service
* designate: DNS service
* magnum: Containers service
* murano: Application catalog
* congress: Non-domain specific policy enforcement
* rally: Benchmark service
* mistral: Workflow service

Requirements include:

Authoring: Information and source must be maintained and reviewed by project
developers with API working group informed and doc team providing support.

Authoring: API reference information could be auto-generated with strings
stored in the code or reviewed and written collaboratively. For more info,
review the :ref:`overview-of-standards` below.

Authoring: API reference information review should use the APIImpact and
DocImpact flags.

Authoring: Need an open-source toolchain for authoring.

Output: Output must offer a great experience for SDK developers and
application developers consuming OpenStack cloud resources. Optionally, it
would offer a "try it out" sandbox for each call against TryStack when using
authenticated credentials.

Output: Output should indicate which version of OpenStack will support a
particular API version, and within extensible APIs like Compute and Identity,
indicate which version a particular extension is available with.

Output: Since we may need a phased approach for timing and scoping, should work
with current docs such as with redirects or integrated displays.

Build: Must be automated based on Gerrit review and workflow.

Scope: Must be viable within six month release period.

Optional features:

Build: Optionally, build pieces that any cloud provider could then consume and
re-use in their customer documentation.

Contract validation: Optionally, provide validation of requests and
responses as valid and would work against a public cloud endpoint.

Compilation of changes: Optionally, provide a list of changes to help
reviewers discover wording that could be fixed, inconsistencies in examples,
parameter naming, potential for better human grouping and so on.

Conceptual API information
--------------------------

As noted above, ideally we enable teams to write and review API information
while bringing all the sources together into one consumable, readable guide.
The work done last release to put the "narrative" information, such as rate
limits, versioning, and so on into each project's managed repository should be
reused for these Developer Guides.

For an interim step, we can start publishing the RST-sourced information to
http://developer.openstack.org/api-guide/compute
from the http://git.openstack.org/cgit/openstack/nova/tree/doc/source/v2
information. Publishing as separate items does mean needing to add a separate
index.rst and conf.py build for each of the services that has these types of
conceptual documents.

Also, add a new column to the developer.openstack.org landing page that links
to conceptual information for each service in a column next to API Reference.

These are the current links to API conceptual information:
http://docs.openstack.org/developer/nova/v2/index.html
http://docs.openstack.org/developer/swift/#object-storage-v1-rest-api-documentation
http://specs.openstack.org/openstack/glance-specs/#image-service-v2-api
http://specs.openstack.org/openstack/glance-specs/#image-service-v1-api
http://specs.openstack.org/openstack/keystone-specs/#v3-api
http://specs.openstack.org/openstack/keystone-specs/#v2-0-api
http://specs.openstack.org/openstack/neutron-specs/#api-specs
http://specs.openstack.org/openstack/cinder-specs/#volume-v2-api

By building and linking more prominently we hope to add to the collection of
helpful information for application and SDK developers.

.. _overview-of-standards:

Overview of standards
---------------------

The reference portion of this documentation should follow an industry standard.
REST API documentation has evolved over the years and a few standards have
recently become popular:

Swagger
   Community-maintained standard, open-source tooling. Allows for
   inclusion of content similar to our current entities. To output the
   information you must run a server that renders the content. Current
   community-maintained specification for content is version 2, see
   https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md.
   Supported by SmartBear Software.

RAML
   Community-maintained standard, proprietary tooling unless you just edit
   in text, but then how do you validate? RAML specification found here:
   http://raml.org/spec.html. Allows for inclusion of content similar to our
   current WADL entities for reuse of content. Based on YAML, supported
   and provided by MuleSoft.

API Blueprint
   The API Blueprint standard was started by Apiary, a company that specializes
   in API design and documentation, but they do accept patches
   from the community. The JSON and YAML specification is at
   https://github.com/apiaryio/api-blueprint-ast. We could write a JSON schema
   to add to the standard "asset element" based on
   https://github.com/apiaryio/api-blueprint-ast#asset-element. However it
   currently does not support a data structure that will allow us to have
   multiple request/response combinations for the same endpoint.

WADL
   Currently all the reference information is housed and maintained in
   openstack/api-site in WADL files. We have a `WADL2Swagger tool <https://github.com/rackerlabs/wadl2swagger>`_
   which has been run on our current WADL files. The `resulting Swagger files <http://rackerlabs.github.io/wadl2swagger/openstack.html>`_ can be used for
   comparison and testing purposes.

With the Python routes approach, we could first write to the Swagger 2.0 spec
but then write another lexer for RAML if needed.

JSON schema could be required for our API requests validation, to see if the
contract is being upheld. JSON Schema is a JSON media type for defining the
structure of JSON data, such as a request from a REST API service. JSON Schema
provides a contract for what JSON data is required for a given application and
how to interact with it. For example, request parameters, many of which are
defined as "plain" parameters, and some of which have multiple array-based
needs in the request that would have to be defined with JSON schema.

Example: Here's a sample request for adding personality to a Create Server
POST /v2/{tenant_id}/servers::

   "personality": [
            {
               "path": "/etc/banner.txt",
               "contents": "ICAgICAgDQo...mQgQmFjaA=="
            }
         ]

Considerations
==============

Russell Sim has done a proof of concept for Volume API v2. He can upload an
example for the rest of the team to start working on. He investigated using
httpdomain, but it seems that it would require depending on Sphinx in
production, angering packagers and operators alike. Instead he is making
a compatible parser written in docutils. That way we hopefully can reuse the
documentation to build with Sphinx later, but not have Sphinx as a runtime
dependency.

The `CORS cross-project specification <https://review.openstack.org/#/c/179866/>`_
should help with display of results using AngularJS as
it's a similar idea.

Identity v3 has the most calls in the core with 74, but Compute v2 plus
extensions has over 120 calls.

Currently the openstack/api-site repo that creates the API reference
information documents the last two stable releases. Our current policy is not
to merge changes to the master version of any API because end users would not
typically have access to a cloud that has that change.

For this new approach in this spec, examples are generated based on walking
through the source, so our tool would have to first apply to cinder
stable/juno and output, for example. Next, apply the tool to the cinder
stable/kilo branch and generate output. For testing purposes, the tool can be
applied against cinder master branch and flag when a backwards-incompatible
change would occur or flag when samples changed and release notes should note
the change. Versions and microversions should be displayed per call.

Alternatives
------------

Could keep what we currently have in api-site and WADL. However this requires
the continued use of clouddocs-maven-plugin for builds, which currently has no
maintainers.

Wait for a standard to emerge that supports microversions, multiple responses,
and all the features we need for our myriad API designs. None of the current
standards (WADL, Swagger, RAML) support microversions so we need to forge our
own path to ensure future maintainability and serving app devs writing code for
OpenStack clouds.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  annegentle

Other contributors:
  cberendt
  russellsim

Work Items
----------

Proof of concept automating API reference information with Volume v2 service.

Proof of concept aggregating information across separate repos in their
respective doc/source directories.

Web design and development of templates for new developer guide.

Information architecture for where the deliverables should be published on
http://developer.openstack.org/.

Fix WADL where inconsistencies are discovered.

Write a JSON Schema for modified Swagger (Swaggerish) to support multiple
request/response types at the same URL, such as Orchestration actions resource:
http://developer.openstack.org/api-ref-orchestration-v1.html#stack_action_suspend
http://developer.openstack.org/api-ref-orchestration-v1.html#stack_action_resume
Or the Compute server actions resource:
http://developer.openstack.org/api-ref-compute-v2.html#compute_server-actions

Define documentation that is included in a Swagger Tag. For example, there
exists a lot of narrative or conceptual information in the WADL and DocBook
that we need to integrate into an overall dev guide. We could develop a Tag
hierarchy with a naming scheme like:

server
server::actions
server::metadata
server::actions

Then use the Tag to design the front-end.

Surface the existing conceptual information by publishing existing content to
developer.openstack.org/api-guides/<servicename>.

Migration work items:
Delete WADL files in api-site/api-ref once replacement is complete.
Create a feature branch for api-site
Prepare the developer.openstack.org website for the transition including
DevStack installation, CORS support, and an overall information architecture
for developer guides.
Create a front-end design for presenting the information. Two POCs:
Default Swagger UI http://fairy-slipper.russellsim.org/swagger-ui/
Stripe-like Swagger UI (from jensoleg):
http://fairy-slipper.russellsim.org/swagger-ui-jensoleg/

Dependencies
============

* In order to place this functionality in oslo, we'll need the co-operation of
  oslo reviewers.

* The API Working Group is following closely and will help with ensuring the
  solution meets our needs.

Testing
=======

Output should be tested for cross-browser, cross-operating-system
compatibility.

Generating the Swagger should not require Sphinx as a run-time, to ensure that
we do not introduce unwanted global dependencies.

References
==========

Previous unimplemented blueprints related to this spec:

* https://blueprints.launchpad.net/openstack-manuals/+spec/autogenerate-api-reference
  Generating API reference information and samples is a good way forward.
  However, we'll supercede this blueprint with this implementation spec as that
  blueprint does not have a detailed spec associated with it.

* https://blueprints.launchpad.net/openstack-manuals/+spec/api-samples-to-api-site
  Moving content to project repos would be the opposite moving direction
  and may work perfectly well for this use case.

* https://blueprints.launchpad.net/openstack-manuals/+spec/api-try-it-out
  I'd see this as a stretch goal, not necessarily required for the main
  goal of making contributions and maintenance better going forward.

Additional information:

* API Archaeology: Complexity and sizing of an interface
  http://justwriteclick.com/2015/01/12/api-archaeology-complexity-and-sizing-of-an-interface/
  This blog post gives counts as of the January post date. As of April 27,
  2015 the counts are now 915 calls.

* List of services with REST APIS:
  http://git.openstack.org/cgit/openstack/governance/tree/reference/projects.yaml

* Issues with WADL2Swagger: The underlying issue is that Swagger
  definitions itself should require JSON schema to be useful and contractual.
  https://github.com/rackerlabs/wadl2swagger/issues/8

* November 2014 User Survey Data http://superuser.openstack.org/articles/openstack-user-survey-insights-november-2014

* April 2015 User Survey Data (app devs) http://superuser.openstack.org/articles/openstack-application-developers-share-insights

* API Docs Working Session Etherpad https://etherpad.openstack.org/p/Documentation__API_Work_Session

* Pecan decorator proof-of-concept for creating swagger https://github.com/elmiko/pecan-swagger
