# Zephyr & Nordic Connect SDK Tips

A collection of useful tips, tricks, and configurations for working with Zephyr RTOS and Nordic Connect SDK (NCS).

## Table of Contents

- [Code Formatting](#code-formatting)
- [Disable Verification Process to Speed Up Flashing](#disable-verification-process-to-speed-up-flashing)
- [Checkpath Usage and Git Hooks](#checkpath-usage-and-git-hooks)

## Code Formatting

### Setting up clang-format in VS Code

Add the following configuration to your VS Code settings file (`~/Library/Application Support/Code/User/settings.json` on macOS):

```json
{
    "editor.rulers": [75,100],
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "ms-vscode.cpptools",
    "C_Cpp.clang_format_path": "/opt/nordic/ncs/toolchains/5c0d382932/bin/clang-format",
    "C_Cpp.clang_format_style": "file:/opt/nordic/ncs/v3.1.0/zephyr/.clang-format",
}
```

## Disable Verification Process to Speed Up Flashing

**Problem:** The verification process during flashing can be very time-consuming, especially for larger firmware images.

**Solution:** Disable the verification step in the nrfutil runner by modifying the `nrf_common.py` file.

#### Steps:

1. Navigate to: `/opt/nordic/ncs/v3.1.0/zephyr/scripts/west_commands/runners/nrf_common.py`
2. Find the `_op_program` method (around line 460)
3. Change the verification option from:
   ```python
   'options': {'chip_erase_mode': erase, 'verify': 'VERIFY_READ'}
   ```
   To:
   ```python
   'options': {'chip_erase_mode': erase, 'verify': 'VERIFY_NONE'}
   ```

## Checkpath Usage and Git Hooks

### Setting up Zephyr's Checkpath with Git Hooks

The Zephyr project includes a `checkpath.pl` script that enforces coding standards and style guidelines. You can integrate this into your Git workflow using hooks to automatically check your code before pushing changes.

#### Setting up a Pre-Push Hook

1. **Create the pre-push hook file:**
   ```bash
   nano .git/hooks/pre-push
   ```

2. **Add the following content** (from [Zephyr Documentation](https://docs.zephyrproject.org/latest/contribute/style/index.html)):
   ```bash
   #!/bin/sh
   remote="$1"
   url="$2"
   
   z40=0000000000000000000000000000000000000000
   
   echo "Run push hook"
   
   while read local_ref local_sha remote_ref remote_sha
   do
       args="$remote $url $local_ref $local_sha $remote_ref $remote_sha"
       exec ${ZEPHYR_BASE}/scripts/series-push-hook.sh $args
   done
   
   exit 0
   ```

3. **Make the hook executable:**
   ```bash
   chmod +x .git/hooks/pre-push
   ```

#### How It Works

- The hook runs automatically before each `git push` operation
- It calls Zephyr's `series-push-hook.sh` script which performs various checks including:
  - Code style validation using `checkpatch.pl`
  - Commit message format validation
  - Signed-off-by verification
- If any checks fail, the push is blocked until issues are resolved

#### Testing the Hook

After setting up the hook, test it by:

1. **Check git status:**
   ```bash
   git status
   ```

2. **Make a test commit:**
   ```bash
   git commit --signoff -m "Test commit message"
   ```

3. **Push changes:**
   ```bash
   git push
   ```

The hook will run and display "Run push hook" along with any checkpath validation results.

#### Environment Requirements

Make sure `ZEPHYR_BASE` environment variable is set correctly:
```bash
export ZEPHYR_BASE=/opt/nordic/ncs/v3.1.0/zephyr
```

This is typically set automatically when you source the Zephyr environment setup script.

## J-Link OB configuration for nRF DK
1) Update J-Link firmware

- Install the latest Segger J-Link package: https://www.segger.com/downloads/jlink/
- Run the J-Link Commander (macOS: `JLinkExe`) and follow prompts to update firmware. Allow the app in System Preferences → Security & Privacy if blocked.

2) macOS: avoid MSD interference

- The DK can enumerate as a Mass Storage Device (MSD). macOS auto-mounting (Finder/Spotlight) can block J-Link operations. Before flashing/debugging either:

   - Disable MSD on the board (if the DK exposes a button/menu), or
   - Run J-Link Commander and issue `MSDDisable`.

- If the MSD is already mounted, unmount it on macOS:

   diskutil list
   diskutil unmountDisk /dev/diskN   # replace N with the disk number

- Re-enable the MSD after you're done with:

   JLinkExe
   MSDEnable

That's it — update firmware, disable or unmount the MSD on macOS, perform your flash/debug, then re-enable the MSD.
