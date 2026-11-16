# Lazy Arranger Midi DAW releases

This repository holds the public binaries and release notes for **Lazy Arranger
Midi DAW**. The application's source code is private.

## Download

Open [Releases](https://github.com/nhth-mibu/lazy-arranger-midi-daw-releases/releases)
and choose the latest release.

Each release has:

- **macOS:** `ArrangerMidiDaw-macos-arm64.dmg` or the current DMG name.
- **Windows:** `ArrangerMidiDaw-windows-x64.zip`, for Windows x64.
- **Linux:** `ArrangerMidiDaw-linux-x64-ubuntu-26.04.tar.gz`, for Ubuntu 26.04
  x64 and newer compatible distributions.
- A `.sha256` file for each package.
- The demo style pack when a release includes it.

Download the package and its `.sha256` file. Check the package before you open
it:

```sh
shasum -a 256 -c ArrangerMidiDaw-macos-arm64.dmg.sha256
```

On Windows, use `certutil -hashfile <file> SHA256` and compare the result with
the `.sha256` file.

## Notes

- The macOS app is ad-hoc signed, not Developer ID signed or notarized. macOS
  may block the first launch. Review System Settings > Privacy & Security >
  Open Anyway only for a build you trust.
- Windows needs the Microsoft Visual C++ v14 x64 Redistributable.
- Linux packages target Ubuntu 26.04 x64. Older distributions may need newer
  system libraries.
- The packages do not include a General MIDI SoundFont. The app can download
  one, or you can choose your own SF2 file.
- Third-party notices are included in each package.
- Releases before v0.0.17 use the former name, Arranger MIDI DAW, in the app
  window and installer.

The "Source code" archives that GitHub adds to a release contain only this
README. They are not the application source.

## Link

- [Latest release](https://github.com/nhth-mibu/lazy-arranger-midi-daw-releases/releases/latest)
