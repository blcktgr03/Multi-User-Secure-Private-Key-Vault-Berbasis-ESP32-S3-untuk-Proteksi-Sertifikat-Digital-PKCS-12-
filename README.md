# Multi-User-Secure-Private-Key-Vault-Berbasis-ESP32-S3-untuk-Proteksi-Sertifikat-Digital-PKCS-12-
NOTES!!!!:Proyek ini masih dijalakan secara lokal
Mini HSM adalah proyek Hardware Security Module berbasis ESP32-S3 untuk menyimpan private key secara aman dan melakukan tanda tangan digital tanpa mengekspos kunci ke komputer. Sistem terdiri dari firmware ESP32, TTP Server, dan Signing App untuk provisioning sertifikat serta penandatanganan dokumen PDF.
# ESP32_HSM

Mini HSM berbasis ESP32-S3 untuk eksperimen penyimpanan identitas digital,
sertifikat, dan layanan tanda tangan digital dengan private key yang tidak
diekspor kembali dari perangkat.

## Struktur Proyek

```text
ESP32_HSM
|
|-- ESP32_HSM.ino
|-- config.h
|-- serial_manager.h
|-- serial_manager.cpp
|-- storage_manager.h
|-- storage_manager.cpp
|-- command_manager.h
|-- command_manager.cpp
`-- README.md
```

## Pembagian Modul

- `ESP32_HSM.ino`: entry point firmware Arduino.
- `config.h`: konfigurasi global perangkat.
- `serial_manager.*`: komunikasi command-response melalui Serial.
- `storage_manager.*`: penyimpanan identitas ke NVS/Preferences.
- `command_manager.*`: parsing dan routing command dari host.

## Command Awal

```text
PING
STATUS
SECURITY_STATUS
HELP
REGISTER fingerprint_id|private_key_pem_escaped|certificate_pem_escaped
GET_CERT fingerprint_id
SIGN fingerprint_id|sha256_hex
DELETE fingerprint_id
```

Command `SIGN` membaca private key yang sudah disimpan melalui `REGISTER`, lalu
membuat RSA-SHA256 signature dan mengirim hasilnya sebagai Base64.

## Arah Pengembangan

1. Tambahkan autentikasi fingerprint sebelum `SIGN` dan `DELETE`.
2. Tambahkan validasi signature di aplikasi host.
3. Perkuat provisioning key dari host ke ESP32-S3 dengan session key atau kanal
   komunikasi yang diautentikasi.
4. Aktifkan fitur keamanan ESP32-S3 seperti Flash Encryption, Secure Boot, dan
   proteksi eFuse pada tahap deployment.

Panduan deployment keamanan ada di:

```text
..\SECURITY_DEPLOYMENT.md
```
# Mini HSM TTP Server

Tampilan Dashboard Third Trust Party
<img width="1918" height="859" alt="image" src="https://github.com/user-attachments/assets/d67cf212-2b58-42f1-b35e-dff314d7b1cd" />

TTP sederhana berbasis Flask untuk registrasi user, generate key pair, membuat
sertifikat digital, export file PKCS#12 `.p12`, dan provisioning private key
serta sertifikat ke ESP32 melalui USB Serial.

## Menjalankan

```powershell
cd C:\Users\L E N O V O\Documents\Arduino\mini_hsm\ttp_server
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Buka browser:

```text
http://127.0.0.1:5000
```

## Alur

1. Isi form registrasi user.
2. Masukkan password PKCS#12 minimal 8 karakter.
3. Klik `Generate`.
4. Server membuat private key, public key, sertifikat digital, dan file `.p12`.
5. Hubungkan ESP32 ke laptop via USB dan pastikan firmware ESP32_HSM sudah berjalan.
6. Pilih port USB ESP32.
7. Klik `Kirim ke ESP & Download`.

Saat tombol tersebut ditekan, TTP akan:

1. Membuka file `.p12` menggunakan password.
2. Mengekstrak private key dan sertifikat.
3. Mengirim command `PING` ke ESP32.
4. Mengirim command `REGISTER user_<id>|private_key|certificate` ke ESP32.
5. Jika ESP membalas `OK IDENTITY_REGISTERED`, file `.p12` diunduh.

## File Lokal

- Database SQLite: `data/ttp.sqlite3`
- CA private key: `ca/ca_private_key.pem`
- CA certificate: `ca/ca_certificate.pem`
- File user `.p12`: `issued/user_<id>.p12`

# Mini HSM Signing App
<img width="1918" height="817" alt="image" src="https://github.com/user-attachments/assets/16cb3ac7-0d1b-4663-9775-d4ce4127a412" />

Aplikasi Flask sederhana untuk upload PDF, mengambil sertifikat dari ESP32_HSM
melalui USB Serial, mengirim hash dokumen ke ESP32 untuk ditandatangani, dan
mengunduh PDF yang sudah ditandatangani.

## Menjalankan

```powershell
cd "C:\Users\L E N O V O\Documents\Arduino\mini_hsm\signing_app"
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Buka:

```text
http://127.0.0.1:5001
```

## Alur

1. Daftar akun Signing App, lalu masuk dengan username dan password.
2. Pastikan identity sudah diprovision dari `ttp_server` ke ESP32, misalnya `user_1`.
3. Tutup Serial Monitor Arduino IDE.
4. Upload dokumen PDF yang akan ditandatangani.
5. Isi Signer ID sesuai data di ESP32, misalnya `user_1`.
6. Pilih port USB ESP32.
7. Klik `Tanda Tangani PDF`.
8. Download PDF yang sudah ditandatangani.

Catatan: jika certificate CA lokal belum dipercaya oleh PDF viewer, status
signature bisa muncul sebagai `Unknown Signature`. Itu normal untuk CA lokal.


