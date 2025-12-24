# Copilot API Project Analysis

## Functionality Overview

The `copilot-api` project is a reverse-engineered proxy that exposes the GitHub Copilot API as an OpenAI and Anthropic compatible service. It allows using GitHub Copilot with tools like Claude Code.

### Key Components

* **Entry Point**: `src/main.ts` uses `citty` to define the CLI.
* **Commands**:
  * `start`: Starts the server (`src/start.ts`).
  * `auth`: Handles GitHub authentication.
  * `check-usage`: Displays usage/quota.
  * `debug`: Shows debug info.
* **Configuration**:
  * Stored in `config.json` (handled by `src/lib/config.ts`).
  * Parameters include `smallModel` (fallback), `extraPrompts`, and `modelReasoningEfforts`.
* **State**:
  * Runtime state is managed in `src/lib/state.ts` (tokens, options like manual approve, rate limits).

## Current Argument Parsing (`start` command)

The `start` command in `src/start.ts` accepts several flags:

* `--port` (`-p`)
* `--verbose` (`-v`)
* `--github-token` (`-g`)
* `--claude-code` (`-c`)
* `--proxy-env`
* etc.

### The `claude-code` Workflow

When `-c` (`--claude-code`) is used:

1. The server starts.
2. It prompts the user interactively (via `consola.prompt`) to select a **Main Model** and a **Small Model** from the list of available models fetched from Copilot.
3. It generates and prints/copies a shell command to configure `claude` CLI environment variables.

## Problem Statement

The user wants to specify the Main Model and Small Model directly via command-line arguments to avoid interactive prompts, e.g.:
`bun run start start --proxy-env -c -p 4141 --main-model gpt-5 --small-model gpt-5-mini`

## Proposed Solution

1. **Modify `src/start.ts`**:
    * Add `--main-model` (alias `-m`?) and `--small-model` (alias `-s`?) to the `start` command arguments.
    * Pass these values to the `runServer` function.
2. **Update `runServer` logic**:
    * **Claude Code Generation**: If `-c` is active, check if models are provided via args. If so, use them/validate them against available models (or just use them if validation isn't strict). If not provided, fall back to interactive prompts.
    * **Server Configuration**: The `small-model` argument should also ideally override the effective `smallModel` used by the server (in `src/lib/config.ts`) for consistent behavior (e.g., warmup requests).

## Implementation Details

* **File**: `src/start.ts`
  * Update `args` definition.
  * Update `RunServerOptions` interface.
  * Update `runServer` function to use options.
* **File**: `src/lib/config.ts`
  * Dependencies: Provide a way to set/override `smallModel` in the in-memory config if the CLI arg is present.
