# 🔁 n8n Loop + IF Filter Reports

An n8n workflow that takes a student CSV uploaded through a form, processes it in batches, filters it with two IF conditions, and automatically downloads **two filtered CSV reports** in a single zip file.

![Workflow canvas](screenshots/workflow-canvas.png)

---

## 🔄 How It Works

```
Form Upload → Extract from File → Loop Over Items ─┬─ loop ─→ Wait (1 sec) ─→ back to Loop
                                                   │
                                                   └─ done ─→ IF: City = Chennai
                                                                ├→ Convert to File  (chennai_list.csv) ─────┐
                                                                └→ IF: Email valid                          │
                                                                     └→ Convert to File (email_vaild_data.csv)
                                                                                                            ↓
                                                              Merge → Compression (zip) → Form Ending (download)
```

| Step | Node | What it does |
|------|------|--------------|
| 1 | **On form submission** | Form to upload the student CSV |
| 2 | **Extract from File** | Splits the CSV into one item per row |
| 3 | **Loop Over Items** | Processes rows in batches of 10 |
| 4 | **Wait** | Pauses 1 second per batch (useful for API rate limits) |
| 5 | **If** | Keeps only students where `City = Chennai` |
| 6 | **If1** | From those, keeps only students where `Email_Valid = True` |
| 7 | **Convert to File** ×2 | Creates `chennai_list.csv` and `email_vaild_data.csv` |
| 8 | **Merge** | Combines both files onto one item |
| 9 | **Compression** | Zips both CSVs into `chennai_reports.zip` |
| 10 | **Form Ending** | Returns the zip to the browser as a download |

---

## 📁 Project Structure

```
├── workflow/
│   └── etl-student-loop-filter.json   # n8n workflow export
├── sample-data/
│   └── File.csv                       # Input student data
├── output/
│   ├── chennai_list.csv               # All Chennai students
│   └── email_vaild_data.csv           # Chennai students with valid emails
├── screenshots/
│   └── workflow-canvas.png
├── README.md
└── .gitignore
```

---

## 🚀 Setup

1. In n8n, go to **Workflows → ⋯ → Import from File** and select `workflow/etl-student-loop-filter.json`.
2. Click **Execute workflow**.
3. Upload `sample-data/File.csv` in the form and keep the tab open.
4. When processing finishes, `chennai_reports.zip` downloads automatically. Unzip it to get both reports.

No API keys or credentials are needed.

---

## 💡 Key Lessons

- **Loop Over Items has two outputs.** `loop` sends one batch at a time to the nodes inside the loop, and the last node must connect back to the loop's input. `done` fires once, with all items, after every batch finishes. The filters belong on `done`.
- **An n8n form can return only one file.** With two Form Ending nodes, only the first file downloads. Merging both files and zipping them with the Compression node solves this.
- **Check the Wait node's unit.** Set it explicitly to **Seconds**; the default may be hours, which makes the workflow appear stuck.
- **Binary field names must match exactly** across Convert to File, Compression and Form Ending, including spelling.
- **Make IF conditions forgiving.** CSV values can arrive as `True`, `true` or with extra spaces. An expression like `{{ String($json.Email_Valid).trim().toLowerCase() }}` compared to `true` handles them all.

---

## 🛠️ Tech Stack

- [n8n](https://n8n.io) – workflow automation
- Nodes used: Form Trigger, Extract from File, Loop Over Items, Wait, If, Convert to File, Merge, Compression, Form

---

## 🙏 Acknowledgements

Built while learning automation with **Social Eagle**.
