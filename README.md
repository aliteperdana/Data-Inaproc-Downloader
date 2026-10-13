<img width="434" height="165" alt="image" src="https://github.com/user-attachments/assets/68486626-6f57-4182-bec9-903c3239de58" />

# Data Inaproc Downloader

Aplikasi baris perintah berbasis Python untuk mengunduh data dari halaman dashboard [Data Inaproc](https://data.inaproc.id/) secara otomatis. Cukup tambahkan tautan ke `links.txt`, jalankan skrip, dan hasilnya tersimpan sebagai CSV dan XLSX dengan nama yang rapi serta log proses.

<img width="214" height="87" alt="image" src="https://github.com/user-attachments/assets/9c546adb-2e2d-4488-ad59-86fcb007ed4b" />
<img width="1903" height="908" alt="image" src="https://github.com/user-attachments/assets/2ffe11c1-e5a7-4316-b6da-06c406484809" />

## Fitur

- Membaca banyak tautan dari satu berkas (`links.txt`), tanpa mengubah kode.
- Membuka halaman dengan Chrome headless dan menekan tombol **Download CSV**.
- Menamai berkas berdasarkan halaman dan waktu sinkronisasi data (badge "Data Tersinkron" di halaman).
- Tidak pernah menimpa berkas lama: nama yang sudah dipakai diberi akhiran `_2`, `_3`, dst., sehingga semua versi bisa dibandingkan.
- Mengonversi CSV ke XLSX (header tebal dan dibekukan, lebar kolom menyesuaikan, kolom berisi "kode" disimpan sebagai teks).
- Menulis log baru untuk setiap eksekusi di folder `Logs`.
- Jika satu tautan gagal, tautan lain tetap diproses.
- Dapat dijalankan terjadwal secara otomatis (Task Scheduler di Windows, cron di Linux/macOS).

## Persyaratan

- Python 3.9 atau lebih baru
- Google Chrome (versi stabil) terpasang. Wajib dipasang terpisah, lihat langkah di bagian Instalasi
- Koneksi internet (termasuk saat pertama kali, untuk mengunduh driver Chrome otomatis)

## Instalasi

### 1. Pasang Google Chrome (wajib)

Aplikasi mengendalikan browser Chrome, dan Chrome **tidak ikut terpasang** oleh `pip`. Tanpa Chrome, aplikasi berhenti dengan galat `cannot find Chrome binary`. Driver Chrome (`chromedriver`) tidak perlu dipasang manual karena diunduh otomatis.

**Windows / macOS:** unduh dan pasang dari <https://www.google.com/chrome/>.

**Ubuntu / Debian (termasuk server tanpa layar):**

```bash
# Pastikan arsitekturnya amd64
dpkg --print-architecture

# Unduh dan pasang Google Chrome Stable
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

# Pastikan terpasang
google-chrome --version
```

Catatan untuk Linux:

- Server tidak memerlukan layar atau desktop karena aplikasi memakai mode headless.
- Pasang lewat `apt install ./nama.deb` (bukan `dpkg -i`) agar dependensinya ikut terpasang.
- Jangan memakai `chromium-browser` dari Ubuntu: paket itu berupa snap dan sering bermasalah dengan Selenium.
- Google Chrome resmi hanya tersedia untuk amd64. Pada arm64 diperlukan Chromium non-snap dan pengaturan `options.binary_location` di fungsi `buat_driver()`.

### 2. Pasang Python dan dependensi

Pada Ubuntu/Debian, pasang dulu Python dan alat pendukungnya:

```bash
sudo apt install python3 python3-pip python3-venv
```

Lalu:

```bash
git clone <url-repositori-anda>
cd <nama-folder-repositori>

# (opsional) buat virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Linux/macOS (gunakan python3 bila perintah python tidak ada)

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

| Kondisi               | Contoh hasil                                                 |
| --------------------- | ------------------------------------------------------------ |
| Nama otomatis         | `afirmasi-pdn_2026_D376_Realisasi_sinkron_20261013_0227.csv` |
| Nama kustom           | `Afirmasi_PDN_Ringkasan_sinkron_20261013_0227.csv`           |
| Nama sudah ada        | `..._sinkron_20261013_0227_2.csv` (dan `.xlsx` pasangannya)  |
| Badge tidak ditemukan | waktu eksekusi dipakai sebagai pengganti waktu sinkron       |

Nama otomatis dibentuk dari nama halaman, `tahun`, `instansi`, dan `view_table` (jika ada) pada tautan.

## Menjalankan Otomatis (Terjadwal)

Aplikasi dapat dijalankan secara berkala tanpa campur tangan. Semua folder (`Downloads`, `Logs`) dihitung dari lokasi `Downloader.py`, jadi hasilnya sama walau dijalankan oleh penjadwal dari folder kerja mana pun.

Prinsip yang berlaku untuk semua penjadwal:

- Gunakan **path absolut** untuk Python dan `Downloader.py`. Jika memakai virtual environment, pakai Python di dalamnya agar dependensi ditemukan.
- Komputer harus menyala dan terhubung internet pada jam yang dijadwalkan.
- Hasil tiap eksekusi dapat dicek di folder `Logs`. Kode keluar `0` berarti semua berhasil, `2` berarti ada tautan yang gagal, `3` berarti Chrome gagal dibuka.

### Windows (Task Scheduler)

**Lewat PowerShell** (ganti path sesuai lokasi proyek Anda):

```powershell
$folder = "D:\Aplikasi\Data Inaproc Downloader"

$aksi = New-ScheduledTaskAction `
    -Execute "$folder\venv\Scripts\python.exe" `
    -Argument "`"$folder\Downloader.py`"" `
    -WorkingDirectory $folder

$pemicu = New-ScheduledTaskTrigger -Daily -At 7:00AM

$pengaturan = New-ScheduledTaskSettingsSet -StartWhenAvailable

Register-ScheduledTask -TaskName "Data Inaproc Downloader" `
    -Action $aksi -Trigger $pemicu -Settings $pengaturan `
    -Description "Unduh data dashboard secara terjadwal"
```

Jika tidak memakai virtual environment, ganti nilai `-Execute` dengan path `python.exe` hasil perintah `(Get-Command python).Source`. Opsi `-StartWhenAvailable` membuat tugas dijalankan begitu komputer menyala jika jadwalnya terlewat.

Untuk jadwal lain, ubah pemicunya, misalnya mingguan setiap Senin:

```powershell
$pemicu = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 7:00AM
```

**Uji dan kelola:**

```powershell
Start-ScheduledTask -TaskName "Data Inaproc Downloader"       # jalankan sekarang
Get-ScheduledTaskInfo -TaskName "Data Inaproc Downloader"     # lihat hasil terakhir
Unregister-ScheduledTask -TaskName "Data Inaproc Downloader" -Confirm:$false   # hapus
```

**Lewat antarmuka grafis:** buka _Task Scheduler_ > _Create Task_, lalu isi:

1. **General:** beri nama tugas. Centang _Run whether user is logged on or not_ bila ingin tetap berjalan saat Anda tidak masuk.
2. **Triggers:** _New_ > pilih jadwal (harian, mingguan, dll.) dan jam.
3. **Actions:** _New_ > _Start a program_.
   - _Program/script:_ path `python.exe`
   - _Add arguments:_ `"D:\Aplikasi\Data Inaproc Downloader\Downloader.py"`
   - _Start in:_ `D:\Aplikasi\Data Inaproc Downloader`
4. **Settings:** centang _Run task as soon as possible after a scheduled start is missed_.

### Linux / macOS (cron)

Buka daftar cron:

```bash
crontab -e
```

Tambahkan satu baris (ganti path sesuai lokasi proyek Anda), contoh setiap hari pukul 07.00:

```cron
0 7 * * * /home/pengguna/Data-Inaproc-Downloader/venv/bin/python /home/pengguna/Data-Inaproc-Downloader/Downloader.py
```

Format jadwal cron: `menit jam tanggal bulan hari-dalam-minggu`.

| Jadwal                        | Penulisan      |
| ----------------------------- | -------------- |
| Setiap hari pukul 07.00       | `0 7 * * *`    |
| Setiap Senin pukul 07.00      | `0 7 * * 1`    |
| Setiap 6 jam                  | `0 */6 * * *`  |
| Setiap hari kerja pukul 08.30 | `30 8 * * 1-5` |

Lihat jadwal yang aktif dengan `crontab -l`. Cron berjalan dengan lingkungan yang minimal, sehingga path absolut wajib dipakai. Kalau tugas tidak berjalan, periksa log sistem (`grep CRON /var/log/syslog` di Debian/Ubuntu) dan folder `Logs` proyek.

### Catatan untuk eksekusi terjadwal

- Setiap eksekusi menambah berkas baru di `Downloads` dan `Logs` dan tidak ada penghapusan otomatis. Bersihkan berkas lama secara berkala agar penyimpanan tidak penuh.
- Gunakan frekuensi yang wajar. Data di halaman hanya berubah saat disinkronkan, jadi unduhan yang terlalu sering akan menghasilkan berkas yang isinya sama.
- Pertama kali dijalankan, driver Chrome diunduh otomatis sehingga membutuhkan internet.

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

| Pengaturan        | Fungsi                               | Bawaan |
| ----------------- | ------------------------------------ | ------ |
| `TIMEOUT_HALAMAN` | Batas tunggu elemen halaman (detik)  | `30`   |
| `TIMEOUT_UNDUHAN` | Batas tunggu unduhan selesai (detik) | `120`  |
| `KONVERSI_XLSX`   | Buat file `.xlsx` dari setiap CSV    | `True` |
| `SIMPAN_CSV`      | Simpan CSV setelah dikonversi        | `True` |

## Kode Keluar

| Kode | Arti                                                          |
| ---- | ------------------------------------------------------------- |
| `0`  | Semua tautan berhasil                                         |
| `1`  | Tidak ada tautan untuk diproses                               |
| `2`  | Ada tautan yang gagal (lihat log)                             |
| `3`  | Browser Chrome gagal dibuka (biasanya Chrome belum terpasang) |

## Pemecahan Masalah

| Gejala                                              | Kemungkinan penyebab                                     | Tindakan                                                                                                         |
| --------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Folder `Downloads` kosong                           | Tombol salah diklik atau situs memblokir mode headless   | Periksa log; coba jalankan tanpa `--headless`; hapus baris `--user-agent`                                        |
| Timeout menunggu unduhan                            | Data besar atau server lambat                            | Naikkan `TIMEOUT_UNDUHAN`                                                                                        |
| Badge tidak ditemukan                               | Teks atau struktur badge di halaman berubah              | Berkas tetap tersimpan; perbarui selektor badge di `proses_satu_link`                                            |
| Konversi XLSX gagal                                 | Format CSV tidak terduga                                 | CSV tetap tersimpan; lihat jejak galat di log                                                                    |
| Galat `cannot find Chrome binary` (kode keluar `3`) | Google Chrome belum terpasang                            | Pasang Chrome sesuai bagian Instalasi, lalu jalankan ulang                                                       |
| Galat driver hanya mendukung versi Chrome tertentu  | Driver di cache tidak cocok dengan Chrome yang terpasang | Hapus cache driver (`rm -rf ~/.wdm` di Linux/macOS, folder `%USERPROFILE%\.wdm` di Windows), lalu jalankan ulang |
| Driver Chrome gagal diunduh                         | Tidak ada akses internet                                 | Periksa koneksi atau pasang driver secara manual                                                                 |

## Catatan Penggunaan

- Peringatan `RequestsDependencyWarning` atau `FutureWarning` tentang `urllib3` dan `chardet` berasal dari ketidakcocokan versi pustaka dan tidak menghentikan aplikasi. Untuk menghilangkannya: `pip install --upgrade requests urllib3 chardet`.

- Aplikasi ini bergantung pada struktur halaman data.inaproc.id (teks tombol **Download CSV** dan badge **Data Tersinkron**). Jika situs berubah, selektor perlu disesuaikan.
- Gunakan secara wajar dan patuhi ketentuan pemakaian situs penyedia data.
- Berkas hasil unduhan dan log sengaja tidak disertakan di repositori (lihat `.gitignore`).

## Lisensi

Belum ditentukan.
