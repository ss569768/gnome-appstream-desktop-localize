# GNOME AppStream Desktop Localizer

Generate localized `.desktop` overrides for GNOME applications using translations from AppStream metadata.

## Why?

Some GNOME applications provide translated application names through AppStream metadata, but their installed `.desktop` files may contain only a generic entry such as:

```ini
Name=Software
```

instead of localized Desktop Entry names such as:

```ini
Name=Software
Name[zh_TW]=GNOME 軟體
Name[zh_CN]=GNOME 软件
Name[ja]=GNOME ソフトウェア
```

KDE Plasma's application launcher uses the Desktop Entry `Name[...]` fields when displaying application names.

This script bridges the two:

```text
GNOME AppStream metadata
        │
        │ localized <name>
        ▼
Desktop Entry Name[locale]
        │
        ▼
~/.local/share/applications/
        │
        ▼
KDE Plasma Application Launcher
```

## Features

* Scans installed `.desktop` files.
* Finds matching AppStream metadata using the `desktop-id` launchable.
* Reads localized application names from AppStream.
* Generates user-level `.desktop` overrides.
* Does not modify files under `/usr/share/applications`.
* Supports normal system installations and Flatpak exports.
* Supports KDE Plasma 5 and Plasma 6.
* Automatically refreshes the KDE application service cache.
* Can generate only selected locales.
* Includes a `--dry-run` mode.

## Requirements

* Python 3
* AppStream metadata installed on the system
* KDE Plasma if the generated files are intended for the Plasma launcher

No additional Python packages are required.

## Usage

Make the script executable:

```bash
chmod +x gnome-appstream-desktop-localize.py
```

Preview the changes:

```bash
python3 gnome-appstream-desktop-localize.py --dry-run
```

Generate the overrides:

```bash
python3 gnome-appstream-desktop-localize.py
```

Only generate Traditional Chinese translations:

```bash
python3 gnome-appstream-desktop-localize.py --locale zh_TW
```

Generate multiple locales:

```bash
python3 gnome-appstream-desktop-localize.py \
    --locale zh_TW \
    --locale zh_CN \
    --locale ja
```

To process all applications with matching AppStream metadata rather than GNOME applications only:

```bash
python3 gnome-appstream-desktop-localize.py --all-matched
```

## Where are the generated files?

The script writes user-level overrides to:

```text
~/.local/share/applications/
```

For example:

```text
~/.local/share/applications/org.gnome.Software.desktop
```

The original system desktop files are not modified.

## How it works

The script:

1. Scans XDG application directories and Flatpak export directories.
2. Finds application `.desktop` files.
3. Reads the corresponding AppStream metadata.
4. Uses the AppStream `<launchable type="desktop-id">` association to identify the application.
5. Reads localized `<name xml:lang="...">` entries.
6. Converts the locale names to Desktop Entry locale syntax.
7. Adds `Name[locale]=...` entries to a user-level override.
8. Runs `kbuildsycoca6` (or `kbuildsycoca5`) to refresh KDE's application database.

## Example

Suppose the installed GNOME Software desktop entry contains:

```ini
[Desktop Entry]
Name=Software
Exec=gnome-software
Type=Application
```

while its AppStream metadata contains:

```xml
<name>Software</name>
<name xml:lang="zh-TW">GNOME 軟體</name>
<name xml:lang="zh-CN">GNOME 软件</name>
<name xml:lang="ja">GNOME ソフトウェア</name>
```

The generated override will contain:

```ini
Name=Software
Name[zh_TW]=GNOME 軟體
Name[zh_CN]=GNOME 软件
Name[ja]=GNOME ソフトウェア
```

Plasma can then display the appropriate localized application name according to the current locale.

## Limitations

The script depends on the AppStream metadata available on the local system.

If an application does not provide a localized name in AppStream, the script cannot create a translation that does not exist.

Different distributions and packaging formats may store AppStream metadata in different locations. The script therefore checks several standard XDG and Flatpak locations.

Existing user-created `.desktop` overrides are not overwritten unless `--force` is specified.

## License

MIT License.
