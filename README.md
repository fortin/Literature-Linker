# Literature Linker Workflow (Zotero + Obsidian + Hookmark + OmniFocus + Alfred)

This Alfred workflow allows you to:

✅ Search your Zotero library from Alfred via a **CSL JSON** export (fast local search)  
✅ Generate literature notes directly into their final Obsidian folder  
✅ Use a Templater-based note template with metadata fields filled from Zotero  
✅ Auto-link the Obsidian note, the Zotero item, and the linked PDF in Hookmark  
✅ Optionally create an **OmniFocus project** for reading the item, with two recurring tasks  
✅ Insert an inline OmniFocus project link into the note using a placeholder comment  
✅ Respect user preferences for whether to link Markdown/PDFs in DEVONthink or elsewhere  
✅ Match PDFs in DEVONthink using citekey-based lookup, with fallback to fuzzy metadata search  

---

## 🔧 Setup Instructions

### 1️⃣ Environment

You’ll need a Python environment with:

- `micromamba activate zotero` (or any conda/venv environment)
- Python packages:
  - `pandas`
  - `pyyaml`
  - `python-decouple`

Your **Alfred Environment Variables** (or `.env`) should include:

```bash
CSL_JSON_PATH=/path/to/My Library.json
PYTHON_PATH=/path/to/python
PDF_FOLDER=/path/to/pdf/folder

ZOTERO_USER_ID=12345678
ZOTERO_API_KEY=your-api-key
ZOTERO_STORAGE=/path/to/Zotero/storage
ZOTERO_SQLITE=/path/to/zotero.sqlite

LINKED_ITEMS=/path/to/linked_items.csv
OBSIDIAN_VAULT=/path/to/Obsidian/vault
TEMPLATE_PATH=/path/to/your/Literature Template.md

SOURCE_MATERIAL=/path/to/Source Material
ARTICLES=532 📄 Articles
BOOKS=542 📖 Books

HOOK_PATH=/usr/local/bin/hook
SCRIPT_PATH=/path/to/standardise_item_types.py

# Optional OmniFocus integration
CREATE_LIT_NOTE_PATH=/path/to/create_lit_note.py
READ_BOOK_AUTOMATION_PATH=/path/to/read-book.omnijs

# Link preferences
MARKDOWN_IN_DEVONTHINK=False
PDF_IN_DEVONTHINK=True
```

**Important:**

Your Zotero library must be auto-exported using **Better CSL JSON** format with `Keep updated` enabled.

### 2. Obsidian Template

Templater must be enabled in Obsidian.

Create a file wherever you keep your templates in Obsidian. For these instructions, we're using:

```
📁 300 💡 Resources/📁 330 㡯 Templates/Literature Template.md
```

Template content:

```markdown
---
title: "@<% tp.file.title %>"
authors: <%* tR += tp.user.citationField("authorString") %>
citation: @<%* tR += tp.user.citationField("citekey") %>
year: <%* tR += tp.user.citationField("year") %>
DOI: <%* tR += tp.user.citationField("DOI") %>
tags: [[literature]], [[ToRead]]
URI: zotero://select/items/@<%* tR += tp.user.citationField("citekey") %>
---
📌 [Open in Zotero](zotero://select/items/@<%* tR += tp.user.citationField("citekey") %>)
📄 [Open PDF](/path/to/pdfs/<%* tR += tp.user.citationField("citekey") %>.pdf)
<!-- OMNIFOCUS_LINK_PLACEHOLDER -->

## Summary
-

## Key Points
-

## Quotes & Highlights
> ""

## My Thoughts
-

## Related Notes
-
```

Also create `templater/user/functions/citationField.js`:

```javascript
module.exports.citationField = (field) => {
  const yaml = app.plugins.plugins["citations"]?.api?.getActiveCitation();
  return yaml?.[field] || "";
};
```

### 3️⃣ Alfred Workflow Setup

- **Script Filter:** Uses `json_search.py` to match queries in your **CSL JSON** file.
- **Run Script:** Calls `create_lit_note.py` to generate the note, apply template, and Hookmark links.
- **Keywords:**
  - `lit` → creates the note and links, but **no OmniFocus project**
  - `read` → same as lit, but also creates an OmniFocus project with two recurring daily tasks:
    - ➡️ Read {title} by {author last name}
    - 🗃️ Turn highlights into Slip Box notes
  - Hookmark links are anchored to the **first recurring task**, ensuring they show up in the Today perspective.

#### Hookmark requirements:

- Hook CLI (hook) must be installed (/usr/local/bin/hook by default). You can install it from [here](https://brettterpstra.com/projects/hook-cli/).
- Files must be indexed in DEVONthink if you want DEVONthink links.

---

## **🔄 Typical Alfred Workflows**

### **🔎 Search and create/update a note**

→ Alfred → type lit keyword followed by query → select result → literature note created

What happens:

- Note created in the correct folder (Articles, Books, Other)
- DEVONthink is searched for the best PDF match using citekey → title/author fallback
- Zotero ⇄ Obsidian ⇄ PDF links created via Hookmark
- linked_items.csv updated to prevent duplicate notes

### **📝 Create note and OmniFocus project**

→ Alfred → type read keyword followed by query → select result → everything above **plus**:

- OmniFocus project named 🧭 Read ‘{title}’ by {author}
- Two recurring tasks created (reading + note processing)
- Hookmark links added to the first task
- Project URI inserted inline in the note by replacing <!-- OMNIFOCUS_LINK_PLACEHOLDER -->

------

## **✅ Features**

- Notes sorted automatically into:
  - 532 📄 Articles (journal articles, chapters, papers)
  - 542 📖 Books (books, theses, proceedings)
  - 572 ⺟ Other (miscellaneous, reports)
- Automatically creates Hookmark links for:
  - Zotero item (zotero://select/items/@citekey)
  - DEVONthink or file-based PDF
  - Obsidian note
  - First OmniFocus task
- Notes include an **inline OmniFocus link placeholder comment** (see the sample template above)
- Paths to Obsidian folder and other locations can be customised in `.env` and Alfred environment variables.

------

## **🧩 Troubleshooting**

- Define all environment variables via .env or the Alfred GUI
- Your Zotero export must use **Better CSL JSON** with Keep updated enabled
- DEVONthink must index the folders containing your PDFs/notes if using DEVONthink links
- Obsidian template **must** include <!-- OMNIFOCUS_LINK_PLACEHOLDER --> to support inline insertion
- PDFs should be named after citekeys for accurate matching

------

## **⚠ Limitations & Known Issues**

### **🔗 Hookmark integration**

- Hookmark can’t dynamically generate links *from* Zotero items but can use zotero://select/... URIs.
- Hookmark links to DEVONthink or Obsidian require those files to be indexed in DEVONthink (unless using file links).
- Hookmark links are local to each machine; syncing requires matching vault paths and DEVONthink databases.
- Moving files breaks Hookmark links unless manually re-established.

### **📅 OmniFocus integration**

- Projects are named 🧭 Read '{title}' by {author}
- First recurring task is used as the Hookmark anchor and appears in the Today perspective
- Inline OmniFocus link is inserted directly into the note via placeholder comment
- Projects/titles are deduplicated using task and project ID logic

### **🔎 Search performance**

- Local CSL JSON search is extremely fast
- SQLite or Web API alternatives are slower and not currently supported

### **🔄 Citations plugin quirks**

- If Obsidian’s Citations plugin stops working, re-sync the Zotero database from the plugin settings

------

Built with ❤️ to make academic research less painful.
