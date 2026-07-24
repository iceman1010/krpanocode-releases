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

## All Options

| Flag | Description |
|------|-------------|
| `-p, --prompt` | Your edit instruction, in plain English (asked interactively if omitted) |
| `-f, --folder` | Path to your tour folder (default: current directory) |
| `-m, --model` | Use a specific AI model for this run (also saves it as your new default) |
| `-c, --clarify` | Ask the AI to confirm it understood before editing |
| `--no-docs` | Skip KRPano documentation lookup (faster, less accurate for obscure features) |
| `--models` | List all models available on your proxy and exit |
| `--setup` | Configure your API key and default model interactively |
| `-y, --yes` | Keep changes automatically, no confirmation prompt |
| `-u, --update` | Check for and install the latest version |
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
  (optional) DocSearch finds relevant KRPano docs
  from 27 curated files covering KRPano 1.23.3
        │
        ▼
  (--clarify) AI checks if instruction is clear —
  asks you a question if it's ambiguous
        │
        ▼
  AI reads and edits the files using tool calls
  (reads each file, then writes back the changes)
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
