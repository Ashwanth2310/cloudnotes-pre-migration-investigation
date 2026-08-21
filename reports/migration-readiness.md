# Migration Readiness Review
CloudNotes is not migration-ready in its current form. The service configuration
declares `127.0.0.1:5000`, which limits access to the local machine and must be
reconciled with the endpoint and traffic-routing configuration before deployment.
The architecture also has single-instance tiers, an unreplicated database, and
unencrypted network traffic, creating availability, recovery, and security risks.

## Virtualization Consideration

Virtual machines provide isolation between the web, application, and database
tiers, but virtualization alone does not provide high availability or disaster
recovery. Each tier currently has one VM, so a VM failure takes that tier out of
service. The database is especially vulnerable because it is the sole source of
truth and has no scheduled replica or automated backup described. Before
migration, define backup retention and restore testing, then deploy redundant
instances across separate failure domains where the workload and budget allow.

## Distributed Systems Risk

The topology has a single point of failure at every tier. There is no traffic
distribution layer, so the Web VM cannot fail over to another web instance, and
the single Application VM cannot absorb a host failure. The single Database VM
also creates both an availability risk and a potential data-loss risk. Adding
load balancing and multiple stateless web/application instances improves
availability, but requires health-based routing, session handling that does not
depend on local VM state, and explicit database replication or managed database
failover. Cross-tier traffic should use authenticated transport encryption.

## Cloud Responsibility Concern

Moving the VMs to a cloud provider would not automatically make the service
resilient or secure. The provider is responsible for the underlying cloud
infrastructure, but the CloudNotes team remains responsible for configuring
network access, identity and permissions, operating-system and application
patching, secrets, encryption, logging, backups, and recovery tests. The team
must also verify that the selected service tier actually provides the required
availability and backup guarantees; a single cloud VM would preserve the
current failure mode.

## Architecture Improvement

Place redundant web and application instances behind a managed load balancer
with health checks. Keep the application tier stateless, or move sessions to a
shared store, so instances can be replaced safely. Use a managed database with
automated backups, point-in-time recovery, encryption, and a tested failover or
replica strategy. Restrict network access so only the required tier-to-tier
paths are open, use TLS for user and internal traffic, and centralize logs and
monitoring with alerts. Finally, standardize the service configuration and
endpoint definition, including a deliberate bind address and port, and verify
the `/health` probe through the same route used by the load balancer.
