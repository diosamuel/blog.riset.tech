---
title: "Running DuckDB Quack Protocol Securely Over Tailscale"
description: "Deploy a DuckDB Quack server on a Raspberry Pi and access it remotely through Tailscale for secure, private database access."
date: "June 23 2026"
draft: false
---

> AI generated content for documentation purposes only, but struggles are real

## Introduction

DuckDB recently introduced Quack, a remote protocol that allows a DuckDB client to connect to a remote DuckDB instance over HTTP. This makes it possible to query, read, and write data on a remote database without manually transferring files.

In this tutorial, we'll deploy a DuckDB Quack server on a Raspberry Pi and access it remotely through Tailscale. This approach avoids exposing the service to the public internet while still providing secure remote access from anywhere.

## Architecture

The setup looks like this:

```
Laptop
   |
   | Tailscale VPN
   v
100.x.x.x
   |
   v
Raspberry Pi
   |
   +-- DuckDB
   +-- Quack Server (Port 9494)
```

Instead of opening firewall ports or configuring port forwarding, Tailscale creates a private network between devices.

## Prerequisites

Before starting, ensure you have:

- A Raspberry Pi running Linux
- DuckDB 1.5.2 or later
- A Tailscale account
- A client machine with DuckDB installed

## Step 1: Install DuckDB

Install DuckDB on both the Raspberry Pi and your client machine.

```bash
curl https://install.duckdb.org | sh
```

Verify installation:

```bash
duckdb --version
```

## Step 2: Create a Database

Launch DuckDB:

```bash
duckdb warehouse.duckdb
```

Create a sample table:

```sql
CREATE TABLE users (
    id INTEGER,
    name VARCHAR
);

INSERT INTO users VALUES
(1, 'dio'),
(2, 'sam');
```

Verify:

```sql
SELECT * FROM users;
```

## Step 3: Install the Quack Extension

Inside DuckDB:

```sql
INSTALL quack FROM core_nightly;
LOAD quack;
```

Verify that the extension loads successfully.

## Step 4: Start the Quack Server

Launch the Quack server and bind it to all interfaces:

```sql
CALL quack_serve('quack:0.0.0.0:9494',allow_other_hostname=true,token='token');
```

By default, Quack listens on port 9494.

Open another terminal and verify:

```bash
ss -tulpn | grep 9494
```

Expected output:

```
0.0.0.0:9494
```

If you see:

```
127.0.0.1:9494
```

the server will not be reachable through Tailscale.

## Step 5: Install Tailscale

Install Tailscale on the Raspberry Pi:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start the daemon:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate:

```bash
sudo tailscale up
```

Follow the login URL shown in the terminal.

## Step 6: Get the Tailscale IP Address

Retrieve the Raspberry Pi's Tailscale address:

```bash
tailscale ip -4
```

Example:

```
100.70.53.55
```

This address will be used by remote DuckDB clients.

## Step 7: Verify Connectivity

From the client machine:

```bash
ping 100.70.53.55
```

Check if the Quack port is reachable:

```bash
nmap -p 9494 100.70.53.55
```

Or:

```bash
nc -vz 100.70.53.55 9494
```

Expected:

```
9494/tcp open
```

## Step 8: Connect from DuckDB Client

Launch DuckDB on your laptop:

```bash
duckdb
```

Install and load Quack:

```sql
INSTALL quack FROM core_nightly;
LOAD quack;
```

Attach the remote database:

```sql
ATTACH 'quack:100.70.53.55:9494' AS remote (
    TOKEN 'token',
    DISABLE_SSL true
);
```

## Why DISABLE_SSL Is Required

When connecting to a non-local host, the DuckDB client automatically attempts an HTTPS connection:

```
https://100.70.53.55:9494/quack
```

However, a basic Quack server does not provide TLS certificates by default. As a result, the client may fail with:

```
SSL connect error
```

Since Tailscale already encrypts traffic end-to-end, disabling SSL at the Quack layer is generally acceptable for a private Tailnet.

```sql
ATTACH 'quack:100.70.53.55:9494' AS remote (
    TOKEN 'token',
    DISABLE_SSL true
);
```

## Step 9: Query Remote Tables

List databases:

```sql
SHOW DATABASES;
```

Example:

```
memory
remote
```

Query a remote table:

```sql
SELECT * FROM remote.users;
```

Result:

```
1  dio
2  sam
```

Count rows:

```sql
SELECT COUNT(*)
FROM remote.users;
```

Insert data remotely:

```sql
INSERT INTO remote.users
VALUES (3, 'tailscale');
```

Verify:

```sql
SELECT * FROM remote.users;
```

## Step 10: Expose via Tailscale Funnel

Tailscale Funnel allows you to expose a service from your Tailnet to the public internet using a Tailscale-provided hostname. This is useful when you want to share the DuckDB Quack server with users outside your Tailnet without exposing your home IP directly.

### Enable Funnel on Port 9494

On the Raspberry Pi, run:

```bash
tailscale funnel 9494
```

This exposes port 9494 publicly. Tailscale will assign a stable HTTPS hostname like:

```
https://raspberry-pi.tail12345.ts.net
```

### Get the DNS Name

To find the assigned hostname, run:

```bash
tailscale funnel status
```

Or check via the admin console at https://login.tailscale.com/admin/machines. The hostname will be visible under your machine's funnel configuration.

### Connect Using the Funnel DNS

Once Funnel is active, you can connect using the Tailscale-assigned DNS name:

```sql
ATTACH 'quack:raspberry-pi.tail12345.ts.net:9494' AS remote (
    TOKEN 'token',
    DISABLE_SSL true
);
```

Tailscale Funnel terminates TLS automatically, so HTTPS traffic reaches your service through Tailscale's infrastructure. This means:

- No need to manage TLS certificates yourself
- Your home IP is not exposed
- The connection is encrypted end-to-end by Tailscale

> **Note:** Funnel requires that your Tailscale account has Funnel enabled and your machine is authenticated. Check https://tailscale.com/kb/1223/tailscale-funnel for details.

## Step 11: Troubleshooting

### Check if Quack is Running

```bash
ss -tulpn | grep 9494
```

### Check Tailscale Status

```bash
tailscale status
```

### Verify Port Reachability

```bash
nc -vz 100.70.53.55 9494
```

### Common SSL Error

```
SSL connect error
```

Solution:

```sql
ATTACH 'quack:100.70.53.55:9494' AS remote (
    TOKEN 'token',
    DISABLE_SSL true
);
```

## Conclusion

Combining DuckDB Quack with Tailscale creates a lightweight and secure remote database architecture without requiring:

- Public IP addresses
- Port forwarding
- Reverse proxies
- Domain names
- Cloud infrastructure

For homelab projects, Raspberry Pi deployments, and lightweight data platforms, this setup provides a simple way to access DuckDB remotely while keeping the service private within your Tailnet.