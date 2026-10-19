# imo-cli

Automates IMO on Android phones over adb: opens IMO clones, likes recent
posts in pinned chats, and switches accounts. Works with or without
Multi Space.

## Requirements

- Python 3.10+
- adb on PATH (Android platform-tools), or set `ADB_PATH`
- Phones with USB debugging enabled, connected over USB

## Install

```bash
cd imo-cli
pip install -e .
```

This installs the `imo` command and its dependencies (uiautomator2, rich,
typer, Pillow).

## Run

```bash
imo doctor        # check adb, devices, and the IMO setup
imo devices       # list connected phones
imo init          # install the uiautomator2 agent on each phone (one time)
imo run           # run the workflow on all connected phones
```

`imo run` shows a live dashboard. Stop it with Ctrl+C.

Other useful commands:

| Command | What it does |
|---|---|
| `imo run --device <S>` | Run on one phone only |
| `imo run --json file.ndjson` | Save the run log to a file |
| `imo disconnect <S>` | Stop one phone cleanly |
| `imo report file.ndjson` | Print a report from a saved `--json` run |

## If the like button isn't found

The Story like button is found by resource id and then by scanning the UI
dump. If your IMO build uses an id this tool doesn't know, the run log prints
a line like `Story nodes: ...` listing every node on the story screen. Read
the like button's id from there and set it before running:

```bash
export IMO_LIKE_BUTTON_ID=com.imo.android.imoim:id/your_id_here
```

On Windows use `set IMO_LIKE_BUTTON_ID=...` instead.

## Troubleshooting

| Problem | Fix |
|---|---|
| `adb not found` | Install platform-tools or set `ADB_PATH` |
| `No devices connected` | Check the USB cable, enable USB debugging, accept the RSA prompt on the phone |
| `agent not reachable` | Run `imo init`; on some phones grant the uiautomator2 app the Accessibility/overlay permission |

## Like accuracy (how a like is confirmed)

Every like is checked, not assumed:

1. After tapping a post, the tool waits for the viewer to fully open, then
   looks at the heart icon. The post only counts as **liked** once the heart
   is actually red.
2. A tap that had no effect is retried (up to 3 times) and the like button is
   located again. It is never silently counted as "already liked".
3. A post that is already red is left alone, so nothing is toggled off.
4. If a like cannot be confirmed on the picture (the video behind the heart is
   red, or the heart is an unexpected colour), the tool goes back to the chat
   and checks for the reaction heart beside the post. If the heart is missing it
   retries the post once. Posts that still cannot be confirmed are counted as
   **failed**, and a screenshot is saved in the `imo_debug` folder.

If a phone is slow, give it more time without editing code (Windows: `set`,
other systems: `export`):

| Setting | Default | Meaning |
|---|---|---|
| `IMO_VIEWER_OPEN_TIMEOUT` | 3.0 | Seconds to wait for a tapped post to open |
| `IMO_VIEWER_SETTLE` | 0.35 | Pause after it opens, before tapping like |
| `IMO_LIKE_MAX_ATTEMPTS` | 3 | Like taps per post before giving up |
| `IMO_LIKE_VERIFY_POLLS` | 4 | Checks for the red heart after each tap |
| `IMO_DEBUG_DIR` | imo_debug | Folder for failure screenshots (empty = off) |
