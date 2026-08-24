# Panduan Deploy ke cPanel — Kusuma Agro Frontend (dashboard admin)

Dashboard admin gabungan untuk mengelola tiga properti web: Kusuma Agro, Kusuma ID, dan Kusuma Industri. Seluruh aplikasi ada dalam **satu file `Login.html`** — statis, tanpa PHP, tanpa database.

> **Deploy paling akhir**, setelah tiga backend lain hidup. Dashboard ini tidak punya data sendiri; semuanya diambil dari mereka.

---

## 0. Pertimbangan penting: ini pintu masuk admin

Meski cuma satu file HTML, di sinilah semua admin masuk untuk mengelola konten. Dua hal yang tidak boleh dilewat:

| Hal | Kenapa |
|---|---|
| **HTTPS wajib** | Halaman ini mengirim username & password saat login, lalu membawa token di header `Authorization`. Tanpa HTTPS semuanya melintas sebagai teks polos |
| **Jangan taruh di domain publik yang mudah ditebak** | Pertimbangkan subdomain terpisah (`dashboard.kusuma.id`) dan, kalau hosting mendukung, tambahkan proteksi ekstra seperti **Directory Privacy** cPanel atau pembatasan IP |

## 1. Upload

```bash
git clone https://github.com/kusumatech-org/kusuma-agro-frontend.git
```

Salin `Login.html` ke document root domain dashboard.

**Disarankan ganti namanya jadi `index.html`** supaya bisa dibuka lewat `https://dashboard.kusuma.id/` saja, tanpa perlu mengetik nama file.

Tidak ada build step dan tidak ada dependency yang perlu di-install — Bootstrap dan SweetAlert2 diambil dari CDN.

## 2. PHP version

Tidak relevan — tidak ada file PHP sama sekali.

## 3. 🔴 WAJIB: ganti 3 URL backend di `Login.html`

Semuanya masih menunjuk domain development `.test`. Kalau tidak diganti, dashboard terbuka tapi **login gagal dan semua panel kosong**.

| Baris | Menunjuk ke | Ganti jadi |
|---|---|---|
| ~2393 | `http://kusuma-agro-backend.test/index.php/api` | `https://api-agro.kusuma.id/index.php/api` |
| ~2396 | `http://kusuma-agro-backend.test/index.php/api` | `https://api-agro.kusuma.id/index.php/api` |
| ~2503 | `http://kusuma-id-admin.test/api` | `https://www.kusuma.id/api` |
| ~2806 | `http://kusuma-industri.test` | `https://industri.kusuma.id` |

Nomor baris bisa bergeser kalau file pernah diedit — lebih aman cari string `.test` di seluruh file dan pastikan **tidak ada yang tersisa**.

Semua harus **`https`**. Kalau dashboard di-HTTPS tapi memanggil API lewat HTTP, browser memblokirnya sebagai *mixed content* dan panel akan kosong tanpa pesan error yang jelas.

## 4. HTTPS

Aktifkan lewat cPanel → **SSL/TLS Status**. Lihat bagian 0 — ini bukan opsional untuk halaman ini.

## 5. Pastikan CORS mengizinkan domain dashboard ini

Dashboard memanggil dua API dari domain berbeda, jadi keduanya harus mengizinkan origin dashboard:

- `kusuma-agro-backend` → lihat `kusuma-agro-backend/DEPLOY.md` bagian 8
- `kusuma-id-admin` → lihat `kusuma-id-admin/DEPLOY.md` bagian 8

Ganti `Access-Control-Allow-Origin: *` di keduanya menjadi domain dashboard yang sebenarnya.

## 6. Panel Kusuma Industri (iframe) — uji khusus

Berbeda dari dua panel lain yang memakai `fetch`, panel Industri menampilkan halaman admin `kusuma-industri` lewat **`<iframe>`**.

> ⚠️ Kalau dashboard dan `kusuma-industri` berada di **domain berbeda**, browser modern bisa memblokir cookie sesi di dalam iframe (kebijakan SameSite). Gejalanya: panel Industri terus meminta login padahal kredensialnya benar.
>
> Pencegahan termudah: taruh keduanya di **subdomain yang sama** (`dashboard.kusuma.id` dan `industri.kusuma.id`). Detail penanganannya ada di `kusuma-industri/DEPLOY.md` bagian 9.
>
> **Uji ini segera setelah domain production siap.**

## 7. Verifikasi setelah live

Buka dashboard, lalu **DevTools → tab Network**:

- Tidak ada request ke `*.test` (tanda masih ada URL terlewat).
- Tidak ada error CORS.
- Tidak ada peringatan *mixed content*.

Lalu uji alurnya:
1. **Login** pakai akun admin asli → berhasil masuk dashboard.
2. **Panel Kusuma Agro** → daftar paket tampil; coba edit satu data → tersimpan.
3. **Panel Kusuma ID** → data tampil; coba edit → tersimpan. (Ini membuktikan token dari `kusuma-agro-backend` diterima juga oleh `kusuma-id-admin` — skema login terpadunya jalan.)
4. **Panel Kusuma Industri** → iframe tampil dan tetap login (lihat bagian 6).
5. **Logout** → kembali ke halaman login, dan data tidak bisa diakses lagi tanpa login ulang.

Kalau langkah 3 gagal dengan 401 padahal langkah 2 berhasil, kemungkinan besar header `Authorization` tidak sampai ke server `kusuma-id-admin`. Cek `CGIPassAuth On` di konfigurasi web server-nya — Apache menahan header itu dari CGI secara default.

## 8. Sisa yang belum dirapikan

- Placeholder `NAME@KUSUMA.CO.ID` di kolom username form login — sisa UI lama. Username sebenarnya **bukan** format email. Kosmetik saja, tidak mengganggu fungsi.
- Semua library (Bootstrap, SweetAlert2, Google Fonts) diambil dari CDN tanpa salinan lokal. Kalau CDN sedang bermasalah, tampilan dashboard rusak.
- File tunggal 5.299 baris dengan >150 fungsi global — sulit dipelihara, kandidat refactor jangka panjang.

---

Konteks lengkap semua project ada di `kusuma-id-admin/HANDOFF.md` (bagian 8 khusus project ini).
