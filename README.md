# Java 25 for macOS High Sierra 

This is a port of Java 25 for macOS High Sierra (Could also probably work on macOS Mojave but hasn't been tested)

You can download it in the releases tab

If you plan to run modern Minecraft you will need to replace the games built in OpenAL with a different one
I downloaded OpenAL through MacPorts by running this command ```sudo port install openal-soft```

## Troubleshooting & Porting Guides

### Git Worktree Error: `unknown switch 'z'` in GitHub Desktop Plus

When porting or running Git-related tools or wrappers (such as **GitHub Desktop Plus**) on macOS High Sierra, you might encounter the following error:

```
error: unknown switch `z'
usage: git worktree add [<options>] <path> [<commit-ish>]
   or: git worktree list [<options>]
...
```

#### Cause
The `-z` flag for `git worktree list --porcelain` was introduced in **Git version 2.36.0** (to output NUL-separated instead of newline-separated records). macOS High Sierra's default bundled Git (Apple Git) is older (usually version 2.15.x or 2.18.x) and does not support this switch, causing any tools utilizing it to fail.

To make it work with older system Git versions without forcing users to install a newer Git version, you can apply one of the following two solutions:

---

#### Solution 1: Use a Custom Git Wrapper Script (Easiest)

You can create a lightweight bash wrapper script for `git` that automatically intercepts the `git worktree list` call, strips the unsupported `-z` flag, runs the command, and translates the output newlines to NUL bytes.

1. Create a script named `git` (with no extension) in a directory that takes priority in the application's `PATH` (e.g., `/usr/local/bin/git` or a dedicated wrapper bin directory):

   ```bash
   #!/bin/bash

   # Path to the real system git binary
   REAL_GIT="/usr/bin/git"

   # Check if this is 'git worktree list'
   if [ "$1" = "worktree" ] && [ "$2" = "list" ]; then
       has_porcelain=0
       has_z=0
       other_args=()
       for arg in "${@:3}"; do
           if [ "$arg" = "--porcelain" ]; then
               has_porcelain=1
           elif [ "$arg" = "-z" ]; then
               has_z=1
           else
               other_args+=("$arg")
           fi
       done

       # If -z was specified, run without it and convert newlines to NUL bytes
       if [ $has_z -eq 1 ]; then
           if [ $has_porcelain -eq 1 ]; then
               "$REAL_GIT" worktree list --porcelain "${other_args[@]}" | tr '\n' '\0'
           else
               "$REAL_GIT" worktree list "${other_args[@]}" | tr '\n' '\0'
           fi
           exit ${PIPESTATUS[0]}
       fi
   fi

   # Fallback for all other git commands
   exec "$REAL_GIT" "$@"
   ```

2. Make the wrapper script executable:
   ```bash
   chmod +x git
   ```

---

#### Solution 2: Modify the Source Code in GitHub Desktop Plus (Recommended for Port Developers)

If you have access to modify the source code of GitHub Desktop Plus (TS/JS):

1. **Detect the Git Version:**
   Query the Git version at application startup by running `git --version` and parsing the major/minor versions.

2. **Conditionally Omit `-z`:**
   In the file where `git worktree list` is called (usually inside the Git execution/worktree utilities), adjust the flags dynamically:
   - If the detected Git version is **older than 2.36.0**:
     - Execute: `git worktree list --porcelain` (omit `-z`).
     - Split the stdout string by `\n` to parse records.
   - If the Git version is **2.36.0 or newer**:
     - Execute: `git worktree list --porcelain -z`.
     - Split the stdout string by `\0` (NUL byte) to parse records.

This allows the application to gracefully degrade and work seamlessly with older system Git versions out-of-the-box!
