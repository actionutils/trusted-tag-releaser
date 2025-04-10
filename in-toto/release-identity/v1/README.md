# Predicate type: Release Identity

Type URI: `https://github.com/actionutils/trusted-tag-releaser/in-toto/release-identity/v1`

Version: 1

Predicate Name: Release Identity

## Purpose

The Release Identity predicate provides a standardized way to attest to the identity and origin of a release. It captures essential metadata about a release, including version information, creation timestamp, and Git repository details. This predicate enables verification of release authenticity by establishing a clear link between a release artifact and its source repository.

This predicate is particularly useful for products where the tag itself is considered a release (GitHub Actions, reusable workflows, Go/Deno modules, Vim plugins, etc.). For these types of products, there might not be concrete artifacts, but the tag and the commit SHA it points to are critical pieces of information for verification. That's why this predicate includes both the tag and the SHA it references.

The Release Identity predicate is primarily designed to be used with the trusted-tag-releaser tool, which creates releases through labeled Pull Requests. In the future, this predicate could be extended to include information about whether the release PR was properly reviewed, providing additional verification capabilities for the release process.

## Use Cases

### Release Verification

When a release is created, it's important to be able to verify its authenticity and origin. The Release Identity predicate addresses this by providing:

1. A link to the specific Git commit that the release was built from
2. The tag associated with the release
3. The timestamp when the release was created
4. The repository where the release originated

Current predicate types like SLSA Provenance focus on how an artifact was built, but don't specifically address the release identity information. This predicate fills that gap by providing a standardized way to attest to release identity.

### Tag-Based Releases

Many GitHub-hosted projects use Git tags as the primary mechanism for releases. This is especially common for:

- GitHub Actions and reusable workflows
- Go and Deno modules
- Vim/Emacs plugins
- Other libraries and tools where the tag itself signifies a release

For these projects, the tag and the commit it points to are the most important pieces of information for verification. The Release Identity predicate provides a standardized way to capture and verify this information, ensuring that consumers can verify they're using a legitimate release from the expected source.

### Supply Chain Security

In the context of supply chain security, it's crucial to be able to verify that a release came from the expected source. The Release Identity predicate enables:

- Verification that a release artifact corresponds to a specific Git commit
- Confirmation that the release was created at the expected time
- Validation that the release came from the authorized repository

## Prerequisites

- The [in-toto Attestation Framework](https://github.com/in-toto/attestation) is required for understanding and using this predicate type.
- Git version control system concepts (commits, tags, repositories)
- Basic understanding of software release processes

## Model

The Release Identity predicate is relevant to the following supply chain phases:

- **Release**: The phase where software is packaged and published for distribution
- **Verification**: The phase where consumers verify the authenticity of the release

The predicate is created by the release functionary (typically a CI/CD system) and consumed by verifiers who want to confirm the authenticity of a release.

## Schema

### Parsing Rules

The Release Identity predicate follows the [standard parsing rules](https://github.com/in-toto/attestation/blob/main/spec/v0.1.0/README.md#parsing-rules) defined in the in-toto Attestation Framework. Additionally:

- Unknown fields in the predicate MUST be ignored by parsers
- The `created_at` field MUST be interpreted as seconds since the Unix epoch (January 1, 1970, 00:00:00 UTC)
- The `version` field SHOULD follow semantic versioning format, but this is not strictly required

### Fields

#### Statement Fields

The standard in-toto Statement fields apply:

- `_type`: MUST be `https://in-toto.io/Statement/v0.1`
- `subject`: MUST contain at least one entry with:
  - `name`: The name of the repository
  - `digest`: MUST contain a `git` key with the commit SHA as its value

#### Predicate Fields

- `predicateType`: MUST be `https://github.com/actionutils/trusted-tag-releaser/in-toto/release-identity/v1`
- `predicate`: An object with the following fields:
  - `version` (required, string): The semantic version tag of the release
  - `created_at` (required, integer): Unix timestamp (seconds since epoch) when the attestation was generated during release
  - `git` (required, object): Git-specific information with the following fields:
    - `tag` (required, string): The Git tag name for the release
    - `repository` (required, string): The GitHub repository URL

## Example

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://github.com/actionutils/trusted-tag-releaser/in-toto/release-identity/v1",
  "subject": [
    {
      "name": "trusted-tag-releaser",
      "digest": {
        "git": "abcdef1234567890abcdef1234567890abcdef12"
      }
    }
  ],
  "predicate": {
    "version": "v1.2.3",
    "created_at": 1712851200,
    "git": {
      "tag": "v1.2.3",
      "repository": "https://github.com/actionutils/trusted-tag-releaser"
    }
  }
}
```

## Verification

Verification of a Release Identity attestation should include:

1. Validating that the attestation follows the in-toto Statement format
2. Validating that the predicate follows the schema defined above
3. Verifying that the commit SHA in the subject's digest matches the SHA that the tag points to
4. Verifying that the tag in the predicate matches the tag in the repository


## Changelog and Migrations

This is the initial version (v1) of the Release Identity predicate type, so no migrations are necessary.