# MULM

MULM is a [Chrome Extension](https://chromewebstore.google.com/detail/mulm/kchcndllnnpddfldhdbphkhkdbcdeakl) that streamlines the workflow between Minerva University's Forum platform and Google's NotebookLM.

Developed between December 2025 and January 2026, it was built to remove the repetitive friction of gathering session readings, copying source links, and moving workbook questions and content into AI tooling by hand. The extension packages that flow into a Manifest V3 product with a React-based popup, cross-origin content scripts, and a NotebookLM-side sync action.

MULM shipped as Version `1.0.0` on January 6, 2026 and is currently used by 50+ students, roughly half of one Minerva cohort.

## Overview

MULM turns a multi-step student workflow into a single extension-assisted flow:

- Extracts the current Minerva class session title and reading links from Forum
- Captures workbook questions and text from the embedded Minerva workbook iframe
- Stores extracted data in `chrome.storage.local` so the popup and NotebookLM integration can share state
- Injects a native-feeling `Sync Minerva` chip into NotebookLM's source UI
- Pastes all reading URLs into NotebookLM's Website import flow automatically
- Tracks failed NotebookLM source imports and surfaces them back in the popup UI

## Why This Exists

Students working between Minerva Forum and NotebookLM often have to:

- open a class page
- gather all assigned readings
- copy links one by one
- move into NotebookLM
- paste and import sources
- separately pull workbook questions and content for context

MULM reduces that process to a few clicks and keeps the relevant material accessible inside the extension popup.

## How It Works

1. On a Minerva class page, the popup sends a `GET_READINGS` message to the Forum content script.
2. The Forum script extracts the session title and the reading links from the page DOM.
3. A second content script runs in the Minerva workbook iframe and stores workbook text in `chrome.storage.local`.
4. The popup reads shared storage state and presents:
   - the session title
   - the discovered sources
   - copy actions for NotebookLM sources and workbook content
5. On `notebooklm.google.com`, a dedicated content script injects a `Sync Minerva` chip next to NotebookLM's existing source options.
6. When clicked, that chip opens NotebookLM's Website import flow, pastes the stored URLs, and triggers source insertion.
7. If NotebookLM reports failed sources, MULM records them and highlights those items in the popup.

## Tech Stack

- React 19
- TypeScript 5
- Tailwind CSS v4
- Vite 7
- `@crxjs/vite-plugin` for Chrome Extension development on Manifest V3
- Chrome Extension APIs:
  - `chrome.storage`
  - `chrome.tabs`
  - runtime messaging between popup and content scripts

## Architecture

### Popup UI

The popup is a React app that:

- validates the user is on a Minerva class page before scraping
- listens for storage updates in real time
- renders discovered sources and workbook content actions
- displays source import failures from NotebookLM

Primary files:

- `src/App.tsx`
- `src/components/Button.tsx`
- `src/components/ErrorMessage.tsx`
- `src/components/LinkList.tsx`

### Minerva Content Scripts

`src/content-scripts.ts` handles two separate page contexts:

- `forum.minerva.edu`
  - extracts the session title
  - finds the `Readings` section
  - returns a normalized list of links plus a newline-delimited URL string
- `sle-collaboration.minervaproject.com`
  - scrapes workbook questions and content from rendered editors
  - updates storage continuously via a `MutationObserver`

### NotebookLM Content Script

`src/notebooklm.ts` runs on `notebooklm.google.com` and:

- injects a custom `Sync Minerva` chip by cloning NotebookLM's native chip UI
- opens the Website import flow
- pastes stored URLs into NotebookLM's input
- clicks `Insert` automatically
- observes NotebookLM source errors and syncs them back to the popup

## Manifest Scope

The extension currently targets three origins:

- `https://forum.minerva.edu/*`
- `https://sle-collaboration.minervaproject.com/*`
- `https://notebooklm.google.com/*`

Permissions in `manifest.json`:

- `storage`
- `activeTab`

## Local Development

### Install

```bash
npm install
```

### Start Development

```bash
npm run dev
```

### Build Production Bundle

```bash
npm run build
```

### Lint

```bash
npm run lint
```

## Load the Extension in Chrome

1. Build the extension with `npm run build`.
2. Open `chrome://extensions`.
3. Enable Developer Mode.
4. Click `Load unpacked`.
5. Select the generated `dist/` directory.

## Project Structure

```text
.
├── manifest.json
├── popup.html
├── vite.config.ts
├── src
│   ├── App.tsx
│   ├── content-scripts.ts
│   ├── notebooklm.ts
│   ├── main.tsx
│   ├── index.css
│   └── components
│       ├── Button.tsx
│       ├── ErrorMessage.tsx
│       └── LinkList.tsx
└── public/images
```

## Notes

- The extension relies on DOM selectors in both Minerva and NotebookLM, so UI changes on either platform may require selector updates.
- Shared state currently lives in `chrome.storage.local`, which keeps the popup and page integrations loosely coupled.
- The current repository reflects a shipped Manifest V3 extension, but it does not yet include automated tests.

## Product Context

MULM represents a full product cycle from problem discovery to public release:

- identifying a concrete student workflow bottleneck inside Minerva's learning platform
- designing and building a React-based extension UI for a fast one-click workflow
- implementing a cross-platform content injection system under Chrome Manifest V3
- automating retrieval of course readings and workbook content across Minerva and NotebookLM contexts
- configuring a Vite + crxjs build pipeline for Chrome Extension delivery
- shipping through the Chrome Web Store review process

This repository is the implementation of a workflow automation product that is already in active use by 50+ students.
