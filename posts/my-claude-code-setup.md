---
title: My Claude Code setup
date: 2025-10-20
tags:
  - english
  - software
  - development
  - ai
layout: layouts/post.njk
---

*Note: This is the continuation of [A new programming paradigm]({% link_to "a-new-programming-paradigm" %}). Claude Code often releases new features, so this post may be updated to reflect them.*

Claude Code is really easy to use. In case it's useful for anyone, here are the details of my current setup:

## Don't forget to Init

When working with Claude Code on a project for the first time, run `/init` so it can evaluate the project structure, read its README file, and generate a `CLAUDE.md` file which will be part of the context every time you work on that project again.

Even better, after running `/init`, edit `CLAUDE.md` and ensure all the information is correct. Make sure to include all your rules and technical details.

## Powerline

[Claude Powerline](https://github.com/Owloops/claude-powerline) is a status line for Claude Code. It includes information about your git repo, current session, and context.

## Use plan mode

Unless your task is super simple, use plan mode. Press Shift+Tab until you see `⏸ plan mode on` below your status line.

Describe the problem providing all the information Claude Code might need to implement the new feature, fix the bug, etc. Claude Code will come up with a plan. You can make changes to the plan until you are OK with all the steps. Pay close attention to the libraries it plans to use and other architectural decisions that might impact your project in the future.

Usually, after approving the plan, you can change the mode to *accept edits*. You will see `⏵⏵ accept edits on` below your status line.

## MCP Servers

These are a few MCP servers I use for almost all my projects:

### PostgresSQL

I mainly work on backend projects connected to a PosgreSQL database, so I usually use [this MCP server for PostgreSQL](https://github.com/crystaldba/postgres-mcp).

To quickly set up the MCP server for the current directory, run:

```bash
claude mcp add \
  -e DATABASE_URI="postgresql://" -- \
  database docker run \
    -i --rm \
    -e DATABASE_URI \
    crystaldba/postgres-mcp \
    --access-mode=unrestricted
```
### Context7

[context7](https://context7.com/) is a hub of up-to-date technical documentation. Using this MCP server, you can ensure Claude Code has access to current documentation for libraries, frameworks, platforms, etc. You can even add repositories that are not part of the directory.

To install the Context7 MCP server to the project in the current directory, run:

```bash
claude mcp add \
  --transport http \
  context7 \
  https://mcp.context7.com/mcp \
  --header "CONTEXT7_API_KEY: YOUR_API_KEY"
```

Make sure to replace `YOUR_API_KEY` with your real key.

### Google Chrome DevTools

The [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp/) allows Claude Code to interact with Chrome DevTools Protocol (CDP). This enables debugging, inspecting, and controlling Chrome browsers programmatically. It's particularly useful for web development, automated testing, and browser automation tasks.

To install the Chrome DevTools MCP server to the project in the current directory, run:

```bash
claude mcp add \
  chrome-devtools \
  npx -y chrome-devtools-mcp@latest
```

### Growthbook

I use growthbook to define feature flags and rollout new features. You can setup the growthbook MCP server by running:

```bash
claude mcp add \
  -e GB_EMAIL=your@email.com \
  -e GB_API_KEY=secret_admin_ABCDEF \
  -- growthbook npx \
    -y @growthbook/mcp@latest \
    -e GB_EMAIL \
    -e GB_API_KEY
```

## Clear the Context Often

After a new feature is implemented or a bug is fixed, clean your context by running `/clean`. This will initialize a new context with the project information but without the noise of any previous task.

## Resume

If you need to exit Claude Code for some reason, you can resume any previous session by running `claude --resume`.

## Hooks

Some Claude Code tasks take time. While waiting for it to finish working on a new feature, I'm usually working on another task in a web browser or another terminal. I don't want to waste time by looking at the terminal while Claude Code is working but I also want to be notified when it's done. This is a good use case for hooks. I have a hook that plays a sound when Claude Code finishes working and the *Stop* event is triggered.

To add this hook, you can edit the file `~/.claude/settings.json`:

```json
{
  ...
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "some-audio-player /path/to/your/file.wav"
          }
        ]
      }
    ]
  },
  ...
}

```

## Memorize

If you realize Claude Code usually makes changes to a project but does not run the linter after that, there's a way to add "memorization". You can just type `#` followed by the thing you want it to remember. For example:

```
# After each change, run `npm run build` and fix any errors
```

Then, Claude Code will ask you where to store this. Since this specific example is valid for a specific project, I would choose to store it in the project's `CLAUDE.md` file.
