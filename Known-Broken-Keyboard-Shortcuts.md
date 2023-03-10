# Keyboard shortcut conflicts

## Application level

In some applications, some keyboard shortcuts no longer work with the AZERTY-NF layout.

Most Windows applications rely on a combination of the _virtual key code_ and modifiers (<kbd>Shift</kbd>, <kbd>Ctrl</kbd>, <kbd>Alt</kbd> or <kbd>AltGr</kbd>) to detect the keyboard shortcut being typed. By modifying the location of certain characters, the AZERTY-NF layout breaks keyboard shortcuts in some applications.

Furthermore, the AZERTY-NF introduces an extensive set of new diacriticals implemented in the form of _dead keys_. Keyboard shortcuts that relied on combinations that were not mapped in the traditional AZERTY layout may be broken as a result.

This page lists known applications and the broken keyboard shortcuts.

### Google Sheets

The following keyboard shortcuts no longer work:

- Paste format only: <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>v</kbd>.

**Solution**: please, use the `Edit | Paste special | Paste format only` menu item instead.

### Microsoft Office

In most Office apps, it is impossible to type an `_` (underscore) character in the _backstage_ area.

For instance. in the `Save As` screen, it is impossible to type an `_` when typing the new name under which the document must be saved as because it conflicts with the <kbd>AltGr</kbd>+<kbd>8</kbd> keyboard shortcut.

**Solution**: please, take advantage of the numeric keypad and punch in the <kbd>Alt</kbd>+<kbd>0</kbd><kbd>9</kbd><kbd>5</kbd> [Alt code](https://en.wikipedia.org/wiki/Alt_code) instead.

![image](https://user-images.githubusercontent.com/8488398/71819285-bbf1e100-308b-11ea-9984-839f6fdbc178.png)

## System level

Some applications make use of global shortcuts to trigger an action from anywhere.
Those are notorious for getting in the way of advanced international keyboard layout.

### Nvidia Geforce Experience In-Game Overlay

By default Nvidia Geforce Experience will break <kbd>AltGr</kbd> functionality for the <kbd>M</kbd>, <kbd>R</kbd> and <kbd>Z</kbd> keys.

To fix it, go to GeForce Experience settings and disable the In-Game Overlay.
If you still want to use the overlay you will need the redefine the conflicting shortcuts instead.
To do so, go to the Keyboard shortcuts section of the In-Game Overlay settings.
Note that it can't make the difference between left and right modifiers which is not helping.
To avoid conflicts with your keyboard layout and application level shortcuts we recommend you use <kbd>Ctrl</kbd>+<kbd>Shift</kbd> as modifiers notably when combining with printable character keys.

### Dell Display Manager

Dell Display Manager lets you define various global shortcuts from the Input Manager and Options tabs depending of your display capabilities. You may want to double check they don't conflict with your keyboard layout.
