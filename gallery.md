# Connecting to Wifi + SSH
### 1. Start -> Network Settings -> Wifi SSD and Wifi Key. Once it is connected, the IP address will appear at the top
### 2. Ensure your computer is connected to the same network. In the terminal
```bash
ssh report@insertipaddress
```

# Theme Editing

### 1. Create the Theme Folder
Follow the RetroPie EmulationStation theme guide:
https://retropie.org.uk/docs/Creating-Your-Own-EmulationStation-Theme/#references
You will be working in storage/.emulationstation/themes

Instead of the example folder names like `nes`, `snes`, and `gb`, use:
```bash
ports
```
From the home directory, place the theme files in:
```bash
/storage/themes
```
### 2. Create a Test Port File
For the `ports` section to appear, you need at least one executable file inside the `ports` folder.
Create the folder:
```bash
mkdir -p /storage/roms/ports
```
Create a test file:
```bash
nano "/storage/roms/ports/View Photos.sh"
```

Add this inside the file:
```bash
#!/bin/bash
echo "View Photos test"
sleep 5
```
Save and exit, then make the file executable:

```bash
chmod +x "/storage/roms/ports/View Photos.sh"
```

### 3. Select the Theme
Turn on the cyberdeck, then go to:
```text
Start → User Interface Settings → Theme Set
```
Select the theme you created.

Go to these menu items and adjust so that only the sections you want appear in the interface:
```text
Start → Game Collection Settings → Systems Displayed
Start → Game Collection Settings → Automatic Game Collections
```
