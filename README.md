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
Synthetic sample files (no real students) for practising, also linked from the Help page:
- `sample_marks_files/`: Student ID + Marks sheets (`<COURSE>_<N>students_max<M>.csv`)
- `sample_erp_files/`: the matching ERP CSVs (keep their names; the course code is read from them)
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
- 📋 The ERP file is the master list: if a student in it (other than W) is missing from your pasted marks, you are asked for their marks before the histogram opens; extra pasted students are ignored
- 🔠 Pasted IDs are converted to capital letters and checked against the BITS ID format (e.g. `2023A7PS0412H`) before the histogram opens
- 🔄 The New Grade and View Grades pages are cleared when you come back to them (including with the browser's Back button)
- 🆔 Students are matched between your marks and the ERP file on **year + roll number + campus** (e.g. `2023`, `0412`, `H` in `2023A7PS0412H`); the programme code (`A7PS`, `B4A7`, …) is ignored
- 🚫 Duplicate students are refused: the same student on two pasted rows, twice in the ERP file, or twice in a saved JSON (compared on year + roll number + campus, like matching)
- 🧪 Try with a small dataset first: the **Practice files** section of the Help page has made-up marks and ERP files (30 to 800 students) and a saved JSON to download

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
