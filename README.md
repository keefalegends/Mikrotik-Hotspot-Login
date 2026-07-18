# 🌐 MikroTik Hotspot Login Page - Jarkom

Tampilan halaman login dan status koneksi hotspot MikroTik dengan desain modern dark theme bergaya Winbox/RouterOS. Dibuat dengan HTML, CSS, dan JavaScript murni — tanpa framework, tanpa dependensi eksternal selain Google Fonts.

![RouterOS](https://img.shields.io/badge/RouterOS-7.x-orange?style=flat-square&logo=mikrotik)
![HTML](https://img.shields.io/badge/HTML-5-red?style=flat-square&logo=html5&logoColor=white)
![CSS](https://img.shields.io/badge/CSS-3-blue?style=flat-square&logo=css3)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

## 📸 Preview

| Login Page | Status Page | Logout Page |
|---|---|---|
| `login.html` | `status.html` | `logout.html` |
| Form autentikasi user hotspot | Status sesi & total data terpakai | Ringkasan sesi setelah koneksi diakhiri |

---

## 📁 Struktur File

```
/
├── login.html     # Halaman login hotspot
├── status.html    # Halaman status setelah login berhasil
├── logout.html    # Halaman setelah user berhasil logout
└── README.md
```

> **Penamaan file wajib sesuai** agar dikenali RouterOS secara otomatis.

---

## ✨ Fitur

### `login.html`
- Form submit ke `$(link-login-only)` — variabel MikroTik untuk autentikasi asli
- Input username & password dengan ikon
- Toggle show/hide password
- Loading spinner saat proses submit
- Pesan error `$(error)` langsung diinjeksi oleh RouterOS beserta auto-translate ke Bahasa Indonesia
- Checkbox "Ingat saya"
- Ripple effect pada tombol LOGIN

### `status.html`
- Semua data real dari variabel template MikroTik
- Timer sesi berjalan dari nilai `$(uptime)`
- Bar progress total Download & Upload menggunakan `$(bytes-out)` & `$(bytes-in)` proporsional terhadap kuota
- Sub-label persentase pemakaian data
- 4 stat card: Durasi, IP Address, Share Users, Limit Speed
- Tombol Logout ke endpoint `/logout` MikroTik

### `logout.html`
- Ringkasan sesi setelah pengguna melakukan logout
- Menampilkan Durasi, IP Address, MAC Address, Total Download, dan Upload
- Tombol untuk Login Kembali ke jaringan hotspot

### Desain
- Dark theme konsisten bergaya Winbox
- Font: Rajdhani + Share Tech Mono + Exo 2
- Animated grid background
- Glow effect pada elemen aktif
- Fully responsive

---

## 🚀 Cara Deploy ke MikroTik

### 1. Persiapkan File

Pastikan file berikut sudah tersedia di direktori lokal kamu: `login.html`, `status.html`, dan `logout.html`.

### 2. Upload via Winbox (Drag & Drop)

1. Buka **Winbox** dan hubungkan ke MikroTik kamu
2. Klik menu **Files** di sidebar kiri
3. Cari dan buka folder **`hotspot`**
4. Buka folder penyimpanan file di komputer kamu
5. **Drag & Drop** `login.html`, `status.html`, dan `logout.html` ke dalam folder `hotspot` di jendela Winbox
6. Jika muncul konfirmasi overwrite → pilih **Yes / Overwrite**

### 3. Cek HTML Directory

Pastikan profil hotspot mengarah ke folder yang benar:

```
IP → Hotspot → Server Profiles → hsprof1 → klik 2x
  Lihat field "HTML Directory" → pastikan isinya: hotspot
```

### 4. Konfigurasi Hotspot Lengkap (Praktik 9)

**a. DHCP Client di Ether1**
```
IP → DHCP Client → + → Interface: ether1
```

**b. DNS**
```
IP → DNS → Servers: 8.8.8.8, 192.168.200.1
             ☑ Allow Remote Request
```

**c. NAT Masquerade**
```
IP → Firewall → NAT → +
  Chain        : srcnat
  Out Interface: ether1
  Action       : masquerade
```

**d. IP Address di Ether3**
```
IP → Addresses → +
  Address  : 192.168.17.1/24
  Interface: ether3
```

**e. DHCP Server di Ether3**
```
IP → DHCP Server → DHCP Setup → ether3 → Next (sampai selesai)
```

**f. Setup Hotspot**
```
IP → Hotspot → Hotspot Setup
  Interface          : ether3
  Local Address      : 192.168.17.1/24
  ☑ Masquerade Network
  Address Pool       : 192.168.17.2-192.168.17.254
  Certificate        : none
  SMTP Server        : 0.0.0.0
  DNS                : 8.8.8.8, 192.168.200.1
  DNS Name           : tjkt.keefa.net
  Local Hotspot User : admin
  Password           : (isi password)
```

**g. Server Profile**
```
IP → Hotspot → Server Profiles → hsprof1 → Login
  ☑ HTTP CHAP
  ☑ HTTP PAP
```

**h. User Profile (dengan limit)**
```
IP → Hotspot → User Profiles → +
  Name            : user hotspot 3M
  Share Users     : 1
  Rate Limit rx/tx: 3M/3M
```

**i. Buat User**
```
IP → Hotspot → Users → +
  Name    : keefa
  Password: keefa
  Profile : user hotspot 3M
```

### 5. Test Tanpa Logout

Untuk preview halaman login tanpa harus logout dari jaringan, buka di browser:
```
http://192.168.17.1/login
```

---

## 🔧 Variabel Template MikroTik

RouterOS secara otomatis mengganti variabel berikut saat halaman dirender:

| Variabel | Digunakan di | Keterangan |
|---|---|---|
| `$(link-login-only)` | `login.html` | URL endpoint autentikasi hotspot |
| `$(link-orig)` | `login.html` | URL tujuan asal sebelum redirect ke login |
| `$(username)` | `status.html`, `logout.html` | Nama user yang sedang login |
| `$(address)` | `status.html`, `logout.html` | IP address client |
| `$(uptime)` | `status.html`, `logout.html` | Durasi sesi aktif (format: `0d0h0m0s`) |
| `$(bytes-out)` | `status.html` | Total bytes dikirim dari router (download oleh client) |
| `$(bytes-in)` | `status.html` | Total bytes diterima oleh router (upload dari client) |
| `$(bytes-out-nice)`| `status.html`, `logout.html`| Total bytes download dengan format yang mudah dibaca |
| `$(bytes-in-nice)`| `status.html`, `logout.html`| Total bytes upload dengan format yang mudah dibaca |
| `$(mac)` | `logout.html` | MAC address client |
| `$(session-time-left)` | — | Sisa waktu sesi |

> Saat file dibuka langsung di browser (bukan via MikroTik), variabel `$(...)` belum dirender — ini normal. Tampilannya baru real setelah file masuk ke RouterOS.

---

## ⚙️ Kustomisasi

### Ganti kuota bar progress (default 1 GB)

Di `status.html`, cari dan ubah baris berikut:

```js
const QUOTA_BYTES = 1 * 1024 * 1024 * 1024; // 1 GB
// Contoh 500 MB:
// const QUOTA_BYTES = 0.5 * 1024 * 1024 * 1024;
```

### Ganti DNS Name

Cari teks `tjkt.keefa.net` di semua file HTML dan ganti sesuai DNS Name yang kamu set di Hotspot Setup.

---

## 📝 Lisensi

MIT License — bebas digunakan dan dimodifikasi.
