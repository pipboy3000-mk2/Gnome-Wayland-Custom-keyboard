# Canary Angle Keyboard Layout for Linux

A custom XKB keyboard layout implementation of the Canary layout with angle mod for Ubuntu and other Linux distributions.

## What is Canary?

Canary is an alternative keyboard layout designed for comfort and efficiency, optimizing for common English letter patterns while reducing finger travel and awkward key combinations. The "angle mod" variant shifts the bottom-left keys to create a more ergonomic hand position on standard staggered keyboards.

## Layout

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬──────────┐
│  `  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │  9  │  0  │  -  │  =  │ Backspace│
│  ~  │  !  │  @  │  #  │  $  │  %  │  ^  │  &  │  *  │  (  │  )  │  _  │  +  │          │
├─────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬───────┤
│  Tab   │  W  │  L  │  Y  │  P  │  K  │  Z  │  X  │  O  │  U  │  ;  │  [  │  ]  │   \   │
│        │     │     │     │     │     │     │     │     │     │  :  │  {  │  }  │   |   │
├────────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴───────┤
│  Caps     │  C  │  R  │  S  │  T  │  B  │  F  │  N  │  E  │  I  │  A  │  '  │  Enter   │
│           │     │     │     │     │     │     │     │     │     │     │  "  │          │
├───────────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────────┤
│  Shift       │  J  │  V  │  D  │  G  │  Q  │  M  │  H  │  /  │  ,  │  .  │    Shift    │
│              │     │     │     │     │     │     │     │  ?  │  <  │  >  │             │
├────────┬─────┴─┬───┴──┬──┴─────┴─────┴─────┴─────┴─────┴──┬──┴───┬─┴────┬┴─────┬───────┤
│  Ctrl  │ Super │ Alt  │             Space                 │ AltGr│ Super│ Menu │ Ctrl  │
└────────┴───────┴──────┴───────────────────────────────────┴──────┴──────┴──────┴───────┘
```

## Features

- Full Canary layout with angle mod
- Compatible with X11 and Wayland (tested on Ubuntu 25.10)
- Installs as a variant of the US keyboard layout
- Accessible through standard system keyboard settings

## Quick Install

### 1. Download the layout file

```bash
git clone https://github.com/yourusername/canary-angle-linux.git
cd canary-angle-linux
```

### 2. Append to US symbols

```bash
cat canaryangle | sudo tee -a /usr/share/X11/xkb/symbols/us > /dev/null
```

### 3. Register the variant

Open the XML configuration:

```bash
sudo nano /usr/share/X11/xkb/rules/evdev.xml
```

Search for `<name>us</name>` (`Ctrl+W`), find the `<variantList>` section, and add:

```xml
        <variant>
          <configItem>
            <name>canaryangle</name>
            <description>English (Canary W/ang)</description>
          </configItem>
        </variant>
```

### 4. Verify installation

```bash
setxkbmap -layout us -variant canaryangle -print
```

### 5. Activate

Log out and back in, then go to **Settings → Keyboard → Input Sources** and add "English (Canary W/ang)".

### 6. (Optional) Enable at login screen and TTY

To use your layout at the login screen and virtual consoles, edit `/etc/default/keyboard`:

```bash
sudo nano /etc/default/keyboard
```

Set these values:

```
XKBMODEL="pc105"
XKBLAYOUT="us"
XKBVARIANT="canaryangle"
XKBOPTIONS=""
```

Apply and reboot:

```bash
sudo dpkg-reconfigure keyboard-configuration
sudo reboot
```

See [INSTALL-CUSTOM-KEYBOARD-LAYOUT.md](INSTALL-CUSTOM-KEYBOARD-LAYOUT.md) for more options.

## Where Does the Layout Work?

| Environment | Works After Basic Install? | Additional Setup |
|-------------|---------------------------|------------------|
| Desktop Session | ✅ Yes | — |
| GUI Terminals | ✅ Yes | — |
| Login Screen (GDM) | ❌ No | See Step 6 |
| Virtual Console (TTY) | ❌ No | See Step 6 |
| SSH Sessions | ✅ Yes | — |

## Files

| File | Description |
|------|-------------|
| `canaryangle` | XKB symbols file for the Canary Angle layout |
| `INSTALL-CUSTOM-KEYBOARD-LAYOUT.md` | Detailed installation guide with troubleshooting |
| `README.md` | This file |

## Compatibility

| Distro | Version | Display Server | Status |
|--------|---------|----------------|--------|
| Ubuntu | 25.10 | Wayland | ✅ Tested |
| Ubuntu | 24.04 LTS | X11 | ✅ Tested |
| Ubuntu | 22.04 LTS | X11/Wayland | Should work |
| Debian | 12+ | X11/Wayland | Should work |
| Fedora | 38+ | X11/Wayland | Should work |

## Troubleshooting

See [INSTALL-CUSTOM-KEYBOARD-LAYOUT.md](INSTALL-CUSTOM-KEYBOARD-LAYOUT.md) for detailed troubleshooting steps.

**Common issues:**

- **Layout not appearing in Settings**: Run `sudo dpkg-reconfigure xkb-data` and log out/in
- **XML errors**: Verify proper nesting and that `<name>` matches the symbols file exactly
- **Keys not working**: Check key codes in the symbols file against the reference diagrams

## Contributing

Feel free to open issues or submit pull requests if you:

- Find bugs or compatibility issues
- Want to add support for other Canary variants
- Have improvements to the installation process

## Resources

- [Canary Layout Official Site](https://github.com/Apsu/Canary)
- [XKB Configuration Guide](https://www.x.org/wiki/XKB/)
- [Keyboard Layout Editor](http://www.keyboard-layout-editor.com/)

## License

This project is released into the public domain. Feel free to use, modify, and distribute as you see fit.

## Acknowledgments

- The Canary layout creators for the original layout design
- The XKB developers for the flexible keyboard configuration system
