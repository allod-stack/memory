---
name: list-models
description: Find out which models each installed agent CLI (pi, codex, claude) declares and whether a named one actually answers, before scripting anything around a model name. Covers where each CLI keeps its list, why a catalog entry is not evidence that a model is served, the one-prompt probe that settles it, and the refusal shapes that exit 0. Ships probe-models, which lists what is present and probes only the models you name. Use when choosing a model for a subagent or reviewer, when a model is refused, or when a note about which models exist is more than a day old.
---

# list-models

A note that says which models exist is wrong within days: providers rotate
catalogs, an account's list changes with its plan, and a model a catalog
declares may not be served. Each CLI has a different place it keeps its list
and a different way of refusing, so this skill records where to look and
ships `probe-models`, which lists what each installed CLI declares and, for
the models you name, asks each one a one-word question and reports whether it
answered. It assumes nothing about accounts: a CLI that is not installed is
reported as such, and nothing is probed unless named.

## Where each CLI keeps its list

| CLI | The list | Readiness | Refusal |
|---|---|---|---|
| `pi` | `pi --list-models [search]` prints provider, model, context, output and thinking columns; `pi update` refreshes the catalogs | `pi auth check --provider <p>` (`--json` for a machine-readable line) | a model the provider does not serve still exits 0; in `--mode json` the assistant's `message_end` event has `stopReason: "error"` and an `errorMessage` naming `unsupported_model`. An unknown provider prints `Unknown provider` |
| `codex` | no list command; the account's list is what the client fetched into `$CODEX_HOME/models_cache.json` (default `~/.codex`), one entry per model with `slug`, `visibility` (`list` or `hide`), `default_reasoning_level` and `supported_reasoning_levels`, refreshed whenever codex runs | `codex doctor` prints the auth mode and the configured model | `codex exec -m <model>` on a model the account lacks prints an `ERROR:` line with a 400 `not supported when using … account` and exits 1; a misspelled model gets the identical message, so check spelling before reading it as an account limit |
| `claude` | no list command; `--model` takes an alias such as `sonnet` or a full model name (`claude --help`, under `--model`) | none beyond running it | `claude -p --model <model>` on an unknown name exits 1 and says the model may not exist or may not be accessible; the wording varies with how it is invoked |

Only codex has a default model to speak of, and it is a line in
`$CODEX_HOME/config.toml`, so pass `-m` explicitly in anything scripted; the
listing prints what the file currently says.

## Running the probe

```
skills/list-models/probe-models                      # what each installed CLI declares
skills/list-models/probe-models pi <provider>/<model>...
skills/list-models/probe-models codex <model>...
skills/list-models/probe-models claude <model>...
```

With no arguments it prints pi's catalog and per-provider readiness, codex's
cached account list with each model's reasoning levels and the configured
default, and claude's `--model` help. Named models each get one prompt and
one line: `<cli> <model> served|refused <detail>`, where the detail is the
answer or the first line of the refusal. Exit 0 when every named model
answered, 2 when any was refused, 1 on usage. `PROBE_TIMEOUT` bounds each
probe in seconds, default 120.

`pi -p` prints nothing and never exits when it has no terminal, so the probe
runs it under `script -qec … /dev/null` and strips the carriage returns that
adds. Anything you script around pi needs the same two things.

## Reading a refusal

A refusal is a fact about today: the account, the provider's served set, or
the spelling. Before changing the model, check the spelling against the
listing, because two of the three CLIs give a typo the same message as an
unavailable model. Then check readiness: `pi auth check` reports `ready` or
the credential problem, and `codex doctor` reports whether auth is
configured. Only after both is a refusal evidence about the model.

## Precise limits

- Served means the model answered one short prompt now. It says nothing about
  quality, rate limits, or context size; the listing's columns are the only
  source for those, and they are declarations too.
- Each probe spends tokens on that model. The script refuses to probe a whole
  catalog for that reason; name what you will use.
- Reachability is a property of this machine: a provider that answers here can
  be unreachable from another VM with the same credential. Probe where the
  work will run.
