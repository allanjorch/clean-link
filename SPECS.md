# clean-link — Specification (Language-Agnostic)

## 1. Overview

`clean-link` is a lightweight command-line utility that takes "share links" from social media and content platforms and produces clean, tracking-free versions of those links.

Its primary goal is to remove unnecessary tracking parameters, referral codes, and other junk while preserving everything required to view or access the original content. The tool is designed to be extremely fast and convenient for daily use, especially via keyboard shortcuts.

## 2. Core Purpose

- Remove tracking and analytics parameters from URLs.
- Canonicalize / normalize links to their simplest reliable form where appropriate.
- Make sharing cleaner and more privacy-friendly.
- Provide an excellent clipboard-centric workflow.

## 3. Primary Use Cases

1. User copies a dirty share link from YouTube, X, Instagram, etc.
2. User triggers `clean-link` (via terminal, hotkey, or script).
3. Tool outputs a clean link and places it in the clipboard.

## 4. Functional Requirements

### 4.1 Input

The tool must accept input in two ways:

- **Command-line argument**: One URL passed as an argument.
- **Clipboard (default when no argument is given)**: Automatically read the current clipboard contents.

When no argument is provided, the tool should:
- Read from the system clipboard.
- Trim leading/trailing whitespace and newlines.
- Treat the result as the input URL.

### 4.2 Output

- Always print the cleaned URL to standard output (stdout).
- When appropriate, copy the cleaned URL back to the system clipboard.

### 4.3 Clipboard Behavior

**Recommended workflow (no arguments):**
- Read from clipboard → Clean URL → Print clean URL → Write clean URL to clipboard.
- After running, the user can immediately paste the clean link anywhere.

**When a URL is provided via argument:**
- Print the clean URL.
- Only copy to clipboard if the user explicitly requests it (e.g. via `--copy` flag).

### 4.4 URL Cleaning Rules

#### General Rules (apply to all URLs)
- Remove common tracking parameters, including but not limited to:
  - `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`, `utm_id`
  - `fbclid`, `gclid`, `dclid`, `msclkid`
  - `si` (YouTube share tracking)
  - `igshid`, `igsh`
  - `mibextid`, `__tn__`
  - `s` (commonly used as tracking on X)
  - Any parameter starting with `utm_`
- Remove fragments (`#...`) unless they are semantically important for the content.
- Prefer `https://` over `http://`.
- Remove unnecessary `www.` and `m.` prefixes from hostnames where safe.

#### Platform-Specific Rules

**YouTube / YouTube Music**
- Extract the video ID from any of the following formats:
  - `youtube.com/watch?v=ID`
  - `youtu.be/ID`
  - `youtube.com/shorts/ID`
  - `youtube.com/embed/ID`
  - `music.youtube.com/watch?v=ID`
- Output the shortest clean form: `https://youtu.be/VIDEO_ID`
- Preserve timestamp parameters (`t=` or `start=`) when present.
- Remove all other tracking parameters.

**X (Twitter)**
- Normalize host to `x.com` (preferred over `twitter.com`).
- Remove tracking parameters (especially `s=`).
- Keep only the essential path (e.g. `/username/status/POST_ID`).

**Instagram**
- Remove tracking parameters (`igshid`, etc.).
- Keep the essential path (`/reel/...`, `/p/...`, etc.).
- Strip `www.` and `m.` prefixes.

**Facebook**
- Remove tracking and referral parameters.
- Keep the essential post or reel path.
- Strip mobile (`m.`) prefixes where appropriate.

**Other Platforms**
- Apply general tracking removal rules.
- Do not rewrite the URL structure unless clearly beneficial.

### 4.5 Flags / Options

At minimum, the tool should support:

- `--copy`, `-c` — Force copying the result to the clipboard (useful when providing a URL as argument).
- `--help`, `-h` — Display usage information.

### 4.6 Error Handling & Robustness

- If the input is not a valid URL, the tool should still attempt to process it or return it unchanged (graceful degradation).
- If clipboard access fails (tools missing or permissions), the tool should still function by printing to stdout and informing the user.
- The tool must not crash on malformed input.
- The tool should be fast and have minimal startup time.

## 5. Non-Functional Requirements

- **Simplicity**: The tool should be small, self-contained, and easy to install.
- **Speed**: Should feel instantaneous.
- **Privacy**: Must not phone home or send any data.
- **Clipboard Integration**: First-class support on Linux (both X11 and Wayland).
- **Extensibility**: Designed so new platforms and cleaning rules can be added easily.
- **No heavy dependencies**: Should work with minimal or no external runtime dependencies beyond common clipboard utilities (`xclip`, `wl-clipboard`).

## 6. Target Environment

- Primary: Linux desktop (X11 and Wayland)
- Should be usable in scripts and via keyboard shortcuts / hotkey daemons.

## 7. Future Considerations (Nice to Have)

- Support for additional platforms (TikTok, Reddit, LinkedIn, Threads, Bluesky, etc.)
- Configurable list of tracking parameters to remove
- Option to preserve or strip timestamps on YouTube
- Support for cleaning multiple URLs at once
- JSON or structured output mode
- Cross-platform support (macOS, Windows) in the future
- Integration as a browser extension or system service

## 8. Success Criteria

A successful run should produce a link that:
- Still opens the original content correctly.
- Contains no obvious tracking or analytics parameters.
- Is as short and clean as reasonably possible.
- Can be safely shared without leaking referrer or tracking information.

---

This specification is intentionally language- and implementation-agnostic so it can serve as a reference for any future version or port of the tool.
