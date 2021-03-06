..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Proprietary driver docs in openstack-manuals
============================================

https://blueprints.launchpad.net/openstack-manuals/+spec/move-driver-docs

The Configuration Reference and Cloud Admin Guide include documentation for
various drivers. This spec clarifies the expectations and handling of such
documentation.

The shared goals for driver documentation includes:

- consistent documentation, comparable for each driver.
- version-independent documentation for each driver, meaning the
  OpenStack version can change but the driver documentation can remain
  the same.
- maintainable documentation for each driver that won't change much
  over time and remains accurate.
- reviewable documentation for each driver.

Some benefits we see for this new approach are:

- more flexibility in changing detailed driver documentation on the
  appropriate website.
- makes maintenance of detailed driver doc easier for contributing
  driver writers since they can choose their source and publishing
  chain that fits with their current workflow.
- reduces maintenance for core OpenStack documentation team.

The driver documentation for Compute, Storage, Networking, and
Databases will change as described with the goal of having accurate
documentation. This can be brief version independent information in
the OpenStack manuals with a link to a vendor page with full details
or full version.

Problem description
===================

Many OpenStack projects include drivers to support specific hardware
or software.

Examples are:

* Cinder: block storage drivers
* Neutron: network plug-ins
* Nova: hypervisors
* Trove: Different databases

Many of these drivers are hardware or software specific and can only
be documented by the third-party driver vendor. Some vendors have
great in-tree documentation and update it regularly, others have none
or only outdated documentation. Many vendors have already on
their web page documentation about the drivers, this spec proposes to
move the documentation to the vendor web pages and simply link to
them.

The spec also documents requirements for full documentation as part of
OpenStack manuals.

This will reduce work for both documentation team and vendor drivers
and give clear requirements.

Proposed change
===============

The documentation team will fully document the reference drivers as
specified below and just add short sections for other drivers. If a
vendor wants to document their driver, they are invited - but not
forced - to include their documentation in the Configuration
Reference if they commit to maintain the documentation. However,
other documentation (including the Cloud Admin Guide and Networking
Guide) will not contain content for third-party drivers. In these books,
where third party drivers exist, add the statement: "For other drivers,
see chapter X in the Configuration Reference Guide".

Guidelines for drivers that will be documented fully by the OpenStack
community in the OpenStack documentation:

* The complete solution must be open source and use standard hardware
* The driver must be part of the respective OpenStack repository
* The driver is considered one of the reference drivers

For documentation of other drivers, the following guidelines apply:

* The Configuration Reference will contain a small section for each
  driver, see below for details
* Only drivers are covered that are contained in the official
  OpenStack project repository for drivers (for example in the main
  project repository or the official third-party repository).

If a vendor wants to add more than the minimal documentation, they
need to commit to the following guidelines:

* Assign an editor that is responsible for the content.
* Review and - if necessary - update their driver for each release
  cycle.

With this policy, the docs team will document in the Configuration
Reference at least the following drivers:

* For cinder: volume drivers: document LVM and NFS; backup drivers:
  document swift
* For glance: Document local storage, cinder, and swift as backends
* For neutron: document ML2 plug-in with the mechanisms drivers
  OpenVSwitch and LinuxBridge
* For nova: document KVM (mostly), send Xen open source call for help
* For sahara: apache hadoop
* For trove: document all supported Open Source database engines like
  MySQL.


Default section format for external drivers
-------------------------------------------

For each external driver, we want to document briefly the driver in a
way that is version independent and include the current configuration
options.

Each section should follow this format:

* A short paragraph explaining the driver.
* A link with detailed instructions to the vendor site (if there is one)
* A default paragraph like::

    Set the following in your cinder.conf, and use the following options
    to configure it.

    volume_driver = cinder.volume.drivers.smbfs.SmbfsDriver

* And finally the autogenerated configuration options

Driver vendors can send in patches for these or create bugs.


Full documentation by vendors
-----------------------------

If a vendor wants full documentation in the Configuration Reference,
they have to add to the `wiki page
<http://wiki.openstack.org/Documentation/VendorDrivers>`_ a contact
editor that will take care of the vendor driver documentation. The
Documentation team will assign bugs to the contact person, include the
contact person in reviews for the vendor driver, and expects timely
responses.

If vendor driver documentation becomes outdated and the contact person
is not reacting to requests, the Documentation team will change the
full documentation to a minimal version.

The documentation team will review vendor drivers and ensure that the
various driver documents follow a consistent standard.

Alternatives
------------

* Keep status quo: Add all drivers to the Configuration Reference.
* Remove drivers, do not link to them at all - or just link to a
  single wiki page.
* Have minimal documentation for all drivers only. This was the
  initial idea but rejected since some vendors do not have
  documentation on their own.

Implementation
==============
The work will be done in three steps:

#. In the Configuration Reference, bring all driver sections that
   are currently just "bare bones" up to the standard mentioned.
#. Work with third-party drivers to convert existing documentation
   in the Configuration Reference to the new standard.
#. Purge third-party driver content from other documentation such
   as the Cloud Admin Guide.


Assignee(s)
-----------

Entire documentation team
loquacities - informing vendors

Work Items
----------

* Inform third-party driver contacts about change (note that we
  have to make this spec known to them earlier to get input on it as
  well)
* Ask vendor drivers to assign a contact person and give deadlines.
* Add minimal content for drivers that have no content right now.
* Enhance content (based on suggestion by driver vendors)
* Purge third-party driver content from other documentation.


Dependencies
============

None.


Testing
=======


References
==========

https://etherpad.openstack.org/p/docstopicsparissummit
