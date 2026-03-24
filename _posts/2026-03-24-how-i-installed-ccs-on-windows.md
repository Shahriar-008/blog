---
title: How I Installed CCS on Windows and Got It Working Fast
date: 2026-03-24 10:30:00 +0600
categories: [tools, productivity]
tags: [ccs, claude, windows, npm, developer-tools]
description: A practical step-by-step post on how I installed CCS on Windows, verified it worked, and handled the small issues that usually slow down setup.
---

## Why I Installed CCS

I installed [CCS](https://github.com/kaitranntt/ccs) for one simple reason: I wanted to use Claude Code with Codex and Gemini without turning my setup into a mess.

I did not want to keep bouncing between separate tools, separate auth flows, and separate terminal habits just to use different models. I wanted one workflow where I could stay inside Claude Code and still reach for Codex or Gemini when I needed them.

That is what made CCS interesting to me.

So here is exactly how I installed it on Windows.

## What I Checked Before Installing

The current recommended install method in the repo is `npm`, so I made sure Node.js and npm were already available in my terminal.

```powershell
node -v
npm -v
```

If those two commands return versions, you are good to go. If not, install Node.js first and then come back.

## The Install Command I Used

I went with the recommended package:

```powershell
npm install -g @kaitranntt/ccs
```

That was the entire install.

One thing I liked here: the project keeps the setup simple. No weird manual file shuffling, no long bootstrap script, no guessing where things ended up.

## How I Verified It Actually Worked

After the install finished, I checked the version:

```powershell
ccs --version
```

Then I opened the help output just to confirm the command was available globally:

```powershell
ccs --help
```

That is usually my rule for CLI tools. I do not trust a successful install message by itself. If `--version` and `--help` work, the tool is probably in the right place and the shell can see it.

## What I Wanted From It

My goal was pretty specific:

1. Use Claude Code as my main coding interface.
2. Switch to Codex when I wanted OpenAI-style code help.
3. Use Gemini when I wanted another model in the same overall workflow.

That is the part that made CCS feel useful instead of just clever. It was not about collecting tools. It was about reducing friction between the tools I actually wanted to use.

## My First Real Run

Once the binary was available, I tried the default command:

```powershell
ccs
```

From there, I could start with the default profile and then move into the model-specific commands I actually cared about.

For my use case, the interesting commands were:

```powershell
ccs glm
ccs kimi
ccs codex --auth
```

I like tools that let me start small. I did not need to learn the whole feature set on day one. I just needed the install to work and the command to respond.

## One Windows Detail Worth Knowing

If you are on Windows, there is one detail in the project docs that is easy to skip: **Developer Mode**.

CCS uses symlink-based behavior on Windows for the better experience. If Developer Mode is disabled, the tool can fall back to copying directories instead. That still works, but it is not as clean or instant.

So if you plan to use CCS heavily on Windows, it is worth enabling:

`Settings -> Privacy & Security -> For Developers -> Developer Mode`

It is not a dramatic step, but it is the kind of small OS setting that saves confusion later.

## What I Liked About the Setup

A lot of CLI tools claim to be easy to install, then immediately make you fight environment variables, shell profiles, or undocumented edge cases.

CCS felt more straightforward than that.

The flow was basically:

1. Make sure Node.js exists.
2. Install the package globally.
3. Run `ccs`.
4. Explore the profile or auth flow I actually need.

That is the kind of setup I appreciate. Fast in, fast out, and no wasted motion.

## If `ccs` Is Not Recognized

This did not happen to me, but it is a common Windows issue with global npm packages.

If PowerShell says `ccs` is not recognized, I would check these first:

- Restart the terminal after installation.
- Run `npm config get prefix` and make sure the global npm bin path is in `PATH`.
- Confirm the package actually installed with `npm list -g --depth=0`.

Most of the time, it is just a shell session problem, not a broken install.

## Final Thoughts

Installing CCS on Windows was refreshingly uneventful, which is honestly the best thing I can say about any developer tool.

I used the repo's recommended npm route, verified the CLI immediately, and had a working command in just a few minutes. If you already have Node.js installed, this is one of those setups that feels more like adding a useful tool to your belt than starting a weekend project.

If you want the shortest version possible, it is this:

```powershell
npm install -g @kaitranntt/ccs
ccs --version
ccs
```

And that is enough to get moving.
