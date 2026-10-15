# Configuration Files

## Purpose

This directory is reserved for sanitized configuration examples and templates related to the home network lab.

Live configuration backups should not be stored in this repository.

## Allowed Content

Examples of acceptable content include:

- sanitized configuration snippets
- generic templates
- redacted examples
- documentation-only configuration samples
- non-sensitive reference files

## Prohibited Content

Do not store:

- raw pfSense configuration backups
- passwords
- private keys
- certificates containing private material
- API tokens
- Wi-Fi credentials
- captive portal credentials
- unredacted exports containing sensitive operational data

## Sanitization Requirement

Any configuration example added to this directory should be reviewed before commit.

Sensitive values should be removed or replaced with clearly identifiable placeholders.

For example:

```text
username = EXAMPLE_USER
password = REDACTED
api_token = REDACTED
```

Internal RFC1918 addressing may remain when it is useful for understanding the network design.