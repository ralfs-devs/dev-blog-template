---
title: V-Server Setup
sidebar_position: 1
---

# V-Server Setup

This guide walks you through setting up and securing your own cloud  
server instance running Ubuntu 24.04 LTS – from the initial SSH login  
to key-only authentication, a custom NGINX web page on port 8081  
and a complete Git/GitHub configuration.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quickstart](#quickstart)
3. [Description](#description)
   - [1. Create an SSH Key Pair](#1-create-an-ssh-key-pair)
   - [2. Copy Your Public Key to the Server](#2-copy-your-public-key-to-the-server)
   - [3. Verify Key-Based Login](#3-verify-key-based-login)
   - [4. Disable Password and Root Login](#4-disable-password-and-root-login)
   - [5. Install and Configure NGINX](#5-install-and-configure-nginx)
   - [6. Configure Git and GitHub Access](#6-configure-git-and-github-access)
   - [7. Final Verification](#7-final-verification)

## Prerequisites

Before you start, make sure you have the following:  
- A virtual server (or VM) running **Ubuntu 24.04 LTS**  
- SSH access to the server with username and password (initial setup)  
- A local machine with a terminal and OpenSSH installed  
- A GitHub account (for the repository interaction in Section 6)  

## Quickstart

1. Generate an SSH key pair on your local machine (never on the server).  
2. Log in to the V-Server via SSH using your username and password.  
3. Add your public key to the server's authorized_keys with:  
   `ssh-copy-id -i $HOME/.ssh/[keyname]_ed25519.pub [user]@[SERVER_IP]`  
4. Log out of the server, then log back in using the key only:  
   `ssh -i $HOME/.ssh/[keyname]_ed25519 [user]@[SERVER_IP]`  
   – you should not be prompted for a password.  
5. Disable password login and root login on the server  
   (see [Section 4](#4-disable-password-and-root-login)).
6. Install NGINX and serve a custom HTML page on port 8081  
   (see [Section 5](#5-install-and-configure-nginx)).
7. Configure your Git identity and a server-side SSH key for GitHub  
   (see [Section 6](#6-configure-git-and-github-access)).

## Description

The following sections describe each step in detail, including the
commands to run and the output to expect.

### 1. Create an SSH Key Pair

Generate the key pair on your **local machine**.
An Ed25519 key is recommended:

```bash
ssh-keygen -t ed25519 -C "[your-email@example.com]"
```

This creates the private key `~/.ssh/[keyname]_ed25519`
and the public key `~/.ssh/[keyname]_ed25519.pub`. 

### 2. Copy Your Public Key to the Server

While password login is still active, transfer the public key:

```
ssh-copy-id -i $HOME/.ssh/[keygen]_ed25519.pub [user]@[SERVER_IP]
```

**Caution:** Don't forget the Extension .pub in order not to transfer  
the private key. Confirm the transfer with your password when prompted.  
The key is then appended to ~/.ssh/authorized_keys on the server.

### 3. Verify Key-Based Login

Log out of the server and log back in using only the key:

```
ssh -i $HOME/.ssh/[keyname]_ed25519 [user]@[SERVER_IP]
```

You should be logged in without any password prompt.  
Do not continue with Section 4 until this works reliably,  
otherwise you will lock yourself out of the server.

### 4. Disable Password and Root Login

Open the SSH daemon configuration:

```
sudo nano /etc/ssh/sshd_config
```

Set or uncomment the following directives:

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

**Warning:** Watch out for drop-in files: on many cloud images,  
the file /etc/ssh/sshd_config.d/50-cloud-init.conf  
overrides the main configuration, because include files are processed first  
and the first value wins.  
Make sure this file does not re-enable password authentication:

```
sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
```

Apply the changes:

```
sudo systemctl restart sshd
```

Verify the effective configuration (do not trust the config files alone):

```
sudo sshd -T | grep -E "passwordauthentication|pubkeyauthentication|permitrootlogin"
```

Expected output:

```
permitrootlogin no (or without-password)  
pubkeyauthentication yes  
passwordauthentication no  
```

### 5. Install and Configure NGINX

Install the web server:

```
sudo apt update && sudo apt install nginx -y
```

Create the directory for your custom page and place your HTML file there:

```
sudo mkdir -p /var/www/alternative
sudo touch alternate-index.html
sudo cp alternate-index.html /var/www/alternative/
```

Configure NGINX to listen on port 8081 and serve this page
as the entry point:

```
sudo nano /etc/nginx/sites-available/default
```

```nginx
server {
    listen 8081;
    listen [::]:8081;
    root /var/www/alternative;
    index alternate-index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Validate the configuration before applying it:

```
sudo nginx -t
```

Expected output: 

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful  
```

Reload NGINX:

```
sudo systemctl reload nginx
```

Open http://[SERVER_IP]:8081 in your browser to confirm  
that your custom page is served.

### 6. Configure Git and GitHub Access

Set your Git identity on the server so it matches  
the data stored in your GitHub account:

```
git config --global user.name "[Your Name]"
git config --global user.email "[your-email@example.com]"
```

To interact with GitHub repositories from the server,  
generate a dedicated SSH key pair on the server:

```
ssh-keygen -t ed25519 -C "[your-email@example.com]"
```

Print the public key  
and add it to your GitHub account under Settings → SSH and GPG keys:

```
cat ~/.ssh/[key_id]_ed25519.pub
```

Verify the connection:

```
ssh -T git@github.com
```

You should see a greeting with your GitHub username.

### 7. Final Verification

Run these checks from your local machine:

Key-based login still works:

```
ssh [user]@[SERVER_IP]
```

Password login is rejected:

```
ssh -o PubKeyAuthentication=no [user]@[SERVER_IP]
```

Expected output:

```
[user]@[SERVER_IP]: Permission denied (publickey).
```

The server announces publickey as the only accepted authentication method  
no password prompt is offered at all.

The web server responds on port 8081: open http://[SERVER_IP]:8081  
in your browser.

