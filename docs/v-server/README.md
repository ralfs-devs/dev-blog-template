---
title: V-Server Setup
sidebar_position: 1
---

# V-Server Setup – Technical Documentation

## Table of Contents

1. [Overview](#overview)
2. [Access and Security](#access-and-security)
3. [Web Server](#web-server)
4. [Version Control](#version-control)
5. [Development Environment](#development-environment)
6. [Lessons Learned](#lessons-learned)

## Overview
- V-Server is my virtual server, which I set up 
for learning and practice purposes as 
part of a DevSecOps training program with my educational provider. 
- server provider: DA Developer Akademie GmbH, Munich 
- operating system: Ubuntu 24.04 LTS

## Access and Security

Password-based login is completely disabled. The server is accessible
exclusively via SSH key authentication:

- **Primary access:** locally generated Ed25519 key pair
- **Fallback credentials:** two hardware security sticks

The effective SSH daemon configuration was verified with `sshd -T`:

    sudo sshd -T | grep -E "passwordauthentication|pubkeyauthentication|permitrootlogin"
    permitrootlogin without-password
    pubkeyauthentication yes
    passwordauthentication no


The hardening was validated with two tests from the client:

    $ ssh user@<SERVER_IP>
    (login succeeds without any password prompt)

    $ ssh -o PubKeyAuthentication=no user@<SERVER_IP>
    user@<SERVER_IP>: Permission denied (publickey).

The second test proves that no password prompt is offered at all –
the server announces `publickey` as the only authentication method.

All access methods are managed via host aliases in the SSH config of
the client. Each alias (standard key, hardware stick, backup stick)
specifies its key file via `IdentitiesOnly yes`, so plugged-in
security sticks do not interfere with the standard login.

### Observed Threat Activity

Within 24 hours of operation,
more than 1,000 automated SSH login attempts
were recorded on the server, targeting common usernames such as `root`:

    Connection closed by authenticating user root <IP> port 44384 [preauth]

The `[preauth]` marker shows that these attempts fail before any
credential check occurs: the server only offers public key
authentication, so password-guessing bots disconnect immediately.
This observation confirms the effectiveness of the key-only policy.

## Web Server
- NGINX installation
- Alternative HTML page at /var/www/alternatives/alternate-index.html
- Permissions on the web root
- Validity checked with `nginx -t`: 
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

## Version Control
- Git configured on the server (user.name, user.email)
- SSH config backup in a dedicated repository
- GitHub integration of the server via its own SSH key

## Development Environment
- VS Code Remote-SSH (isolated instance)

## Lessons Learned

### Configuration Precedence Pitfall
After setting `PasswordAuthentication no` in `/etc/ssh/sshd_config`, password
prompts still appeared. Diagnosis: `/etc/ssh/sshd_config.d/50-cloud-init.conf`
(created by the cloud-init template) overrides the main config, because
include files are processed first and the first value wins in OpenSSH.

**Fix:** Align the drop-in with the intended policy (`PasswordAuthentication no`).

**Lesson:** Always verify effective configuration with `sshd -T` instead of
trusting the main config file.

### NGINX Syntax Sensitivity
Adding a semicolon to the `try_files` directive resolved a configuration
error. The parser fails at the next token (closing brace), making the error
message ("unexpected `}`") misleading – the real issue was the missing
semicolon on the previous line.

**Lesson:** NGINX (like C-style languages) requires semicolons after every
directive. Python developers need to be aware of this context switch.

### Security Through Hardening
With password authentication disabled, all 1,000+ automated login attempts
failed immediately with `[preauth]` markers, proving the effectiveness of
key-only authentication. Without this measure, the server would be
vulnerable to brute-force attacks.

## References

- Course materials and setup guide, DA Developer Akademie GmbH
- Project FAQs provided by DA Developer Akademie GmbH
- `man sshd_config` – OpenSSH daemon configuration reference
- `man nginx` – NGINX command-line reference
