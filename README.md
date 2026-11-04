# 🎓 BITS Pilani Grade Moderator

A simple, browser-based tool to help instructors **upload marks, visualize distributions, assign grades, and export results** — all without installing anything.


---

## 🚀 Features

### 🆕 New (Enhanced Workflow)
- ✨ **Paste directly from Excel / Google Sheets** (no CSV needed!)
- 🧹 Cleaner and faster grading interface (`New Grade`)
- ⚡ Automatic cleanup of empty rows and formatting
- ✨ NC cutoff criteria asked before printing the pdf

### 📊 Smart Grading & Analytics
- Interactive **histogram-based grading**
- 📈 Grade distribution visualization (bar + pie charts)
- 🔍 Search students instantly
- ✏️ Edit grades manually when needed

### 📂 Flexible Export Options
- 💾 Export ERP-ready CSV
- 🗂️ Save your work as JSON (resume later)
- 🖨️ Print reports as PDF

### 🔀 Sorting & Organization (NEW)
- Sort students by:
  - 🏫 Campus  
  - 🎓 Branch  
  - 🔀 Both (Campus + Branch)
- 📤 Or export directly in original order (old method still supported!)

---

## 🧭 How to Use

### 1️⃣ Download the Repository
- Click **Code → Download ZIP**
- Extract it anywhere on your system

### 2️⃣ Open the Application
- Open `index.html` in your browser

### 3️⃣ Choose a Workflow

#### ✅ *New Grade*
- Paste marks directly from Excel
- Upload ERP CSV
- Generate histogram and assign grades

#### 📂 View Grades
- Load a previously saved JSON file

#### 🧪 Practice data
Synthetic sample files (no real students; 30 to 800 per course, including CS F101 with a transfer `…GH` student and NC candidates) for practising, also linked from the Help page:
- `sample_marks_files/`: Student ID + Marks sheets (`<COURSE>_<N>students_max<M>.csv`)
- `sample_erp_files/`: the matching ERP CSVs (keep their names; the course code is read from them) (BITS F464 + CS U464 is one course with two ERP files, UG and PhD)
- `sample_jsons/`: a saved grades file to open with View Grades

---

## 🌐 Browser Support

✅ Best experience: **Google Chrome**  
Also works on:
- Firefox
- Edge
- Safari

---

## 💡 Tips for First-Time Users

- 💾 Always save a JSON backup
- 📋 The ERP file is the master list: if a student in it is missing from your pasted marks, you are asked for their marks before the histogram opens (W students: pasted marks are kept, a missing W student gets 0 without asking). After the ERP file(s) are loaded, the preview shows each student's ERP name and any non-letter ERP grade (such rows are orange); extra pasted students are ignored
- 🔠 Pasted IDs are converted to capital letters and checked against the BITS ID format (e.g. `2023A7PS0412H`) before the histogram opens
- 🔄 The New Grade and View Grades pages are cleared when you come back to them (including with the browser's Back button)
- 🆔 Students are matched between your marks and the ERP file on **year + roll number + campus** (e.g. `2023`, `0412`, `H` in `2023A7PS0412H`); the programme code (`A7PS`, `B4A7`, …) is ignored, except for PhD IDs with `PH` after the year (e.g. `2026PHXP2000H`), which must match in full because a PhD candidate can share year, roll number and campus with a first-degree student
- 📚 **Several ERP files for one course** (code change, separate UG/PhD codes): tick the box on New Grade and select all of them; students are combined (each once; a student on several ERP rows is fine, rows that disagree only give a warning), a Name column from ERP is added to the preview, and every ERP file is saved in the JSON and exported separately
- 🕒 Exported ERP CSVs are named after the ERP file plus `_CSV_saved_<date-time>` (like the JSON)
- 🎓 Transfer students' IDs may end in two or three different campus letters from G, H, P (e.g. `2023A7PS0412GH`, `…HP`, `…GHP`); these are accepted and matched on the full campus part
- 🧮 **NC Calculator** (histogram page, left button): NC line = a percentage (40 by default) of the median marks or a percentage (30 by default) of the average of the top K students (percentages and K of your choice), lists the students below it, and can shade the NC zone light red on the histogram
- ❌ The red ✕ button next to it gives NC to everyone below the NC line after a Yes/Cancel confirmation, downloads an `AUTO_BACKUP_` JSON first, and restarts the histogram at the new lowest mark; the rule it used is saved and prefills the NC criterion when you Print PDF
- 🚫 Duplicate students are refused: the same student on two pasted rows, or twice in a saved JSON (compared on year + roll number + campus, like matching)
- 🧪 Try with a small dataset first: the **Practice files** section of the Help page has made-up marks and ERP files (8 courses, 30 to 800 students, one with two ERP files) and a saved JSON to download

---

## 🛠️ Issues & Support

If something breaks (or you have ideas!):

- 🐛 Open an issue on GitHub  
- 📧 Email:  
  <a href="mailto:a.mukherjee@hyderabad.bits-pilani.ac.in">a.mukherjee@hyderabad.bits-pilani.ac.in</a>

---

## 📜 License

Licensed under the **MIT License**  
See `LICENSE.md` for details.

---

✨ Built to make grading smoother, faster, and less painful.
