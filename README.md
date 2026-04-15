# Software Inventory Scripts

A collection of scripts for generating a CSV report of installed software across Windows, Linux, and macOS. Each script outputs a file named `software_report.csv` to the current user's Desktop.

## Output Format

All three scripts produce a CSV with the following columns:

| Column | Description |
|---|---|
| `Package` / `DisplayName` | The name of the installed application |
| `Version` / `DisplayVersion` | The installed version string |
| `Publisher` / `Maintainer` | The developer or signing authority |

---

## Scripts

### Windows (PowerShell)

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher |
Export-Csv "$env:USERPROFILE\Desktop\software_report.csv" -Encoding UTF8
```

**Requirements:** Windows PowerShell 3.0+ or PowerShell Core  
**Output:** `%USERPROFILE%\Desktop\software_report.csv`

Reads installed software entries from the Windows Registry uninstall key and exports them as a UTF-8 encoded CSV. Note that this only queries the `HKLM` (machine-wide) hive — software installed per-user under `HKCU` will not be included. To capture both, append a second `Get-ItemProperty` call for `HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*`.

---

### Linux (Bash / dpkg)

```bash
echo "Package,Version,Publisher" > ~/Desktop/software_report.csv
dpkg-query -W -f='${Package},${Version},${Maintainer}\n' >> ~/Desktop/software_report.csv
```

**Requirements:** Debian/Ubuntu-based distro with `dpkg`  
**Output:** `~/Desktop/software_report.csv`

Queries the `dpkg` package database for all installed packages and their maintainer metadata. This covers packages managed by `apt`/`dpkg` only — software installed via `snap`, `flatpak`, `AppImage`, or compiled from source will not appear.

---

### macOS (Bash)

```bash
find /Applications -maxdepth 1 -name "*.app" | while read app; do
  name=$(basename "$app" .app)
  version=$(defaults read "$app/Contents/Info" CFBundleShortVersionString 2>/dev/null)
  publisher=$(codesign -dv --verbose=4 "$app" 2>&1 | grep "Authority=" | head -n 1 | cut -d= -f2)
  echo "\"$name\",\"${version:-N/A}\",\"${publisher:-Unknown}\""
done >> ~/Desktop/software_report.csv
```

**Requirements:** macOS with Xcode Command Line Tools (`codesign`, `defaults`)  
**Output:** `~/Desktop/software_report.csv`

Scans the top-level `/Applications` directory for `.app` bundles, reads the version from each app's `Info.plist`, and extracts the publisher from its code-signing certificate. Apps that are unsigned will show `Unknown` for the publisher, and those missing a version string will show `N/A`. Apps installed in user-specific locations (e.g. `~/Applications`) are not scanned.

> **Note:** This script appends to the CSV rather than overwriting it. If you're running it standalone (without the Windows or Linux scripts), prepend a header line first:
> ```bash
> echo "Package,Version,Publisher" > ~/Desktop/software_report.csv
> ```

---

## Notes

- On **macOS**, `codesign` may be slow for large application directories since it inspects each app individually.
- On **Windows**, entries with a blank `DisplayName` (common for system components) will appear as empty rows. These can be filtered out by piping through `Where-Object { $_.DisplayName }`.
- None of the scripts require elevated/admin privileges, though results may be incomplete without them on systems with restricted registry or filesystem access.
