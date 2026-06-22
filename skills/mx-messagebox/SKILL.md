---
name: mx-messagebox
description: The Eremex MxMessageBox modal dialog — how to show messages and confirmation prompts in an Avalonia app. This skill should be used when the user asks about MxMessageBox, a message box, an alert/confirm/question dialog, "show yes/no", "show an error dialog", or mentions the Show / ShowAsync methods. Covers the public API signatures, MessageBoxButtons / MessageBoxIcon / MessageBoxResult enums, configurable properties, sync vs async usage, and customization (button alignment, clipboard text, allow-close, default button, localization).
---

# MxMessageBox

`MxMessageBox` is a modal message/question dialog for Eremex Avalonia apps. It exposes a **static API** with both synchronous (`Show`) and asynchronous (`ShowAsync`) methods. Source: [MxMessageBox.cs](../../../../controls/Source/Eremex.Avalonia.Controls/Common/MessageBox/MxMessageBox.cs).

## Namespace

```csharp
using Eremex.AvaloniaUI.Controls; // MxMessageBox + the three enums below
```

> The namespace is `Eremex.AvaloniaUI.Controls` (one word: `AvaloniaUI`), **not** the project name `Eremex.Avalonia.Controls`. See the `_shared` skill for why.

## API signatures

There are two overloads each for `Show` and `ShowAsync` — a full-params form and a fluent `Action<MxMessageBox> configure` form:

```csharp
// Synchronous — blocks the caller until the user dismisses the dialog.
public static MessageBoxResult Show(
    Window? owner, string text, string? title = null,
    MessageBoxButtons buttons = MessageBoxButtons.Ok,
    MessageBoxIcon icon = MessageBoxIcon.None,
    MessageBoxResult defaultButton = MessageBoxResult.None,
    Action<MxMessageBox>? configure = null);

public static MessageBoxResult Show(Action<MxMessageBox> configure);

// Asynchronous — awaitable, non-blocking.
public static Task<MessageBoxResult> ShowAsync(/* same params as Show */);
public static Task<MessageBoxResult> ShowAsync(Action<MxMessageBox> configure);
```

Returns a `MessageBoxResult` (see table) telling which button was clicked.

> The `configure` callback runs while building the dialog and can set any writable property (Text, Title, Buttons, Icon, DefaultButton, ButtonAlignment, ClipboardText, AllowClose). It **cannot** set `Owner` — see [customization.md](references/customization.md).

## Enums

**`MessageBoxButtons`** — which buttons appear:

| Member | Buttons rendered | Can close with ESC? |
|--------|------------------|---------------------|
| `Ok` | OK | yes (→ `Ok`) |
| `OkCancel` | OK, Cancel | yes (→ `Cancel`) |
| `YesNoCancel` | Yes, No, Cancel | yes (→ `Cancel`) |
| `YesNo` | Yes, No | **no** |
| `AbortRetryIgnore` | Abort, Retry, Ignore | **no** |
| `RetryCancel` | Retry, Cancel | yes (→ `Cancel`) |

**`MessageBoxIcon`** — the icon on the left (aliases grouped):

| Value | Aliases | Meaning |
|-------|---------|---------|
| `None` | — | no icon |
| `Hand` | `Stop`, `Error` | error / stop |
| `Question` | — | question |
| `Exclamation` | `Warning` | warning |
| `Asterisk` | `Information` | information |

**`MessageBoxResult`** — the return value: `None` (0, dismissed without a choice), `Ok`, `Cancel`, `Yes`, `No`, `Abort`, `Retry`, `Ignore`.

## Configurable properties

Set via the full-params arguments or inside the `configure` action:

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Text` | `string?` | — | main message body |
| `Title` | `string?` | — | window title |
| `Buttons` | `MessageBoxButtons` | `Ok` | button set |
| `Icon` | `MessageBoxIcon` | `None` | left-side icon |
| `DefaultButton` | `MessageBoxResult` | `None` | focused button; `None`/invalid → first button |
| `ButtonAlignment` | `HorizontalAlignment` | `Right` | `Left` / `Center` / `Right` |
| `ClipboardText` | `string?` | — | custom Ctrl+C payload |
| `AllowClose` | `bool` | `true` | allow X / ESC close; forced `false` for `YesNo` & `AbortRetryIgnore` |
| `Owner` | `Window?` | — | **read-only via `configure`**; set it via the `owner` parameter |

## Quick examples

```csharp
// 1. Minimal OK message (sync). 'owner' is your parent Window; null = active window.
MxMessageBox.Show(owner, "Saved successfully.");

// 2. Yes/No question, async.
var result = await MxMessageBox.ShowAsync(
    owner, "Delete this item?", "Confirm",
    MessageBoxButtons.YesNo, MessageBoxIcon.Warning, MessageBoxResult.No);

if (result == MessageBoxResult.Yes) { /* delete */ }

// 3. Fluent configure overload — set many options at once.
var result = await MxMessageBox.ShowAsync(mb =>
{
    mb.Text     = "Operation failed.";
    mb.Title    = "Error";
    mb.Buttons  = MessageBoxButtons.OkCancel;
    mb.Icon     = MessageBoxIcon.Error;
    mb.AllowClose = false;
    mb.ClipboardText = "ERR-1234";
});
```

More copy-paste examples (confirmation flows, retry loops, owner handling) are in [usage-examples.md](references/usage-examples.md). Property deep-dives and gotchas are in [customization.md](references/customization.md).

## Sync vs async — when to use which

- **`ShowAsync`** in async contexts, event handlers, commands, ViewModels — the normal choice. `await` it.
- **`Show`** (sync) blocks via a nested dispatcher loop. Use only when you genuinely cannot be async (rare). Avoid from the UI thread if re-entrancy is a concern.

## Modality note

- With a resolvable **owner**, the dialog opens modally (`ShowDialog`) and disables the parent until dismissed.
- Without an owner and with no active window, it opens non-modally (`Show`). Pass an `owner` (or ensure a window is active) for a true modal dialog.

## Related skills

- `_shared` — namespace gotcha, theming, localization.
