# Repository Policy

## Canonical Ownership

Every editable artifact has exactly one canonical repository. ISO profiles and
package recipes consume pinned artifacts or generated outputs; they do not keep
independently editable copies of catalogs, application source, artwork, or
transaction logic.

`governance/repositories.yaml` is the organization ownership registry. A product
repository must identify its executable surface, data contracts, privileged
boundary, test command, release artifact, and downstream consumers.

## Creation Gate

A new system-software repository requires all of the following:

1. An approved product boundary distinct from existing repositories.
2. Linxira-owned implementation or code ready to migrate from a named source.
3. Automated tests for its public and privileged contracts.
4. A maintainer and release/update responsibility.
5. A packaging and consumer migration plan.

Empty roadmap repositories are prohibited. Update Agent, Driver Manager, Kernel
Manager, and Recovery gain repositories when their first independently testable
implementation exists.

## Migration

Repository extraction is atomic at the ownership level:

1. Establish and test the new canonical source.
2. Update packages and consumers to use a pinned artifact from that source.
3. Remove the old editable implementation.
4. Leave only a migration notice where external consumers require it.

Copying a component while both copies remain editable is not a migration.

## Deprecation

Superseded repositories are archived, remain readable, and identify their
replacement in the description and README. They must not publish packages,
release artifacts, or active documentation after archival.

## Protection

Active system repositories require pull-request tests, non-force-pushed default
branches, pinned CI actions, least-privilege workflow permissions, and no secrets
in source. Signing credentials are available only to the release publication
environment.
