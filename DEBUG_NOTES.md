# Debugging Session: Zero-Window Bug

## Symptom
App stops closing zero-window apps after a few hours of operation. Restarting the app fixes it temporarily.

## Current Hypothesis
Swindler's AX (Accessibility) observer or its internal window cache goes stale.

## Changes Made for Testing
1. **SwiftQuit.swift**: Added a `print` statement inside `activateAutomaticAppClosing()` to log `pid`, `knownWindows` count, and `trusted` status every time a window is destroyed.
2. **AppDelegate.swift**: Restored the standard `AXSwift.checkIsProcessTrusted` guard at launch. (We temporarily bypassed this during local builds due to macOS TCC signature cache issues, but it is now back in place for the permanent build).

## How to Check Logs
When the bug occurs (app stops closing zero-window apps), open **Console.app**, search for `WindowDestroyed`, and look for the output from Swift Quit.

- **If logs stop appearing entirely**: Swindler's observer died. We need to rebuild/re-initialize it on wake or periodically (Option B fix).
- **If logs appear but `knownWindows` > 0** for an app with no windows: Swindler's cache is stale. We need to bypass the cache and query the AX API directly (Option A fix).
