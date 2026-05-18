# EarnHub — Design Brief

## Problem: Kenapa sekarang keliatan "AI banget"?

1. **Terlalu seragam** — setiap card identik, setiap section pakai grid yang sama
2. **Warna terlalu ramai** — amber, pink, orange, purple, teal semua muncul sekaligus
3. **Badge overpowering** — TOP PICK dan FAST CASH bersaing visual dengan konten card
4. **Typography flat** — ukuran dan weight terlalu mirip satu sama lain, tidak ada hierarchy yang kuat
5. **Terlalu banyak emoji** di nav, tag, dan section header — nuansanya jadi toyish
6. **Section header monoton** — icon box + title + desc, semua sama polanya
7. **Gradient text di hero** — ciri khas website AI/template

---

## Directions

### 1. Warna — Simpelkan
Fokus ke **2 warna inti** saja:
- **Navy** `#1a4a8c` — authority, primary actions
- **Teal** `#00c896` — accent, CTA, highlight

Warna kategori (amber, pink, orange) tetap ada tapi hanya di payout pill — bukan di button. Semua button pakai navy atau teal.

### 2. Typography — Bikin hierarchy jelas
- Hero H1: besar, bold, **plain color** (tidak gradient)
- Section title: pakai **left accent border** berwarna — lebih editorial
- Card title: sedikit lebih besar, weight 700 (bukan 800)
- Kurangi ALL CAPS, kurangi letter-spacing berlebihan

### 3. Cards — Bedakan yang penting
- Card biasa: clean, minimal border, flat
- Card TOP PICK: punya **left colored border** (4px solid teal), tidak perlu badge pill
- Card AI Trainer: background tint subtle, lebih lebar (full row atau 2-col max)
- Kurangi badge — hapus teks label, pakai visual cue saja

### 4. Layout — Campur grid dan list
- AI Trainer section: **horizontal card** (landscape), bukan kotak
- Top Picks: grid 2 kolom wide
- Games/Surveys/Passive: grid 3 kolom standar
- Satu section boleh punya featured item pertama yang lebih lebar

### 5. Section headers — Lebih editorial
- Ganti icon box dengan **colored left border** (3–4px) di section title
- Hilangkan box icon, pakai emoji kecil inline saja
- Tambah subtle divider line di antara section, bukan full-width hr

### 6. Micro-details
- Ref code: pakai monospace chip yang lebih rapi, copy button lebih kecil/subtle
- Button: kurangi radius sedikit (8px), lebih "korporat"
- Nav: kurangi jumlah link — gabung beberapa section, atau pakai dropdown
- Footer: lebih singkat, tambah tahun

---

## Priority Implementation Order

1. [ ] Simplify warna button — semuanya navy atau teal
2. [ ] Ganti badge pill dengan left border pada card
3. [ ] Hero H1 → plain bold, hapus gradient text
4. [ ] Section header → left border accent, hapus icon box
5. [ ] AI Trainer card → landscape/horizontal layout
6. [ ] Kurangi emoji di nav (teks saja)
7. [ ] Typography hierarchy — naikkan contrast antara title dan desc
