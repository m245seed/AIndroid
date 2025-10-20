# AI File Organizer — Implementation Plan

**Last updated:** 2025-10-20T12:11:44Z

This document captures a practical, end‑to‑end plan for building and shipping the AI File Organizer across desktop (Python/Tkinter) and Android (Kotlin + Jetpack Compose + SAF). It reflects the current dual‑root design (SOURCE → LIBRARY), _top 5 biggest files per folder_ hints, name‑clash handling, and undo support.

---

## 1) Goals & Scope

- Move top‑level items from an **unsorted SOURCE** into an organized **LIBRARY** structure with exactly **two levels**: `Category/Subcategory` (or `Category/_root`).
- Prefer **existing** Category/Subcategory in LIBRARY; create minimal new folders only if necessary.
- Show a preview plan before applying moves, then **apply** moves with a confirmation.
- Track a **moves log** so the operation can be fully **undone**.
- For each top‑level folder in SOURCE, include up to **5 largest file names** (recursive) to guide the LLM’s placement decisions.
- Handle name clashes by **renaming** incoming items to `name (2)`, `name (3)`, etc., without merging.

---

## 2) Desktop (Python) Versions

### 2.1 Single‑root (earlier) vs. Dual‑root (current)
- **Single‑root:** Organizes in place. (Legacy)
- **Dual‑root (current):** Moves from SOURCE to LIBRARY; indexes LIBRARY (two levels) for reuse‑first placement.

### 2.2 Files & Temp Storage
- Per target pair workspace: `{temp}/ai_downloads_organizer_dual/<sha1(SOURCE::LIBRARY)[:16]>`.
- Saved artifacts:
  - `source_inventory_*.json` + `source_inventory_latest.json`
  - `library_index_*.json`   + `library_index_latest.json`
  - `plan_*.json`            + `plan_latest.json`
  - `moves_*.json`           + `moves_latest.json`

### 2.3 Credentials
- **Gemini**: `GEMINI_API_KEY`
- **Other provider (OpenAI‑compatible)**: `ANTHROPIC_API_KEY` and optional `OPENAI_BASE_URL`

### 2.4 Key Behaviors
- **Create Plan**: scan SOURCE top‑level, compute `top_big_files` for folders; index LIBRARY two levels; request LLM plan; save to `plan_*.json`.
- **Organise Files**: apply plan; write `moves_*.json` with source/destination rel paths.
- **Undo Moves**: read last `moves_*.json` and reverse operations.
- **Name clash**: use `name (n)`; for files, number is added before extension.

---

## 3) Android App (Kotlin + Compose + Storage Access Framework)

### 3.1 UX Flow
1. **Pick SOURCE** and **Pick LIBRARY** via `ACTION_OPEN_DOCUMENT_TREE` (SAF), persist permissions.
2. **Create Plan**: 
   - `scanSource(sourceUri)` → top‑level items + `top_big_files` per directory.
   - `indexLibrary(libraryUri)` → two‑level Category/Subcategory map.
   - Send both JSONs to a **Planner**: 
     - **RemotePlanner** (recommended): call your backend `/plan` with the two JSONs.
     - **LocalHeuristicPlanner** (fallback): simple, on‑device heuristic to get started.
   - Save to `cache/organizer/<hash>/`.
3. **Organise Files**: apply plan using SAF; handle name clashes; log to `moves.json`.
4. **Undo Moves**: reverse from `moves.json` with SAF operations.

### 3.2 Data Models (Kotlin)
- `SourceOverview` with `entries: List<SourceEntry>` where a directory entry includes `top_big_files: List<String>`.
- `LibraryIndex` → `categories: List<LibCategory>` each with `subcategories: List<LibSubcategory>` and optional pseudo `"_root"`.
- `Plan` → `placements: List<Placement>` and optional `new_folders`.
- `MoveOp` → records the actual destination (URI on Android; relative path on desktop).

### 3.3 SAF Implementation Notes
- Use `DocumentFile` & `ContentResolver` to enumerate and move.
- Prefer `DocumentsContract.moveDocument` when available; otherwise **copy+delete**.
- **uniqueChildName()** generates `name (n)` resolution; always check with `destDir.findFile()`.
- For big‑file hints: recursively traverse directories and keep a min‑heap of `(size, name)` to retain top 5.

### 3.4 Workspace & Logs
- `cache/organizer/<sha1(SOURCE::LIBRARY)[:16]>`.
- Files: `source_inventory.json`, `library_index.json`, `plan.json`, `moves.json`.

### 3.5 Planner Backend (recommended)
- Place your API keys server‑side. Android app calls a lightweight proxy.
- Endpoint: `POST /plan`
  - **Request**
    ```json
    {
      "source": <SourceOverview>,
      "library": <LibraryIndex>
    }
    ```
  - **Response** (Plan)
    ```json
    {
      "placements": [
        { "path": "top_level_item", "category": "Cat", "subcategory": "Sub" | null, "reason": "..." }
      ],
      "new_folders": [ { "category": "Cat", "subcategory": "Sub" | null, "reason": "..." } ],
      "notes": "..."
    }
    ```
- **Prompt key points** (match desktop prompt): reuse existing categories, only create when needed; never split a top‑level SOURCE folder; only two levels; use `top_big_files` to infer content; output strict JSON.

---

## 4) JSON Schemas (informal)

### 4.1 SourceOverview
```json
{
  "source_root": "<uri-or-path>",
  "generated_at": "ISO-8601",
  "total_entries": 0,
  "entries": [
    {
      "type": "file",
      "rel_path": "name.ext",
      "size": "12.3MB",
      "suffix": ".ext",
      "mime": "type/subtype"
    },
    {
      "type": "directory",
      "rel_path": "FolderName",
      "counts": { "directories": 0, "files": 0 },
      "sample_children": ["a/", "b/", "c"],
      "top_big_files": ["big1.mkv", "big2.zip", "..."]
    }
  ],
  "file_extension_counts": [[".pdf", 10], [".jpg", 5]]
}
```

### 4.2 LibraryIndex
```json
{
  "library_root": "<uri-or-path>",
  "generated_at": "ISO-8601",
  "categories": [
    {
      "name": "Category",
      "subcategories": [
        { "name": "Subcategory", "file_count": 0, "sample_files": ["a.ext", "b.ext"] },
        { "name": "_root", "file_count": 2, "sample_files": ["loose1.ext"] }
      ],
      "notes": ""
    }
  ]
}
```

### 4.3 Plan
```json
{
  "placements": [
    { "path": "ItemA", "category": "Media", "subcategory": "Photos", "reason": "..." },
    { "path": "ItemB", "category": "Documents", "subcategory": null, "reason": "..." }
  ],
  "new_folders": [
    { "category": "Research", "subcategory": "Datasets", "reason": "no good fit" }
  ],
  "notes": "..."
}
```

### 4.4 Moves Log (desktop)
```json
{
  "executed_at": "ISO-8601",
  "source_root": "/path/to/source",
  "library_root": "/path/to/library",
  "operations": [
    { "source_rel": "ItemA", "destination_rel": "Media/Photos/ItemA", "reason": "..." }
  ]
}
```

---

## 5) Error Handling & Edge Cases
- **Permissions**: desktop — OS permissions; Android — SAF persisted permissions.
- **Missing items**: skip with reason `source not found`.
- **Missing category** in plan: skip with reason `missing category`.
- **Existing destination**: rename to `name (n)`; never merge.
- **Undo**: if an item is missing at destination, record a failure entry and continue.
- **Long paths / invalid characters**: sanitize or fall back to renaming.

---

## 6) Testing Checklist
- Plan preview lists correct counts; `top_big_files` present for folder entries.
- Moves apply into existing categories without creating duplicates; new folders created only when needed.
- Name clash behavior: files & folders become `name (2)`, `name (3)` …
- Undo fully reverses.
- Android: SAF move and copy fallback both work; large file copy progress acceptable.

---

## 7) Roadmap (optional)
- Progress UI with per‑file counters and cancel.
- “Strict reuse only” mode (forbid creating new categories).
- Merge mode: allow merging folders with same name (with safeguards).
- On‑device embeddings as a fallback when offline (no LLM).
- iOS variant via Kotlin Multiplatform + SwiftUI wrappers.

---

## 8) Build Notes Summary
- **Desktop**: Python 3.10+, `openai` (if using OpenAI‑compatible), `google-genai` (for Gemini). Run `python sorter_dualroot.py`.
- **Android**: Compose app + `documentfile`, `kotlinx-serialization-json`, optional `okhttp` for networking; set `PLANNER_BASE_URL` to your proxy.

---

### Appendix A — Prompt Tips (for your backend)
- **System**: “You are a meticulous file librarian…”
- **User**: include both JSON blocks; restate constraints; emphasize: two levels, reuse‑first, no splitting top‑level folders, obey `top_big_files` hints; demand strict JSON object output.

### Appendix B — Security
- Keep API keys **server‑side**. Android app never stores keys; use your proxy service.
