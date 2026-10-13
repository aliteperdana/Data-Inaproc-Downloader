# Inaproc Data Downloader

Aplikasi baris perintah berbasis Python untuk mengunduh data tabel dari dashboard [data.inaproc.id](https://data.inaproc.id) (misalnya Realisasi RUP dan Afirmasi PDN) secara otomatis. Cukup tambahkan tautan ke `links.txt`, jalankan skrip, dan hasilnya tersimpan sebagai CSV dan XLSX dengan nama yang rapi serta log proses.

## Fitur

- Membaca banyak tautan dari satu berkas (`links.txt`), tanpa mengubah kode.
- Membuka halaman dengan Chrome headless dan menekan tombol **Download CSV**.
- Menamai berkas berdasarkan halaman dan waktu sinkronisasi data (badge "Data Tersinkron" di halaman).
- Tidak pernah menimpa berkas lama: nama yang sudah dipakai diberi akhiran `_2`, `_3`, dst., sehingga semua versi bisa dibandingkan.
- Mengonversi CSV ke XLSX (header tebal dan dibekukan, lebar kolom menyesuaikan, kolom berisi "kode" disimpan sebagai teks).
- Menulis log baru untuk setiap eksekusi di folder `Logs`.
- Jika satu tautan gagal, tautan lain tetap diproses.

## Persyaratan

- Python 3.9 atau lebih baru
- Google Chrome terpasang
- Koneksi internet (termasuk saat pertama kali, untuk mengunduh driver Chrome otomatis)

## Instalasi

```bash
git clone <url-repositori-anda>
cd <nama-folder-repositori>

# (opsional) buat virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Linux/macOS

pip install -r requirements.txt
```

## Penggunaan

1. Isi `links.txt` dengan tautan yang ingin diunduh (lihat format di bawah).
2. Jalankan:

```bash
python Downloader.py
```

3. Hasil ada di folder `Downloads`, log ada di folder `Logs`.

### Format `links.txt`

Satu tautan per baris. Baris kosong dan baris berawalan `#` diabaikan.

```text
# Nama otomatis dari tautan
https://data.inaproc.id/realisasi-rup?tahun=2026&section=detail&jenis_klpd=3&instansi=D376

# Nama kustom: tulis nama, tanda |, lalu tautan
Afirmasi PDN Ringkasan | https://data.inaproc.id/afirmasi-pdn?tahun=2026&jenis=3&instansi=D376
```

### Penamaan berkas

| Kondisi | Contoh hasil |
|---|---|
| Nama otomatis | `afirmasi-pdn_2026_D376_Realisasi_sinkron_20261013_0227.csv` |
| Nama kustom | `Afirmasi_PDN_Ringkasan_sinkron_20261013_0227.csv` |
| Nama sudah ada | `..._sinkron_20261013_0227_2.csv` (dan `.xlsx` pasangannya) |
| Badge tidak ditemukan | waktu eksekusi dipakai sebagai pengganti waktu sinkron |

Nama otomatis dibentuk dari nama halaman, `tahun`, `instansi`, dan `view_table` (jika ada) pada tautan.

## Struktur Folder

```text
.
├── Downloader.py      # aplikasi utama
├── links.txt          # daftar tautan yang diunduh
├── requirements.txt   # dependensi Python
├── Downloads/         # hasil unduhan (dibuat otomatis, tidak masuk Git)
└── Logs/              # log per eksekusi (dibuat otomatis, tidak masuk Git)
```

## Pengaturan

Ubah di bagian atas `Downloader.py`:

| Pengaturan | Fungsi | Bawaan |
|---|---|---|
| `TIMEOUT_HALAMAN` | Batas tunggu elemen halaman (detik) | `30` |
| `TIMEOUT_UNDUHAN` | Batas tunggu unduhan selesai (detik) | `120` |
| `KONVERSI_XLSX` | Buat file `.xlsx` dari setiap CSV | `True` |
| `SIMPAN_CSV` | Simpan CSV setelah dikonversi | `True` |

## Kode Keluar

| Kode | Arti |
|---|---|
| `0` | Semua tautan berhasil |
| `1` | Tidak ada tautan untuk diproses |
| `2` | Ada tautan yang gagal (lihat log) |

## Pemecahan Masalah

| Gejala | Kemungkinan penyebab | Tindakan |
|---|---|---|
| Folder `Downloads` kosong | Tombol salah diklik atau situs memblokir mode headless | Periksa log; coba jalankan tanpa `--headless`; hapus baris `--user-agent` |
| Timeout menunggu unduhan | Data besar atau server lambat | Naikkan `TIMEOUT_UNDUHAN` |
| Badge tidak ditemukan | Teks atau struktur badge di halaman berubah | Berkas tetap tersimpan; perbarui selektor badge di `proses_satu_link` |
| Konversi XLSX gagal | Format CSV tidak terduga | CSV tetap tersimpan; lihat jejak galat di log |
| Driver Chrome gagal diunduh | Tidak ada akses internet | Periksa koneksi atau pasang driver secara manual |

## Catatan Penggunaan

- Aplikasi ini bergantung pada struktur halaman data.inaproc.id (teks tombol **Download CSV** dan badge **Data Tersinkron**). Jika situs berubah, selektor perlu disesuaikan.
- Gunakan secara wajar dan patuhi ketentuan pemakaian situs penyedia data.
- Berkas hasil unduhan dan log sengaja tidak disertakan di repositori (lihat `.gitignore`).

## Lisensi

Belum ditentukan.
