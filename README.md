# Cinnamon Windows Quick List Applet With Custom Keyboard Shortcut to Trigger

## Changes

- Added support for configurable keyboard shortcut to open/close the window list.
- The shortcut can be customized in the applet settings.
- The default shortcut is `Super + a`.
- The applet now responds to the following key events when focused:
 - `Space` or `Enter`: Toggle the window list
 - `Escape`: Close the window list if open
 - `Down Arrow`: Open the window list if closed and navigate focus to the first item

## Installation

1. Clone the repo in `/usr/share/cinnamon/applets/windows-quick-list@cinnamon.org`
2. Enable the applet in Cinnamon settings.

## Configuration

The keyboard shortcut to open/close the window list can be configured in the applet settings. To access the settings:

1. Right-click on the applet.
2. Select "Configure".
3. Modify the "Show List" keybinding to your desired shortcut.

## Usage

- Click on the applet icon to open the window list.
- Click on a window item to activate that window.
- Use the configured keyboard shortcut to open/close the window list.
- When the applet is focused:
 - Press `Space` or `Enter` to toggle the window list.
 - Press `Escape` to close the window list if it is open.
 - Press `Down Arrow` to open the window list if it is closed and navigate focus to the first item.

## Changes

- Added the `Settings` import:
```
const Settings = imports.ui.settings;
```

- Added settings handling in the `CinnamonWindowsQuickListApplet` constructor:
```
this.settings = new Settings.AppletSettings(
  this,
  metadata.uuid,
  instance_id
);
this.settings.bindProperty(
  Settings.BindingDirection.IN,
  "keyOpen",
  "keybinding",
  this.on_keybinding_changed,
  null
);
this.actor.connect(
  "key-press-event",
  Lang.bind(this, this._onSourceKeyPress)
);

this.on_keybinding_changed();
```

- Added the `_onSourceKeyPress` method to handle key press events:
```
_onSourceKeyPress(actor, event) {
  let symbol = event.get_key_symbol();

  if (symbol == Clutter.KEY_space || symbol == Clutter.KEY_Return) {
    this.menu.toggle();
    return true;
  } else if (symbol == Clutter.KEY_Escape && this.menu.isOpen) {
    this.menu.close();
    return true;
  } else if (symbol == Clutter.KEY_Down) {
    if (!this.menu.isOpen) this.menu.toggle();
    this.menu.actor.navigate_focus(this.actor, Gtk.DirectionType.DOWN, false);
    return true;
  } else return false;
}
```

- Added the `on_keybinding_changed` method to update the hotkey binding:
```
on_keybinding_changed() {
  Main.keybindingManager.addHotKey(
    "must-be-unique-id",
    this.keybinding,
    Lang.bind(this, this.on_hotkey_triggered)
  );
}
```

- Added the `on_hotkey_triggered` method to handle the hotkey trigger:
```
on_hotkey_triggered() {
  if (this.menu.isOpen) {
    this.menu.close(false);
  } else {
    this.updateMenu();
    this.menu.open(true);
  }
}
```

- Added a new file `settings-schema.json`
```
{
  "section1": {
    "type": "section",
    "description": "Behavior"
  },
  "keyOpen": {
    "type": "keybinding",
    "description": "Show List",
    "default": "<Super>a",
    "tooltip": "Shortcut to open/close the window list"
  }
}
```
