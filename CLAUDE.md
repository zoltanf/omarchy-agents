# omarchy-agents: agent notes

Omarchy bar widget `zoltanf.agents` ("Agents & Forecast"), cloned from
`omarchy.agents` with `omarchy plugin clone` on 2026-11-07. README.md covers
what it changes; upstream's README is kept as README.omarchy.md. **Public**
GitHub repo `zoltanf/omarchy-agents`, created 2026-11-07: keep home-network
details, usage numbers and screenshots of the author's machine (balances,
spend) out of it.

## State (2026-11-07)

- Installed as a **symlink** `~/.config/omarchy/plugins/zoltanf.agents` →
  this checkout, in place of `omarchy.agents` (the clone switched the bar
  entry). Apply edits with `omarchy restart shell`; open the panel for a
  screenshot with `omarchy-shell omarchy.agents open` (IPC target keeps the
  built-in id), then `grim -o eDP-1`.
- The first commit is the clone verbatim, so `git diff <first-commit>` shows
  every change and newer upstream versions can be merged against it.
- Change 1: a red dashed vertical line (3 dashes of `thickness` px with
  half-thickness gaps, about 16 px; 5 dashes was too tall; Column of
  Rectangles; theme `urgent` = the theme's red) on each limit-window
  meter, where the window lands at its reset if use keeps the average rate
  since the window opened: `used × span / elapsed` (`projectedPercent` in
  `Panel.qml`, `projection` on `Meter`). Pinned to the end of the bar above
  100%. Only windows with a reset time and a known length (`windowSpanMs`:
  Session 5h, Weekly 7d, Monthly 30d, from the label or title). Hidden for
  the first 5% of a window (15 min of 5h, 8.4 h of 7d), where one prompt
  swings it. Not on the balance bar. OpenRouter's "Monthly key limit" has
  no `resetsAt`, so no tick. The row tooltip adds "On pace for N% at reset"
  (English only, no i18n key), which shows how far past 100% it goes.
- Change 2: model-scoped windows ("Fable Weekly") get their own row and bar
  (with their own forecast), right after the window they share a clock with
  (`displayWindows`). Upstream drew them as a black marker on that row.
- Verified live 2026-11-07: Session, Weekly and Fable Weekly lines at the
  computed positions; tooltip line shown by a temporary always-visible edit,
  since reverted; OpenRouter's key limit without a line.
- Screenshots: the panel opens on the monitor whose bar has focus; with the
  external monitor focused it opened on eDP-1 only on the second try.
- Not ours: `assets/claude-light.svg` "Cannot open" warnings in the journal;
  the stock plugin lacks that file too.
