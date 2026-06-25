# Plan: Artikel Blog Template — Premium Reading UX

> **Goal:** Template artikel yang nyaman dibaca, responsif, dan mendorong konversi — tanpa HTML markup berantakan.

**File utama:** `src/pages/blog/[slug].astro`
**File konten:** `src/content/blog/*.md` (14 file)

---

## 🔍 Audit Masalah Saat Ini

| Masalah | Root Cause | Dampak |
|---|---|---|
| Markdown gak dirender proper | Custom `renderBody()` parser lemah — gak handle list, table, bold inline | Heading OK, tapi list & bold sering ilang |
| `.md` file WP berantakan | Masih ada artifact WP + format campur aduk | Teks jadi dinding tanpa visual break |
| Gak ada reading progress | User gak tau posisi scroll | Bounce rate tinggi |
| Gak ada table of contents | Artikel panjang, gak ada navigasi internal | User overwhelmed |
| CTA cuma di footer | Gak ada CTA mid-content | Konversi rendah |
| Font size statis di mobile | `prose-sm` aja, gak fluid | Terlalu kecil di HP kecil |

---

## 🎯 Target UI/UX

### Prinsip: "Medium.com meets Indonesian ecommerce"

| Elemen | Target |
|---|---|
| **Lebar baca** | `max-w-2xl` (672px) — optimal 65-75 karakter per baris |
| **Font size** | `text-[17px] sm:text-[18px]` — lebih besar dari default |
| **Line height** | `leading-[1.8]` — lega, gak dempet |
| **Spacing** | `space-y-5` antar paragraf, `mt-10` sebelum H2 |
| **Warna teks** | `#1a1a1a` — bukan `#374151` (gray-700), lebih kontras |
| **CTA mid-content** | Banner kecil di antara section — "Butuh Asxone? 💬 Order WA" |
| **Reading progress** | Bar hijau tipis di atas — progress scroll |
| **Table of Contents** | Sticky sidebar di desktop, dropdown di mobile |
| **Bold** | Warna hijau `#15803d` — biar nendang |
| **Blockquote** | Card dengan background hijau muda — buat highlight |
| **List** | Checkmark hijau — bukan bullet default |

---

## 📐 Layout

```
┌──────────────────────────────────┐
│  ▓▓▓▓▓▓▓▓▓▓  (reading progress) │
├──────────────────────────────────┤
│                                  │
│  H1  ─── Title (center)          │
│  P   ─── Description (center)    │
│                                  │
│  📑 Table of Contents            │
│     ├─ Section 1                 │
│     ├─ Section 2                 │
│     └─ Section 3                 │
│                                  │
│  ────────────────────────────    │
│                                  │
│  H2  ─── Heading                 │
│  P   ─── Paragraph               │
│  P   ─── Paragraph               │
│  💬 ┌──────────────────────┐    │  ← CTA mid-content
│     │ Butuh Asxone? Order WA│    │
│     └──────────────────────┘    │
│  P   ─── Paragraph               │
│  BQ  ─── Blockquote              │
│                                  │
│  ────────────────────────────    │
│                                  │
│  H2  ─── FAQ                     │
│  DTLS ─── Accordion              │
│                                  │
│  ────────────────────────────    │
│                                  │
│  💬 ┌──────────────────────┐    │  ← CTA footer
│     │  Hubungi via WhatsApp │    │
│     └──────────────────────┘    │
│                                  │
│  ← Artikel terkait              │
└──────────────────────────────────┘
```

---

## 📦 Task (5 task, ±35 menit)

### Task 1: Replace renderBody with `marked` + Custom Renderer

**File:** `src/pages/blog/[slug].astro` — fungsi `renderBody()`

Ganti parser kustom dengan `marked` + custom renderer:
- Heading → `<h2 class="...">` dengan styling premium
- List → checklist hijau `✓`
- Bold → `<strong class="text-green-800">`
- Blockquote → `<blockquote class="...">`
- Table → responsive scroll wrapper

```js
import { marked } from 'marked';

marked.setOptions({
  gfm: true,
  breaks: true,
});

function renderBody(text) {
  // Bersihkan WP artifacts
  text = text.replace(/By\s*\n+\s*Gink Digital\s*\/\s*\n+\s*\w+ \d+, \d+/g, '');
  
  // Parse + wrap list items dengan checkmark
  let html = marked.parse(text);
  
  // Tambah class ke elemen
  html = html.replace(/<h2/g, '<h2 class="..."');
  html = html.replace(/<ul/g, '<ul class="..."');
  // ...dst
  
  return html;
}
```

### Task 2: CSS Premium Reading — Global Styles

**File:** `src/pages/blog/[slug].astro` — di `<style>` section

```css
/* Article reading experience */
.article-body {
  font-size: 17px;
  line-height: 1.8;
  color: #1a1a1a;
  max-width: 680px;
  margin: 0 auto;
}

/* Heading 2 — signature Fraunces */
.article-body h2 {
  font-family: 'Fraunces', serif;
  font-size: clamp(1.5rem, 3vw, 1.8rem);
  font-weight: 600;
  color: #14532d;
  margin-top: 2.5rem;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #dcfce7;
}

/* Paragraph */
.article-body p {
  margin-bottom: 1.25rem;
}

/* Bold — green accent */
.article-body strong {
  color: #15803d;
  font-weight: 600;
}

/* List — green checkmark */
.article-body ul {
  list-style: none;
  padding-left: 0;
}
.article-body ul li {
  padding-left: 1.75rem;
  position: relative;
  margin-bottom: 0.6rem;
}
.article-body ul li::before {
  content: "✓";
  position: absolute;
  left: 0;
  color: #22c55e;
  font-weight: 700;
}

/* Blockquote — green card */
.article-body blockquote {
  background: #f0fdf4;
  border-left: 4px solid #22c55e;
  border-radius: 0 12px 12px 0;
  padding: 1rem 1.25rem;
  margin: 1.5rem 0;
  font-style: italic;
}

/* Horizontal rule */
.article-body hr {
  border: none;
  height: 1px;
  background: linear-gradient(90deg, transparent, #e2e8f0, transparent);
  margin: 2.5rem 0;
}

/* Table — responsive */
.article-body table {
  width: 100%;
  border-collapse: collapse;
  margin: 1.5rem 0;
  font-size: 0.9rem;
}
.article-body th {
  background: #f0fdf4;
  color: #14532d;
  padding: 0.75rem;
  text-align: left;
  font-weight: 600;
}
.article-body td {
  padding: 0.75rem;
  border-bottom: 1px solid #e2e8f0;
}

/* CTA mid-content */
.article-cta {
  background: linear-gradient(135deg, #166534, #15803d);
  color: white;
  border-radius: 12px;
  padding: 1rem 1.5rem;
  margin: 2rem 0;
  text-align: center;
}
.article-cta a {
  color: white;
  background: #22c55e;
  padding: 0.6rem 1.5rem;
  border-radius: 50px;
  font-weight: 600;
  text-decoration: none;
  display: inline-block;
  margin-top: 0.5rem;
}

@media (max-width: 640px) {
  .article-body { font-size: 16px; }
  .article-body h2 { font-size: 1.3rem; }
}
```

### Task 3: Reading Progress Bar

**File:** `src/pages/blog/[slug].astro` — tambah di `<header>` sebelum artikel

```html
<div id="progress-bar" class="fixed top-0 left-0 h-1 bg-green-500 z-50 transition-all" style="width: 0%"></div>
<script>
  window.addEventListener('scroll', () => {
    const winScroll = document.documentElement.scrollTop;
    const height = document.documentElement.scrollHeight - window.innerHeight;
    document.getElementById('progress-bar').style.width = Math.min(100, (winScroll / height) * 100) + '%';
  });
</script>
```

### Task 4: Table of Contents (Auto-generated)

**File:** `src/pages/blog/[slug].astro` — setelah header, sebelum konten

```html
<nav class="bg-gray-50 rounded-xl p-4 mb-8 text-sm" id="toc">
  <details class="md:hidden">
    <summary class="font-bold cursor-pointer">📑 Daftar Isi</summary>
    <div class="mt-2 space-y-1" id="toc-mobile"></div>
  </details>
  <div class="hidden md:block">
    <div class="font-bold mb-2 text-sm">📑 Daftar Isi</div>
    <div class="space-y-1" id="toc-desktop"></div>
  </div>
</nav>
<script>
  // Auto-generate TOC from h2 tags
  const h2s = document.querySelectorAll('.article-body h2');
  const tocDesktop = document.getElementById('toc-desktop');
  const tocMobile = document.getElementById('toc-mobile');
  h2s.forEach((h2, i) => {
    h2.id = 'section-' + i;
    const link = `<a href="#section-${i}" class="block text-green-700 hover:underline py-0.5">${h2.textContent}</a>`;
    tocDesktop.innerHTML += link;
    tocMobile.innerHTML += link;
  });
</script>
```

### Task 5: Clean Up 14 `.md` Files

**File:** `src/content/blog/*.md` (semua 14)

Bersihkan WP artifacts dari semua file:
- Hapus "By Gink Digital / June 22, 2026"
- Hapus baris kosong berlebihan
- Hapus "💡 Ringkasan" yang redundan
- Hapus "📑 Daftar Isi" bawaan WP (nanti auto-generated)

Script batch:
```bash
for f in src/content/blog/*.md; do
  sed -i '/^By\s*$/d' "$f"
  sed -i '/^Gink Digital/d' "$f"
  # ...etc
done
```

---

## 📊 Target Skor

| Metrik | Sebelum | Sesudah |
|---|---|---|
| Reading comfort | ⚠️ Wall of text | ✅ Bernapas, 17px, 1.8 line-height |
| Markdown rendering | ⚠️ Custom parser lemah | ✅ `marked` + custom renderer |
| Visual hierarchy | ❌ Flat | ✅ H2 border + green accent |
| CTA conversion | ⚠️ 1 di footer | ✅ 2-3 mid-content + footer |
| Navigation | ❌ None | ✅ TOC + progress bar |
| SEO | ✅ Title/meta/URL utuh | ✅ Tetap utuh |

---

## ⏱ Timeline

| Task | Waktu |
|---|---|
| T1: marked + custom renderer | 10 mnt |
| T2: CSS premium reading | 10 mnt |
| T3: Reading progress bar | 5 mnt |
| T4: Auto TOC | 5 mnt |
| T5: Clean 14 .md files | 5 mnt |
| Build + deploy | 3 mnt |
| **Total** | **±38 mnt** |

---

**Setuju eksekusi?**
