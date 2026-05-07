# Project-C-Triosik-

![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)
![Database](https://img.shields.io/badge/Database-Relational-blue?style=for-the-badge)

👨‍💻 Anggota Kelompok :
- Muhammad Zaky Arrasyid (25)
- Salatin Nibras Bama Kerti (32)
- Rinzani Fauziah (31)

---

# 🎸 Studio Booking System

**Sistem Booking Studio Musik Offline** - Kiosk berbasis desktop dengan 3 aktor (Guest, Admin, Kasir). Dirancang khusus untuk studio musik yang menginginkan mesin booking mandiri dengan pengalaman pengguna seperti kiosk restoran cepat saji.

## ✨ Fitur Utama

- 🎤 **Guest Booking Tanpa Login** – Pelanggan cukup mengisi nama band, nama customer, dan no HP.
- 🎫 **Nomor Antrian Otomatis** – Setiap pemesanan mendapatkan nomor antrian unik yang direset setiap hari.
- ⏰ **Batas Waktu Bayar 15 Menit** – Sistem otomatis membatalkan (*expired*) booking jika tidak dibayar dalam 15 menit.
- 💰 **Metode Pembayaran** – Mendukung pembayaran Cash (input nominal oleh kasir) & QRIS (verifikasi manual).
- 🕐 **Jam Operasional Fleksibel** – Pengaturan operasional pukul 10:00 - 21:00 dengan maksimal durasi booking 5 jam.
- 👥 **3 Aktor Sistem** – Akses khusus untuk Guest (Kiosk), Admin, dan Kasir.
- 📊 **Laporan & Monitoring** – Laporan harian, cek ketersediaan studio secara real-time, dan pantau antrian.

## 🏗️ Teknologi

- **Database**: Microsoft SQL Server
- **Aplikasi**: C# Desktop (WinForms / WPF) - 
- **Arsitektur**: Offline (Local Network), Tidak membutuhkan koneksi internet.

## 📊 Struktur Database
<img width="1531" height="661" alt="WhatsApp Image 2026-05-06 at 15 20 27" src="https://github.com/user-attachments/assets/aba5cc37-82ae-4815-bf21-069f49d01a74" />



### Tabel Utama

| Tabel | Fungsi | Jumlah Kolom |
|-------|--------|--------------|
| `users` | Akun kredensial admin & kasir | 4 |
| `studio` | Master data studio musik | 4 |
| `booking` | Data pemesanan oleh pelanggan | 15 |
| `pembayaran` | Rekam jejak transaksi kasir | 6 |

## 📋 Detail Tabel

### 1. `users` - Akun Pengguna

| Field | Tipe | Keterangan |
|-------|------|-------------|
| id_user | INT (PK) | Auto increment |
| username | VARCHAR(50) | Unique |
| password | VARCHAR(255) | |
| role | VARCHAR(10) | 'admin' / 'kasir' |

**Data Default:**
| username | password | role |
|----------|----------|------|
| admin | admin123 | admin |
| kasir | kasir123 | kasir |

### 2. `studio` - Data Studio

| Field | Tipe | Keterangan |
|-------|------|-------------|
| id_studio | INT (PK) | Auto increment |
| nama_studio | VARCHAR(100) | |
| harga_per_jam | INT | Dalam Rupiah |
| status_studio | VARCHAR(10) | 'aktif' / 'nonaktif' |

### 3. `booking` - Pemesanan

| Field | Tipe | Keterangan |
|-------|------|-------------|
| id_booking | INT (PK) | Auto increment |
| nomor_antrian | INT | Auto per hari |
| nama_band | VARCHAR(100) | Diisi guest |
| nama_customer | VARCHAR(100) | Diisi guest |
| no_hp | VARCHAR(15) | Diisi guest |
| id_studio | INT (FK) | → studio.id_studio |
| tanggal | DATE | Tanggal booking |
| jam_mulai | TIME | 10:00 - 21:00 |
| jam_selesai | TIME | Max 21:00, min 1 jam, max 5 jam |
| total_harga | INT | Auto hitung |
| status | VARCHAR(10) | 'waiting' / 'paid' / 'expired' / 'cancelled' |
| metode | VARCHAR(10) | 'cash' / 'qris' |
| created_at | DATETIME | Default GETDATE() |
| expired_at | DATETIME | Default 15 menit setelah created_at |

**Constraint:**
- `jam_mulai >= '10:00'`
- `jam_selesai <= '21:00'`
- `DATEDIFF(HOUR, jam_mulai, jam_selesai) BETWEEN 1 AND 5`

### 4. `pembayaran` - Transaksi

| Field | Tipe | Keterangan |
|-------|------|-------------|
| id_pembayaran | INT (PK) | Auto increment |
| id_booking | INT (FK) | → booking.id_booking (UNIQUE) |
| id_kasir | INT (FK) | → users.id_user |
| jumlah_bayar | INT | Total dibayar |
| kembalian | INT | Default 0 |
| waktu_bayar | DATETIME | Default GETDATE() |

## ⚡ Trigger (5 Trigger)

| No | Nama Trigger | Tabel | Event | Fungsi |
|----|--------------|-------|-------|--------|
| 1 | `trg_generate_nomor_antrian` | booking | INSTEAD OF INSERT | Generate nomor antrian otomatis per hari |
| 2 | `trg_hitung_total_harga` | booking | INSTEAD OF INSERT | Hitung total_harga dari jam_mulai, jam_selesai, dan harga_per_jam |
| 3 | `trg_cek_konflik_booking` | booking | INSTEAD OF INSERT | Cek apakah jam yang dipilih sudah dibooking orang lain |
| 4 | `trg_auto_expired` | booking | AFTER INSERT, UPDATE | Ubah status 'waiting' menjadi 'expired' jika melewati expired_at |
| 5 | `trg_update_status_paid` | pembayaran | AFTER INSERT | Ubah status booking menjadi 'paid' setelah pembayaran masuk |

## 📦 Stored Procedure

### `sp_cek_ketersediaan`
Cek apakah suatu studio tersedia di jam tertentu.

**Parameter:**
- `@id_studio INT`
- `@tanggal DATE`
- `@jam_mulai TIME`
- `@jam_selesai TIME`

**Output:** Jumlah konflik (0 = tersedia)

### `sp_laporan_harian`
Melihat laporan booking dan pembayaran per hari.

**Parameter:**
- `@tanggal DATE`

## 👁️ View

### `vw_daftar_antrian`
Menampilkan daftar antrian booking hari ini dengan status display:
- `MENUNGGU BAYAR` - Booking waiting & belum expired
- `EXPIRED` - Booking waiting & sudah lewat 15 menit
- `LUNAS` - Booking sudah paid

## 🚀 Cara Install

### 1. Prasyarat
- SQL Server (2016 atau lebih baru)
- SQL Server Management Studio (SSMS)

### 2. Setup Database

```sql
-- Jalankan script lengkap yang tersedia di file database.sql
-- Script akan:
-- 1. Membuat database StudioBookingDB
-- 2. Membuat semua tabel
-- 3. Memasukkan data awal (admin & kasir, contoh studio)
-- 4. Membuat semua trigger
-- 5. Membuat stored procedure dan view
## 🚀 Alur Kerja (Workflow)

### A. Alur Guest (Kiosk Mode)
1. Guest membuka aplikasi pada mesin kiosk.
2. Memilih studio yang tersedia dan menentukan tanggal.
3. Memilih jam mulai dan jam selesai (Maks. 5 jam).
4. Mengisi identitas: Nama Band, Nama Customer, dan No HP.
5. Memilih metode pembayaran (Cash/QRIS).
6. Sistem mencetak/menampilkan nomor antrian.
7. Pelanggan menuju kasir untuk melakukan pembayaran sebelum batas waktu 15 menit berakhir.

### B. Alur Kasir
1. Kasir melakukan login.
2. Mencari data booking berdasarkan nomor antrian, nama band, atau no HP.
3. Menerima pembayaran (Input nominal untuk cash atau verifikasi manual untuk QRIS).
4. Klik tombol konfirmasi.
5. Sistem mencetak struk fisik.
6. Status booking otomatis berubah menjadi **'paid'**.

### C. Alur Admin
1. Admin melakukan login.
2. **Manajemen Studio**: Menambah, mengedit, atau menonaktifkan studio.
3. **Monitoring**: Melihat semua data booking dengan filter status tertentu.
4. **Kontrol**: Membatalkan booking jika terjadi kendala teknis.
5. **Reporting**: Melihat dan mengekspor laporan pendapatan harian.

## 🛠️ Pengembangan (SQL Snippets)

Gunakan perintah berikut untuk pengecekan database pada SQL Server Management Studio (SSMS):

```sql
USE StudioBookingDB;

-- Cek data pengguna dan studio
SELECT * FROM users;
SELECT * FROM studio;

-- Cek trigger yang aktif pada database
SELECT name, is_instead_of_trigger 
FROM sys.triggers;
