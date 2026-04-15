# Software Inventory Scripts

A collection of scripts for generating a CSV report of installed software across Windows, Linux, and macOS. Each script outputs a file named `software_report.csv` to the current user's Desktop.

---

## 👋 Not sure what a "script" is? Start here

These scripts are small text-based programs that, when run, will automatically scan your computer and create a spreadsheet listing all installed software. You don't need to be technical to use them — just follow the step-by-step instructions for your operating system below.

The spreadsheet (`software_report.csv`) will appear on your **Desktop** when the script finishes. You can open it with Excel, Google Sheets, or any spreadsheet app.

---

## How to run the script

### 🪟 Windows — Step by step

1. Click the **Start** menu and type `PowerShell`.
2. Right-click **Windows PowerShell** and choose **Run as administrator**. Click **Yes** if a confirmation dialog appears.
3. Click inside the PowerShell window (the dark window with a blinking cursor).
4. Copy the script below, paste it in with **Ctrl + V**, and press **Enter**.
5. Wait a few seconds. When the `>` prompt returns, the script is done.
6. Go to your **Desktop** — you'll find a file called `software_report.csv`.

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher |
Export-Csv "$env:USERPROFILE\Desktop\software_report.csv" -Encoding UTF8
```

> **Tip:** If you see a message about script execution being disabled, run this first and then try again:
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

---

### 🐧 Linux (Ubuntu/Debian) — Step by step

1. Open the **Terminal** app. You can usually find it by searching "Terminal" in your applications menu, or pressing **Ctrl + Alt + T**.
2. Click inside the Terminal window.
3. Copy the two lines below, paste them in with **Ctrl + Shift + V**, and press **Enter**.
4. When the `$` prompt returns, the script is done.
5. Go to your **Desktop** — you'll find a file called `software_report.csv`.

```bash
echo "Package,Version,Publisher" > ~/Desktop/software_report.csv
dpkg-query -W -f='${Package},${Version},${Maintainer}\n' >> ~/Desktop/software_report.csv
```

---

### 🍎 macOS — Step by step

1. Open **Terminal**. To find it, press **Cmd + Space**, type `Terminal`, and press **Enter**.
2. Click inside the Terminal window.
3. Copy the full script below, paste it in with **Cmd + V**, and press **Enter**.
4. The script may take a minute or two — this is normal. When the `$` prompt returns, it's done.
5. Go to your **Desktop** — you'll find a file called `software_report.csv`.

```bash
echo "Package,Version,Publisher" > ~/Desktop/software_report.csv
find /Applications -maxdepth 1 -name "*.app" | while read app; do
  name=$(basename "$app" .app)
  version=$(defaults read "$app/Contents/Info" CFBundleShortVersionString 2>/dev/null)
  publisher=$(codesign -dv --verbose=4 "$app" 2>&1 | grep "Authority=" | head -n 1 | cut -d= -f2)
  echo "\"$name\",\"${version:-N/A}\",\"${publisher:-Unknown}\""
done >> ~/Desktop/software_report.csv
```

> **Note for macOS:** If prompted to install developer tools, click **Install** and re-run the script once it completes.

---

## 📂 Opening the report

Once the script finishes, a file called `software_report.csv` will be on your Desktop. To open it:

- **Windows:** Double-click it — it should open in Excel. If it doesn't, right-click → *Open with* → *Excel* or *Notepad*.
- **macOS:** Double-click it — it should open in Numbers or Excel.
- **Linux:** Double-click it or open it with LibreOffice Calc.

You can also upload it to **Google Sheets**: go to [sheets.google.com](https://sheets.google.com), click the folder icon, and upload the file.

---

---

## Output Format

All three scripts produce a CSV with the following columns:

| Column | Description |
|---|---|
| `Package` / `DisplayName` | The name of the installed application |
| `Version` / `DisplayVersion` | The installed version string |
| `Publisher` / `Maintainer` | The developer or signing authority |

---

## Scripts & technical notes

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

Scans the top-level `/Applications` directory for `.app` bundles, reads the version from each app's `Info.plist`, and extracts the publisher from its code-signing certificate. Apps that are unsigned will show `Unknown` for the publisher, and those missing a version string will show `N/A`. 

> **Note:** This script appends to the CSV rather than overwriting it. If you're running it standalone (without the Windows or Linux scripts), prepend a header line first:
> ```bash
> echo "Package,Version,Publisher" > ~/Desktop/software_report.csv
> ```

---
