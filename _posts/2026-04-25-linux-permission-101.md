---
title: Linux Permission 101
date: 2026-04-25 09:00:00 +0600
categories: [linux, fundamentals]
tags: [linux, permissions, chmod, security]
description: A beginner-friendly guide to Linux users, groups, and numeric permissions with practical chmod examples.
---

Linux permissions are one of the first things you should understand if you want to work comfortably and safely on Linux systems.

This guide covers the essentials in a practical way.

## 1. Identity: Users vs. Groups

Linux is a multi-user operating system. To manage access, each file and directory is tied to:

- A **user owner** (the primary account that owns it)
- A **group owner** (a collection of users associated with it)

Why this matters:

- You can keep one primary owner while granting team access through a group.
- You can control security without constantly changing file ownership.

### Switching Users with `su`

You can switch between user accounts with the `su` command.

```bash
su -l username
```

Using `-l` (login) is a best practice because it loads that user's shell environment and starts you in their home directory.

## 2. Actions: Read, Write, and Execute

Linux permissions are built from three core actions:

- **Read (`r`)**: View the contents of a file.
- **Write (`w`)**: Modify a file (and, depending on context, remove or replace it).
- **Execute (`x`)**: Run a file as a program or script.

These actions are applied separately to:

- **Owner**
- **Group**
- **Others**

## 3. The Math: Numeric Permissions

Linux shows permissions as letters (`rwx`), but it stores and applies them as numbers.

| Permission | Numeric Value |
| --- | --- |
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |
| None (`-`) | 0 |

### How to Calculate

Permissions always come in three sets: **Owner**, **Group**, and **Others**.

Add the values in each set:

Example: `rwx r-x r--`

- Owner (`rwx`) = 4 + 2 + 1 = **7**
- Group (`r-x`) = 4 + 0 + 1 = **5**
- Others (`r--`) = 4 + 0 + 0 = **4**

Result: **754**

## Common Permission Codes

- **777**: Everyone can read, write, and execute. High security risk.
- **755**: Owner can edit; others can read and execute.
- **644**: Owner can edit; others can only read.
- **700**: Only owner has access.

## Why This Matters in Real Work

You apply numeric permissions with `chmod` (change mode):

```bash
chmod 750 secret.txt
```

This means:

- Owner gets `7` (read, write, execute)
- Group gets `5` (read, execute)
- Others get `0` (no access)

## Quick Mental Model

- Start from least privilege.
- Give only the access needed.
- Avoid `777` unless you fully understand the risk and have a temporary reason.

Once permissions click, Linux administration gets much easier and much safer.