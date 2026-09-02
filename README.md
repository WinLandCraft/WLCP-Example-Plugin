# WLCP Example Plugin

An independent Java 21 / Fabric plugin mod for **WinLandCraft API v1**. This project does not need the WinLandCraft source repository, Minecraft mappings, Loom, Rust, or a separate MCEF dependency. It compiles against the small public API JAR only.

## Requirements and quick build

1. Install **JDK 21**. Windows `dev.ps1` discovers Temurin in Program Files; otherwise set `JAVA_HOME`. Linux/macOS: set `JAVA_HOME` to your JDK 21.
2. Put exactly one `winlandcraft-*-plugin-api.jar` in `libs/`. The initial local setup already includes the 0.1.82-dev API JAR. It is ignored by Git, so a fresh clone needs this step. Do not use the full mod JAR as the compile dependency.
3. Build from this folder:

```powershell
powershell -ExecutionPolicy Bypass -File .\dev.ps1 clean check build
```

Linux/macOS:

```sh
JAVA_HOME=/path/to/jdk-21 sh ./gradlew clean check build
```

The bundled Gradle 8.12 wrapper downloads Gradle on first use; no global Gradle installation is needed. `check` verifies the entrypoint/metadata and rejects accidental bundling of the API or host classes. Output: **`build/libs/wlcp-example-plugin-1.0.0.jar`**. Build version comes from `build.gradle` and is expanded into Fabric metadata automatically.

## Install and test

Install Minecraft 1.21.4, Fabric Loader 0.16.9+, Fabric API, and WinLandCraft 0.1.82-dev or newer. Add this project's built plugin JAR to the same client `mods` folder. Do **not** install the API JAR, a sources JAR, or another MCEF JAR. Remove the old `winlandcraft-*-example-plugin.jar` first: both examples use mod ID `wlc_example` and cannot coexist. This is a client-only plugin; the server does not need it.

Open a world and find three new entries in WinLandCraft's Apps panel:

| App | What it demonstrates |
| --- | --- |
| Example Notes | Native Canvas drawing, host-managed typing, and an asynchronous `.wlcnotes` file handler. Click to type, use Backspace, and press Escape to return to Minecraft. This minimal example does not save edits or implement a full editor. |
| Example Web | A full-window managed Chromium view loading example.com. |
| Example Hybrid | A native Reload toolbar above a managed browser rectangle; browser input is forwarded automatically. |

Use File Manager to open `sample.wlcnotes`, choose an edge, and verify the separate Notes window. Also test dragging that file onto Notes, resizing versus Ctrl-scaling, grouped movement/curves, close/reopen, and leaving the world. Browser tests need a working WinLandCraft Chromium runtime and internet access. A successful Java build cannot verify in-game rendering or native CEF behavior.

## How the code fits together

- `src/main/resources/fabric.mod.json` declares the client-only `winlandcraft:plugins` entrypoint and host version requirement.
- `src/main/java/example/ExamplePlugin.java` registers three immutable AppDefinitions. Each factory creates independent per-window app state.
- `Notes` draws in logical pixel coordinates. It requests typing through WindowContext instead of intercepting Minecraft input. File reads run in a worker, are capped at 8 KiB, and return through session-bound `execute`, so a late completion cannot update a closed window.
- `Web` asks the host's BrowserView to navigate. The host creates and closes Chromium resources.
- `Hybrid.onResize` reserves a 52-pixel native toolbar; the remaining rectangle belongs to Chromium. Its native pointer callback handles Reload without forwarding web input itself.
- The API dependency is compile-only. At runtime WinLandCraft supplies those classes and all windowing/input behavior.

## Adapt this into your own plugin

Change the Fabric mod ID, plugin name, entrypoint package/class, and AppDefinition IDs together. Each app ID's namespace must equal your Fabric mod ID. Choose NATIVE, CHROMIUM, or HYBRID; omit file associations for apps that do not open files. Add extensions/exact file names to opt into File Manager. Add an icon under `src/main/resources/assets/<modid>/textures/` and reference it with `.icon("<modid>:textures/icon.png")`.

Keep heavy I/O out of callbacks, bound your data/queues, and handle file errors in the app. The host owns locking, focus, Escape, picking, dragging, grouping, scaling, and browser forwarding; do not add input mixins. Use the public v1 package rather than host implementation classes.

[API-GUIDE.md](API-GUIDE.md) contains the API reference, lifecycle, input contract, and compatibility rules, copied from WinLandCraft 0.1.82-dev's PLUGINS.md. For this standalone project's build and installation, use this README. Future API upgrades are explicit: replace the JAR in libs/, review/synchronize the guide, adjust the minimum host version only if using new capabilities, and rebuild/test. Existing v1 signatures are intended to remain compatible.

## Repository independence

The source, wrapper, sample file, build configuration, and documentation are all here. There is no parent Gradle build, sibling-path dependency, source inclusion, or automatic host checkout/build. The API JAR is the only external build input. No remote is configured initially; publish this repository wherever you prefer.
