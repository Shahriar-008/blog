---
title: Process 101 on Linux
date: 2026-04-25 16:10:00 +0600
categories: [linux, fundamentals]
tags: [linux, processes, pid, systemctl, terminal, notes]
description: My personal notes on Linux process management, including PIDs, signals, systemd, systemctl, and foreground/background jobs.
---

Today I revised Linux process management and wrote this note so I can quickly review it later.

Managing processes in Linux feels like conducting an orchestra. Every running program is a process, and Linux gives me clear tools to monitor, control, and organize them.

## 1. Tracking Process IDs (PIDs)

Each process gets a unique Process ID (PID) when it starts.

- PID 1 is usually `systemd` on modern Linux systems.
- New processes get new PIDs as they are created.

Commands I should remember:

```bash
ps
```

Shows a snapshot of processes in the current shell session.

```bash
ps aux
```

Shows almost all running processes across the system.

```bash
top
```

Live, real-time process monitor for CPU and memory usage.

## 2. Managing the Life of a Process (Signals)

If a process hangs or needs control, I can send a signal using its PID.

```bash
kill PID
```

Default signal is SIGTERM. This is the polite way to ask a process to stop.

```bash
kill -9 PID
```

Sends SIGKILL. This forcefully stops a process immediately.

```bash
kill -STOP PID
```

Pauses a running process.

```bash
kill -CONT PID
```

Resumes a paused process.

My rule: try SIGTERM first, use SIGKILL only when needed.

## 3. How Processes Are Organized (systemd, Parent/Child, Namespaces)

On most Linux distributions, `systemd` starts first and acts as the main parent process.

- Services and apps become child processes in a process tree.
- I can inspect this structure with commands like `pstree`.

Quick note on namespaces:

- Namespaces help isolate processes so one group cannot easily interfere with another.
- This is a key idea behind containers.

## 4. Service Automation with systemctl

For long-running services (like web servers), I should use `systemctl`.

| Command | What it does |
| --- | --- |
| `systemctl start SERVICE` | Starts the service now |
| `systemctl stop SERVICE` | Stops the service now |
| `systemctl restart SERVICE` | Restarts the service |
| `systemctl enable SERVICE` | Starts service automatically at boot |
| `systemctl disable SERVICE` | Stops auto-start at boot |
| `systemctl status SERVICE` | Shows current status and recent logs |

Example:

```bash
systemctl status apache2
```

## 5. Foreground vs Background Jobs

Linux terminal jobs can run in two ways:

- Foreground: command takes over the terminal.
- Background: command runs while I keep using the terminal.

Useful job control shortcuts:

```bash
long_command &
```

Starts a command directly in background.

```bash
Ctrl + Z
```

Suspends the current foreground job.

```bash
bg
```

Continues a suspended job in background.

```bash
fg
```

Brings a background/suspended job back to foreground.

```bash
jobs
```

Lists current jobs in this shell.

## Quick Revision Table

| If I want to... | Use this |
| --- | --- |
| See running processes | `ps aux` or `top` |
| Stop a stuck process politely | `kill PID` |
| Force stop a process | `kill -9 PID` |
| Pause and resume a process | `kill -STOP PID` and `kill -CONT PID` |
| Start a service now | `systemctl start SERVICE` |
| Start a service on boot | `systemctl enable SERVICE` |
| Suspend current task | `Ctrl + Z` |
| Resume in background | `bg` |
| Bring task back to focus | `fg` |

## What I Learned Today

- Every program is a process with a PID.
- Signals are the safest way to control process behavior.
- `systemd` and `systemctl` are central for service management.
- Job control (`Ctrl + Z`, `bg`, `fg`) is very useful for terminal multitasking.

This is one of those Linux fundamentals I need to stay sharp on for both system administration and security work.
