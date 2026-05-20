# KRpanoCode

LLM-powered KRPano XML editor for virtual tours.

## Quick start

```bash
chmod +x krpanocode.phar
./krpanocode.phar
```

## Options

| Flag | Description |
|------|-------------|
| `-p`, `--prompt` | Instruction for the edit (e.g. "add a info hotspot to each scene") |
| `-f`, `--folder` | Path to your tour folder (default: current directory) |
| `-c`, `--clarify` | Ask clarifying questions before making changes |
| `--no-docs` | Skip fetching KRPano documentation |
| `-y`, `--yes` | Auto-confirm changes without prompting |
| `-u`, `--update` | Check for and install updates |
| `--version` | Show version |

## Examples

```bash
# Interactive mode
./krpanocode.phar

# One-shot command
./krpanocode.phar -p "add a floorplan thumbnail hotspot" -f /path/to/tour

# With clarifying questions
./krpanocode.phar -p "change skin" -c

# Update to latest version
./krpanocode.phar --update
```

## Requirements

- PHP 8.1 or later

## Downloads

Download the latest PHAR from the [Releases](https://github.com/iceman1010/krpanocode-releases/releases) page.
