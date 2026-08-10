# KRpanoCode

Edit your KRPano virtual tour XML files by just describing what you want in plain English.

Instead of manually hunting through `tour.xml`, `skin.xml`, `panel.xml` and dozens of `<include>` files, you tell KRpanoCode what you want to change. It reads your tour files, looks up the relevant KRPano documentation, and sends the instruction to an AI model — which reads, edits, and writes the files for you. You get a clear diff preview and can keep or undo every change.

---

## Requirements

- **PHP 8.1** or later (`php -v` to check)
- An **API key** from your LiteLLM proxy administrator

---

## Install

### Step 1: Download

Download the latest `krpanocode.phar` from the [releases page](https://github.com/iceman1010/krpanocode-releases/releases).

### Step 2: Install as a system command

Follow the instructions for your operating system.

#### Linux

Move the PHAR to a directory in your PATH and make it executable:

```bash
sudo mv krpanocode.phar /usr/local/bin/krpanocode
sudo chmod +x /usr/local/bin/krpanocode
```

Now you can run `krpanocode` from any folder.

To verify:

```bash
krpanocode --version
```

#### macOS (OS X)

Same as Linux — move it to a directory in your PATH:

```bash
sudo mv krpanocode.phar /usr/local/bin/krpanocode
sudo chmod +x /usr/local/bin/krpanocode
```

If `/usr/local/bin` does not exist, create it first:

```bash
sudo mkdir -p /usr/local/bin
```

Verify:

```bash
krpanocode --version
```

#### Windows

1. Move the PHAR somewhere permanent, e.g. `C:\bin\krpanocode.phar`

2. Open the **Environment Variables** editor:
   - Press `Windows + R`, type `sysdm.cpl`, press Enter
   - Go to **Advanced** → **Environment Variables**

3. Add `C:\bin` to your **PATH**:
   - Under "User variables", find **Path**, click **Edit**
   - Click **New**, type `C:\bin`, click **OK** on all three dialogs

4. Create a wrapper batch file so you don't have to type `php` manually. Open Notepad, paste:

   ```bat
   @echo off
   php "%~dp0krpanocode.phar" %*
   ```

   Save it as `C:\bin\krpanocode.bat`

5. Open a **new** terminal and verify:

   ```cmd
   krpanocode --version
   ```

### Step 3: Configure

Run the one-time setup to enter your API key and pick a default model:

```bash
krpanocode --setup
```

This asks for your API key, verifies it works, lets you pick a default AI model from the available list, and saves everything to `~/.krpanocode/.env` (on Windows: `%USERPROFILE%\.krpanocode\.env`).

**Quick API key setup (non-interactive):**

```bash
krpanocode --key SK-your-api-key-here
```

This sets only the API key without running the full interactive setup. Use `--json --key` for machine mode.

---

## How to Use

### Interactive mode (easiest)

Navigate to your tour folder and run KRpanoCode without arguments:

```bash
cd /path/to/my-tour
krpanocode
```

It will find the tour automatically (looks for `index.html`), show you the editable files, and ask what you'd like to change:

```
  +---------------------------------------------------+
  |   KRpanoCode v0.3.2 - LLM-powered KRPano Editor   |
  +---------------------------------------------------+


  Configuration
  ─────────────────────────────────────
  Model        glm-5.2-coding
  API          https://ai.panomatics.com/v1/
  Tour         huahinsportcenter (5 editable files)
  Docs         Enabled
  Key          ...WaGuU8Pw

  Tour Files
  ─────────────────────────────────────
   Path                  Size               Status
   tour-d.xml            328 bytes          editable
   skin/skin.xml         3,974 bytes        editable
   skin/loadingbar.xml   1,093 bytes        editable
   panel.xml             18,479 bytes       editable
   blend.xml             encrypted/binary   locked
   tour.xml              8,722 bytes        editable

  Backup created.

  What would you like to change?
  > _
```

Type your instruction in plain English — for example:

> rename the Main Pool to "Swimming Pool"

KRpanoCode sends it to the AI, which reads the relevant files, makes the edits, and writes them back. You then see a **color-coded diff** of exactly what changed:

```
  Editing
  -------
    Instruction: rename the Main Pool to 'Swimming Pool'
    [Edit] Sending request to glm-5.2-coding...
      [Tool] read_file(tour.xml)
      [Tool] write_file(tour.xml) (8,494 bytes)
    [Edit] Completed in 64.3s

  Changes
  -------
    FILE: tour.xml (1 line changed)

  -  123 <scene name="scene_poolsideday" title="Main Pool" ...>
  +  123 <scene name="scene_poolsideday" title="Swimming Pool" ...>

  Keep these changes?
    [y] Yes - keep         (default)
    [n] No  - undo
    [d] Show full diff
  > _
```

### One-shot mode

Skip the interactive prompt by passing the instruction directly:

```bash
krpanocode -p "add a hotspot from scene1 to scene2"
krpanocode -p "change the autorotation speed to 3" -y
krpanocode -p "remove the floorplan thumbnail" -f /path/to/tour
```

`-y` auto-confirms and keeps the changes without asking.

---

## Clarification Step

By default, KRpanoCode sends your instruction straight to the AI and lets it work. This works great for clear, specific instructions.

Add the `--clarify` flag when your instruction might be ambiguous, or when you want the AI to confirm it understood before making changes:

```bash
krpanocode -p "change the skin colors" --clarify
```

With `--clarify`, the AI first analyses your instruction and the tour files, then tells you whether the instruction is clear or asks a follow-up question before proceeding:

```
  [Clarify] Asking glm-5.2-coding if instruction is clear...
  [Clarify] Response in 12.3s
  Status:  CLEAR
  Reason:  I will change the title of scene "scene_poolsideday"
           in tour.xml from "Main Pool" to "Swimming Pool".
```

If the instruction is ambiguous, the AI will say `CLARIFY` and ask a question:

```
  [Clarify] Asking glm-5.2-coding if instruction is clear...
  [Clarify] Response in 8.1s
  Status:  CLARIFY
  Reason:  The instruction mentions "skin colors" but there are
           multiple color settings spread across different files.
           Need to know which elements to recolor.
  Question: Which colors would you like to change — the overall
            skin background, the button colors, or the hotspot
            tooltip background?
```

Your answer is then sent back to the AI as part of the editing instruction.

**When to use `--clarify`:**
- Instructions that could be interpreted multiple ways ("change the colors", "improve the thumbnails")
- When you want reassurance the AI understood before it touches your files
- Complex multi-file edits where precision matters

**When to skip it:**
- Simple, specific instructions ("rename scene1 to lobby", "add a hotspot")
- When you already know exactly what you want

---

## Documentation Context

KRpanoCode includes 27 curated KRPano 1.23.3 documentation files. Before editing, it searches these docs for anything relevant to your instruction and includes that context for the AI. This makes the AI smarter about KRPano-specific syntax and elements.

To skip this (faster, but the AI relies only on its training knowledge):

```bash
krpanocode -p "rename scene1" --no-docs
```

---

## Choosing a Model

The default model is set during `--setup`. To see all available models on your proxy:

```bash
krpanocode --models
```

To use a different model for one run — this also **saves it as your new default** for future runs:

```bash
krpanocode -p "add little planet view" -m glm-5.2-nvidia
```

To change the model interactively (model picker):

```bash
krpanocode --setup
```

---

## Machine Mode (`--json`)

`--json` switches all output from human-readable boxes/tables to **NDJSON**
(Newline-Delimited JSON): one JSON object per line on `stdout`, flushed as
events happen. No ANSI colors, no banner, no prompts.

It is the interface the **KRPano Code desktop UI** (a separate Tauri app) drives
the PHAR through. You can also use it for scripting, CI, or any other
automation that wants structured progress instead of plain text.

The contract for every event type lives in
[`PLAN-JSON-MODE.md`](./PLAN-JSON-MODE.md) and is the authoritative spec.

### Quick examples

```bash
# Version (one line)
krpanocode --json --version
# -> {"type":"version","version":"0.3.2"}

# List available models
krpanocode --json --models
# -> {"type":"models","models":["glm-5.2-coding","glm-5.2-nvidia",...]}

# Non-interactive setup: writes ~/.krpanocode/.env
krpanocode --json --setup --key SK-xxx --model glm-5.2-coding --backup-keep 10
# -> {"type":"setup","ok":true,"model":"glm-5.2-coding","models":[...]}

# Undo: restore the most recent backup for a tour folder
krpanocode --json --restore -f ./my-tour
# -> {"type":"restored","files":["tour.xml","skin/skin.xml"],"backup":"/abs/path"}
#    {"type":"done"}

# Edit a tour — streams events live
krpanocode --json -p "rename scene_poolsideday to lobby" -f ./my-tour --yes
# -> {"type":"start","tour":"my-tour","backup":"/abs/path","editable":[...],"locked":[...]}
#    {"type":"tool","name":"read_file","file":"tour.xml","ms":4}
#    {"type":"tool","name":"write_file","file":"tour.xml","bytes":8490,"ms":0}
#    {"type":"diff","file":"tour.xml","hunks":[{"line":123,"context":"...","old":"title=\"A\"","new":"title=\"B\""}]}
#    {"type":"done","ms":45291}

# Clarify flow — blocks on stdin for one line after each clarify event
krpanocode --json --clarify -p "..." -f ./my-tour
# -> {"type":"start",...}
#    {"type":"clarify","status":"clarify","question":"Which colors?"}
#    <the process blocks here, reading one line from stdin>
#    ...when the UI writes "<answer>\n" to stdin, the PHAR proceeds...
#    {"type":"clarify","status":"clear","reason":"..."}
```

### Event types at a glance

| Event type   | Emitted by | Key fields (besides `type`) |
|--------------|------------|------------------------------|
| `start`      | edit       | `tour`, `backup`, `editable[]`, `locked[]` |
| `reasoning`  | edit (optional) | `text` |
| `tool`       | edit       | `name`, `file` (when N/A), `ms`, `bytes` (write_file only), `query` (docsearch only), `files[]`+`reason` (plan_files only) |
| `plan_files` | edit (user-gated pre-read) | `status` (`ask`, `yes`, `no`), `files[]`, `reason` (on ask), optional `reason` (on no) |
| `clarify`    | `--clarify` | `status` (`clear` or `clarify`), `reason` (when clear), `question` (when clarify) |
| `validation_retry` | edit (user-gated self-correction) | `status` (`ask`, `retry`, `abort`), `attempt`, `maxAttempts`, `kind`, `file`, `line`, `message`, `details[]` |
| `diff`       | edit       | `file`, `hunks[]` (each: `line`, `context`, `old`, `new`) |
| `restored`   | `--restore` | `files[]`, `backup` |
| `done`       | edit, restore | `ms` (edit only) |
| `models`     | `--models` | `models[]` |
| `setup`      | `--setup`  | `ok`, `model`, `models[]` (additive), `error` (when `ok:false`) |
| `version`    | `--version` | `version` |
| `error`      | all (on failure) | `message` — plus `kind`, `model`, `reset_at`, `retry_after_seconds` on rate limits |

### Rate-limit errors (429)

When the LLM proxy returns a 429, `--json` mode emits an `error` event with
additive fields so a UI can show a countdown + Retry button without parsing
the message:

```json
{
  "type": "error",
  "message": "Rate limit reached on glm-5.2-coding. Limit resets at 2026-07-25T10:44:41Z.",
  "kind": "rate_limit",
  "model": "glm-5.2-coding",
  "reset_at": "2026-07-25T10:44:41Z",
  "retry_after_seconds": 37623
}
```

A consumer that only reads `message` still works. `reset_at` is canonical
ISO-8601 UTC (`...Z`). The CLI does **not** auto-retry or auto-fallback on
429 — it restores the backup, emits this error, exits non-zero, and leaves
the retry decision to the caller.

### Notes for `--json` consumers

- **Backup scheme:** `start.backup` is the tour directory itself, not a
  timestamped per-edit folder. Undo (`--restore`) reverts to the single
  backup set the CLI keeps. See [`backupproblem.md`](./backupproblem.md)
  for the full rationale.
- **Clarify + stdin:** after a `{"type":"clarify","status":"clarify",...}`
  event, the process **blocks** reading one line from `stdin`. The
  consumer must write `<answer>\n` to stdin AND keep draining stdout, or
  PHP's buffering can deadlock when the stdin write blocks.
- **Clarify abort:** writing `skip` (or EOF with no input) aborts the
  clarify round-trip and the edit is rolled back.
- **Plan files + stdin:** after a `{"type":"plan_files","status":"ask",...}`
  event, the process **blocks** reading one line from `stdin` — same channel
  and contract as `clarify`. Write `"yes"`/`"y"` to approve (the model then
  receives all requested file contents inline), anything else or EOF to
  decline (the model falls back to individual `read_file` calls). `--yes`
  auto-approves so headless runs aren't blocked. `--auto-approve-file-scope`
  also auto-approves but emits `status:"auto-approved"` so the conversation
  log still records the model's proposed file list.
- **Manifest enrichment:** the text manifest in the edit user-message now
  carries bracketed structural hints per file (e.g. `[defines-styles:textnames]`,
  `[uses-styles:textnames]`, `[align:bottomleft]`). This is invisible to
  `--json` consumers (the `start.editable` array is unchanged) but improves
  the model's file-targeting accuracy.
- **`--update` has no `--json` mode** today; the UI should call `--update`
  out of band or without `--json`.

---

## All Options

| Flag | Description |
|------|-------------|
| `-p, --prompt` | Your edit instruction, in plain English (asked interactively if omitted) |
| `-f, --folder` | Path to your tour folder (default: current directory) |
| `-m, --model` | Use a specific AI model for this run (also saves it as your new default) |
| `-c, --clarify` | Ask the AI to confirm it understood before editing |
| `--no-docs` | Skip KRPano documentation lookup (faster, less accurate for obscure features) |
| `--models` | List all models available on your proxy and exit |
| `--setup` | Configure your API key and default model interactively (or non-interactively with `--json --key --model`) |
| `-y, --yes` | Keep changes automatically, no confirmation prompt |
| `--auto-approve-file-scope` | Auto-approve the AI's proposed file-scope plan without asking (skips the `plan_files` confirmation) |
| `-u, --update` | Check for and install the latest version |
| `--json` | Machine mode: NDJSON events on stdout instead of human output |
| `--restore` | Restore the most recent backup for the `-f` tour folder and exit |
| `--key` | Setup (non-interactive): API key to write to `~/.krpanocode/.env` |
| `--backup-keep` | Setup: number of backups to keep per tour (default 10) |
| `-h, --help` | Show help |
| `-V, --version` | Show version |

---

## Self-Update

To check for and install the latest version:

```bash
krpanocode --update
```

This downloads the new PHAR from the [releases page](https://github.com/iceman1010/krpanocode-releases/releases) and replaces itself in place.

---

## How It Works

```
Your instruction (plain English)
        │
        ▼
  KRpanoCode reads your tour files
  (index.html → XML → <include> → all linked files)
        │
        ▼
  File manifest is built with structural hints
  (style defs, style refs, align values per file)
        │
        ▼
  (optional) DocSearch finds relevant KRPano docs
  from 27 curated files covering KRPano 1.23.3
        │
        ▼
  (--clarify) AI checks if instruction is clear —
  asks you a question if it's ambiguous.
  Its verdict is carried forward as an explicit
  target anchor for the edit phase.
        │
        ▼
  AI optionally declares the files it intends to
  read/edit via the plan_files tool — you confirm
  before it reads them
        │
        ▼
  AI reads and edits the files using tool calls
  (reads each file, then writes back the changes)
        │
        ▼
  CLI validates the resulting XML — if invalid,
  asks you whether to let the AI self-correct
  (up to max_validation_retries times)
        │
        ▼
  You see a color-coded diff showing every changed line
        │
        ▼
  Keep the changes, or undo with one keystroke
```

A **backup** of all your tour files is created before editing, so if anything goes wrong you can always restore the originals.

---

## Config Location

Settings are stored in your home directory:

- **Linux / macOS:** `~/.krpanocode/.env`
- **Windows:** `%USERPROFILE%\.krpanocode\.env`

This file contains your API key and default model. It is created by `--setup` (readable/writable by you only). You can edit it manually if needed:

```
PANOMATICS_API_KEY=your-key-here
KRpanocode_MODEL=glm-5.2-coding
```

---

## License

Private project. All rights reserved.
