# Converting a React + Vite + TypeScript App into an Electron App

This guide walks through converting an existing **React + Vite + TypeScript** application into an Electron desktop application using modern **ES Modules (ESM)**.

> **Tested with**
>
> - React 19
> - Vite 6
> - TypeScript 5
> - Electron 43

---

# Project Structure

A recommended project layout is:

```text
my-app/
├── electron/
│   ├── main.ts
│   └── preload.ts
├── src/
│   ├── App.tsx
│   ├── main.tsx
│   └── ...
├── dist/
├── dist-electron/
├── package.json
├── tsconfig.json
├── tsconfig.electron.json
└── vite.config.ts
```

---

# Step 1 - Install Dependencies

```bash
npm install -D electron electron-builder concurrently wait-on typescript
```

---

# Step 2 - Configure package.json

Since Vite already uses ES Modules, keep:

```json
{
  "type": "module"
}
```

Add the Electron entry point:

```json
{
  "main": "dist-electron/main.js"
}
```

Add these scripts:

```json
{
  "scripts": {
    "dev": "vite --port=3000",

    "dev:electron": "concurrently \"npm run dev\" \"npm run electron\"",

    "electron": "wait-on tcp:3000 && tsc -p tsconfig.electron.json && electron .",

    "build": "vite build && tsc -p tsconfig.electron.json"
  }
}
```

> **Important**
>
> If Vite runs on port **3000**, Electron must also connect to **3000**.
>
> Don't leave it pointing to **5173**.

---

# Step 3 - Create the Electron Main Process

Create:

```text
electron/main.ts
```

```ts
import { app, BrowserWindow } from "electron";
import path from "node:path";
import { fileURLToPath } from "node:url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

function createWindow() {
  const win = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, "preload.js"),
      contextIsolation: true,
      nodeIntegration: false
    }
  });

  if (app.isPackaged) {
    win.loadFile(path.join(__dirname, "../dist/index.html"));
  } else {
    win.loadURL("http://localhost:3000");
    win.webContents.openDevTools();
  }
}

app.whenReady().then(createWindow);

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});
```

---

# Step 4 - Create the Preload Script

Create:

```text
electron/preload.ts
```

```ts
import { contextBridge } from "electron";

contextBridge.exposeInMainWorld("electron", {
  version: process.versions.electron
});
```

---

# Step 5 - Configure TypeScript

Create:

```text
tsconfig.electron.json
```

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist-electron",

    "strict": true,
    "esModuleInterop": true,

    "types": ["node"]
  },

  "include": ["electron/**/*.ts"]
}
```

This is **very important**.

Since the project uses:

```json
"type": "module"
```

Electron should also compile using:

```json
"module": "NodeNext"
```

Using `"CommonJS"` here will eventually cause:

```
ReferenceError: exports is not defined in ES module scope
```

---

# Step 6 - Start the Application

Run:

```bash
npm run dev:electron
```

The process is:

1. Start Vite
2. Wait for port 3000
3. Compile Electron
4. Launch Electron

---

# Step 7 - Production Build

Run:

```bash
npm run build
```

This builds:

```
dist/
```

and

```
dist-electron/
```

Electron automatically loads:

```ts
win.loadFile("../dist/index.html")
```

instead of the Vite development server.

---

# Common Errors

## Error

```
Cannot find module '<project-folder>'
```

### Cause

Electron doesn't know which file to launch.

### Fix

Ensure package.json contains:

```json
{
  "main": "dist-electron/main.js"
}
```

Also verify:

```
dist-electron/main.js
```

actually exists.

---

## Error

```
Cannot find module 'dist-electron/main.js'
```

### Cause

TypeScript never compiled the Electron files.

### Fix

Run:

```bash
npx tsc -p tsconfig.electron.json
```

Then verify:

```
dist-electron/
    main.js
    preload.js
```

exists.

---

## Error

```
ReferenceError: exports is not defined in ES module scope
```

### Cause

The Electron code was compiled as CommonJS while the project is configured as ES Modules.

Example:

```json
"type": "module"
```

combined with:

```json
"module": "CommonJS"
```

### Fix

Compile Electron using:

```json
"module": "NodeNext"
```

instead.

---

## Error

```
__dirname is not defined
```

### Cause

`__dirname` does not exist in ES Modules.

### Fix

Replace it with:

```ts
import path from "node:path";
import { fileURLToPath } from "node:url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

---

## Electron Opens a Blank Window

Usually one of these:

- Wrong Vite port
- Wrong `loadURL()`
- Wrong `loadFile()` path
- Renderer crashed

Verify:

```ts
win.loadURL("http://localhost:3000");
```

matches the Vite server.

---

## Electron Never Starts

Usually:

```bash
wait-on tcp:5173
```

while Vite is actually running on:

```bash
3000
```

Update the script:

```json
"electron": "wait-on tcp:3000 && tsc -p tsconfig.electron.json && electron ."
```

---

# Packaging

Install:

```bash
npm install -D electron-builder
```

Example configuration:

```json
{
  "build": {
    "appId": "com.example.app",
    "files": [
      "dist/**/*",
      "dist-electron/**/*"
    ]
  }
}
```

Then package:

```bash
npx electron-builder
```

---

# Summary

A working Electron + React + Vite setup should have:

- ✅ `"type": "module"` in `package.json`
- ✅ `"main": "dist-electron/main.js"`
- ✅ `module: "NodeNext"` in `tsconfig.electron.json`
- ✅ `loadURL()` pointing to the correct Vite port
- ✅ `wait-on` using the same Vite port
- ✅ `fileURLToPath(import.meta.url)` instead of `__dirname`
- ✅ `preload.ts` compiled into `dist-electron`

Following these steps avoids the most common startup errors, including:

- `Cannot find module`
- `exports is not defined`
- `__dirname is not defined`
- Blank Electron windows
- Incorrect Vite port configuration
