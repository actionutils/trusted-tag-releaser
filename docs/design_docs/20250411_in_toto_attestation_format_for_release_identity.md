---
title: "In-toto Attestation Format for Release Identity"
date: 2025-04-11
author: haya14busa
status: draft
---

# 2025-04-11: In-toto Attestation Format for Release Identity

## Background

The trusted-tag-releaser project currently generates a `release-identity.json` file that contains metadata about a release, including repository name, version, timestamp, and git information. This file is used to verify the authenticity of releases.

As part of aligning with the in-toto Attestation Framework, we should also consider renaming this file to follow in-toto conventions.

Currently, this file uses a custom JSON format that doesn't follow any standard. There's an opportunity to align this with the [in-toto Attestation Framework](https://github.com/in-toto/attestation), which would provide better interoperability with other supply chain security tools and standards.

## Current Format

The current format of the `release-identity.json` file is:

```json
{
  "name": "<repository-name>",
  "version": "<tag-name>",
  "timestamp": "<ISO-8601-timestamp>",
  "description": "This file contains identity information about this GitHub release. It can be verified with the release-identity.intoto.jsonl file to confirm the authenticity and build source of this release.",
  "git": {
    "commit": "<commit-sha>",
    "tag": "<tag-name>",
    "repository": "https://github.com/<owner>/<repo>"
  }
}
```

## In-toto Attestation Framework Overview

The in-toto Attestation Framework defines a standard format for supply chain security attestations. The basic structure is:

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "<URI identifying the predicate type>",
  "subject": [
    {
      "name": "<name>",
      "digest": {"<algorithm>": "<digest value>"}
    }
  ],
  "predicate": {
    // Predicate-specific fields
  }
}
```

Key components:
- `_type`: Identifies this as an in-toto Statement
- `predicateType`: URI identifying the type of attestation
- `subject`: The artifact(s) this attestation is about
- `predicate`: Type-specific data about the subject

## Standard Predicate Types Analysis

The in-toto ecosystem defines several standard predicate types:

### 1. SLSA Provenance (`https://slsa.dev/provenance/v0.2`)

This predicate type describes how an artifact was built, including:
- Builder information
- Build parameters
- Materials used in the build

**Applicability**: While our release identity contains some similar information, it doesn't include build details. The SLSA Provenance is focused on build provenance rather than release identity.

### 2. SPDX (`https://spdx.dev/Document`)

This predicate type represents a Software Bill of Materials (SBOM) in SPDX format.

**Applicability**: Not applicable, as our file doesn't contain dependency information.

### 3. CycloneDX (`https://cyclonedx.org/bom`)

Another SBOM format.

**Applicability**: Not applicable for the same reason as SPDX.

### 4. VEX (`https://openvex.dev/ns/v0.1.0`)

Vulnerability Exploitability Exchange format for communicating vulnerability information.

**Applicability**: Not applicable, as our file doesn't contain vulnerability information.

## Custom Predicate Type Options

Since none of the standard predicate types perfectly match our use case, we have several options:

### Option 1: Define a Custom Release Identity Predicate Type

We could define a custom predicate type specifically for release identity information:

```
https://github.com/actionutils/trusted-tag-releaser/in-toto/release-identity/v1
```

This URL structure makes the spec directly visible and follows a more standard pattern for in-toto predicates.

This would allow us to structure our data in a way that makes sense for our specific use case while still conforming to the in-toto Statement format.

### Option 2: Use SLSA Provenance with Extensions

We could use the SLSA Provenance predicate type and extend it with our specific fields. This would leverage an existing standard but might require some field mapping that isn't perfectly aligned.

### Option 3: Use a Minimal Statement without a Specific Predicate Type

We could use a generic predicate type just to conform to the basic in-toto Statement structure, without committing to a specific predicate schema.

## Recommendation

**Option 1: Define a Custom Release Identity Predicate Type** is recommended for the following reasons:

1. It allows us to precisely model our release identity data
2. It maintains compatibility with in-toto tooling
3. It clearly communicates the purpose of the attestation
4. It provides flexibility for future extensions

The proposed format would be:

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://github.com/actionutils/trusted-tag-releaser/in-toto/release-identity/v1",
  "subject": [
    {
      "name": "<repository-name>",
      "digest": {
        "git": "<commit-sha>"
      }
    }
  ],
  "predicate": {
    "version": "<tag-name>",
    "created_at": <unix-timestamp>,
    "git": {
      "tag": "<tag-name>",
      "repository": "https://github.com/<owner>/<repo>"
    }
  }
}
```

Key changes:
- Added `_type` field to identify this as an in-toto Statement
- Added `predicateType` with a custom URI
- Restructured data into `subject` and `predicate` sections
- Used the git commit SHA as a digest in the subject
- Moved other metadata to the predicate section

Field descriptions:
- `created_at`: Unix timestamp (seconds since epoch) when the attestation was generated during the release process
- `version`: The semantic version tag of the release
- `git.tag`: The Git tag name for the release
- `git.repository`: The GitHub repository URL

## Implementation Plan

1. Create a new branch from the current PR
2. Update the JSON generation in `trusted-release-workflow.yml` to use the new format
3. Update the verification logic to handle the new format
4. Add documentation about the in-toto compatibility
5. Test the changes with the existing verification workflow

## Specification Documentation

To properly document our custom predicate type, we should create a specification that follows in-toto conventions:

1. **Specification README**: Create a `README.md` file at the path corresponding to our predicate type URL:
   ```
   in-toto/release-identity/v1/README.md
   ```
   This file would document the purpose, structure, and usage of our predicate type, making it accessible at the URL we reference in the attestation.

   The README should follow the [in-toto predicate template](https://github.com/in-toto/attestation/blob/main/spec/predicates/template/template.md) and include:
   
   - Overview and purpose of the predicate
   - URI identifier
   - Predicate schema (JSON and protobuf)
   - Examples
   - Security considerations
   - Compatibility with other attestation types

2. **Formal Schema Definition**: Define the format using a schema definition language like Protocol Buffers (protobuf) or JSON Schema. This provides:
   - Clear, machine-readable specification of the format
   - Type checking and validation capabilities
   - Code generation for multiple languages
   - Versioning and evolution of the schema

Example protobuf definition for our release identity predicate:

```protobuf
syntax = "proto3";

package actionutils.trusted_tag_releaser.in_toto.release_identity.v1;

message ReleaseIdentityPredicate {
  // The semantic version tag of the release
  string version = 1;
  
  // Unix timestamp (seconds since epoch) when the attestation was generated during release
  int64 created_at = 2;
  
  // Git-specific information about the release
  GitInfo git = 3;
  
  message GitInfo {
    // The Git tag name for the release
    string tag = 1;
    
    // The GitHub repository URL
    string repository = 2;
  }
}
```

This schema would be stored alongside the README.md in the specification directory.

## Security Model and Attestation Chain

It's important to clarify the security model for the release identity file:

1. The `release-identity.intoto.jsonl` file itself is **not directly signed**
2. Instead, it is treated as an **artifact** that:
   - Has its hash included in SLSA provenance attestations
   - Gets uploaded to the GitHub release
   - Is referenced by other attestations

The security chain works as follows:

```
release-identity.intoto.jsonl
  ↓ (hash included in)
provenance.intoto.jsonl (SLSA provenance attestation)
  ↓ (signed by)
GitHub OIDC token (via SLSA GitHub generator)
```

This approach follows the in-toto principle of having attestations about artifacts rather than self-signed artifacts. The authenticity and integrity of the release identity file are ensured by:

1. The SLSA provenance attestation that includes its hash
2. The GitHub OIDC token that signs the provenance attestation
3. The verification of the entire chain during the release process

This model provides stronger security guarantees than simply signing the file directly, as it establishes a verifiable chain of custody from the GitHub workflow to the release artifacts.

### Provenance File Naming

The default behavior of the SLSA GitHub generator is to name the provenance file after the artifact with an additional `.intoto.jsonl` extension, which would result in the awkward name `release-identity.intoto.jsonl.intoto.jsonl`. We should specify a custom name for the provenance file to avoid this.

Options for the provenance file name:

| Option | Pros | Cons |
|--------|------|------|
| `provenance.intoto.jsonl` | - Clear purpose<br>- Standard term in SLSA<br>- Simple and concise | - Generic name if multiple attestations exist |
| `attestation.intoto.jsonl` | - Generic term for any attestation<br>- Follows in-toto terminology | - Less specific about the type of attestation |
| `release-provenance.intoto.jsonl` | - Clearly relates to the release<br>- Distinguishes from other potential provenances | - Longer name |
| `slsa-provenance.intoto.jsonl` | - Explicitly indicates SLSA compliance<br>- Clear about the standard being followed | - Longer name |

**Recommendation**: Use `provenance.intoto.jsonl` as it's concise, clear, and aligns with SLSA terminology. If multiple attestation types are added in the future, we can adopt a more specific naming convention at that time.

This can be specified in the SLSA GitHub generator workflow using the `provenance-name` parameter.

## File Naming Convention

In the in-toto framework, attestation files typically use the `.intoto.jsonl` extension. Following this convention, we should rename our file from `release-identity.json` to `release-identity.intoto.jsonl`.

This naming convention has several benefits:
- Clearly identifies the file as an in-toto attestation
- Follows established community standards
- Enables automatic recognition by in-toto tooling
- The `.jsonl` extension indicates that the file contains a single JSON object per line, which is the standard format for in-toto attestations

## Implementation Details

Since there are no existing users yet, we can implement this change without backward compatibility concerns. The implementation should focus on:

1. Creating the specification documentation and schema
2. Structuring the data according to the in-toto Statement format
3. Renaming the file from `release-identity.json` to `release-identity.intoto.jsonl`
4. Updating the verification logic to validate the new format
5. Adding documentation about the in-toto compatibility and how to verify the attestation

## Future Extensions

While this design document focuses on adopting the in-toto framework structure, there are several potential extensions that could be explored in future design documents. These extensions are motivated by concrete security goals, particularly advancing toward higher SLSA levels.

### Motivation

1. **Achieving SLSA Builder Level 4**: SLSA Level 4 requires additional security controls such as:
   - Two-person review of all changes
   - Hermetic, reproducible builds
   - Provenance verification
   
2. **Enhanced Supply Chain Security**: Providing more comprehensive attestations about the release process enables:
   - More thorough verification of release integrity
   - Better audit trails for security incidents
   - Compliance with emerging supply chain security regulations

3. **Ecosystem Integration**: Aligning with broader supply chain security initiatives like:
   - SLSA framework
   - Sigstore ecosystem
   - OpenSSF best practices

### Potential Extensions

1. **PR Data Collection**:
   - Adding pull request metadata (author, reviewers, approval status)
   - Enables verification of proper review processes
   - Supports SLSA Level 4 requirement for two-person review
   - Allows policy enforcement (e.g., requiring specific reviewers for sensitive changes)

2. **Build Environment Attestation**:
   - Including information about the build environment
   - Supports SLSA requirements for hermetic builds
   - Enables verification that releases were built in trusted environments

3. **Dependency Attestation**:
   - Linking to SBOMs or other dependency information
   - Enables tracking of transitive dependencies
   - Supports vulnerability management across the supply chain

4. **Custom Verification Rules**:
   - Defining project-specific rules for what constitutes a valid release
   - Enables policy-as-code for release verification
   - Supports organizational governance requirements

These extensions would build upon the in-toto framework structure established in this design, but would be specified and implemented separately as the project matures and security requirements evolve.

## References

- [in-toto Attestation Framework](https://github.com/in-toto/attestation)
- [SLSA Provenance Format](https://slsa.dev/provenance/v0.2)
- [in-toto Statement Format](https://github.com/in-toto/attestation/tree/main/spec/v0.1.0#statement)