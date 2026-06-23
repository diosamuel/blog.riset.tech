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
CALL quack_serve(
    'quack:0.0.0.0:9494'
);
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

## Step 10: Troubleshooting

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