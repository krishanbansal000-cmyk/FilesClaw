# FilesClaw - Quick Reference

## 🎯 What is FilesClaw?

A GitHub Pages-powered file repository with built-in viewers for:
- **Markdown files** (rendered as HTML)
- **Images** (PNG, JPG, GIF, WebP, SVG)
- **PDFs** (embedded viewer)
- **Text/Code** (syntax display)

---

## 📁 Repository Location

```
/root/projects/FilesClaw/
```

---

## 🚀 Upload Files

### Single File
```bash
cd /root/projects/FilesClaw
cp /path/to/file.md .
git add file.md
git commit -m "Add file.md"
git push
```

### Multiple Files
```bash
cd /root/projects/FilesClaw
cp /path/to/*.md .
git add *.md
git commit -m "Add analysis files"
git push
```

### From Astrology Project
```bash
cd /root/projects/FilesClaw
cp /home/angle/projects/astrology/astrology/backtests/*.json .
git add *.json
git commit -m "Add backtest results"
git push
```

---

## 🌐 View Files

**Live Site:** `https://krishanbansal000-cmyk.github.io/FilesClaw/`

**Direct File URL:**
```
https://krishanbansal000-cmyk.github.io/FilesClaw/?path=filename.md
```

---

## 📊 Supported File Types

| Extension | Viewer Type | Example Use |
|-----------|-------------|-------------|
| `.md` | Rendered Markdown | Analysis reports, notes |
| `.txt` | Text viewer | Raw text, logs |
| `.json` | Code viewer | Data, configs |
| `.csv` | Code viewer | Spreadsheets, data |
| `.png` | Image viewer | Charts, screenshots |
| `.jpg` / `.jpeg` | Image viewer | Photos, charts |
| `.gif` | Image viewer | Animations |
| `.webp` | Image viewer | Optimized images |
| `.svg` | Image viewer | Vector graphics |
| `.pdf` | PDF embed | Documents, reports |
| `.html` | Code viewer | Web pages |
| `.js` | Code viewer | Scripts |
| `.css` | Code viewer | Styles |

---

## 💡 Example Workflows

### Daily Trading Analysis
```bash
cd /root/projects/FilesClaw

# Generate analysis (from your astrology app)
python3 /home/angle/projects/astrology/astrology/scripts/generate_daily.py > daily-analysis-$(date +%Y-%m-%d).md

# Upload
git add daily-analysis-*.md
git commit -m "Add daily analysis $(date +%Y-%m-%d)"
git push

# Share link
echo "https://krishanbansal000-cmyk.github.io/FilesClaw/?path=daily-analysis-$(date +%Y-%m-%d).md"
```

### Backtest Results
```bash
cd /root/projects/FilesClaw

# Copy backtest results
cp /home/angle/projects/astrology/astrology/backtests/*.json .

# Upload
git add *.json
git commit -m "Add backtest results"
git push
```

### Charts & Screenshots
```bash
cd /root/projects/FilesClaw

# Copy screenshots
cp ~/Downloads/chart-*.png .

# Upload
git add *.png
git commit -m "Add chart screenshots"
git push
```

---

## 🔗 Share Files

**To share a file via Telegram/Discord:**

1. Open: `https://krishanbansal000-cmyk.github.io/FilesClaw/`
2. Click the file
3. Copy URL from browser
4. Paste in chat

**URL Format:**
```
https://krishanbansal000-cmyk.github.io/FilesClaw/?path=filename.md
```

---

## 📋 Git Commands Cheat Sheet

```bash
# Check status
git status

# View changes
git diff

# Add all changes
git add .

# Commit
git commit -m "Your message"

# Push
git push

# Pull latest
git pull

# View history
git log --oneline -10
```

---

## ⚙️ GitHub Pages Setup

**If site is not live:**

1. Go to: `https://github.com/krishanbansal000-cmyk/FilesClaw/settings/pages`
2. Source: **Deploy from a branch**
3. Branch: **main**
4. Folder: **/ (root)**
5. Click **Save**
6. Wait 1-2 minutes

---

## 🎨 Features

### Markdown Rendering
- Headers, lists, tables
- Code blocks with syntax highlighting
- Blockquotes, links, images
- Full CommonMark support

### Image Viewer
- Responsive display
- Full-size viewing
- All major formats

### File Browser
- Grid layout with icons
- File type badges
- Size display
- Folder navigation

### Auto-Deploy
- GitHub Actions deploys on every push
- Usually takes 30-60 seconds
- Check: `https://github.com/krishanbansal000-cmyk/FilesClaw/actions`

---

## 📞 Troubleshooting

### Site shows 404
- Enable GitHub Pages in repo settings
- Wait 1-2 minutes after enabling

### Files not updating
- Check GitHub Actions: `https://github.com/krishanbansal000-cmyk/FilesClaw/actions`
- Hard refresh browser (Ctrl+Shift+R)

### Large files failing
- GitHub limit: 100 MB per file
- Use Git LFS for larger files

---

**Created:** April 10, 2026  
**Repo:** `krishanbansal000-cmyk/FilesClaw`
