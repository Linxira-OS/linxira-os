# Release Manifests

Every promoted ISO has one immutable JSON manifest validated against
`manifest.schema.json`. A manifest binds the artifact to its exact source,
package cohort, build environment, and acceptance evidence.

Rejected diagnostics may be recorded, but their `acceptance` value must remain
`rejected-diagnostic`; they are not release candidates.

Required evidence includes source commits, Arch and Linxira package manifests,
repository database hashes, ISO SHA-256, SBOM path and hash, builder identity,
firmware boot results, disk-install results, both kernel/initramfs results, first
boot, Package Center transaction, and recovery acceptance.
