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
- Change 1: a dotted vertical line (4 dots of 2×2 device px with 2 px
  gaps, 14 device px ≈ 11 px tall; Column of Rectangles; color see "Color"
  below) on each limit-window
  meter, where the window lands at its reset if use keeps the average rate
  since the window opened: `used × span / elapsed` (`projectedPercent` in
  `Panel.qml`, `projection` on `Meter`). Pinned to the end of the bar above
  100%. Only windows with a reset time and a known length (`windowSpanMs`:
  Session 5h, Weekly 7d, Monthly 30d, from the label or title). Hidden for
  the first 5% of a window (15 min of 5h, 8.4 h of 7d), where one prompt
  swings it. Not on the balance bar. OpenRouter's "Monthly key limit" has
  no `resetsAt`, so no tick. The row tooltip adds "On pace for N% at reset"
  (English only, no i18n key), which shows how far past 100% it goes.
- Change 4 (2026-11-07): two more tooltip lines (English only, like "On
  pace"). `runOutText`: when the forecast passes 100%, "Full by <time>, <d h>
  before reset" (time to full = `(1 − used) × elapsed / used`; weekday added
  unless today; Qt's narrow no-break space before AM/PM replaced by a space,
  the monospace font drew "2:36AM"). `dailyAllowanceText`: Weekly/Monthly
  only (`spanMs` ≥ 7 d, now on each window object), "N% a day left to stay
  under 100%" = `(1 − used) / days to reset`, hidden in the last day.
  Verified live: Weekly and Fable Weekly per-day values match the math; with
  usage temporarily doubled in `runOutText` (reverted) a Full-by line showed
  the computed time; Session shows neither line. The today-only time format
  was not seen live.
- Change 5 (2026-11-08): `todayLeftText`, above the per-day line, Weekly/
  Monthly only: "N% left today to stay under 100%" = even-pace target at the
  next local midnight (`elapsed at midnight / span`, capped at 100%) − used;
  hidden below 0.05%. No state saved. The user chose this over "today's
  share" (save the % at the first reading after midnight, subtract usage
  since), knowing it can read more than the per-day line because unused
  earlier days count toward today. `shareText` formats both lines. Verified
  live 2026-11-08 on Weekly and Fable Weekly (both lines match the math);
  with used +30% (reverted) the line hid.
- Per-day line wording (2026-11-08): the user asked for the day count, so
  it reads "N% a day for D days to stay under 100%" (D = days to reset, one
  decimal). Verified live on Fable Weekly (matches the math).
- Hover testing: `hyprctl dispatch` takes Lua on Hyprland 0.56
  (`hyprctl dispatch 'hl.dsp.cursor.move({x=1950,y=1334})'`, layout coords),
  but a warp sends no motion to the panel, so tooltips neither appear nor
  follow; reopening the panel under the cursor didn't help either. Use a
  temporary forced `visible:` on the row's PanelToolTip, then revert.
- Change 6 (2026-11-10, marketplace review #14317, Low at `4e1666c`): the
  sync snapshot (stock `Main.qml`, `syncSnapshotFile` FileView) was written
  0644 and `mkdir -p` made 0755 folders under umask 022. `runSync` now runs
  `bash -c 'umask 077 && mkdir -p -- "$0" && touch -- "$1" && chmod 600 --
  "$1"'` before the write; the FileView's atomic write (QSaveFile) keeps an
  existing file's mode. `syncPreparedPath` holds the path that step
  prepared; if the settings change the path before the write, the write is
  skipped and a new run scheduled (else FileView would `mkpath` a 0755
  folder and QSaveFile create a 0644 file; Fable review). Only `runSync` and
  `writeSyncSnapshot` differ from stock in `Main.qml`.
  Verified live with `syncMode On` + `syncDir` in a scratch folder (keys
  removed afterwards, shell.json byte-identical): new folders 700, snapshot
  600, valid JSON, no leftover temp file; an existing 0644 snapshot became
  600 on the next sync. Existing sync folders keep their mode on purpose
  (a user's Syncthing/Dropbox folder).
- Change 7 (2026-11-10): the row tooltip's reset line ("%1: %2% used ·
  resets in %3", stock i18n key `agents.windowUsedResets`) gets the clock
  time after the duration: `%3` = "10h 7m (Wed 7:59 AM)", so every language
  keeps its own sentence. `clockText()` (shared with `runOutText`): today's
  time alone, the next 6 days with the weekday, later "MMM d, time" (Monthly).
  Verified live on Weekly and Fable Weekly (Tue 9:52 PM → "Wed 7:59 AM");
  the today-only and date formats were not seen live.
- Change 2: model-scoped windows ("Fable Weekly") get their own row and bar
  (with their own forecast), right after the window they share a clock with
  (`displayWindows`). Upstream drew them as a black marker on that row.
- Verified live 2026-11-07: Session, Weekly and Fable Weekly lines at the
  computed positions; tooltip line shown by a temporary always-visible edit,
  since reverted; OpenRouter's key limit without a line.
- Screenshots: the panel opens on the monitor whose bar has focus; with the
  external monitor focused it opened on eDP-1 only on the second try.
- Line width (2026-11-07): at the laptop's 1.25 scale a 2 px line is 2.5
  device px, so lines rendered 2 or 3 px wide depending on position. Now
  sized in device pixels (2 px wide, dashes `round(thickness × dpr)`) and x
  snapped to the device grid. The ratio must come from
  `QsWindow.window.devicePixelRatio` (1.25); `Screen.devicePixelRatio`
  reports 2 there (that gave 1 px lines). Verified by pixel count: all three
  lines 2 px, dashes 5 px, gaps 3 px.
- Dots (2026-11-07): the user found the dashes (3 × 5 px, 21 device px
  tall) too big and too long; replaced by 4 small dots, two thirds of the
  height. y snapped to the device grid like x. Verified by pixel count: dots
  2×2, gaps 2, 14 px on all three meters.
- Color (2026-11-07): the dots blend from `foreground` (black in a light
  theme) to `urgent` as the forecast goes from 50% to 100% (`tint` on
  `projectionTick`, `mix()` helper); fully red at 100% and above. Started
  at 60%; the user lowered it to 50%. Verified on the 50% ramp: 42% black
  (29,29,31), 73% about halfway (107,39,31); on the 60% ramp with a
  temporary +0.35 offset (reverted) 108% was full red (200,50,30).
- Change 3: row titles wrap to two lines (`compactTitle`: WordWrap,
  maximumLineCount 2) instead of eliding; OpenRouter's "Monthly key limit"
  was cut to "Monthly key li…". Verified live.
- OpenRouter "Monthly key limit" = the credit limit set on the API key the
  collector uses (`/usr/bin/omarchy-agent-usage-openrouter`, `/api/v1/key`:
  `limit`, `limit_remaining`, `limit_reset`; key from OPENROUTER_API_KEY or
  opencode's auth.json). The collector writes `resetsAt: ""`, so no forecast
  line on it.
- `collectors.py` removed (2026-11-07): the clone copied it, but nothing
  loads it. The `/usr/bin/omarchy-agent-usage-*` collectors import
  `$OMARCHY_PATH/shell/plugins/agents/collectors.py` (package `omarchy`),
  and the widget only reads their JSON in `~/.local/state/omarchy/agents/
  usage/`. README.omarchy.md still mentions it "beside this README"
  (upstream text, left verbatim). Risk: an Omarchy update that changes the
  record format; merge the new stock `Panel.qml` against the first commit.
- `preview.png` (2026-11-07, marketplace card + README): crop of the Claude
  Code rows (Session, Weekly, Fable Weekly) from a real screenshot, with the
  plan label ("Max 5x") painted over in the panel background. No token
  counts, balances or spend. Redo the same way; never commit the full panel.
- Not ours: `assets/claude-light.svg` "Cannot open" warnings in the journal;
  the stock plugin lacks that file too.
