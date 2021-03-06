===================================
OpenStack manuals project migration
===================================

Problem description
~~~~~~~~~~~~~~~~~~~

The documentation team are rapidly losing key contributors and core reviewers.
We are not alone, this is happening across the board. It is making things
harder, but not impossible.
Since our inception in 2010, we’ve been climbing higher and higher trying to
achieve the best documentation we could, and uphold our high standards.
However, we now need to take a step back and realise that the amount of work
we are attempting to maintain is out of reach for the team size that we have.
At the moment we have 13 cores, of whom none are full time contributors or
reviewers.

Until this point, the documentation team has owned several manuals that
include content related to multiple projects, including an installation
guide, admin guide, configuration guide, networking guide, and security
guide. Because the team no longer has the resources to own that content,
we want to invert the relationship between the doc team and project teams,
so that we become liaisons to help with maintenance instead of asking for
project teams to provide liaisons to help with content. As a part of that
change, we plan to move the existing content out of the central manuals
repository, into repositories owned by the appropriate project teams.
Project teams will then own the content and the documentation team will
assist by managing the build tools, helping with writing guidelines and
style, but not writing the bulk of the text.

We currently have the infrastructure set up to empower project teams to
manage their own documentation in their own tree, and many do. As part of
this change, the rest of the existing content from the install guide and
admin guide will also move into project-owned repositories.

Proposed change
~~~~~~~~~~~~~~~

Combine all of the documentation builds for a given repository, so
that each repository has a single doc/source directory, a single
sphinx conf.py, and a single job for building and publishing the
content including developer, contributor, and user documentation. This
option would reduce the number of build jobs we have to run, and cut
down on the number of separate sphinx configurations in each
repository.

Move most of the content from various guides in openstack-manuals into
the same repository as the thing being documented -- the documentation
should live with the code. For example, the installation instructions
for glance move to the glance repository and the reference for using
the glance client command line tool move to the python-glanceclient
repository.

In order to make it easy to include links from the landing pages on
docs.openstack.org, we need to ensure a minimum level of consistency
in the organization of the docs directory. The sections are based on
the existing high-level groupings for which there are already landing
pages on docs.openstack.org that will need to be updated. Within each
top-level directory, project teams are free to organize their content
however seems most appropriate to them. This is the proposed layout
for phase 1:

* ``doc/source/``

  * ``install/`` -- anything having to do with installation of the
    project
  * ``contributor/`` -- anything related to contributing to the
    project or how the team is managed. Applies to some of the current
    content under /developer, we are changing the name to emphasize
    that not all contributors are developers and sometimes developers
    are users but not contributors. Service projects should place
    their automatically generated class documentation under this part
    of the tree, e.g. in ``contributor/api`` or
    ``contributor/internals``.
  * ``configuration/`` -- automatically generated configuration
    reference information based on oslo.config's sphinx integration
    (or manually written for projects not using
    oslo.config). Step-by-step guides for doing things like enabling
    cells or configuring a specific driver should be placed in the
    ``admin/`` section.
  * ``cli/`` -- command line tool reference docs, similar to man pages
    These may be automatically generated with cliff's sphinx
    integration, or manually written when auto-generation is not
    possible. Tutorials or other step-by-step guides *using* these
    tools should go in either the ``user/`` or ``admin/`` sections,
    depending on their audience. Because the documentation for each
    project should live in the repository with the code, this
    directory may not be present for all service repositories but will
    be present for most of the client library repositories.
  * ``admin/`` -- any content from the old admin guide or anything
    else that discusses how to run or operate the software
  * ``user/`` -- end-user content such as concept guides, advice,
    tutorials, step-by-step instructions for using the CLI to perform
    specific tasks, etc.
  * ``reference/`` -- any reference information associated with a
    project that is not covered by one of the above categories.
    Library projects should place their automatically generated class
    documentation here.

This layout is the *minimum* set. Projects are free and encouraged to
add whatever other docs they need beyond this list, but these items
are listed here explicitly because there are already links to most of
them from landing pages, and landing pages can be created for the
others.

During a later phase, we will merge the API reference and release notes builds
into the same job, along with the rest of the documentation for a project.
Both of those builds have custom considerations, though, and it is more
important to move the content that is no longer going to be maintained
by the documentation team.

* ``doc/source/``

  * ``api/`` -- the REST API reference and Guide content, when it exists
  * ``releasenotes/`` -- reno directions (the actual release notes
    inputs will stay in /releasenotes/notes, where they are now)

.. note::

   Further discussion of the layout of the ``api/`` and
   ``releasenotes/`` directories is deferred until we are farther
   along with the initial migration work.

A new documentation build job will be set up to take the output produced from
``tox -e venv -- python setup.py build_sphinx`` (matching the existing
`Consistent Testing Interface"
<https://governance.openstack.org/tc/reference/project-testing-interface.html>`_)
and publish it to a new location under `<http://docs.openstack.org>`_
The results will go to:

* ``docs.openstack.org/$project-name/latest`` -- build from master
* ``docs.openstack.org/$project-name/$series`` -- build from
  stable/$series
* ``docs.openstack.org/$project-name/latest/$lang`` -- build
  translated version from master
* ``docs.openstack.org/$project-name/$series/$lang`` -- build
  translated version from stable/$series

Because we plan to reuse the existing `doc/source` directory in each project,
some of the existing content will need to be rearranged as part of importing
the content from openstack-manuals.

Ultimately, this changes the way we publish results, and redirects will be
required to be setup from all of the existing locations to the new locations,
and move all of the existing documentation under the new structure. We will
retain landing pages for the high level categories such as the install guides,
the configuration reference, and contributor documentation. Those pages will
continue to be maintained by the documentation team, and will deep-link into
the project team documentation.

For example, we will keep pages like
https://docs.openstack.org/project-install-guide/ocata/ but they will
provide lists of links to URLs like
https://docs.openstack.org/nova/ocata/install, which will be part of
the single doc build for nova.

What is happening to each guide?
--------------------------------

* Installation Guide

  * Most of the content will move from openstack-manuals into each project
    tree.
  * The chapters not directly related to specific OpenStack projects,
    such as the parts related to installing ntp and RabbitMQ, will be
    retained in openstack-manuals in a shared guide for setting up
    common dependencies so that content does not need to be reproduced
    several times.
  * The current guide is actually built 3 separate times, to cover 3
    separate deployment platforms. The new build will not support
    that, so the migration for the installation guide will involve
    breaking the content up into separate pages for each platform (as
    needed). Patch https://review.openstack.org/473579 splits the
    content up into separate patches, one per OS.

* Project Installation guides, already in tree

  * We recommend any installation guides already in-tree also move to the new
    organization.

* Administrator Guide

  * This content will move from openstack-manuals into each project tree. No
    part will be retained in openstack-manuals. This spec was already
    approved:
    https://review.openstack.org/#/c/439122/

* High Availability Guide

  * This guide will remain in openstack-manuals and be managed by the HA team.
    For more information: https://blueprints.launchpad.net/openstack-manuals/+spec/implement-ha-guide-todos

* Operations Guide

  * This guide will eventually move from openstack-manuals into the wiki.
    Nothing will be done with it until a volunteer is found to manage that
    move.

* Security Guide

  * This content will stay in openstack-manuals, and be managed by the
    security team.
  * A notice is being added to indicate the last time it was updated
    and which release is relevant
    (https://review.openstack.org/#/c/470059).

* Architecture Design Guide

  * This content will stay in openstack-manuals, and be deprecated.
  * A notice will be added to each page indicating that the guide is up to
    date as of $RELEASE after the finalisation of the current set of goals.
    For more information on those goals:
    https://blueprints.launchpad.net/openstack-manuals/+spec/arch-design-pike

* Networking Guide

  * This content will move from openstack-manuals to the neutron repository
    under docs/source/admin.

* Configuration Reference

  * A few pages will move from openstack-manuals to the user-facing
    documentation in oslo.config.
  * The remainder will be removed, and replaced with new pages in the
    in-tree documentation built using oslo_config.sphinxext.
  * For tracking purposes, please see:
    https://blueprints.launchpad.net/openstack-manuals/+spec/automate-config-ref

* API Documentation

  * No changes.

* End User Guide

  * This content will be divided between the horizon repository and
    python-openstackclient repository.

* Command-Line Reference

  * This content will move the project-specific client documentation
    trees under doc/source/cli. For example, the information about
    using the ``glance`` command line tool would move to the
    python-glanceclient repository.

* Virtual Machine Image Reference
  * This content will stay in openstack-manuals.

Migration process
-----------------

We will need to parallelize the migration work as much as possible if we are
going to complete it by the end of the Pike cycle. We will therefore need
project teams to find volunteers to "pull" the content into their
repositories, instead of having the documentation team "push" it.

.. note::

   Use the topic ``doc-migration`` for all patches.

.. note::

   Repeat these steps for all server projects, clients, and other
   libraries.

#. Move the existing contributor-focused content to fit the layout
   above. Submit that change with ``Depends-On:
   Ia750cb049c0f53a234ea70ce1f2bbbb7a2aa9454`` to tie it to this
   spec.
#. If your project docs are not already building using
   warning-is-error in setup.cfg, turn that on and fix any build
   errors. Submit these as patches on top of the first patch.
#. Pull in the content being migrated, following the layout above.

   * Go through the list of manuals in
     https://etherpad.openstack.org/p/doc-migration-tracking and take
     any actions needed to import content.
   * Prepare one patch per manual (so one to import the install guide,
     one to import the user guide, etc.). Submit these as patches on
     top of any previous patches.
   * `:term:` needs to be removed when importing and
     performing the migration. This is due to the glossary remaining in the
     openstack-manuals repo. Teams can choose to link to the glossary
     on their own project pages if they so desire.

#. Ensure that there is an index.rst in each subdirectory of
   doc/source so that the various landing pages managed by the
   documentation team can link directly to that portion of the
   documentation for your project. For example, in addition to moving
   installation documentation into ``install/`` create
   ``install/index.rst`` with a ``toctree`` directive that shows all of
   the installation.
#. Ensure that there is a top-level index.rst in doc/source that
   incorporates all of the documentation for the project by including
   all of the subdirectories in a toctree.
#. Update the theme for the in-tree docs to use the openstackdocstheme
   instead of oslosphinx.
#. Add auto-generated config reference section(s) under the
   ``configuration/`` directory.
#. If pbr's autodoc feature is being used, update the ``api_doc_dir``
   setting in the ``pbr`` section of ``setup.cfg`` to point to either
   ``reference/api`` (for libraries) or ``contributor/api`` (for other
   types of projects).
#. Update project-config to have the doc build use the new jobs instead of the
   old jobs by replacing 'openstack-server-publish-jobs' with
   'openstack-unified-publish-jobs'.

   Set ``Depends-On`` to the Change-Id from the patch created in
   step 1. This ensures that we do not publish the old content to the
   new location.

#. Add links to the reviews for individual TODO items below those
   items in the sections dedicated to each manual. That way the docs
   team will know when it is safe to start deleting content.

Alternatives
------------

#. We could retain the existing trees for developer and API docs, and add a new
   one for "user" documentation. The installation guide, configuration guide,
   and admin guide would move here for all projects. Neutron's user
   documentation would include the current networking guide as well. This
   option would add 1 new build to each repository, but would allow us to
   easily roll
   out the change with less disruption in the way the site is organized and
   published, so there would be less work in the short term.
#. We could move the content under separate repositories owned by the project
   teams, rather than in-tree with the code. This would allow project teams to
   delegate management of the documentation to a separate review
   project-sub-team, but would complicate the process of landing code and
   documentation updates together so that the docs are always up to date.
#. Do nothing, and watch the world burn.

We did consider using "service type" instead of "project name" for the
publishing URLs, but not all of the projects that need documentations
are services. We will have user-facing documentation coming from several
Oslo libraries, for example.

Implementation
~~~~~~~~~~~~~~

Assignee(s)
-----------

Primary assignee:

* Alexandra Settle (asettle)
* Doug Hellmann (dhellmann)
* Project teams
* Documentation team PTL for Queens
* Documentation team

Work items
----------

The task list is quite long, so rather than repeat it here we give a summary.
There is more detail in the tracking pad mentioned in step 3.

#. Define new doc build and gate jobs that work like the current job, using
   "tox -e venv -- python setup.py build_sphinx`" in a repository, but publish
   to the new location of docs.o.o/$project-name/latest (dhellmann)

   * https://review.openstack.org/#/c/471881/

#. Define doc build jobs for stable branches that run the same command but
   publish to docs.o.o/$project-name/$series (dhellmann)

   * The same job will work for all branches.

#. In parallel, in each repository, perform the migration steps listed above to
   copy the new content into the doc/source directory. Refer to
   https://etherpad.openstack.org/p/doc-migration-tracking for details about
   which pages go into which project trees.
#. Define new translation jobs based on the ones for the release notes build
   but using the main doc build.
#. Create a separate build for the openstack-manuals glossary.

Dependencies
~~~~~~~~~~~~

- Project team(s) collaboration
- Infra team assistance
- Reviews from multiple sources

References
~~~~~~~~~~

* https://etherpad.openstack.org/p/doc-planning
* The list of all URLs and where the content will move can be found
  in: https://etherpad.openstack.org/p/doc-migration-tracking
* Documentation Publishing future thread:
  http://lists.openstack.org/pipermail/openstack-dev/2017-May/117162.html
* Operations Guide Future thread:
  http://lists.openstack.org/pipermail/openstack-dev/2017-June/117799.html
