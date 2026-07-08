# Process Capability Record Tool

> เครื่องมือบันทึกค่าความสามารถของกระบวนการ (Cpk) แบบเว็บแอป — ทดแทนฟอร์ม Excel `FM8.3.2-PE-17`

**Author:** Sittisak C. (PE Engineer, DENSO Thailand)
**Version:** v1.0.1
**Type:** Single-file HTML app · No build tools required

---

## 🇹🇭 ภาพรวม

เครื่องมือเว็บสำหรับบันทึกข้อมูลการวัด, คำนวณ `Cp`, `Cpk` และแสดงผลในรูปแบบกราฟ 3 แบบ — ใช้ทดแทนเทมเพลต Excel `FM8.3.2-PE-17 (22.06R)` ของ DENSO Thailand

ออกแบบมาเพื่อ:
- เปิดในเบราว์เซอร์ได้ทันที (ไม่ต้องติดตั้งอะไร)
- เปรียบเทียบ dataset ได้ 2–8 ชุดในมุมมองเดียว
- Export กราฟลง PowerPoint ได้สะดวก (PNG/SVG/Clipboard)
- รองรับ 3 ภาษา (TH/EN/JP)

### ฟีเจอร์หลัก

- **Multi-dataset 2–8 ชุด** — เพิ่ม/ลบ/ตั้งชื่อ/เลือกสี/เลือกรูป marker ได้ในแต่ละชุด
- **3 กราฟ** — Capability View · Distribution View (Bell curve) · Run Chart
- **Drag-able UI** — ลากปรับ spacing, ตำแหน่ง USL/LSL label, ตำแหน่ง Cpk ของแต่ละ dataset ได้อิสระ
- **ปรับขนาด chart** — drag-resize + aspect presets (16:9, 4:3, 1:1, Free)
- **Export** — Copy PNG/SVG ลง clipboard, Save PNG/SVG เป็นไฟล์
- **Import** — จาก Excel (รองรับเทมเพลต FM8 เดิม) หรือจากรูป (OCR ผ่าน Tesseract.js)
- **i18n** — ไทย / English / 日本語
- **localStorage persistence** — บันทึก state อัตโนมัติ
- **Print** — A4 พิมพ์ได้สวย

### วิธีใช้

```
1. ดาวน์โหลด cpk_tool.html
2. เปิดไฟล์ในเบราว์เซอร์ (Chrome/Edge/Firefox)
3. กรอกข้อมูล → กราฟอัปเดตอัตโนมัติ
```

---

## 🇬🇧 Overview

Web-based tool for recording measurement data, computing `Cp` / `Cpk` statistics, and displaying results in three chart types. Built as a replacement for the DENSO Thailand `FM8.3.2-PE-17 (22.06R)` Excel template.

Designed to:
- Open instantly in any browser (no install required)
- Compare 2–8 datasets side-by-side
- Export charts to PowerPoint via PNG/SVG/Clipboard
- Support TH/EN/JP languages

### Key features

- **Multi-dataset 2–8** — add/remove/rename/color/marker shape per dataset
- **3 chart views** — Capability View, Distribution View (Bell curve), Run Chart
- **Draggable UI** — adjust dataset spacing, USL/LSL label positions, per-dataset Cpk labels
- **Resizable charts** — drag-resize + aspect presets (16:9, 4:3, 1:1, Free)
- **Export** — Copy PNG/SVG to clipboard, Save PNG/SVG files
- **Import** — from Excel (FM8 template compatible) or image (Tesseract.js OCR)
- **i18n** — Thai / English / Japanese
- **localStorage persistence** — auto-save state
- **Print** — A4-ready styles

---

## 🛠 Tech Stack

- **Single HTML file** — no build step, no bundler, no npm
- **Vanilla JavaScript** — no frameworks
- **CDN libraries:**
  - [Chart.js 4.4.0](https://www.chartjs.org/) — Run Chart
  - [SheetJS 0.18.5](https://sheetjs.com/) — Excel import
  - [Tesseract.js 5](https://tesseract.projectnaptha.com/) — OCR
- **Fonts:** IBM Plex Sans · IBM Plex Mono · Noto Sans Thai · Noto Sans JP (Google Fonts)
- **Custom SVG** — Capability + Distribution charts drawn from scratch

## 🚀 Quick Start

No installation needed — just open `cpk_tool.html` in a modern browser.

For development:

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/cpk-tool.git
cd cpk-tool

# Just open the file
open cpk_tool.html        # macOS
xdg-open cpk_tool.html    # Linux
start cpk_tool.html       # Windows
```

To serve locally (optional, for testing OCR features that need same-origin):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/cpk_tool.html
```

## 📁 Repository Structure

```
cpk-tool/
├── cpk_tool.html        # The entire app (HTML + CSS + JS in one file)
├── README.md            # This file
├── CLAUDE.md            # Context for Claude Code (AI assistant)
├── CHANGELOG.md         # Version history
└── .gitignore
```

## 📝 License

Internal tool. License TBD by author.

## 👤 Author

**Sittisak C.** — Process Engineering, DENSO (Thailand) Co., Ltd.

---

*Built iteratively with [Claude](https://claude.ai) as a pair-programming assistant.*
