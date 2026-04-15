# contributing

## scope

This repo is a small Emacs Lisp package plus repo-local docs and fixtures.

Keep changes tied to the tmux `CSI-u` problem. Avoid drive-by feature growth.

## local setup

- Emacs with batch mode available as `emacs`
- tmux available for manual repro work
- python3 available for `script/qa-smoke`
- run `script/bootstrap-package-lint` once on a new machine to populate the repo-local ELPA dir

## required commands

Run from the repo root.

```bash
script/format
script/check
```

`script/check` runs format check, lint, byte-compilation, and tests.

## when mappings or protocol change

Update the owning docs and fixtures in the same change.

- printable baseline contract: `test/fixture/generated-matrix.json`
- punctuation capture contract: `test/fixture/punctuation.json`
- protocol notes and limits: `doc/ref/protocol.md`
- user install, migration, troubleshooting, manual verification notes: `README.md`

## issue and PR shape

- include a reproduction path for behavior changes
- include exact tmux, Emacs, and terminal versions when the issue is environment-sensitive
- note whether the problem reproduces inside a daemon-started TTY client
- keep one logical change per PR

## before opening a PR

- run `script/check`
- run `script/qa-smoke` when TTY behavior changed
- rerun the relevant manual verification steps from `README.md`
- mention fixture updates when `test/fixture/generated-matrix.json` or `test/fixture/punctuation.json` changed
- mention preserved conflicts if the change touches install behavior
