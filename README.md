## Cursor Installer & Updater Script

This Bash script installs Cursor AI IDE as an AppImage, sets up its desktop entry, and can uninstall. The download URL has been updated to point at the stable release on `cursor.com`.

---

### Prerequisites

- **Linux distro** with `bash` (tested on Ubuntu/Debian)
- `sudo` privileges for writing to `/opt` and `/usr/share/applications`
- Internet access to download from `cursor.com`

---

### Installation - Copy paste every command below on your terminal

1. **Save the script**
save it as cursor-installer.sh
```
#!/bin/bash
# cursor-manager.sh
# Usage:
#   sudo ./cursor-manager.sh install   — install Cursor AppImage & desktop entry
#   sudo ./cursor-manager.sh uninstall — remove Cursor AppImage & desktop entry

set -euo pipefail

APPIMAGE_PATH="/opt/cursor.appimage"
ICON_PATH="/opt/cursor.png"
DESKTOP_ENTRY_PATH="/usr/share/applications/cursor.desktop"
CURSOR_URL="https://www.cursor.com/download/stable/linux-x64"
ICON_URL="https://custom.typingmind.com/assets/models/cursor.png"

install_cursor() {
  echo "→ Installing Cursor AI IDE…"

  # ensure /opt exists
  sudo mkdir -p /opt

  # download AppImage
  echo "  • Downloading AppImage"
  sudo curl -sSL "$CURSOR_URL" -o "$APPIMAGE_PATH"
  sudo chmod +x "$APPIMAGE_PATH"

  # download icon
  echo "  • Downloading icon"
  sudo curl -sSL "$ICON_URL" -o "$ICON_PATH"

  # create desktop entry
  echo "  • Creating desktop entry"
  sudo tee "$DESKTOP_ENTRY_PATH" >/dev/null <<EOF
[Desktop Entry]
Name=Cursor AI IDE
Exec=$APPIMAGE_PATH
Icon=$ICON_PATH
Type=Application
Categories=Development;
EOF

  echo "✔️  Cursor installed. Find it in your application menu."
}

uninstall_cursor() {
  echo "→ Uninstalling Cursor AI IDE…"

  # remove files
  sudo rm -f "$APPIMAGE_PATH" "$ICON_PATH" "$DESKTOP_ENTRY_PATH"
  sudo rm -f "${DESKTOP_ENTRY_PATH%.*}.cache" || true

  echo "✔️  Cursor removed."
}

case "${1:-}" in
  install)
    install_cursor
    ;;
  uninstall)
    uninstall_cursor
    ;;
  *)
    echo "Usage: sudo $0 {install|uninstall}"
    exit 1
    ;;
esac
```
2. **Make the script executable**

   ```bash
   sudo chmod +x ~/cursor-installer.sh
   ```

---

### Usage

* **Initial install:**

  ```bash
  sudo ~/cursor-installer.sh install
  ```

  * Installs the AppImage to `/opt/cursor.appimage`
  * Downloads the icon to `/opt/cursor.png`
  * Creates (or replaces) the desktop file so Cursor appears in your application menu

* **Uninstall:**

  ```bash
  sudo ~/cursor-installer.sh uninstall
  ```
---

Feel free to adjust paths, URLs, or schedule to suit your environment.
