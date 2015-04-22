..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Neutron IPv6 Prefix Delegation Driver
=====================================

https://blueprints.launchpad.net/neutron/+spec/neutron-ipv6-prefix-delegation-driver

Prefix delegation is being added to the L3 agent in the liberty cycle,
and the initial implementation is designed to leverage an external service
called Dibbler to handle prefix delegation. We wish to add an alternative
in-tree implementation of DHCPv6 prefix delegation.

Problem Description
===================

There a couple of problems with the Dibbler implementation which we hope to
solve with this blueprint:

* The L3 agent has to run one copy of Dibbler for every subnet that needs a
  prefix delegated to it, this could cause issues at scale because of resource
  usage on the networking node.

* Any changes that need to be performed to the underlying service have to be
  proposed and merged outside of the OpenStack ecosystem, and then packaged for
  distributions, which slows down release of this feature.

* Dibbler is a full DHCPv6 client that provides many more functions than are
  needed for just prefix delegation. A dedicated prefix delegation driver will
  be much lighter weight and only perform the one task.

Proposed Change
===============

We intend to add an in-tree service and matching driver that provides an
extremely light weight implementation of the DHCPv6 protocol focusing
specifically on the prefix delegation conversation. There will only be one
process running alongside the L3 agent managing all subnets that require prefix
delegation. This will be written entirely in python with no external software
dependencies.

The driver will communicate with the service via Unix sockets, for local
inter-process messaging.

Python HTTP sockets will be used to provide the interface for DHCPv6
communication and it will be multi-threaded to allow for multiple prefix
delegations to happen at the same time.

The service will be started using the "neutron-dhcpv6-client" command, and run
alongside the l3_agent process.

Data Model Impact
-----------------

Handled by original Prefix Delegation BP

REST API Impact
---------------

Handled by original Prefix Delegation BP

Security Impact
---------------

* Running the DHCPv6 PD service will require sudo or elevated privileges in
  order to communicate on the ports required for DHCPv6.

* The feature will include a protocol implementation, based on the DHCPv6
  and Prefix Delegation RFCs.

Notifications Impact
--------------------

None

Other End User Impact
---------------------

None

Performance Impact
------------------

Reduced resource usage compared to the Dibbler implementation of
prefix delegation.

IPv6 Impact
-----------

IPv6 impact should be no different from the Dibbler implementation. 

Other Deployer Impact
---------------------

Config options to be added include:

* pd_socket_loc: The location to store the driver communication socket.

* pd_interface: The network interface to bind to for the DHCPv6
                communication.

When pd_dhcp_driver is configured to use the internal driver, the new
service also needs to be configured and running. Therefore, deployer's
will now have to add 'neutron-dhcpv6-client' to their OpenStack deployments,
if they want to use the driver.

Developer Impact
----------------

None

Community Impact
----------------

None

Alternatives
------------

The alternative is to stick with the Dibbler implementation of prefix
delegation.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Sam Betts <sam@code-smash.net>

Other contributors:
  Bradley Jones <jones.bradley@me.com>

Dependencies
============

Dependent on the original prefix delegation blueprint:
  https://blueprints.launchpad.net/neutron/+spec/ipv6-prefix-delegation

Testing
=======

Documentation Impact
====================

The new config options will need to be added to the user documentation.

References
==========

Relevant DHCPv6 RFCs: 

* http://www.rfc-base.org/txt/rfc-3315.txt

* https://www.ietf.org/rfc/rfc3633.txt
