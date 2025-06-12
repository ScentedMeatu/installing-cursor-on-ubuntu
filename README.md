## Cursor Installer & Updater Script

This Bash script installs Cursor AI IDE as an AppImage, sets up its desktop entry, and can auto-update itself when new releases are available. The download URL has been updated to point at the stable release on `cursor.com`.

---

### Prerequisites

- **Linux distro** with `bash` (tested on Ubuntu/Debian)
- `sudo` privileges for writing to `/opt` and `/usr/share/applications`
- Internet access to download from `cursor.com`

---

### Installation - Copy paste every command below on your terminal

1. **Save the script**  
   Copy paste below commad:

   ```bash
   sudo tee /usr/local/bin/cursor-installer.sh >/dev/null << 'EOF'
   #!/bin/bash
   set -euo pipefail
   SCRIPT_NAME="$(basename "$0")"
   APPIMAGE_PATH="/opt/cursor.appimage"
   ICON_PATH="/opt/cursor.png"
   DESKTOP_ENTRY_PATH="/usr/share/applications/cursor.desktop"
   # Updated stable download endpoint
   CURSOR_URL="https://www.cursor.com/download/stable/linux-x64"
   ICON_URL="https://custom.typingmind.com/assets/models/cursor.png"

   usage() {
     cat <<EOM
   Usage: $SCRIPT_NAME [install|update]

     install  – remove old desktop entry, install AppImage & icon, create .desktop
     update   – download a new AppImage only if the remote is newer, refresh desktop entry
   EOM
     exit 1
   }

   remove_old_desktop() {
     if [ -f "$DESKTOP_ENTRY_PATH" ]; then
       echo "→ Removing existing desktop entry"
       sudo rm -f "$DESKTOP_ENTRY_PATH"
     fi
   }

   download_appimage_if_newer() {
     echo "→ Checking for Cursor update…"
     sudo curl -sSL -z "$APPIMAGE_PATH" "$CURSOR_URL" -o "$APPIMAGE_PATH.new" \
       && sudo chmod +x "$APPIMAGE_PATH.new" \
       && sudo mv "$APPIMAGE_PATH.new" "$APPIMAGE_PATH" \
       && echo "✔️  Updated Cursor AppImage"
   }

   download_icon() {
     echo "→ Downloading Cursor icon"
     sudo curl -sSL "$ICON_URL" -o "$ICON_PATH"
   }

   create_desktop_entry() {
     echo "→ Creating desktop entry"
     sudo bash -c "cat > '$DESKTOP_ENTRY_PATH'" <<EOI
   [Desktop Entry]
   Name=Cursor AI IDE
   Exec=$APPIMAGE_PATH
   Icon=$ICON_PATH
   Type=Application
   Categories=Development;
   EOI
     sudo update-desktop-database >/dev/null 2>&1 || true
   }

   install() {
     remove_old_desktop
     if [ ! -f "$APPIMAGE_PATH" ]; then
       echo "→ Installing Cursor AI IDE..."
       if ! command -v curl &>/dev/null; then
         echo "  • curl not found → installing"
         sudo apt-get update -qq
         sudo apt-get install -y curl
       fi
       echo "  • Downloading AppImage"
       sudo curl -sSL "$CURSOR_URL" -o "$APPIMAGE_PATH"
       sudo chmod +x "$APPIMAGE_PATH"
       download_icon
       create_desktop_entry
       echo "🎉  Installation complete!"
     else
       echo "⚠️  Cursor is already installed. Use '$SCRIPT_NAME update' to check for updates."
     fi
   }

   update() {
     remove_old_desktop
     if [ ! -f "$APPIMAGE_PATH" ]; then
       echo "❌  No existing installation found. Run '$SCRIPT_NAME install' first."
       exit 1
     fi
     download_appimage_if_newer
     download_icon
     create_desktop_entry
   }

   case "${1:-}" in
     install) install ;;
     update)  update  ;;
     *)       usage ;;
   esac
   EOF

2. **Make the script executable**

   ```bash
   sudo chmod +x /usr/local/bin/cursor-installer.sh
   ```

---

### Usage

* **Initial install:**

  ```bash
  sudo /usr/local/bin/cursor-installer.sh install
  ```

  * Installs the AppImage to `/opt/cursor.appimage`
  * Downloads the icon to `/opt/cursor.png`
  * Creates (or replaces) the desktop file so Cursor appears in your application menu
    
### Next time when cursor asks for update

* **Check for & apply updates:**

  ```bash
  sudo /usr/local/bin/cursor-installer.sh update
  ```

  * Uses `curl -z` to only re-download if a newer AppImage exists
  * Always refreshes icon and desktop entry

---

### Automating with Cron

To auto-check for Cursor updates daily at 4 AM, add this to root’s crontab:

```cron
0 4 * * * /usr/local/bin/cursor-installer.sh update >> /var/log/cursor-update.log 2>&1
```

1. Edit root’s cron:

   ```bash
   sudo crontab -e
   ```
2. Paste the above line and save.

The script will run each day, logging output to `/var/log/cursor-update.log`, and only download a new AppImage when one’s available.

---

Feel free to adjust paths, URLs, or schedule to suit your environment.
