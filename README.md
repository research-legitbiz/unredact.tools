<div align="center">

# 🕵️ unredact.tools

### See what your files give away before someone else does. Then scrub it.

[![tests](https://github.com/NPCmillionaire/unredact.tools/actions/workflows/tests.yml/badge.svg)](https://github.com/NPCmillionaire/unredact.tools/actions/workflows/tests.yml)
![python](https://img.shields.io/badge/python-3.10%2B-3776AB?logo=python&logoColor=white)
![local-first](https://img.shields.io/badge/runs-100%25%20local-2ea44f)
![formats](https://img.shields.io/badge/formats-images%20%20PDF%20%20office%20%20media%20%20text-6f42c1)
![version](https://img.shields.io/badge/version-0.2.0-555)

[**Install**](#-install) · [**Usage**](#-usage) · [**What it detects**](#-what-it-detects) · [**How sanitizing works**](#-how-sanitizing-works) · [**Roadmap**](#-roadmap)

<sub>Local-first OPSEC for files: metadata, hidden content, redactions, pixels, prints and shadows.</sub>

<br>

<img width="820" alt="unredact.tools demo: scanning a photo and a court filing, catching a failed redaction, sanitizing, and re-scanning clean" src="docs/demo.gif" />

</div>

---

## Why

People publish sensitive files thinking they are safe. They aren't.

| Scenario | What goes wrong |
| :--- | :--- |
| **2019** | EU Police land-use PDF | **Black boxes** placed over text without deleting the text beneath; journalists copy-pasted it right out |
| **2021** | DOJ filing in high-profile court case | PDF **bookmarks and outline hidden text** retained classified witness names |
| **2023** | Leaked military slides | **EXIF metadata** embedded exact GPS coordinates, camera serial number, and timestamp |
| **2024** | Corporate earnings report | **Cropped image** in a slide still contained the full uncropped original inside the file structure |
| **2024** | Whistleblower photo | **AI vision check** extracted room reflection, weather, and camera angle to pinpoint location |
| **2019** | Paul Manafort's lawyers file a redacted PDF | Black boxes drawn **over** text that was still there; reporters copied it out within minutes |
| **Always** | Everyday Office and PDF files | Tracked changes, comments, author usernames, and every incrementally saved earlier version |

Good tools exist (ExifTool, mat2, Dangerzone), but each one does a single job, and none of them **explains the risk**.
`unredact.tools` combines detection, explanation, and sanitizing in one command.

---

## 📦 Install

**One line** (macOS and Linux: Homebrew, apt, dnf, pacman, zypper, apk):

```bash
curl -fsSL https://raw.githubusercontent.com/NPCmillionaire/unredact.tools/main/install.sh | bash
```

The installer finds Python 3.10+, installs `tesseract` and `ffmpeg` (it asks first), and puts everything in its own virtualenv under `~/.local/share/unredact.tools`. Nothing touches your system Python. It links `unredact.tools`, `unredact` and `unredact-file` into `~/.local/bin`, then checks which features are working.

```bash
curl -fsSL .../install.sh | bash -s -- --with-ai      # + Ollama and the vision model for --scene / sun --auto
curl -fsSL .../install.sh | bash -s -- --minimal      # metadata only, no OCR/vision
curl -fsSL .../install.sh | bash -s -- --uninstall
```

Re-running it updates to the latest version. The installer is tested on clean Ubuntu and macOS machines in CI on every push.

**From source:**

```bash
git clone https://github.com/NPCmillionaire/unredact.tools
cd unredact.tools
pip install -e ".[all]"         # or just `pip install -e .` for metadata-only
brew install tesseract ffmpeg   # apt install tesseract-ocr ffmpeg on Linux
```

This installs three equivalent commands: **`unredact.tools`**, the short **`unredact`**, and the legacy `unredact-file`.

## 🌐 Chrome Extension
[](https://github.com/research-legitbiz/unredact.tools#%EF%B8%8F-supported-formats)

Run local-first OPSEC checks directly inside your browser. The companion extension brings `unredact.tools` to your web workflow—inspecting downloads, shared attachments, and web uploads before they leave your device.

* **Instant File Scanning:** Drag-and-drop or right-click any file in your browser to scan for GPS coordinates, hidden metadata, and failed redactions.
* **One-Click Sanitization:** Automatically clean images, PDFs, and Office documents before submitting forms or uploading to cloud storage.
* **100% Local Processing:** Powered by WebAssembly (WASM), your files never touch an external server—everything executes locally within your browser tab.

### Installation & Usage
Install the extension directly from the [Chrome Web Store](https://chromewebstore.google.com/detail/ipjgdfniodlnhmnbbgcepnkckhddengj?utm_source=item-share-cb).

<details>
<summary><b>Optional extras</b></summary>

| Extra | Command | Adds |
|---|---|---|
| HEIC / HEIF | `pip install -e ".[heic]"` | iPhone photos |
| Audio / video | `brew install ffmpeg` · `apt install ffmpeg` | MP4, MOV, MKV, MP3, FLAC… |
| **Visual checks** | `pip install -e ".[vision]"` + `brew install tesseract` · `apt install tesseract-ocr` | OCR, QR/barcodes, plates, faces, pixelation, printer dots, camera ID |
| AI scene review *(optional)* | [Ollama](https://ollama.com) + `ollama pull qwen3-vl:8b` | `scan --scene`: reflections, landmarks, regional clues |
| City odds | `pip install -e ".[geo]"` | offline city list for `sun`'s where-people-live breakdown |
| Everything | `pip install -e ".[all]"` | all of the above |
| Dev / tests | `pip install -e ".[dev]"` | pytest + leaky-fixture generators |

</details>

---

## 🚀 Usage

### Scan

```bash
unredact.tools scan photo.jpg                 # human-readable report
unredact.tools scan *.pdf *.docx               # many files at once
unredact.tools scan memo.docx --extract ./loot # save recovered artifacts
unredact.tools scan filing.pdf --json          # machine-readable
unredact.tools scan photo.jpg --all            # include info-level findings
```

See what `unredact.tools` catches:

* **Hidden text & failed redactions:** Black boxes drawn *over* text without deleting the text layer beneath, text hidden off-canvas or outside print boundaries, font subsetting leaks.
* **EXIF & structural metadata:** GPS coordinates, camera serials, author usernames, edit history, software versions, creation/modification timestamps.
* **Embedded & cropped content:** Original high-res images inside low-res thumbnails, cropped regions stored full-size inside the container, embedded original attachments.
* **Incremental saves & revisions:** Past versions stored inside PDFs and Office docs (`.docx`, `.xlsx`, `.pptx`), tracked changes, deleted comments, author notes.
* **Visual & pixel-level artifacts:** Font anomalies, color-matched redacted text, blurred/pixelated sensitive regions, hidden layers, printer tracking dots (MIC), watermarks.

---

## 🔮 Sanitizing

```bash
unredact.tools clean photo.jpg                # strips EXIF, scrubs hidden layers
unredact.tools clean memo.pdf --output safe.pdf # writes cleaned copy
unredact.tools clean filing.pdf --rasterize   # flattens PDF to pure images (lossy, max safety)
```

Sanitizing removes or neutralizes all detected risks while preserving visual appearance where possible.

---

## 🛣️ Roadmap

- [x] CLI core, EXIF scrubbing, basic PDF/Office inspection
- [x] Chrome companion extension (WASM)
- [ ] Automated CI/CD action for repository file security checks
- [ ] Desktop GUI wrapper (Electron / Tauri)
- [ ] Expanded AI vision heuristics for ambient location leaks

---

## 📜 License

MIT License. See [LICENSE](LICENSE) for details.
