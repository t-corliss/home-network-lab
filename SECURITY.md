# Security Guidelines

## Purpose

This repository contains network documentation and sanitized examples for the home network lab.

Sensitive operational data should not be committed to Git, even when the repository is private.

## Do Not Commit

The following material should remain outside the repository:

- pfSense configuration backups
- passwords
- Wi-Fi credentials
- captive portal credentials
- API tokens
- private keys
- certificates containing private material
- authentication cookies or session data
- raw credential exports
- sensitive public IP information
- device serial numbers when not required for documentation
- unredacted screenshots containing secrets or account information

## Sanitized Documentation

Configuration examples may be included only when sensitive values have been removed or replaced with placeholders.

Internal RFC1918 addressing may be documented when useful for understanding the lab architecture.

## Review Before Commit

Before committing a new file, verify that it does not contain:

- credentials
- secrets
- private keys
- authentication tokens
- live configuration backups
- unnecessary personally identifiable or device-specific information

If a sensitive file is accidentally committed, do not assume that deleting it in a later commit removes it from repository history.