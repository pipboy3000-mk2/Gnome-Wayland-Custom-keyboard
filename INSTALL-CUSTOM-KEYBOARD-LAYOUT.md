# Installing a Custom XKB Keyboard Layout on Ubuntu (X11 & Wayland)

This guide walks you through adding a custom keyboard layout (such as Canary, Colemak variants, etc.) to Ubuntu. These instructions have been tested on Ubuntu 25.10 with Wayland but should work on Ubuntu 22.04+ with either X11 or Wayland.

## Overview

XKB (X Keyboard Extension) is the keyboard configuration system used by both X11 and Wayland on Linux. Custom layouts are added by:

1. Creating a symbols file with your key mappings
2. Appending it to an existing layout file (like `us`)
3. Registering the variant in the XML configuration
4. Testing and activating the layout

## Prerequisites

- Ubuntu 22.04 or later (tested on 25.10)
- Root/sudo access
- A text editor (nano, vim, etc.)
- Your custom keyboard layout symbols file

---

## Step 1: Prepare Your Symbols File

Your symbols file should follow this format:

```
// Description of your layout
// Author information (optional)

partial alphanumeric_keys
xkb_symbols "yourlayoutname" {
    include "us(basic)"
    
    name[Group1]= "English (Your Layout Name)";

    // Key definitions
    key <AD01>	{[	 w,	 W		]};
    // ... rest of your key mappings
};
```

### Important syntax notes:

- **Do NOT use `default`** in your variant — only the base layout should have this
- **Use a unique variant name** (e.g., `"canaryangle"`, not `"basic"`)
- **Include the base layout** with `include "us(basic)"` if you want to inherit unmapped keys
- **Key codes reference:**
  - `AD01-AD12` = Top letter row (Q-] on QWERTY)
  - `AC01-AC11` = Home row (A-' on QWERTY)
  - `AB01-AB10` = Bottom letter row (Z-/ on QWERTY)
  - `AE01-AE12` = Number row (1-=)
  - `TLDE` = Tilde/Grave key
  - `BKSL` = Backslash

---

## Step 2: Append Your Layout to the US Symbols File

Copy your symbols file to a convenient location first (especially if the path contains spaces):

```bash
cp "/path/to/your/layout-file" /tmp/mylayout
```

Then append it to the US symbols file:

```bash
cat /tmp/mylayout | sudo tee -a /usr/share/X11/xkb/symbols/us > /dev/null
```

### Verify it was added:

```bash
tail -50 /usr/share/X11/xkb/symbols/us
```

You should see your layout at the end of the file.

---

## Step 3: Register the Variant in evdev.xml

Open the XML configuration file:

```bash
sudo nano /usr/share/X11/xkb/rules/evdev.xml
```

### Find the US layout section:

Press `Ctrl + W` and search for:

```
<name>us</name>
```

You may need to repeat the search (`Ctrl + W`, then `Enter`) until you find the one inside a `<layout>` block that looks like this:

```xml
    <layout>
      <configItem>
        <name>us</name>
        <shortDescription>en</shortDescription>
        <description>English (US)</description>
        ...
      </configItem>
      <variantList>
        <variant>
          ...
```

### Add your variant:

Inside the `<variantList>` section (alongside other variants like `euro`, `intl`, `dvorak`), add:

```xml
        <variant>
          <configItem>
            <name>yourlayoutname</name>
            <description>English (Your Layout Name)</description>
          </configItem>
        </variant>
```

**Make sure:**
- The `<name>` matches exactly what you used in your symbols file (e.g., `canaryangle`)
- The `<description>` is human-readable and will appear in Settings
- Your XML is properly indented and nested within `<variantList>`

Save and exit (`Ctrl + X`, then `Y`, then `Enter` in nano).

---

## Step 4: (Optional) Update evdev.lst

This step is optional but recommended for compatibility with some tools:

```bash
sudo nano /usr/share/X11/xkb/rules/evdev.lst
```

Find the `! variant` section and add a line in the US variants area:

```
  yourlayoutname  us: English (Your Layout Name)
```

Save and exit.

---

## Step 5: Safety Checks (Before Logging Out)

Before logging out, run these tests to verify your configuration is valid.

### Test 1: Validate XKB can parse your layout

```bash
setxkbmap -layout us -variant yourlayoutname -print
```

**Expected output:** You should see an `xkb_keymap` block that includes your variant:

```
xkb_keymap {
    xkb_keycodes  { include "evdev+aliases(qwerty)" };
    xkb_types     { include "complete" };
    xkb_compat    { include "complete" };
    xkb_symbols   { include "pc+us(yourlayoutname)+inet(evdev)" };
    xkb_geometry  { include "pc(pc105)" };
};
```

If you see an error instead, check your symbols file syntax.

### Test 2: Compile the layout (dry run)

```bash
setxkbmap -layout us -variant yourlayoutname -print | xkbcomp -xkb - /tmp/test.xkb
```

**Expected output:** Many warnings about `<I###>` keys — **this is normal!** These are extended media keys that X11 doesn't support. As long as you don't see errors specifically mentioning your key definitions, you're good.

### Test 3: Check system recognition (optional)

```bash
localectl list-x11-keymap-variants us | grep yourlayoutname
```

If this doesn't show your variant, run:

```bash
sudo dpkg-reconfigure xkb-data
```

Then try again. Note: This test may not work until after a logout/login even if everything is configured correctly.

---

## Step 6: Activate Your Layout

### Option A: Log out and back in

1. Log out of your session
2. Log back in
3. Go to **Settings → Keyboard → Input Sources**
4. Click **+** to add an input source
5. Search for your layout name (e.g., "Canary")
6. Select it and add

### Option B: Reload XKB data (may avoid logout)

```bash
sudo dpkg-reconfigure xkb-data
```

Then check Settings → Keyboard → Input Sources.

---

## Troubleshooting

### Layout doesn't appear in Settings

1. Verify the variant name in `evdev.xml` matches your symbols file exactly
2. Check for XML syntax errors: `xmllint --noout /usr/share/X11/xkb/rules/evdev.xml`
3. Run `sudo dpkg-reconfigure xkb-data`
4. Try logging out and back in

### Keys not mapping correctly

1. Verify your key codes are correct (AD01, AC01, etc.)
2. Check that you're using the right syntax: `key <CODE> {[ lowercase, UPPERCASE ]};`
3. Test with `xev` to see what keycodes your keyboard sends

### "Cannot load symbols" error

1. Check for typos in your symbols file
2. Verify all brackets and semicolons are balanced
3. Make sure the variant name contains only alphanumeric characters and underscores

### Wayland-specific notes

- The `setxkbmap` command will show a warning about Xwayland — this is normal
- XKB symbols files work identically on X11 and Wayland
- GNOME Settings is the primary way to switch layouts on Wayland

---

## File Locations Reference

| File | Purpose |
|------|---------|
| `/usr/share/X11/xkb/symbols/us` | US keyboard layout symbols (append your layout here) |
| `/usr/share/X11/xkb/rules/evdev.xml` | Layout registry for GUI tools (register your variant here) |
| `/usr/share/X11/xkb/rules/evdev.lst` | Legacy layout list (optional) |

---

## Backup Recommendation

Before making changes, back up the original files:

```bash
sudo cp /usr/share/X11/xkb/symbols/us /usr/share/X11/xkb/symbols/us.backup
sudo cp /usr/share/X11/xkb/rules/evdev.xml /usr/share/X11/xkb/rules/evdev.xml.backup
```

To restore if something goes wrong:

```bash
sudo cp /usr/share/X11/xkb/symbols/us.backup /usr/share/X11/xkb/symbols/us
sudo cp /usr/share/X11/xkb/rules/evdev.xml.backup /usr/share/X11/xkb/rules/evdev.xml
```

---

## Example: Canary Angle Layout

Here's a complete example symbols file for the Canary layout with angle mod:

```
// Canary Angle keyboard layout variant for US
// Custom layout based on the Canary layout with angle mod

partial alphanumeric_keys
xkb_symbols "canaryangle" {
    include "us(basic)"
    
    name[Group1]= "English (Canary W/ang)";

    // Upper alpha row (AD01-AD10)
    key <AD01>	{[	 w,	 W		]};
    key <AD02>	{[	 l,	 L		]};
    key <AD03>	{[	 y,	 Y		]};
    key <AD04>	{[	 p,	 P		]};
    key <AD05>	{[	 k,	 K		]};
    key <AD06>	{[	 z,	 Z		]};
    key <AD07>	{[	 x,	 X		]};
    key <AD08>	{[	 o,	 O		]};
    key <AD09>	{[	 u,	 U		]};
    key <AD10>	{[ semicolon,	 colon		]};

    // Home row (AC01-AC10)
    key <AC01>	{[	 c,	 C		]};
    key <AC02>	{[	 r,	 R		]};
    key <AC03>	{[	 s,	 S		]};
    key <AC04>	{[	 t,	 T		]};
    key <AC05>	{[	 b,	 B		]};
    key <AC06>	{[	 f,	 F		]};
    key <AC07>	{[	 n,	 N		]};
    key <AC08>	{[	 e,	 E		]};
    key <AC09>	{[	 i,	 I		]};
    key <AC10>	{[	 a,	 A		]};

    // Bottom row (AB01-AB10)
    key <AB01>	{[	 j,	 J		]};
    key <AB02>	{[	 v,	 V		]};
    key <AB03>	{[	 d,	 D		]};
    key <AB04>	{[	 g,	 G		]};
    key <AB05>	{[	 q,	 Q		]};
    key <AB06>	{[	 m,	 M		]};
    key <AB07>	{[	 h,	 H		]};
    key <AB08>	{[   slash,	 question	]};
    key <AB09>	{[   comma,	 less		]};
    key <AB10>	{[  period,	 greater	]};
};
```

And the corresponding `evdev.xml` entry:

```xml
        <variant>
          <configItem>
            <name>canaryangle</name>
            <description>English (Canary W/ang)</description>
          </configItem>
        </variant>
```

---

---

## Visual Keyboard Diagrams

### Standard QWERTY Layout (Reference)

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬──────────┐
│  `  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │  9  │  0  │  -  │  =  │ Backspace│
│  ~  │  !  │  @  │  #  │  $  │  %  │  ^  │  &  │  *  │  (  │  )  │  _  │  +  │          │
├─────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬───────┤
│  Tab   │  Q  │  W  │  E  │  R  │  T  │  Y  │  U  │  I  │  O  │  P  │  [  │  ]  │   \   │
│        │AD01 │AD02 │AD03 │AD04 │AD05 │AD06 │AD07 │AD08 │AD09 │AD10 │AD11 │AD12 │ BKSL  │
├────────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴───────┤
│  Caps     │  A  │  S  │  D  │  F  │  G  │  H  │  J  │  K  │  L  │  ;  │  '  │  Enter   │
│           │AC01 │AC02 │AC03 │AC04 │AC05 │AC06 │AC07 │AC08 │AC09 │AC10 │AC11 │          │
├───────────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────────┤
│  Shift       │  Z  │  X  │  C  │  V  │  B  │  N  │  M  │  ,  │  .  │  /  │    Shift    │
│              │AB01 │AB02 │AB03 │AB04 │AB05 │AB06 │AB07 │AB08 │AB09 │AB10 │             │
├────────┬─────┴─┬───┴──┬──┴─────┴─────┴─────┴─────┴─────┴──┬──┴───┬─┴────┬┴─────┬───────┤
│  Ctrl  │ Super │ Alt  │             Space                 │ AltGr│ Super│ Menu │ Ctrl  │
└────────┴───────┴──────┴───────────────────────────────────┴──────┴──────┴──────┴───────┘
```

### Canary Angle Layout

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

### XKB Key Code Map

Use this reference when creating your own layout:

```
Number Row:
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ TLDE │ AE01 │ AE02 │ AE03 │ AE04 │ AE05 │ AE06 │ AE07 │ AE08 │ AE09 │ AE10 │ AE11 │ AE12 │
│  `~  │  1!  │  2@  │  3#  │  4$  │  5%  │  6^  │  7&  │  8*  │  9(  │  0)  │  -_  │  =+  │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

Top Row:
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ AD01 │ AD02 │ AD03 │ AD04 │ AD05 │ AD06 │ AD07 │ AD08 │ AD09 │ AD10 │ AD11 │ AD12 │ BKSL │
│  Q   │  W   │  E   │  R   │  T   │  Y   │  U   │  I   │  O   │  P   │  [{  │  ]}  │  \|  │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

Home Row:
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ AC01 │ AC02 │ AC03 │ AC04 │ AC05 │ AC06 │ AC07 │ AC08 │ AC09 │ AC10 │ AC11 │
│  A   │  S   │  D   │  F   │  G   │  H   │  J   │  K   │  L   │  ;:  │  '"  │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

Bottom Row:
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ AB01 │ AB02 │ AB03 │ AB04 │ AB05 │ AB06 │ AB07 │ AB08 │ AB09 │ AB10 │
│  Z   │  X   │  C   │  V   │  B   │  N   │  M   │  ,<  │  .>  │  /?  │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
```

### Key Code Naming Convention

| Prefix | Meaning | Row |
|--------|---------|-----|
| `AE` | Alphanumeric E (number row) | Top |
| `AD` | Alphanumeric D (top letter row) | Second |
| `AC` | Alphanumeric C (home row) | Third |
| `AB` | Alphanumeric B (bottom letter row) | Fourth |
| `TLDE` | Tilde key | Top left |
| `BKSL` | Backslash key | Right of top row |

---

## License

This guide is provided as-is for educational purposes. Feel free to adapt it for your own layouts.
