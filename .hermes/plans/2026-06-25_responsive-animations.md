# Plan: Tokotani Homepage — Responsive + Animations

> **Status:** ⏳ Review, baru eksekusi

**Goal:** Bikin homepage Tokotani smooth di mobile + animasi subtle yang ningkatin konversi.

**File:** `src/pages/index.astro` + `src/layouts/Layout.astro` (tambah CSS animasi global)

---

## 📊 Audit Sekarang

| Section | Mobile | Animasi | Masalah |
|---|---|---|---|
| Hero | ✅ OK | ❌ | H1 agak kecil di HP kecil (320px) |
| Product image | ⚠️ | ❌ | -mt-8 bikin overlap di mobile |
| Konten article | ✅ OK | ❌ | Flat — gak ada visual break antar section |
| Produk card | ✅ OK | ❌ | Hover doang, gak ada reveal |
| Area grid | ✅ OK (2 col) | ❌ | Flat |
| Lokasi (owner photo) | ✅ OK | ❌ | Flat |
| FAQ | ⚠️ | ❌ | 3 item panjang — HP scroll lama |
| CTA section | ✅ OK | ❌ | Statis |

---

## 🎯 Rencana: 5 Task (+-30 menit)

### Task 1: Global Animation CSS (Layout.astro)

**File:** `src/layouts/Layout.astro` — tambah di `<style is:global>`

**Yang ditambah:**
```css
/* Scroll reveal */
.reveal {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.6s ease-out, transform 0.6s ease-out;
}
.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}

/* Hover lift */
.hover-lift {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.hover-lift:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0,0,0,0.08);
}

/* Pulse CTA */
@keyframes pulse-cta {
  0%, 100% { box-shadow: 0 4px 16px rgba(22,163,74,0.3); }
  50% { box-shadow: 0 8px 32px rgba(22,163,74,0.5); }
}
.pulse-cta {
  animation: pulse-cta 2.5s ease-in-out infinite;
}

/* Mobile safe area */
@media (max-width: 640px) {
  .mobile-px { padding-left: 1rem; padding-right: 1rem; }
}
```

**Tambahan JS di akhir Layout (sebelum </body>):**
```html
<script>
  // Scroll reveal
  const revealObserver = new IntersectionObserver((entries) => {
    entries.forEach(e => { if(e.isIntersecting) e.target.classList.add('visible'); });
  }, { threshold: 0.15 });
  document.querySelectorAll('.reveal').forEach(el => revealObserver.observe(el));
</script>
```

**Verifikasi:** Buka halaman, scroll — setiap `.reveal` muncul dengan fade-up.

---

### Task 2: Hero Mobile Fix (index.astro)

**File:** `src/pages/index.astro` — baris 14-31

**Perubahan:**
```astro
<section class="bg-gradient-to-br from-green-800 via-green-700 to-emerald-800 text-white py-12 sm:py-16 md:py-24 px-4 text-center reveal">
  <div class="max-w-3xl mx-auto">
    <h1 class="text-2xl sm:text-3xl md:text-5xl font-bold mb-4 leading-tight">
      Jual Asxone 138 SL di Kalimantan
    </h1>
    <!-- ... -->
```

**Key changes:**
- `py-16 md:py-24` → `py-12 sm:py-16 md:py-24` (lebih kecil di mobile)
- Tambah class `reveal` untuk animasi masuk
- `text-3xl md:text-5xl` → `text-2xl sm:text-3xl md:text-5xl` (fluid)

---

### Task 3: Product Card + Area Grid Animation (index.astro)

**File:** `src/pages/index.astro` — baris 58-67 (produk), 89-103 (area grid)

**Produk card:**
```astro
<div class="bg-white border rounded-xl p-6 hover-lift transition reveal">
```

**Area grid:**
```astro
<div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-2 sm:gap-3 not-prose text-sm reveal">
```
- `grid md:grid-cols-3` → `grid-cols-2 sm:grid-cols-3 md:grid-cols-4` (selalu minimal 2 kolom, gak pernah 1)
- 12 item → 2 col mobile = 6 baris (OK), 3 col tablet, 4 col desktop
- Tambah `reveal` + `hover-lift` di tiap link area

**Setiap link area:**
```astro
<a href="..." class="... hover-lift reveal">
```

---

### Task 4: FAQ Accordion + Owner Photo (index.astro)

**File:** `src/pages/index.astro` — baris 123-137 (FAQ), 107-110 (owner)

**FAQ jadi accordion (mobile friendly):**
```astro
<div class="space-y-2 reveal">
  <details class="bg-gray-50 rounded-xl p-4 group">
    <summary class="font-bold cursor-pointer list-none flex justify-between items-center">
      Dimana bisa membeli Asxone 138 SL di Kalimantan?
      <span class="text-green-600 text-lg transition group-open:rotate-45">+</span>
    </summary>
    <p class="text-gray-600 text-sm mt-3">Anda bisa membeli langsung di Toko Tani Green Al Fatih...</p>
  </details>
  <!-- ulangi untuk 2 FAQ lainnya -->
</div>
```

**Owner photo:**
```astro
<img src="/images/tokotani-owner.webp" alt="..." class="w-full max-w-xs sm:max-w-md mx-auto rounded-xl shadow-md my-4 reveal" loading="lazy" />
```
- `max-w-md` → `max-w-xs sm:max-w-md` (lebih kecil di mobile, gak ganggu layout)

---

### Task 5: CTA Section Polish (index.astro)

**File:** `src/pages/index.astro` — baris 148-153

**Perubahan:**
```astro
<section class="py-12 sm:py-16 md:py-20 px-4 bg-gradient-to-r from-green-700 to-emerald-700 text-white text-center reveal">
  <h2 class="text-2xl sm:text-3xl font-bold mb-4">Siap Panen Lebih Baik?</h2>
  <p class="text-lg sm:text-xl text-green-100 mb-8 max-w-xl mx-auto">Hubungi kami sekarang untuk info harga dan pemesanan. Gratis konsultasi!</p>
  <a href="https://wa.me/6285251988382" class="inline-block bg-white text-green-800 px-8 sm:px-10 py-3 sm:py-4 rounded-full text-base sm:text-lg font-bold hover:bg-green-50 transition shadow-lg pulse-cta">
    💬 Chat WhatsApp Sekarang
  </a>
</section>
```

---

## 📊 Sebelum vs Sesudah

| Aspek | Sebelum | Sesudah |
|---|---|---|
| Hero font | text-3xl (statis) | text-2xl sm:text-3xl md:text-5xl (fluid) |
| Product image overlap | -mt-8 (fixed) | Dihilangkan, ganti margin normal |
| Area grid mobile | 1 kolom (grid) | 2 kolom (grid-cols-2) |
| FAQ | 3 item panjang | Accordion `<details>` — 1 item terbuka per klik |
| Owner photo | max-w-md | max-w-xs sm:max-w-md |
| Animasi | 0 | Scroll reveal di SEMUA section + hover lift + pulse CTA |
| Mobile CTA | Statis | pulse-cta + ukuran fluid |

---

## ⚠️ Batas Dharuriyat

```
✅ Animasi = Tahsiniyat (baju) — aman
✅ Grid columns = Hajiyat (kulit) — gak ubah konten
✅ FAQ accordion = Hajiyat — konten teks TETAP SAMA, cuma bungkus <details>
❌ JANGAN ubah teks konten
❌ JANGAN ubah URL/link
❌ JANGAN tambah/hapus section
```

---

## ⏱ Timeline

| Task | Waktu |
|---|---|
| T1: Global animation CSS + JS | 5 mnt |
| T2: Hero mobile fix | 5 mnt |
| T3: Card + grid animation | 5 mnt |
| T4: FAQ accordion + owner fix | 5 mnt |
| T5: CTA polish | 5 mnt |
| Build + deploy | 3 mnt |
| **Total** | **±28 mnt** |

---

**Setuju eksekusi?**
