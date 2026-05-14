<div align="center">

<img src="https://img.shields.io/badge/Dict-Odia%20Dictionary-1565C0?style=for-the-badge&logo=bookstack&logoColor=white" alt="Dict"/>

# 📖 Dict — Best Odia Dictionary

### A multilingual offline dictionary & reader for English · ଓଡ଼ିଆ · हिंदी

[![License: MIT](https://img.shields.io/badge/License-MIT-f9a825?style=flat-square)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20Browser-1565C0?style=flat-square&logo=android&logoColor=white)](.)
[![Languages](https://img.shields.io/badge/Languages-EN%20%7C%20OD%20%7C%20HI-880e4f?style=flat-square)](.)
[![Offline](https://img.shields.io/badge/Works-Offline-1b5e20?style=flat-square&logo=wifi&logoColor=white)](.)
[![GitHub](https://img.shields.io/badge/GitHub-abhayattcc-181717?style=flat-square&logo=github)](https://github.com/abhayattcc)

</div>

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔍 **Auto-detect language** | Type in English, ଓଡ଼ିଆ, or हिंदी — the app detects instantly |
| 📖 **Odia ↔ Odia Dictionary** | Full Odia monolingual dictionary with Odia meanings |
| 🔄 **Reverse lookup** | Search Odia or Hindi words to find English meanings |
| 🔊 **Text-to-Speech** | Read results aloud in EN / OD / HI with word highlighting |
| 📝 **Fullscreen Reader** | Paginated reader for TXT, HTML, DOCX & PDF files |
| 💾 **Saves your notes** | Persist custom text across sessions via localStorage |
| 📴 **Fully offline** | Databases cached after first load — no internet needed |
| 🤖 **Android app ready** | Native Android bridges for TTS, file picker & floating window |
| 🫧 **Floating dictionary** | Start/stop a floating overlay service on Android |

---

## 🌐 Supported Languages

<div align="center">

| Badge | Language | Direction |
|:---:|---|---|
| ![EN](https://img.shields.io/badge/EN-English-1976D2?style=flat-square) | English | EN → Odia + Hindi |
| ![OD](https://img.shields.io/badge/OD-ଓଡ଼ିଆ-880e4f?style=flat-square) | Odia (Orissan) | OD → English |
| ![HI](https://img.shields.io/badge/HI-हिंदी-1b5e20?style=flat-square) | Hindi | HI → English |
| ![OD](https://img.shields.io/badge/ODIA_DICT-ଓଡ଼ିଆ↔ଓଡ଼ିଆ-4a148c?style=flat-square) | Odia Monolingual | Odia word → Odia meaning |

</div>

---

## 🗄️ Databases

The app uses three compressed SQLite databases hosted in this repository:

```
data/
├── english_odia.db.gz      # English ↔ Odia word pairs
├── english_hindi.db.gz     # English ↔ Hindi word pairs + images
└── odia_meaning.db.gz      # Odia word → Odia meaning (monolingual)
```

Databases are downloaded once and cached via the **Cache API** for fully offline use. All queries run in-browser using **sql.js** (WebAssembly SQLite) — no server required.

---

## 🏗️ Architecture

```
index.html  (single self-contained file)
│
├── Language Detection       — Unicode range checks (Odia U+0B00–U+0B7F, Hindi U+0900–U+097F)
├── Odia Normaliser          — Handles NFC, anusvara/chandrabindu, ba/va, ya/yya variants
├── Database Layer           — sql.js (WASM SQLite), pako (gzip), Cache API
├── Search Engine            — English prefix, Odia reverse, Hindi reverse, Odia monolingual
├── TTS Engine               — Queued, multilingual, with word-level highlighting
├── Fullscreen Reader        — Paginated (configurable words/page), tap-to-read, themes
├── File Extractor           — TXT · HTML · DOCX (custom unzip) · PDF (pdf.js)
└── Android Bridges          — window.AndroidTTS · window.AndroidApp · window.AndroidFile
```

### Android Bridges

When embedded in an Android WebView, the app communicates with native code via JavaScript interfaces:

| Bridge | Methods | Purpose |
|---|---|---|
| `window.AndroidTTS` | `speak(text, lang, id)` · `stop()` | Native TTS playback |
| `window.AndroidApp` | `startFloatingService()` · `stopFloatingService()` · `openUrl(url)` | App control |
| `window.AndroidFile` | `openFilePicker()` | Native file chooser |

Callbacks from Android to JS:
- `window._attsOnEnd(id)` — called when TTS utterance finishes
- `window._androidFileResult(name, base64)` — delivers selected file as base64
- `window._androidFileError(msg)` — reports file picker errors
- `window.showPopup(word)` — triggers a dictionary search (used by floating service)

---

## 🚀 Usage

### Browser (Standalone)

Just open `index.html` in any modern browser. No build step, no dependencies to install.

```bash
git clone https://github.com/abhayattcc/Best-odia-dictionary.git
cd Best-odia-dictionary
# Open index.html in your browser
```

> **Note:** Databases are fetched from GitHub on first load and cached. An internet connection is required only for the initial setup.

### Android WebView

Embed `index.html` as a WebView asset and register the three Java bridges:

```java
webView.addJavascriptInterface(new AndroidTTSBridge(this), "AndroidTTS");
webView.addJavascriptInterface(new AndroidAppBridge(this), "AndroidApp");
webView.addJavascriptInterface(new AndroidFileBridge(this), "AndroidFile");
```

Enable required WebView settings:

```java
WebSettings settings = webView.getSettings();
settings.setJavaScriptEnabled(true);
settings.setDomStorageEnabled(true);     // localStorage
settings.setCacheMode(WebSettings.LOAD_DEFAULT);
settings.setAllowFileAccess(true);
```

---

## 🔍 Search Modes

### 1. English Search
Type any English word → returns Odia and Hindi translations with optional image.

### 2. Odia Reverse Search (`ଓଡ଼ିଆ → EN`)
Type Odia script → finds matching English headwords. Handles Unicode variant normalisation automatically (ବ/ଵ, ଯ/ୟ, ଁ/ଂ, ଡ଼/ଢ଼, etc.).

### 3. Hindi Reverse Search (`हिंदी → EN`)
Type Devanagari → finds matching English headwords.

### 4. Odia Dictionary Mode (`ଓ` button)
Toggle the **ଓ** button for the Odia monolingual dictionary. Type an Odia word or its romanised spelling to get the full Odia meaning.

> All modes use a **1-second debounce** with animated typing indicator — no need to press Enter.

---

## ⚙️ Configuration

Key constants in `index.html`:

```javascript
const SEARCH_DELAY = 1000;   // Debounce delay in ms

// Database URLs (change to self-host)
const DB_URLS = {
  eo: '...english_odia.db.gz',
  eh: '...english_hindi.db.gz',
  om: '...odia_meaning.db.gz'
};

// localStorage keys
const SAVE_KEY   = 'dict-note-v1';   // TTS / reader saved text
const PAGE_PFX   = 'rpage:';         // Reader last-page per filename
const CACHE      = 'float-dict-v1';  // Cache Storage bucket name
```

---

## 📁 File Support (Reader & TTS)

| Extension | Method |
|---|---|
| `.txt` `.md` `.rtf` | FileReader (UTF-8) |
| `.html` `.htm` | DOMParser → innerText |
| `.docx` `.doc` | Custom ZIP parser + pako inflate |
| `.pdf` | pdf.js (loaded on demand) |

---

## 🔠 Odia Script Normalisation

The app includes a robust Odia normaliser that handles real-world encoding inconsistencies:

- **NFC normalisation** and invisible character stripping
- **Anusvara/Chandrabindu** interchange (`ଁ` ↔ `ଂ`)
- **Ba/Va** interchange (`ବ` ↔ `ଵ`)  
- **Ya/Yya** interchange (`ଯ` ↔ `ୟ`)
- **Wa → Va** mapping (`ୱ` → `ଵ`)
- **Nukta** combinations (`ଡ+଼` ↔ `ଡ଼`, `ଢ+଼` ↔ `ଢ଼`)
- **Virama spacing** cleanup
- **Romanised spelling** normalisation (ā→a, ī→i, ū→u, ḍ→d, ṭ→t, w→v, etc.)

---

## 🛠️ Dependencies

All loaded from CDN — no npm/build step required.

| Library | Version | Purpose |
|---|---|---|
| [pako](https://github.com/nodeca/pako) | 2.0.4 | Gzip decompression for databases |
| [sql.js](https://github.com/sql-js/sql.js) | 1.8.0 | In-browser SQLite (WASM) |
| [pdf.js](https://github.com/mozilla/pdf.js) | 3.11.174 | PDF text extraction (on demand) |

---

## 🤝 Contributing

Contributions are welcome! Ideas for improvement:

- 📚 Expand the word databases
- 🌍 Add more languages (Sambalpuri, Bengali, Telugu…)
- 🔤 Improve romanisation → Odia script matching
- 🖼️ Add more word images
- 🤖 Android app improvements (Sketchware Pro / Android Studio)

Please open an issue or pull request on [GitHub](https://github.com/abhayattcc/Best-odia-dictionary).

---

## 💲 Support

If Dict has been useful to you, please consider donating to help keep the project alive:

[![Donate](https://img.shields.io/badge/Donate-Razorpay-072654?style=for-the-badge&logo=razorpay&logoColor=white)](https://razorpay.me/@abhayabehera4628)

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 🔒 Privacy

Dict does not collect any personal data. All dictionary lookups run entirely on-device. See [PRIVACY.md](PRIVACY.md) for the full privacy policy.

---

<div align="center">

Made with ❤️ for the Odia language community

**[abhayattcc](https://github.com/abhayattcc)** · github.com/abhayattcc/Best-odia-dictionary

</div>
