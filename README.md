# Chess Assistant

On-device chess study assistant for iOS. It analyses positions locally with Stockfish 18 or Maia 3 and displays candidate moves, evaluations, opening recognition, and move-quality feedback.

Everything runs locally on the device. No network requests, and no move data leaves your phone.

## Features

> Current development version: **2.6.0**.

Two engines to pick from. Stockfish 18 is statically linked with its NNUE nets embedded and searches strong candidate moves at the selected strength. Maia 3 runs as a Core ML model and selects moves in a human-like style, with configurable skill levels.

- ELO slider from 400 to 3500, mapped to search depth plus skill level and UCI_LimitStrength
- Up to 3 candidate arrows, each labeled with its own evaluation
- Fast and Deep search profiles: Fast caps Stockfish search at depth 12 and limits MultiPV to the 1–3 lines the UI can display; Deep keeps the full ELO-mapped search depth
- Opening repertoire selector with Alien, King's, Evans, Danish, Smith-Morra, Budapest, Englund, Blackmar-Diemer, Halloween, Stafford, and Latvian Gambits
- Selected gambit lines are followed only while the recorded game history matches the exact UCI move prefix; the engine resumes normal analysis as soon as the game leaves the line
- Centipawn or win percentage on the floating button
- Move Analysis grades every move you play from Brilliant to Blunder, tracks accuracy, and shows the full breakdown in the settings panel
- Optional automated move execution for practice and test environments where it is explicitly allowed. Keep it off in rated or competitive games where assistance is prohibited.
- Long-press the floating ♟ button to pause or resume the assistant right away
- Live detection of about 40 openings, including the Ruy Lopez, Najdorf, Caro-Kann and Queen's Gambit, shown in the panel
- One tap copies the current FEN to your clipboard
- Accuracy stats, move grades and opening tracking reset when a game ends, detected on-device from checkmate or stalemate
- Arrow styling: color by eval or solid green, thin to thick shafts, adjustable opacity
- A ⚡ button that applies study defaults in one tap (ELO 1000, Maia on, Auto Play off)

The Alien Gambit entry is the Caro-Kann line 1.e4 c6 2.d4 d5 3.Nd2 dxe4 4.Nxe4 Nf6 5.Ng5 h6 6.Nxf7. It is an aggressive practical surprise, not a sound opening recommendation; the repertoire picker labels that risk rather than presenting it as a forced win.

The UI and engine settings are designed for local study. The feature set does not send positions to a remote analysis service.

## Install

### Jailbroken (rootless)

Add the repo in Sileo or Zebra and install:

```
https://itzzace.github.io/chess-assistant/
```

Respring after installing, then open Chess.com.

### TrollStore (no jailbreak)

Download a compatible IPA from the [releases page](https://github.com/itzmnyzzzXD/chess-assistant/releases) and sideload it with TrollStore. The IPA injection workflow requires a decrypted Chess.com IPA supplied by the maintainer.

## Usage

1. Open Chess.com and start any game.
2. Tap the floating ♟ button to open the settings panel. Long-press it to pause or resume instantly.
3. Set your strength, choose Stockfish or Maia, and configure the display and move review. Use automated move execution only in environments that explicitly permit it. The ⚡ button applies the study preset.
4. Candidate arrows and evaluations appear on the board when a position is ready for analysis.

Debug Log in the panel shows recent events such as FEN detection, engine responses and which hooks installed. FEN copies the current position.

> Fair-play notice: real-time engine assistance or automated move execution may violate the rules of online chess services and can result in account restrictions. Use the assistant for analysis, puzzles, offline play, and environments where assistance is explicitly permitted. It does not attempt to hide engine use.

## Building

Requires [Theos](https://theos.dev) with the iOS SDK:

```bash
# clone Stockfish and fetch its NNUE nets
git clone --depth 1 --branch sf_18 https://github.com/official-stockfish/Stockfish.git sf
cd sf/src && make net && cd ../..

# cross-compile Stockfish for iOS arm64 into a static lib
cd sf/src
SDK=$(xcrun -sdk iphoneos --show-sdk-path)
CXX=$(xcrun -sdk iphoneos -f clang++)
for f in $(find . -name '*.cpp'); do
  $CXX -arch arm64 -isysroot "$SDK" -miphoneos-version-min=15.0 \
       -std=c++17 -O3 -DNDEBUG -fno-exceptions \
       -DUSE_PTHREADS -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8 -I. -w \
       -c "$f" -o "$f.o"
done
ar rcs libstockfish.a $(find . -name '*.cpp.o' ! -name 'main.cpp.o')
cd ../..

# build the deb (Maia model is fetched separately)
make package FINALPACKAGE=1
```

CI does all of this automatically. See [.github/workflows/build.yml](.github/workflows/build.yml). Pull requests build the package without publishing a release. The Maia 3 CoreML model (`maia3_5m.mlpackage`) is downloaded from the repo's `maia-model` release during packaging and installed to `/var/jb/Library/Application Support/Chess/`.

## Project layout

| File | Purpose |
|------|---------|
| `Tweak.xm` | Main tweak: board/FEN detection, arrow rendering, settings UI, auto play |
| `engine.mm` | Stockfish embedded via UCI stream redirection (`EngineGo`, FEN legality, legal-move enumeration) |
| `maia.mm` | Maia 3 CoreML inference wrapper |
| `web/` | Responsive landing page / repo site |
| `Makefile` | Theos tweak build (`com.chess.assistant`, rootless) |
| `.github/workflows/` | CI: builds libstockfish.a + .deb; pull requests verify builds without publishing |

## Credits

- Made by @epicccccc ([YouTube](https://youtube.com/@epicccccc), Discord: `itzzace.`)
- Yousseif ([usif-x](https://github.com/usif-x)) upgraded the engine to Stockfish 18 and added Auto Play with human-like timing and puzzle solving, quick pause, opening detection, Copy FEN, auto stats reset and other tweak updates
- [Stockfish](https://stockfishchess.org), the GPL v3 chess engine
- [Maia 3](https://github.com/CSSLab/maia3), human-like chess AI

Licensed under GPL v3 where applicable, because of the Stockfish linkage.
