# clean-link

A lightweight CLI tool that strips tracking parameters from social media share links. Works with YouTube, X (Twitter), Instagram, Facebook, and any URL with common tracking tags.

## Usage

```bash
# Clean a URL from the clipboard (auto-copies result back)
clean-link

# Clean a specific URL
clean-link "https://www.youtube.com/watch?v=dQw4w9WgXcQ&si=abc123"
# → https://youtu.be/dQw4w9WgXcQ

# Clean a URL and copy the result to clipboard
clean-link -c "https://twitter.com/user/status/123?s=20"
```

## Installation

### From source

```bash
git clone https://github.com/allanjorch/clean-link.git
cd clean-link
cargo build --release
cp target/release/clean-link ~/.local/bin/
```

### Dependencies

Requires a clipboard utility for the default clipboard-first workflow:

- **Wayland**: `wl-clipboard` (`wl-paste` / `wl-copy`)
- **X11**: `xclip`

The tool works fine without either — you can always pass a URL as an argument and read the result from stdout.

## How it works

### Input

- **No argument**: reads from the system clipboard
- **URL argument**: cleans the given URL
- **`--copy` / `-c`**: forces clipboard copy (useful with a URL argument)

### Cleaning rules

Two-phase tracking removal:

1. **General** — parameters removed from every URL (`utm_source`, `fbclid`, `gclid`, etc.)
2. **Platform-specific** — parameters removed only when the host matches a known platform (`si` → YouTube, `s` → X, `igshid` → Instagram, `mibextid` → Facebook)

Additional normalization:
- Upgrades `http://` → `https://`
- Strips `www.` and `m.` subdomain prefixes
- Normalizes `twitter.com` → `x.com`
- Strips URL fragments (`#...`)
- Reconstructs YouTube URLs to the shortest form (`youtu.be/ID`), preserving timestamps

### Output

- Always prints the cleaned URL to stdout
- Automatically copies to clipboard when reading from clipboard (default workflow)
- Only copies with `--copy` when a URL is provided as an argument

## Configuration

On first run, `clean-link` creates `~/.config/clean-link/config.toml` with documented defaults. Edit this file to customize tracking parameters or add new platforms:

```toml
[general]
tracking_params = ["utm_source", "fbclid", "gclid"]
tracking_prefixes = ["utm_"]

[platforms.tiktok]
domains = ["tiktok.com", "www.tiktok.com", "m.tiktok.com"]
tracking_params = ["_t"]
```

Platforms with complex URL restructuring (currently only YouTube) require a built-in cleaner and can't be defined purely via config.

## Adding a platform

If the new platform only needs tracking-parameter removal and host normalization, add a `[platforms.<name>]` block to the config:

```toml
[platforms.reddit]
domains = ["reddit.com", "www.reddit.com", "old.reddit.com"]
tracking_params = ["utm_source", "share_id"]
normalize_host = "reddit.com"
```

If the platform needs custom URL reconstruction (like YouTube → `youtu.be/ID`), open an issue or send a pull request.

## License

MIT

---

Built with [Allan Jørch](https://github.com/allanjorch) and [Claude Code](https://claude.ai) (opencode).
