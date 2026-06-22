# MxMessageBox — usage examples

Copy-pasteable examples for common scenarios. API reference lives in [../SKILL.md](../SKILL.md); property nuances in [customization.md](customization.md).

All examples assume:

```csharp
using Eremex.AvaloniaUI.Controls; // MxMessageBox + MessageBoxButtons/Icon/Result
using Avalonia.Controls;          // Window, HorizontalAlignment
```

`owner` below is the parent `Window` (e.g. `(Window)this.VisualRoot` or your `MainWindow`). Pass `null` to resolve to the currently active window.

## 1. Minimal OK message

```csharp
MxMessageBox.Show(owner, "Document saved.");
```

Defaults: single **OK** button, no icon, title from owner. Returns `MessageBoxResult.Ok` (you can ignore it).

## 2. Information with a title

```csharp
MxMessageBox.Show(owner,
    text:  "The report was generated successfully.",
    title: "Report",
    icon:  MessageBoxIcon.Information);
```

## 3. Yes / No confirmation — async with branching

```csharp
var result = await MxMessageBox.ShowAsync(
    owner,
    "Discard all unsaved changes?",
    "Confirm discard",
    MessageBoxButtons.YesNo,
    MessageBoxIcon.Warning,
    MessageBoxResult.No);            // safer default focused button

if (result == MessageBoxResult.Yes)
{
    await DiscardChangesAsync();
}
```

`DefaultButton: MessageBoxResult.No` means **No** gets focus and reacts to Enter — preferred for destructive confirms so a stray Enter doesn't destroy data.

## 4. OK / Cancel with cancellation

```csharp
var result = await MxMessageBox.ShowAsync(owner,
    "Continue with the upgrade?",
    "Upgrade",
    MessageBoxButtons.OkCancel,
    MessageBoxIcon.Question,
    MessageBoxResult.Ok);

// result is Ok, Cancel, or None
if (result != MessageBoxResult.Ok) return;
```

## 5. Fluent configure overload

Use when you set several options or non-parameterized ones (`ButtonAlignment`, `ClipboardText`, `AllowClose`):

```csharp
var result = await MxMessageBox.ShowAsync(mb =>
{
    mb.Text            = "License expired. Contact support.";
    mb.Title           = "License";
    mb.Buttons         = MessageBoxButtons.Ok;
    mb.Icon            = MessageBoxIcon.Error;
    mb.ButtonAlignment = HorizontalAlignment.Center;
    mb.ClipboardText   = "LIC-EXPIRED-2026-001"; // Ctrl+C copies this exact text
});
```

## 6. Abort / Retry / Ignore — retry loop

```csharp
while (true)
{
    try
    {
        await DoRiskyNetworkCallAsync();
        break; // success
    }
    catch (Exception ex)
    {
        var choice = await MxMessageBox.ShowAsync(
            owner,
            $"Upload failed:\n{ex.Message}",
            "Upload error",
            MessageBoxButtons.AbortRetryIgnore,
            MessageBoxIcon.Error,
            MessageBoxResult.Retry);

        if (choice == MessageBoxResult.Abort)   return;       // give up
        if (choice == MessageBoxResult.Ignore)  break;        // skip
        // Retry → loop again
    }
}
```

> `AbortRetryIgnore` cannot be dismissed with ESC or the window X — the user must pick a button. See [customization.md](customization.md#allowclose--esc-behavior).

## 7. Retry / Cancel

```csharp
var choice = await MxMessageBox.ShowAsync(owner,
    "Connection lost. Try again?",
    "Reconnect",
    MessageBoxButtons.RetryCancel,
    MessageBoxIcon.Warning,
    MessageBoxResult.Retry);

if (choice == MessageBoxResult.Retry) await ReconnectAsync();
```

## 8. Owner resolution

```csharp
// Explicit owner → always modal (ShowDialog), parent disabled while open.
await MxMessageBox.ShowAsync(this, "Hi", "Demo");

// null owner → uses WindowManager active window; modal if one is active.
await MxMessageBox.ShowAsync(null, "Hi", "Demo");

// Configure-only overload has no 'owner' param; Owner stays null and resolves
// to the active window at show time. You CANNOT set Owner from the action.
await MxMessageBox.ShowAsync(mb => mb.Text = "Active-window owned");
```

## 9. Sync usage (avoid when possible)

```csharp
// Blocks via a nested dispatcher loop. Only when you truly can't await.
var result = MxMessageBox.Show(owner, "Done.", "Info");
```

Prefer `ShowAsync` in all async code paths.

## Choosing an overload

| Situation | Use |
|-----------|-----|
| Async code (handlers, VMs, commands) | `ShowAsync(...)` |
| Genuinely synchronous call site | `Show(...)` |
| Many options / non-param options (`ButtonAlignment`, `ClipboardText`, `AllowClose`) | the `Action<MxMessageBox>` overload |
| Just text + a couple of flags | the full-params overload |
