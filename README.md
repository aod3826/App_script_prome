# 📱 วิธีสร้าง PWA Icon แบบ Programmatic
> บันทึกเทคนิคจากโปรเจกต์ AI Image Vault

---

## PWA คืออะไร และทำไมต้องมี Icon?

**PWA (Progressive Web App)** คือเว็บแอปที่ถูก "ยกระดับ" ให้ทำงานคล้าย Native App โดยที่ไม่ต้องอยู่ใน App Store สิ่งที่ทำให้เว็บกลายเป็น PWA มีองค์ประกอบหลัก 3 อย่าง:

```
เว็บปกติ  +  manifest.json  +  Service Worker  =  PWA
```

Icon เป็นสิ่งสำคัญเพราะเมื่อผู้ใช้กด "Add to Home Screen" ระบบจะดึง icon จาก manifest ไปแสดงบนหน้าจอเหมือน App จริงๆ

---

## โครงสร้างไฟล์ที่ต้องมี

```
project/
├── index.html       ← เว็บหลัก (ต้องใส่ <link rel="manifest">)
├── manifest.json    ← บอก OS ว่าแอปนี้ชื่ออะไร icon ไหน สีอะไร
├── sw.js            ← Service Worker (cache + offline)
├── icon-192.png     ← Icon สำหรับ Android Home Screen
└── icon-512.png     ← Icon สำหรับ Splash Screen + ความละเอียดสูง
```

---

## ขั้นตอนที่ 1 — สร้าง Icon ด้วย Python (Pillow)

แทนที่จะวาดใน Figma แล้ว export ผมใช้ **Python + Pillow** วาด icon แบบ programmatic โดยตรง ข้อดีคือ:
- ควบคุม pixel ได้ทุกจุด
- สร้างหลายขนาดจาก function เดียว
- แก้ไขได้ทันทีโดยไม่ต้องเปิด design tool

### ติดตั้ง Library

```bash
pip install Pillow
```

### โครงสร้างโค้ดสร้าง Icon

```python
from PIL import Image, ImageDraw, ImageFont
import math

def make_icon(size):
    # 1. สร้าง canvas เปล่า (RGBA = มี transparency)
    img = Image.new("RGBA", (size, size), (0, 0, 0, 0))
    d   = ImageDraw.Draw(img)

    # 2. วาด background (rounded rectangle)
    d.rounded_rectangle([0, 0, size, size], radius=int(size*0.22), fill="#0d0d0f")

    # 3. วาด grid lines จาง ๆ (ให้ดู techy)
    step = size // 8
    for i in range(1, 8):
        d.line([(i*step, 0), (i*step, size)], fill="#1a1814", width=1)
        d.line([(0, i*step), (size, i*step)], fill="#1a1814", width=1)

    cx, cy = size//2, size//2

    # 4. วาดวงแหวน Orbital (arc ไม่ครบ 360° = มีช่องว่าง)
    r1 = int(size * 0.38)
    d.arc([cx-r1, cy-r1, cx+r1, cy+r1],
          start=30, end=320,
          fill="#c8502a", width=int(size*0.032))

    r2 = int(size * 0.26)
    d.arc([cx-r2, cy-r2, cx+r2, cy+r2],
          start=210, end=140,          # หมุนสวนทิศ
          fill="#d4a843", width=int(size*0.025))

    # 5. วาด dot ที่ปลายวงแหวน (เหมือน planet)
    ang = math.radians(322)
    dx  = int(cx + r1 * math.cos(ang))
    dy  = int(cy + r1 * math.sin(ang))
    dot_r = int(size * 0.025)
    d.ellipse([dx-dot_r, dy-dot_r, dx+dot_r, dy+dot_r], fill="#c8502a")

    # 6. วาดไอคอนรูปภาพตรงกลาง
    fw, fh = int(size*0.18), int(size*0.14)
    d.rounded_rectangle(
        [cx-fw//2, cy-fh//2, cx+fw//2, cy+fh//2],
        radius=int(size*0.02),
        fill="#1e1c19", outline="#c8502a", width=int(size*0.015)
    )

    # 7. ใส่ข้อความ "AI" พร้อม pill background
    fnt = ImageFont.truetype("DejaVuSans-Bold.ttf", int(size*0.10))
    bbox = d.textbbox((0,0), "AI", font=fnt)
    tw, th = bbox[2]-bbox[0], bbox[3]-bbox[1]
    tx = cx - tw//2
    ty = cy + int(size*0.30)
    pad = int(size*0.02)
    d.rounded_rectangle([tx-pad, ty-pad, tx+tw+pad, ty+th+pad],
                         radius=pad, fill="#c8502a")
    d.text((tx, ty), "AI", font=fnt, fill="#ffffff")

    return img

# สร้างทั้ง 2 ขนาดจาก function เดียว
for size in [192, 512]:
    icon = make_icon(size)
    icon.save(f"icon-{size}.png", "PNG")
```

### เทคนิคสำคัญในการวาด

| เทคนิค | วิธีใช้ใน Pillow | ผลลัพธ์ |
|--------|-----------------|---------|
| **Rounded rectangle** | `d.rounded_rectangle([x0,y0,x1,y1], radius=r)` | พื้นหลังมุมมน |
| **Arc (ไม่เต็มวง)** | `d.arc(bbox, start=30, end=320)` | วงแหวนที่มีช่องว่าง |
| **หมุนสวนทิศ** | สลับค่า start/end เช่น `start=210, end=140` | วงแหวนชั้นในหมุนกลับ |
| **Dot บน arc** | คำนวณจาก `cos/sin` ของมุมที่ต้องการ | จุดที่ปลายวงแหวน |
| **Text + Pill** | วาด rounded_rect ก่อน แล้ววาง text ทับ | ป้ายข้อความ |
| **Scale อัตโนมัติ** | คูณทุก value ด้วย `size` | icon คมชัดทุกขนาด |

> **หลักการสำคัญ:** อย่า hardcode pixel — ใช้ `int(size * 0.xx)` เสมอ เพื่อให้ขนาด 192 และ 512 ดูสัดส่วนเดียวกัน

---

## ขั้นตอนที่ 2 — สร้าง manifest.json

```json
{
  "name": "AI Image Vault",
  "short_name": "AI Vault",
  "description": "จัดการคลังภาพ AI Prompt ของคุณ",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#0d0d0f",
  "theme_color": "#c8502a",
  "orientation": "any",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

### อธิบาย fields สำคัญ

| Field | ความหมาย | ตัวอย่าง |
|-------|----------|---------|
| `display` | `standalone` = ซ่อน browser bar | ดูเหมือน native app |
| `background_color` | สีพื้น Splash Screen | ควรตรงกับ bg ของ icon |
| `theme_color` | สี status bar บน Android | ใช้ accent color |
| `purpose: maskable` | icon จะถูกตัดเป็นวงกลม/รูปทรงอื่นบาง OS | ต้องมี safe zone 80% |

---

## ขั้นตอนที่ 3 — สร้าง Service Worker (sw.js)

```javascript
const CACHE  = "aiv-v1";
const ASSETS = ["./index.html", "./manifest.json",
                "./icon-192.png", "./icon-512.png"];

// ติดตั้ง: cache ไฟล์หลักทั้งหมด
self.addEventListener("install", e => {
  e.waitUntil(
    caches.open(CACHE)
      .then(c => c.addAll(ASSETS))
      .then(() => self.skipWaiting())
  );
});

// Activate: ลบ cache เวอร์ชันเก่า
self.addEventListener("activate", e => {
  e.waitUntil(
    caches.keys()
      .then(keys => Promise.all(
        keys.filter(k => k !== CACHE).map(k => caches.delete(k))
      ))
      .then(() => self.clients.claim())
  );
});

// Fetch: เสิร์ฟจาก cache ก่อน ถ้าไม่มีค่อยไปเน็ต
self.addEventListener("fetch", e => {
  // API calls ต้องไปเน็ตเสมอ — ข้ามไป
  if (e.request.url.includes("script.google.com") ||
      e.request.url.includes("api.anthropic.com")) return;

  e.respondWith(
    caches.match(e.request)
      .then(cached => cached || fetch(e.request))
  );
});
```

### Cache Strategy ที่ใช้: Cache First

```
Request → มีใน Cache? → YES → เสิร์ฟจาก Cache (เร็ว)
                      → NO  → ไปดึงจากเน็ต → เก็บ Cache → เสิร์ฟ
```

---

## ขั้นตอนที่ 4 — เชื่อมทุกอย่างใน index.html

### ใส่ใน `<head>`

```html
<!-- บอก browser ว่ามี manifest -->
<link rel="manifest" href="manifest.json">

<!-- สี status bar (Android Chrome) -->
<meta name="theme-color" content="#c8502a">

<!-- iOS Safari -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="AI Vault">
<link rel="apple-touch-icon" href="icon-192.png">

<!-- Favicon -->
<link rel="icon" type="image/png" sizes="192x192" href="icon-192.png">
```

### Register Service Worker ใน `<script>`

```javascript
if ("serviceWorker" in navigator) {
  window.addEventListener("load", () => {
    navigator.serviceWorker.register("./sw.js")
      .then(reg => console.log("SW registered:", reg.scope))
      .catch(err => console.log("SW error:", err));
  });
}
```

### Install Banner (ปุ่มติดตั้ง)

```javascript
let deferredPrompt = null;

// browser ยิง event นี้เมื่อเว็บผ่านเกณฑ์ PWA ครบ
window.addEventListener("beforeinstallprompt", e => {
  e.preventDefault();           // ไม่ให้ browser แสดง popup เอง
  deferredPrompt = e;           // เก็บ event ไว้ใช้ทีหลัง
  showInstallBanner();          // แสดง UI ของเราเอง
});

// เมื่อผู้ใช้กดปุ่ม "ติดตั้ง"
installBtn.addEventListener("click", async () => {
  deferredPrompt.prompt();                        // แสดง native dialog
  const { outcome } = await deferredPrompt.userChoice;
  deferredPrompt = null;
  if (outcome === "accepted") showSuccessMessage();
});
```

---

## ข้อควรรู้เพิ่มเติม

### Maskable Icon คืออะไร?

Android บางรุ่นจะตัด icon ให้เป็นรูปทรงต่างๆ (วงกลม, รูปสี่เหลี่ยม, หยดน้ำ) ถ้าตั้งค่า `purpose: maskable` ต้องออกแบบให้ **เนื้อหาสำคัญอยู่ใน safe zone 80% ตรงกลาง** ส่วนขอบนอกอาจถูกตัดออก

```
┌─────────────────┐
│  ░░░░░░░░░░░░░  │  ← อาจถูกตัด
│  ░┌───────┐░░  │
│  ░│ SAFE  │░░  │  ← เนื้อหาหลักอยู่ตรงนี้
│  ░│ ZONE  │░░  │
│  ░└───────┘░░  │
│  ░░░░░░░░░░░░░  │  ← อาจถูกตัด
└─────────────────┘
```

### เช็ค PWA ผ่าน Chrome DevTools

1. เปิด DevTools → แท็บ **Application**
2. ดูที่ **Manifest** — ควรเห็นข้อมูลครบ ไม่มี error
3. ดูที่ **Service Workers** — ควรเห็นสถานะ `activated and running`
4. กด **Lighthouse** → เลือก Progressive Web App → Analyze

### เงื่อนไขที่ต้องครบก่อน Install Banner จะโชว์

- ✅ มี `manifest.json` ครบถ้วน
- ✅ มี Service Worker registered
- ✅ เปิดผ่าน **HTTPS** เท่านั้น (GitHub Pages รองรับ)
- ✅ ผู้ใช้เข้าเว็บแล้วมี engagement (browser ดูพฤติกรรม)

---

## สรุป Flow ทั้งหมด

```
Python Pillow
  └─ make_icon(192) → icon-192.png
  └─ make_icon(512) → icon-512.png
       │
       ▼
manifest.json ← อ้างอิง icon ทั้งสอง
       │
       ▼
index.html ← link manifest + register SW
       │
       ▼
sw.js ← cache ไฟล์ทั้งหมด
       │
       ▼
GitHub Pages (HTTPS)
       │
       ▼
ผู้ใช้เห็นปุ่ม "Add to Home Screen" / Install Banner
       │
       ▼
📱 Icon บน Home Screen เหมือน Native App
```

---

*บันทึกโดย AI Image Vault Project — ขอให้โค้ดสนุก! 🚀*
