# Multi-User-Secure-Private-Key-Vault-Berbasis-ESP32-S3-untuk-Proteksi-Sertifikat-Digital-PKCS-12-
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
