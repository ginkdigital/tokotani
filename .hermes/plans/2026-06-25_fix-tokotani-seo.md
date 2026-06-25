# Plan: Perbaiki Tokotani — Merge Konten SEO + Desain Astro

> **Status:** ⏳ Review dulu, baru eksekusi

**Masalah:** Build Astro baru menghancurkan 14 artikel SEO + 4 halaman konten dari WordPress. URL, meta, konten — hilang semua.

**Goal:** Gabungkan konten SEO asli (dari `tokotani.web.id`) ke dalam desain Astro baru. URL tetap sama, konten tetap utuh, desain jadi premium.

**Waktu:** ±45 menit

---

## 📊 Inventaris Konten yang Hilang

### Halaman (4) — sitemap `page-sitemap.xml`

| URL | Title | Status Baru |
|---|---|---|
| `/` | Jual Asxone 138 SL Kalimantan | ⚠️ Ada tapi konten beda |
| `/produk/` | Produk | ⚠️ Ada tapi konten beda |
| `/tentang-kami/` | Tentang Kami | ⚠️ URL ganti jadi `/tentang` |
| `/kontak/` | Kontak | ⚠️ URL ganti jadi `/kontak` |

### Artikel SEO (15) — sitemap `post-sitemap.xml`

14 artikel "jual asxone 138 SL [kota]" — ini **SEO backbone** situs. Masing-masing target keyword long-tail spesifik per kota. Semua hilang di build baru.

| # | URL Slug | Target Keyword |
|---|---|---|
| 1 | `toko-tani-green-al-fatih-pangkalan-banteng-pusat-herbisida-pupuk-bibit-di-kalimantan-tengah/` | pusat herbisida kalimantan tengah |
| 2 | `toko-tani-terdekat-di-kalimantan-tengah/` | toko tani terdekat |
| 3 | `jual-asxone-138-sl-pangkalan-bun/` | asxone pangkalan bun |
| 4 | `jual-asxone-138-sl-pangkalan-banteng/` | asxone pangkalan banteng |
| 5 | `jual-asxone-138-sl-sukamara/` | asxone sukamara |
| 6 | `jual-asxone-138-sl-kalimantan-tengah/` | asxone kalteng |
| 7 | `jual-asxone-138-sl-kalimantan-barat/` | asxone kalbar |
| 8 | `jual-asxone-138-sl-kalimantan-timur/` | asxone kaltim |
| 9 | `jual-asxone-138-sl-seruyan/` | asxone seruyan |
| 10 | `jual-asxone-138-sl-kalimantan-utara/` | asxone kaltara |
| 11 | `beli-asxone-138-sl-pangkalan-banteng/` | beli asxone |
| 12 | `jual-asxone-138-sl-di-kalimantan-toko-tani-green-al-fatih/` | asxone kalimantan |
| 13 | `jual-asxone-138-sl-kalimantan-selatan/` | asxone kalsel |
| 14 | `jual-asxone-138-sl-sampit/` | asxone sampit |

---

## 🎯 Rencana Eksekusi

### Task 0: Parse konten dari live site (READ-ONLY)

**Objective:** Ambil SEMUA konten dari `tokotani.web.id` tanpa merusak apapun.

**Cara:** `wget --mirror` atau parse via Python — ambil:
- Title + meta description + H1 + body content
- URL slug
- Gambar (sudah ada di `public/images/`)

**Output:** File `content-data.json` berisi semua halaman.

### Task 1: Buat content collection Astro

**Objective:** Setup Astro content collection untuk blog posts.

**Files:**
- Modify: `src/content/config.ts` — schema blog collection
- Create: `src/content/blog/*.md` — 14 file markdown, 1 per artikel
- Create: `src/pages/[slug].astro` — dynamic route untuk artikel

**Isi setiap artikel (.md):**
```markdown
---
title: "Jual Asxone 138 SL di Pangkalan Bun"
description: "Butuh Asxone 138 SL di Pangkalan Bun? Toko Tani Green Al Fatih..."
pubDate: 2026-03-07
category: "produk"
tags: ["asxone", "pangkalan bun", "herbisida"]
---

[KONTEN ASLI DARI WORDPRESS]
```

**Verifikasi:** Semua 14 URL dari sitemap bisa diakses di local dev.

### Task 2: Kembalikan URL halaman statis

**Objective:** URL `/tentang` → `/tentang-kami`, `/kontak` tetap, `/pengiriman` tetap.

**Files:**
- Rename: `src/pages/tentang.astro` → `src/pages/tentang-kami.astro`

### Task 3: Tambah redirect untuk URL yang berubah

**Objective:** Kalau ada URL berubah, tambah redirect biar SEO gak putus.

**Files:**
- Create: `public/_redirects` (Cloudflare Pages format)
```
/tentang /tentang-kami 301
```

### Task 4: Perbaiki homepage — ambil konten asli

**Objective:** Homepage baru harus pakai title + meta + konten dari WordPress asli, bukan bikinan saya.

**File:**
- Modify: `src/pages/index.astro` — inject konten dari WordPress + retain desain baru

### Task 5: Build, push, deploy

**Objective:** Deploy versi final.

```bash
npm run build
git add -A && git commit -m "fix: merge original SEO content with Astro design"
git push origin main
npx wrangler pages deploy dist --project-name=tokotani --branch=main
```

### Task 6: Verifikasi SEO

- [ ] Semua 14 URL bisa diakses di `tokotani.avathur.id`
- [ ] Title + meta description cocok dengan asli WordPress
- [ ] `sitemap.xml` ter-generate oleh Astro
- [ ] Internal linking antar artikel berfungsi
- [ ] Gambar produk tampil di semua halaman

---

## ⚠️ Yang Gak Boleh Terjadi Lagi

- ❌ Jangan ubah URL slug artikel — itu SEO
- ❌ Jangan hapus konten — merge, jangan replace
- ❌ Jangan build dari nol tanpa cek konten asli dulu
- ❌ Jangan deploy sebelum build sukses lokal

---

## 🔧 Prinsip Baru WAKi

```
1. Konversi dulu (waki-convert.py) — ambil SEMUA konten
2. Evaluasi — apa yang bisa di-improve tanpa merusak?
3. Upgrade — desain/casing baru, konten TETAP
4. Deploy — URL + konten same = SEO aman
```

---

## ⏱ Timeline

| Task | Waktu |
|---|---|
| Task 0: Parse konten | 5 menit |
| Task 1: Content collection | 15 menit |
| Task 2-3: URL fix | 5 menit |
| Task 4: Homepage fix | 10 menit |
| Task 5: Build & deploy | 5 menit |
| Task 6: Verifikasi SEO | 5 menit |
| **Total** | **±45 menit** |

---

**Setuju? Atau ada yang mau disesuaikan?**
