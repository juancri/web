---
title: My Claude Code setup
date: 2025-09-02
tags:
  - english
  - software
  - development
layout: layouts/post.njk
---

Claude Code is really easy to use. In case it's useful for anyone, here are the details of my current setup:

### ccusage

I usually run *ccusage* in another terminal. This allows me to monitor the usage and the current session. I currently have the USD 100 / month plan and have passed the limit only once.

To install *ccusage*, run `npm install -g ccusage`.

I run it with these parameters:

```bash
ccusage blocks --live
```

### Use plan mode

TBD

### Use Opus Plan Mode

I have found that using "Opus Plan Mode" is really useful so the heavy listing of planning is passed to Opus while the code changes are made by sonnet.

Change the mode running `/mode` inside Claude Code.

### MCP Servers

I mainly work on backend projects connected to a PosgreSQL database, so I usually use [this MCP server for PostgreSQL](https://github.com/crystaldba/postgres-mcp).

To quickly setup the MCP server for the current directory, run:

```bash
claude mcp add -e DATABASE_URI="postgresql://" -- database docker run -i --rm -e DATABASE_URI crystaldba/postgres-mcp --access-mode=unrestricted
```
