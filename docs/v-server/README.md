---
title: V-Server Setup
sidebar_position: 1
---

# V-Server Setup – Technical Documentation

## Overview
Purpose, server provider, operating system (Ubuntu 24.04 LTS).

## Access and Security
- Password login is disabled
- The server is therefore accessible exclusively via SSH key authentication
- The primary access is a locally generated Ed25519 key pair
- Two hardware security sticks serve as redundant fallback credentials

The access methods are managed via host aliases in the SSH config **of the client**:
Each access method (standard key, Thetis stick, backup stick) has its own
alias, which defines which key is used (`IdentitiesOnly yes`).
This way, plugged-in sticks do not interfere with the standard login.

## Web Server
- NGINX installation
- Alternative HTML page at /var/www/html/index.html
- Permissions on the web root
- Validity checked with `nginx -t`

## Version Control
- Git configured on the server (user.name, user.email)
- SSH config backup in a dedicated repository
- GitHub integration of the server via its own SSH key

## Development Environment
- VS Code Remote-SSH (isolated instance)

## Lessons Learned / References