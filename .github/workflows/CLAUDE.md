# GitHub Actions Workflow Security Guidelines

## Preventing Command Injection Vulnerabilities

### Rule: Never use `${{ }}` directly in shell scripts

**IMPORTANT**: Direct interpolation of GitHub Actions context expressions (`${{ }}`) in shell scripts can lead to command injection vulnerabilities.

### ❌ BAD: Direct interpolation
```yaml
- name: Example step
  run: |
    echo "Version is ${{ steps.version.outputs.value }}"
    gh release create "${{ github.event.inputs.tag }}"
```

### ✅ GOOD: Use environment variables
```yaml
- name: Example step
  env:
    VERSION: ${{ steps.version.outputs.value }}
    TAG: ${{ github.event.inputs.tag }}
  run: |
    echo "Version is ${VERSION}"
    gh release create "${TAG}"
```

## Why This Matters

GitHub Actions context values can contain arbitrary user input from:
- PR titles and descriptions
- Issue comments
- Workflow dispatch inputs
- Branch names
- Commit messages

If these values are directly interpolated into shell scripts, they can execute arbitrary commands.

## Example Attack Scenario

If a malicious user creates a PR with title: `"; rm -rf /; echo "` and this is used directly:
```yaml
# DANGEROUS
run: echo "PR title: ${{ github.event.pull_request.title }}"
```

This would execute: `echo "PR title: "; rm -rf /; echo ""`

## Safe Patterns

1. **Always use environment variables** for any `${{ }}` values in shell scripts
2. **Quote variables** in shell scripts: `"${VAR}"` not `$VAR`
3. **For complex cases**, consider using intermediate files or GitHub Actions outputs

## References

- [GitHub Security Lab: Keeping your GitHub Actions and workflows secure](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)
- [GitHub Docs: Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
