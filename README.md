# 🖐️ Finger Explosion: Game AI Berbasis Gerakan

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![AI: MediaPipe](https://img.shields.io/badge/AI-MediaPipe-brightgreen)](https://mediapipe.dev/)
[![Library: jQuery](https://img.shields.io/badge/Library-jQuery-orange)](https://jquery.com/)

Game web bertema neon yang seru di mana jari telunjukmu berubah menjadi senjata! Menggunakan **MediaPipe Hands AI**, pemain akan bertarung dalam uji kecepatan dan presisi secara *real-time* langsung di browser.

---

## 🚀 Fitur Utama

* **AI Motion Tracking**: Pelacakan tangan *real-time* menggunakan MediaPipe (tidak butuh stik/controller!).
* **Mode Single & Dual Player**: Duel lawan teman (PVP) atau tantang **Bot AI** yang lincah.
* **Estetika Neon & Cyberpunk**: UI futuristik dengan animasi *canvas* yang halus dan efek **Screen Shake** (guncangan layar) saat ledakan.
* **Sistem Partikel**: Efek ledakan partikel dinamis setiap kali target berhasil dihancurkan.
* **HUD Interaktif**: Skor *real-time*, pengaturan target kemenangan, dan sistem *countdown* arcade.
* **Efek Suara Dinamis**: *Feedback* audio saat memotong target, memulai game, dan saat menang.

---

## 🛠️ Dibuat Menggunakan

| Teknologi | Kegunaan |
| :--- | :--- |
| **MediaPipe Hands** | Deteksi titik koordinat jari tangan yang akurat. |
| **HTML5 Canvas** | Rendering game, efek guncangan, dan sistem partikel. |
| **jQuery** | Manipulasi UI dan transisi menu yang *seamless*. |
| **Google Fonts** | Menggunakan font **Orbitron** untuk tampilan visual ala sci-fi. |

---

## 📖 Panduan Pemakaian (User Guide)

### 1. Persiapan Perangkat
* Pastikan laptop/komputer memiliki **Webcam** yang berfungsi.
* Pencahayaan ruangan harus cukup agar AI bisa mendeteksi tangan dengan jelas.

### 2. Memulai Game
1.  Buka file `index.html` di browser (Disarankan Google Chrome).
2.  Klik **"Allow/Izinkan"** saat browser meminta akses kamera.
3.  Pilih kamera pada menu dropdown dan masukkan **Target Skor**.
4.  Centang **"LAWAN BOT (AI)"** untuk mode Single Player.
5.  Klik tombol **"ENTER ARENA"**.

### 3. Cara Bermain
* **Pemain 1 (Biru)**: Meledakkan bola biru di sisi kanan layar.
* **Pemain 2 / Bot (Pink)**: Meledakkan bola pink di sisi kiri layar.
* Gunakan **Jari Telunjuk** untuk menyentuh bola-bola yang muncul.

---

## 📜 Kredit & Atribusi Suara

Proyek ini menggunakan aset suara dari **Pixabay**:
* **Sfx Slice**: *Sword Blade Slicing* oleh Pixabay.
* **Sfx Start**: *Arcade Countdown* oleh Pixabay.
* **Sfx Over**: *Winning* oleh Pixabay.

## ⚖️ Lisensi

Proyek ini menggunakan lisensi **MIT**. Kamu bebas untuk menggunakan, memodifikasi, dan membagikan ulang kode ini.
