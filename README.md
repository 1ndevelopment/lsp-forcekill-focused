# ForceKill Focused - LSPosed module (OneUI 8.5)

Adds a **Force kill** item to the per-task popup menu in the
Recents / background-apps screen, directly **below Keep open**.
Tap swipes that task card straight up like a manual dismiss and closes the menu, then kills the app's app (focused app only) and removes its Recents card.

## Magisk vs KernelSU?

No separate builds. This is a plain LSPosed APK hooking
`com.android.systemui` (+ OneUI launcher), so it works wherever LSPosed runs:

- Magisk + Zygisk + LSPosed, or
- KernelSU + ZygiskNext + LSPosed

## Install

Use the debug APK (already signed, installable):

```
app/build/outputs/apk/debug/app-debug.apk
```

Rebuild: `gradle --offline :app:assembleDebug` (offline works; Xposed
API stub is vendored in `app/libs/`).

1. Install APK on the phone.
2. LSPosed manager → Modules → enable **ForceKill Focused**.
3. Scope: `com.android.systemui` (+ `com.sec.android.app.launcher` on OneUI).
4. Reboot (or restart SystemUI).
5. Open Recents → tap a task's icon → tap **Force kill** (below Keep open) → toast confirms the package killed.

## How it works (one file: `MainHook.java`)

- OneUI's task menu is a custom DataBinding view (`task_menu_layout` rows of
  `task_menu_item_layout`), not a framework `PopupMenu` - verified via
  on-device resource + stack-trace inspection.
- Hooks `LayoutInflater.inflate()`; every `task_menu_item_layout` inflation
  schedules a `post()` that clones the row natively below the Keep-open row
  (matched by the `task_option_dedicated_app` string). Marker tag = one copy.
- Target pkg: the card itself via TaskView.getTaskPackageName (exact), else view-tag chain
  (`TaskIconView` args), else top non-system recent.
- Kill: hidden `ActivityManager.forceStopPackage()` + `killBackgroundProcesses()`
  + `removeTask()`. Runs as the launcher uid, so it holds the permission.

## Caveats

- OneUI 8.5 moves Recents internals between builds. If the button doesn't
  appear, grab the Recents activity name via
  `adb shell dumpsys activity activities | grep -i recents` and check it
  matches the substring filter; if the wrong app gets killed, the fallback
  path needs the exact OneUI getter name from a `jadx` dump of SecSystemUI.
- Skipped: whitelist UI, kill-all mode, Magisk/KSU flashable zip (unnecessary -
  LSPosed abstracts the root backend). Add when needed.
