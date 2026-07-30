# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

Single-file static HTML app (`ThinkNotes.html`) — an offline credential/account notes manager with a One Piece (海贼王) theme. Contains embedded CSS, JavaScript, and data (account passwords, server URLs, database credentials). No build tools, no dependencies, no external services.

### Key characteristics
- **Single file**: Everything lives in `ThinkNotes.html` — HTML structure, `<style>`, `<script>`, and inline `DATA` object.
- **No build step**: Opens directly in a browser via `file://`. Edit the source and refresh.
- **Data model**: `DATA.platforms[]` array holds platform entries; each platform may have `instances[]` for database/server credentials. Fields are rendered dynamically by `renderFields()` / `renderInstanceFields()`.
- **Security**: Passwords are masked behind a master password gate (`MASTER_PW`, hardcoded as `'123456'`). Clicking a masked password triggers a verification dialog; once unlocked, subsequent clicks copy directly. Copyable fields use `data-value` attributes.
- **UI sections**: Fixed left sidebar (navigation), scrollable right content area (resource cards). Toast notifications for copy feedback.

### Common edits
- **Add accounts**: Append entries to the `DATA.platforms[]` array in the `<script>` block. Follow the existing shape: `{ name, url, loginMethod, username?, password?, note?, instances?: [...] }`.
- **Modify styling**: Changes go in the `<style>` block. CSS uses `--var()` custom properties defined in `:root`.
- **Add features**: Since this is vanilla JS, new logic belongs inside the `<script>` block, respecting the existing `DOMContentLoaded` bootstrap pattern.
