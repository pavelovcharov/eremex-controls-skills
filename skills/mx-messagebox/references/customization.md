# MxMessageBox — customization & nuances

Property deep-dives and the non-obvious behaviors. API reference in [../SKILL.md](../SKILL.md); examples in [usage-examples.md](usage-examples.md). All claims verified against [MxMessageBox.cs](../../../../controls/Source/Eremex.Avalonia.Controls/Common/MessageBox/MxMessageBox.cs) and [MessageWindow.axaml.cs](../../../../controls/Source/Eremex.Avalonia.Controls/Common/MessageBox/MessageWindow.axaml.cs).

## `Owner` is read-only from `configure` ⚠️

```csharp
public Window? Owner { get; private set; }   // private setter
```

You **cannot** assign `Owner` inside the `Action<MxMessageBox>` callback — it won't compile:

```csharp
// ❌ does not compile
MxMessageBox.ShowAsync(mb => mb.Owner = this);

// ✅ pass it as the 'owner' argument instead
MxMessageBox.ShowAsync(this, "Hi");
```

What happens with each overload:

| Overload | Owner source |
|----------|--------------|
| `Show(owner, …)` / `ShowAsync(owner, …)` | the `owner` argument |
| `Show(Action<MxMessageBox>)` / `ShowAsync(Action<MxMessageBox>)` | `null` → resolves to the **active window** at show time |

At show time (`MessageWindow.ShowMessageBoxAsync`): `owner = MessageBox.Owner ?? WindowManager.Default.ActiveWindow`.

- If an owner resolves → dialog opens **modally** via `ShowDialog(owner)` and inherits the owner's `Icon`.
- If none resolves → opens non-modally via `Show()` (and may appear in the taskbar if no other window is there). Pass an owner for predictable modal behavior.

## `ButtonAlignment`

```csharp
mb.ButtonAlignment = HorizontalAlignment.Left;   // or Center, Right (default)
```

Controls the horizontal placement of the button row. `HorizontalAlignment` comes from `Avalonia.Layout`.

## `DefaultButton`

```csharp
mb.DefaultButton = MessageBoxResult.Yes;
```

- That button receives focus on open and reacts to **Enter** (it is the `IsDefault` button).
- If you set `None` (the default) or a value **not present** in the chosen `Buttons` set, the **first button** is used as default (see `MessageWindowPresenter.GetButtons`).
- For destructive confirms, default to the safe choice (`No`, `Cancel`) so an accidental Enter doesn't cause harm.

## `AllowClose` & ESC behavior

`AllowClose` (default `true`) governs whether the dialog can be dismissed without picking a button (window **X** and **ESC**).

```csharp
mb.AllowClose = false; // force the user to click a button
```

Important interactions, from `MessageWindow.axaml.cs`:

1. **Forced off for two combos.** Regardless of what you set, `AllowClose` is forced to `false` for:
   - `MessageBoxButtons.YesNo`
   - `MessageBoxButtons.AbortRetryIgnore`

   (`CanCloseOnEscape` returns `false` for these.) The window X is then ignored (`OnClosing` cancels), so the user **must** click a button.

2. **ESC activates the cancel button** (Avalonia's `IsCancel`), not a custom handler:
   - Combos containing `Cancel` (`OkCancel`, `YesNoCancel`, `RetryCancel`) → ESC yields `MessageBoxResult.Cancel`.
   - `Ok`-only → the OK button is cancelable → ESC yields `MessageBoxResult.Ok`.
   - `YesNo` / `AbortRetryIgnore` → no cancelable button → ESC is a no-op (and AllowClose is false anyway).

3. **Setting `AllowClose = false`** on the other combos keeps ESC working through the cancel button but disables the X. Combine with care.

## `ClipboardText` & Ctrl+C

Pressing **Ctrl+C** copies to the clipboard (`OnKeyDown` matching the platform Copy gesture):

1. If text is **selected** in the message body → copies the **selection**.
2. Else if `ClipboardText` is set → copies `ClipboardText`.
3. Else → copies an **auto-generated** block: title, a separator line, the message text, another separator, and the button labels (classic Win32-style).

```csharp
mb.ClipboardText = "ERR-1234 — full diagnostics here"; // exact Ctrl+C payload when nothing selected
```

Useful for error dialogs where you want users to copy a clean error code instead of the formatted block.

## Icon mapping

`MessageBoxIcon` → displayed glyph (sourced from Eremex icon resources inside the presenter):

| `MessageBoxIcon` | Glyph group |
|------------------|-------------|
| `None` | no icon (icon area hidden) |
| `Hand` / `Stop` / `Error` | error |
| `Question` | question |
| `Exclamation` / `Warning` | warning |
| `Asterisk` / `Information` | information |

Aliases (e.g. `Stop` == `Error` == `Hand`) map to the **same** value and glyph — pick whichever reads best at the call site.

## Localization

Built-in button labels come from `.resx` resources and follow the current UI culture:

- Base: `Source/Eremex.Avalonia.Controls/Common/MessageBox/MessageWindow.resx` (English).
- Shipped cultures: `MessageWindow.ru.resx` (Russian), `MessageWindow.zh-Hans.resx` (Chinese Simplified), and others across the codebase.
- Resource keys for buttons: `Ok`, `Cancel`, `Yes`, `No`, `Abort`, `Retry`, `Ignore` (see `MxMessageBox.GetButtonContent`).

**To add a language:** copy the base `.resx` to `MessageWindow.<culture>.resx` (e.g. `MessageWindow.fr.resx`) and translate the values; the strongly-typed `MessageWindow.Designer.cs` accessor exposes the keys automatically. Switch the UI culture at startup (`CultureInfo.CurrentUICulture`) before the dialog is shown.

See the `_shared` skill for the broader localization story across all Eremex controls.

## Window chrome

- The message window is an `MxWindow` with minimal system decorations (`BorderOnly` on Windows/macOS; full or border on Linux depending on X11 forwarding).
- The client-area title bar hint is set to 1 — the dialog uses its own compact title strip rather than the OS chrome.
- The window `Icon` is inherited from the resolved owner.
