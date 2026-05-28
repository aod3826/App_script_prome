GitHub Pages
└── index.html          ← Frontend (Gallery + Upload Form)

Google Apps Script
└── Code.gs             ← API (GET/POST)
    ├── doGet()         → ดึงข้อมูลทั้งหมดจาก Sheet
    └── doPost()        → รับรูป + บันทึก Sheet + upload Drive

Google Sheet (Prompts)
└── id, timestamp, prompt, model, tags, rating, imageUrl, driveFileId

Google Drive (Folder)
└── ไฟล์รูปภาพทั้งหมด (Public, View only)
