# Java 25 for macOS High Sierra 

This is a port of Java 25 for macOS High Sierra (Could also probably work on macOS Mojave but hasn't been tested)

You can download it in the releases tab

If you plan to run modern Minecraft you will need to replace the games built in OpenAL with a different one
I downloaded OpenAL through MacPorts by running this command ```sudo port install openal-soft```

## Troubleshooting

### Git Worktree Error: `unknown switch 'z'`

When using Git-related tools or wrappers (such as GitHub Desktop Plus) on macOS High Sierra, you might encounter the following error:

```
error: unknown switch `z'
usage: git worktree add [<options>] <path> [<commit-ish>]
   or: git worktree list [<options>]
...
```

#### Cause
The `-z` flag for `git worktree list --porcelain` was introduced in **Git version 2.36.0**. macOS High Sierra's default bundled Git (Apple Git) is older (usually version 2.15.x or 2.18.x) and does not support this switch, causing commands utilizing it to fail.

#### How to Fix

You can easily resolve this by upgrading the system Git version to **2.36.0 or newer**:

1. **Via MacPorts:**
   Install the latest Git version by running:
   ```bash
   sudo port install git
   ```

2. **Via Homebrew:**
   If you use Homebrew, run:
   ```bash
   brew install git
   ```

3. **Verify the Installation:**
   Ensure your shell's `PATH` prioritized the upgraded Git (usually in `/opt/local/bin` or `/usr/local/bin`) over the system Git in `/usr/bin`. Verify with:
   ```bash
   which git
   git --version
   ```
   It should report a Git version of **2.36.0** or newer. Once upgraded, GitHub Desktop Plus and other modern Git tools will work seamlessly with worktrees.
