# Standalone WinLandCraft example plugin

- Java 21, plain Gradle Java project. The only compile dependency is the API JAR in libs/.
- Use dev.winlandcraft.api.v1 only; do not import host internals or add Minecraft input hooks.
- WinLandCraft owns focus, typing capture, input forwarding, browser lifecycle, and window geometry.
- Keep README.md, API-GUIDE.md, and the examples consistent. API-GUIDE.md is a versioned reference copy; synchronize it explicitly when upgrading the API dependency.
- Run ./dev.sh clean check build (Windows: powershell -ExecutionPolicy Bypass -File .\dev.ps1 clean check build), then git diff --check and report the JAR SHA-256.
- Do not commit build output, caches, or libs/*.jar. Preserve user changes. Commit completed changes; do not push unless asked.
- Check configured remotes before editing; this project initially has no remote.
