# ForceKill Focused

LSPosed module for **OneUI 8.5** that adds a **Force kill** item to the per-task popup menu in Recents, directly below **Keep open**.

Tapping it swipe-dismisses the task card, closes the menu, force-stops that one app (the tapped task only), and removes its Recents card.

## Requirements

- Samsung device on OneUI 8.5
- Root via one of:
  - Magisk + Zygisk + LSPosed
  - KernelSU + ZygiskNext + LSPosed

There are no separate Magisk/KernelSU builds. It's a plain LSPosed APK hooking `com.android.systemui` (and the OneUI launcher), so it works anywhere LSPosed runs.

## Install

1. Download the APK from [Releases](https://github.com/1ndevelopment/lsp-forcekill-focused/releases) and install it.
2. In LSPosed Manager, go to **Modules** and enable **ForceKill Focused**.
3. Set the scope to `com.android.systemui` and `com.sec.android.app.launcher`.
4. Reboot, or restart SystemUI.
5. Open Recents, tap a task's icon, then tap **Force kill**.

## How it works

Everything lives in one file, `MainHook.java`.

- **Menu injection:** OneUI's task menu is a custom DataBinding view (`task_menu_layout` containing `task_menu_item_layout` rows), not a framework `PopupMenu`. This was verified via on-device resource and stack-trace inspection.
- **Hook point:** `LayoutInflater.inflate()` is hooked. Each `task_menu_item_layout` inflation schedules a `post()` that clones the row below the Keep open row, which is matched by its `task_option_dedicated_app` string. A marker tag ensures only one copy is added.
- **Target resolution**, in order:
  1. The tapped card's `TaskView.getTaskPackageName()` (exact)
  2. The view-tag chain (`TaskIconView` args)
  3. The top non-system recent task
- **Kill sequence:** hidden `ActivityManager.forceStopPackage()`, then `killBackgroundProcesses()`, then `removeTask()`. This runs inside the hooked process (SystemUI / launcher), which holds the required permission.

## Troubleshooting

Recents internals change between OneUI builds.

- **Button doesn't appear:** get the Recents activity name and check that it matches the module's substring filter:
  ```sh
  adb shell dumpsys activity activities | grep -i recents

