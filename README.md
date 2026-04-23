# tmux-csi-u.el

Emacs-side decoder for tmux `CSI-u` sequences in terminal Emacs.

tmux stays on `csi-u`. This package fixes the Emacs TTY decode gap instead of downgrading tmux key reporting.

Scope: the delta over Emacs native tmux/xterm decode, not a replacement for the native path.

## what it solves

- systematic printable ASCII coverage from generated data
- explicit overrides for the non-native space / return / tab delta, modified backspace / escape, and shifted punctuation
- terminal-local install into `input-decode-map`
- warn-and-preserve conflict handling for already-customized setups

## install

Once on MELPA:

```
M-x package-install RET tmux-csi-u RET
```

Manual install from a clone:

```elisp
(add-to-list 'load-path (expand-file-name "path/to/tmux-csi-u"))
(require 'tmux-csi-u)
```

`tmux-csi-u-auto-enable` defaults to `t`; the package installs from `tty-setup-hook` for supported TTY frames. To enable explicitly:

```
M-x tmux-csi-u-enable RET
```

Optional configuration:

```elisp
;; only for daemon/client edge cases where tty detection cannot
;; see tmux directly
(setq tmux-csi-u-force-enable t)

;; extra mappings applied after package defaults
(setq tmux-csi-u-local-overrides '(("\e[59;2u" . [f13])))
```

## public entrypoints

- `tmux-csi-u-enable` — install candidate mappings for the current TTY terminal and return a report plist
- `tmux-csi-u-supported-p` — return non-nil when the current frame looks like a supported tmux TTY context
- `tmux-csi-u-describe` — return the latest report plist; interactively, render a human summary buffer
- `tmux-csi-u-force-enable` — explicit opt-in for daemon/client edge cases
- `tmux-csi-u-local-overrides` — local mappings applied after package defaults

## conflict behavior

Policy: warn-and-preserve.

- package-owned candidates install when the sequence is free
- already-matching bindings count as already enabled, including stock tty equivalents such as `M-<tab>` / `M-<return>` / `M-ESC`
- conflicting external bindings stay in place
- warnings point at `(tmux-csi-u-describe)` for the full report

## migration from ad hoc bindings

Delete the ad hoc `input-decode-map` entries for sequences this package owns:

- `\e[32;2u`, `\e[32;5u`, `\e[32;6u`, `\e[32;8u`
- `\e[13;3u`, `\e[13;8u`
- `\e[9;3u`
- `\e[127;2u`, `\e[127;3u`, `\e[127;5u`, `\e[127;6u`, `\e[127;7u`
- `\e[27;3u`, `\e[27;5u`, `\e[27;6u`
- `\e[59;2u` and the shifted punctuation family derived from `test/fixture/punctuation.json` (`;2`, `;4`, `;6`, `;8` for the captured local targets)

Emacs already covers part of the tmux -> xterm path natively or effectively, so this package does not claim `\e[32;3u`, `\e[32;7u`, `\e[9;2u`, `\e[9;5u`, `\e[9;6u`, `\e[13;2u`, `\e[13;5u`, `\e[13;6u`, or `\e[13;7u`.

Codepoint-form xterm-native lossy punctuation such as `\e[58;6u` is documented skip behavior; the fixture-derived base-keycode family such as `\e[59;6u` stays package-owned.

Keep unrelated terminal bindings that are not tmux `CSI-u` sequences. Keep local fallbacks for sequences this package does not own, for example `\e[13;4u` (`M-S-RET`).

If an old snippet stays in place, the package does not overwrite it silently. Existing external bindings are preserved and reported.

## troubleshooting

- `tmux-csi-u-supported-p` returns `nil`: check for a TTY frame, a live terminal, and `tty-type` equal to `tmux` or `tmux-256color`; use `tmux-csi-u-force-enable` only for daemon/client edge cases
- `tmux-csi-u-describe` reports conflicts: remove old ad hoc bindings or keep them intentionally; the package preserves them either way
- punctuation still follows shifted base characters such as `S-;`: verify the explicit override is present and the old local mapping is gone

## protocol reference

See `doc/ref/protocol.md` for the tmux CSI-u wire shape, modifier model, generated printable baseline, documented skips, and the canonical special-key overrides.
