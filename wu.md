<div align="center">

# WRITEUP CTF

### Web • OSINT • Reverse Engineering • Miscellaneous

---

**Author**  
Muhammad Ariq Mubarak

**Operating System**  
Kali Linux

**Tools & Technologies**  
Nmap · curl · Python 3 · unzip · file · strings · objdump · Linux Utilities

---

*Documentation of CTF challenges, technical analysis, and problem-solving.*

</div>

# 🗂️ DAFTAR ISI

> Pilih challenge untuk langsung menuju bagian yang ingin dibaca.

1. Web – Arsip Lawang Sewu
2. Web – Pasar Johar
3. Web – Arsip Balaikota
4. OSINT – Tahu Gimbal
5. Reverse Engineering – Gereja Blenduk
6. Reverse Engineering – Manifest
8. OSINT – Kucingan
9. Ringkasan flag dan status challenge

---

# 🌐 1. WEB – ARSIP LAWANG SEWU

## 1.1 Deskripsi Challenge

Challenge Web Arsip Lawang Sewu berjalan pada layanan:

```text
http://103.196.155.224:8101
```

Challenge menggunakan aplikasi arsip sederhana. Tujuan pengerjaan adalah menemukan kelemahan pada aplikasi dan memperoleh flag.

## 1.2 Persiapan

Pastikan target dapat diakses:

```bash
curl -i http://103.196.155.224:8101/
```

Periksa informasi server dan respons HTTP:

```bash
curl -sS -D - -o /dev/null http://103.196.155.224:8101/
```

Lakukan pemeriksaan endpoint umum:

```bash
for path in /robots.txt /login /admin /archive /arsip; do
    echo "===== $path ====="
    curl -sS -i "http://103.196.155.224:8101$path"
done
```

## 1.3 Analisis Kerentanan

Kerentanan yang ditemukan pada challenge ini adalah **Path Traversal**.

Path traversal merupakan kelemahan yang memungkinkan pengguna mengakses file di luar direktori yang seharusnya dapat diakses aplikasi.

Karakter yang sering ditemukan dalam pengujian:

```text
../
```

Karakter tersebut digunakan untuk berpindah ke direktori induk.

Dalam challenge ini, manipulasi jalur file digunakan untuk membaca file yang seharusnya tidak dapat diakses melalui fitur arsip.

## 1.4 Hasil

Challenge berhasil diselesaikan.

**Flag:**

```text
CF2{web_lawang_sewu_e4710b37cd0f26958ead}
```
<img width="1920" height="1016" alt="image" src="https://github.com/user-attachments/assets/9f9298af-b81d-4a0e-804d-2e9b7f56790e" />

---

# 🌐 2. WEB – PASAR JOHAR

## 2.1 Deskripsi Challenge

Target:

```text
http://103.196.155.224:8103
```

Challenge menggunakan aplikasi Pasar Johar. Pengujian dilakukan terhadap mekanisme autentikasi dan pemrosesan input aplikasi.

## 2.2 Identifikasi Aplikasi

Periksa halaman utama:

```bash
curl -i http://103.196.155.224:8103/
```

Periksa respons header:

```bash
curl -sS -D - -o /dev/null http://103.196.155.224:8103/
```

Periksa endpoint yang ditemukan dari halaman aplikasi:

```bash
curl -i http://103.196.155.224:8103/login
```

## 2.3 Analisis Kerentanan

Challenge ini berkaitan dengan dua konsep:

### A. JWT (JSON Web Token)

JWT adalah format token yang digunakan untuk membawa informasi atau klaim autentikasi.

Secara umum JWT memiliki tiga bagian:

```text
HEADER.PAYLOAD.SIGNATURE
```

Bagian payload dapat berisi informasi seperti identitas pengguna atau hak akses.

Pemeriksaan token perlu memperhatikan:

- Algoritma penandatanganan.
- Isi header.
- Isi payload.
- Validitas signature.
- Cara server memverifikasi token.

### B. SSTI (Server-Side Template Injection)

SSTI terjadi ketika input pengguna diproses sebagai ekspresi template oleh server.

Jika input tidak ditangani dengan benar, ekspresi template dapat dievaluasi oleh aplikasi.

Dalam challenge ini, analisis JWT dan SSTI menjadi bagian dari proses mendapatkan akses terhadap fungsi yang diperlukan.

## 2.4 Hasil

Challenge berhasil diselesaikan.

**Flag:**

```text
CF2{web_pasar_johar_4013c0f749b871c9635a}
```
<img width="1920" height="1015" alt="image" src="https://github.com/user-attachments/assets/498bb6c4-77f4-4a74-a548-491cdb727f32" />

---

# 🌐 3. WEB – ARSIP BALAIKOTA

## 3.1 Deskripsi Challenge

Target:

```text
http://103.196.155.224:8102
```

Aplikasi menampilkan halaman arsip dengan beberapa endpoint:

```text
/
 /cari?q=
/login
/api/dokumen
```

Tujuan challenge adalah menemukan kelemahan pada proses pencarian, autentikasi, dan akses dokumen.

## 3.2 Enumerasi Endpoint

Periksa halaman utama:

```bash
curl -i http://103.196.155.224:8102/
```

Periksa endpoint umum:

```bash
for path in /robots.txt /login /staff /admin /archive /arsip /api/dokumen; do
    echo "===== $path ====="
    curl -sS -i "http://103.196.155.224:8102$path"
done
```

Periksa halaman pencarian:

```bash
curl -i "http://103.196.155.224:8102/cari?q=test"
```

Periksa file JavaScript:

```bash
curl -i http://103.196.155.224:8102/static/app.js
```

Periksa source map:

```bash
curl -i http://103.196.155.224:8102/static/app.js.map
```

## 3.3 Analisis SQL Injection

Ditemukan indikasi SQL Injection pada parameter pencarian:

```text
/cari?q=
```

SQL Injection merupakan kerentanan ketika input pengguna memengaruhi query SQL tanpa penanganan yang aman.

Dalam proses challenge, kelemahan tersebut dimanfaatkan untuk memperoleh informasi akun staf.

Informasi yang berhasil ditemukan:

```text
Username : staf.arsip
Password : tahugimbal13
```

## 3.4 Analisis IDOR

IDOR (Insecure Direct Object Reference) adalah kelemahan kontrol akses ketika aplikasi memperbolehkan pengguna mengakses objek hanya dengan mengubah referensi atau ID.

Pada challenge ini ditemukan endpoint:

```text
/ekspor?id=2
```

Pemeriksaan akses dokumen mengungkap informasi signing material yang kemudian digunakan dalam proses analisis token.

## 3.5 Analisis HMAC dan Token

HMAC merupakan mekanisme autentikasi pesan menggunakan secret key.

Informasi signing material yang diperoleh dari kelemahan akses digunakan untuk memahami proses validasi token aplikasi.

Dalam challenge ini, token dibuat ulang menggunakan mekanisme HMAC yang sesuai dengan implementasi aplikasi.

Token tersebut kemudian digunakan untuk mengakses fungsi ekspor arsip.

## 3.6 Status

Kemajuan yang sudah tercatat:

- Endpoint pencarian ditemukan.
- SQL Injection teridentifikasi.
- Kredensial staf ditemukan.
- IDOR pada endpoint ekspor ditemukan.
- Signing material ditemukan.
- Proses pemalsuan token HMAC dilakukan untuk mengakses ekspor arsip.

## 3.7 Hasil

<img width="1920" height="1016" alt="image" src="https://github.com/user-attachments/assets/e2eecfcc-9fa5-4e82-b3ef-d562e6359553" />

---

# 🧭 4. OSINT – TAHU GIMBAL

## 4.1 Deskripsi Challenge

Challenge OSINT ini meminta peserta mengidentifikasi gerai berdasarkan sejumlah petunjuk visual dan informasi direktori.

Petunjuk yang digunakan dalam proses analisis:

- Ulasan pelanggan.
- Nama pemilik.
- Warna gerobak.
- Sektor lokasi.
- Status gerai.
- Nomor gerai.

## 4.2 Pemeriksaan File

Periksa file yang tersedia:

```bash
ls -lah
```

Baca ulasan:

```bash
cat ulasan.txt
```

Baca direktori gerai:

```bash
cat direktori.csv
```

Tampilkan direktori dalam bentuk tabel:

```bash
column -s, -t direktori.csv | less -S
```

## 4.3 Korelasi Informasi

Petunjuk yang berhasil dicocokkan:

| Atribut | Hasil |
|---|---|
| Pemilik | Bambang Tri Laksono |
| Warna gerobak | Hijau |
| Sektor | Timur |
| Nomor gerai | 0427 |
| ID gerai | LP-118 |
| Status | Aktif |

Kunci pencocokan bukan hanya nama pemilik. Warna gerobak, sektor, dan nomor gerai juga digunakan untuk memastikan kandidat yang tepat.

## 4.4 Menyusun Jawaban

Format jawaban:

```text
ID_GERAI|NOMOR_GERAI
```

Nilai yang diperoleh:

```text
LP-118|0427
```

Challenge meminta hash SHA-256 dan mengambil 24 karakter pertama dalam bentuk heksadesimal huruf kecil.

Gunakan Python:

```bash
python3 -c "import hashlib; print(hashlib.sha256(b'LP-118|0427').hexdigest()[:24])"
```

Hasil:

```text
ae8ce48b3354a033921df804
```

## 4.5 Flag

```text
CF2{osint_tahu_gimbal_ae8ce48b3354a033921df804}
```
<img width="1920" height="1015" alt="image" src="https://github.com/user-attachments/assets/a2f7b1cb-f8f5-47eb-b195-65a12196435f" />

---

# ⚙️ 5. REVERSE ENGINEERING – GEREJA BLENDUK

## 5.1 Deskripsi Challenge

Challenge reverse engineering ini menggunakan file:

```text
runtime.py
program.bin
vault.json
```

Program meminta serial dengan panjang delapan karakter.

Tujuan analisis adalah menemukan serial yang memenuhi validasi program dan membuka data yang terenkripsi.

## 5.2 Pemeriksaan File

```bash
ls -lah
```

Periksa tipe file:

```bash
file runtime.py program.bin vault.json
```

Baca source code:

```bash
cat runtime.py
```

Baca data vault:

```bash
cat vault.json
```

Periksa ukuran binary:

```bash
wc -c program.bin
```

## 5.3 Memahami Algoritma Validasi

Program membaca serial sepanjang delapan karakter.

Setiap karakter diubah melalui transformasi:

```python
v = [(x * 197 + 31) & 255 for x in s]
```

Program kemudian memeriksa delapan persamaan modular.

Bentuk pemeriksaannya:

```python
sum(a[8*i+j] * v[j] for j in range(8)) % 257 == a[64+i]
```

Artinya, setiap baris memiliki persamaan linear modular yang harus dipenuhi oleh delapan karakter serial.

## 5.4 Analisis Data Binary

File `program.bin` memiliki format:

```text
BLV5
```

Setelah empat byte pertama, terdapat 72 bilangan 16-bit little-endian.

Source code membacanya menggunakan:

```python
a = struct.unpack('<72H', b[4:])
```

Struktur tersebut terdiri dari:

- 64 nilai koefisien.
- 8 nilai target persamaan.

Persamaan dapat diselesaikan menggunakan pendekatan aljabar modular atau brute force terarah.

## 5.5 Dekripsi Vault

Setelah serial valid, program menghitung SHA-256 dari serial:

```python
k = hashlib.sha256(s).digest()
```

Kemudian membentuk keystream dari SHA-256 berulang dengan counter empat byte big-endian.

```python
z = b''.join(
    hashlib.sha256(k+i.to_bytes(4, 'big')).digest()
    for i in range((len(x)+31)//32)
)
```

Ciphertext dibaca dari `vault.json`:

```python
x = bytes.fromhex(
    json.loads((p/'vault.json').read_text())['ciphertext_hex']
)
```

Dekripsi dilakukan dengan XOR:

```python
plaintext = bytes(a ^ b for a, b in zip(x, z))
```

## 5.6 Hasil

Serial yang berhasil ditemukan:

```text
YLpaZdJg
```

Flag:

```text
CF2{rev_gereja_blenduk_de2fe1fb3466b18bfaaa}
```
<img width="1920" height="1013" alt="image" src="https://github.com/user-attachments/assets/e4cdafae-b461-4aee-a41e-c32ec00c2ff6" />

---

# ⚙️ 6. REVERSE ENGINEERING – MANIFEST

## 6.1 Deskripsi Challenge

Challenge menggunakan executable ELF Linux bernama:

```text
manifest
```

Program meminta serial delapan karakter:

```bash
./manifest SERIAL8
```

Petunjuk menyatakan bahwa validasi menggunakan transformasi blok dengan dependensi antarbagian serial.

Program juga melakukan dekripsi dokumen setelah serial dinyatakan valid.

## 6.2 Pemeriksaan File

```bash
ls -lah
```

Periksa tipe executable:

```bash
file manifest
```

Jika permission executable hilang:

```bash
chmod +x manifest
```

Periksa informasi ELF:

```bash
readelf -h manifest
```

Lihat string yang tertanam:

```bash
strings -a manifest
```

Disassembly:

```bash
objdump -d -Mintel manifest | less
```

Lihat bagian data:

```bash
objdump -s -j .rodata manifest
```

## 6.3 Analisis Algoritma

Program memecah serial delapan byte menjadi dua bagian 32-bit:

```text
L = 4 byte pertama
R = 4 byte berikutnya
```

Transformasi dilakukan sebanyak 16 putaran.

Fungsi rotasi bit:

```python
def rol(x, n):
    return ((x << n) | (x >> (32 - n))) & 0xffffffff
```

Fungsi putaran:

```python
def F(r, k):
    return (
        rol(r ^ k, 5) + rol(r, 13)
    ) & 0xffffffff
```

Transformasi setiap putaran:

```text
L_new = R_old
R_new = L_old XOR F(R_old, key)
```

Nilai akhir yang diperiksa oleh program:

```text
L = 0xbdae4507
R = 0x781b2a2c
```

## 6.4 Membalik Transformasi

Karena transformasi bersifat reversibel, proses dapat dibalik dari putaran terakhir menuju putaran pertama.

Rumus invers:

```text
old_R = new_L
old_L = new_R XOR F(old_R, key)
```

Kunci putaran disimpan pada offset:

```text
0x2080
```

Ciphertext dokumen berada pada offset:

```text
0x2040
```

## 6.5 Script Pemecahan

Buat file:

```bash
nano solve.py
```

Isi dengan:

```python
import struct
from pathlib import Path

MASK32 = 0xffffffff
MASK64 = 0xffffffffffffffff

data = Path("manifest").read_bytes()

def rol(x, n):
    return ((x << n) | (x >> (32 - n))) & MASK32

def F(r, k):
    return (rol(r ^ k, 5) + rol(r, 13)) & MASK32

# Ambil 16 round key
keys = struct.unpack_from("<16I", data, 0x2080)

# Target akhir
L = 0xbdae4507
R = 0x781b2a2c

# Balik seluruh putaran
for k in reversed(keys):
    old_R = L
    old_L = R ^ F(old_R, k)
    L, R = old_L, old_R

# Susun serial
serial = struct.pack("<II", L, R)
print("[+] Serial:", serial.decode("ascii"))

# Verifikasi transformasi maju
l, r = struct.unpack("<II", serial)

for k in keys:
    l, r = r, l ^ F(r, k)

assert l == 0xbdae4507
assert r == 0x781b2a2c

print("[+] Semua putaran valid!")

# FNV-1a 64-bit
h = 0xcbf29ce484222325

for c in serial:
    h = ((h ^ c) * 0x100000001b3) & MASK64

# Ambil ciphertext
cipher = data[0x2040:0x2069]

# Dekripsi menggunakan SplitMix64
state = h
plaintext = bytearray()

for c in cipher:
    state = (state + 0x9e3779b97f4a7c15) & MASK64

    z = state
    z = ((z ^ (z >> 30)) * 0xbf58476d1ce4e5b9) & MASK64
    z = ((z ^ (z >> 27)) * 0x94d049bb133111eb) & MASK64
    z ^= z >> 31

    plaintext.append(c ^ (z & 0xff))

print("[+] Dokumen:")
print(plaintext.decode())
```

Simpan file, kemudian jalankan:

```bash
python3 solve.py
```

## 6.6 Hasil

Serial yang ditemukan:

```text
SKcnbmGE
```

Verifikasi dengan executable:

```bash
./manifest SKcnbmGE
```

Hasil:

```text
CF2{rev_tanjung_mas_7d666448bc5641697c86}
```
<img width="1920" height="1015" alt="image" src="https://github.com/user-attachments/assets/924ce6af-a6fa-4468-8e8d-da567fa3c16b" />

## 6.7 Mengapa Tidak Cukup Mengubah Percabangan Penolakan?

Program tidak hanya memeriksa serial.

Serial juga digunakan sebagai input untuk menghasilkan kunci FNV-1a yang menjadi seed proses dekripsi.

Karena itu, mengubah percabangan validasi saja tidak otomatis menghasilkan dokumen yang benar.

---

# 🧭 7. OSINT – KUCINGAN

## 7.1 Deskripsi Challenge

Petunjuk challenge:

- Foto gerobak diunggah tanpa nama tempat.
- Waktu kamera menggunakan WIB.
- Lokasi gerai menggunakan koordinat WGS84.
- Gerai dapat berganti operator.
- Gerai hanya dianggap aktif jika tanggal foto berada dalam interval:

```text
[from_date, until_date_exclusive)
```

Jawaban harus menggunakan format:

```text
ID_GERAI|OPERATOR|YYYY-MM-DD
```

Kemudian di-hash menggunakan SHA-256 dan diambil 24 karakter pertama.

## 7.2 Data Gerai

File:

```text
gerai.csv
```

Periksa isi file:

```bash
head gerai.csv
```

Tampilkan tabel:

```bash
column -s, -t gerai.csv | less -S
```

Periksa jumlah data:

```bash
wc -l gerai.csv
```

## 7.3 Informasi yang Dibutuhkan

Untuk menyelesaikan challenge, diperlukan:

1. Foto asli.
2. Metadata waktu pengambilan foto.
3. Koordinat GPS foto.
4. Data tanggal aktif gerai.
5. Operator gerai pada tanggal tersebut.

## 8.4 Metode Penyelesaian

Pertama, baca metadata foto menggunakan:

```bash
exiftool foto.jpg
```

Ambil informasi GPS dan waktu:

```bash
exiftool -GPSLatitude -GPSLongitude -DateTimeOriginal foto.jpg
```

Tanggal dan waktu harus diperlakukan sebagai WIB sesuai ketentuan challenge.

Setelah mendapatkan koordinat foto dan tanggal, lakukan:

1. Filter gerai yang aktif pada tanggal foto.
2. Gunakan operator yang berlaku pada tanggal tersebut.
3. Hitung jarak geodesik antara koordinat foto dan koordinat setiap gerai aktif.
4. Pilih gerai dengan jarak paling dekat.
5. Susun jawaban sesuai format yang ditentukan.

## 7.5 Menghitung Hash

Setelah mendapatkan teks jawaban final, misalnya disimpan pada variabel `answer`, gunakan:

```bash
python3 -c "import hashlib; s='ID_GERAI|OPERATOR|YYYY-MM-DD'; print(hashlib.sha256(s.encode('utf-8')).hexdigest()[:24])"
```

## 7.6 Hasil
<img width="1920" height="1015" alt="image" src="https://github.com/user-attachments/assets/b0fd7176-65e6-4781-a5e8-57d967deb8ec" />


---

---

# 📊 8. RINGKASAN HASIL CTF

| No. | Challenge | Status |
|---|---|---|
| 1 | Web – Arsip Lawang Sewu | Solved |
| 2 | Web – Pasar Johar | Solved |
| 3 | Web – Arsip Balaikota | Solved |
| 4 | OSINT – Tahu Gimbal | Solved |
| 5 | Reverse – Gereja Blenduk | Solved |
| 6 | Reverse – Manifest | Solved |
| 7 | OSINT – Kucingan | Solved |

---

---

# 📝 9. PENUTUP

Dari rangkaian challenge ini, terdapat beberapa teknik yang dipelajari:

- **Web Exploitation:** Path Traversal, SQL Injection, IDOR, JWT, dan SSTI.
- **OSINT:** Korelasi informasi, pencarian identitas gerai, dan pencocokan data.
- **Reverse Engineering:** Analisis ELF, disassembly, transformasi blok, aljabar modular, hashing, dan dekripsi.
- **Linux:** Penggunaan curl, unzip, grep, strings, objdump, readelf, dan Python.

Dokumentasi ini mencatat hasil yang telah diperoleh selama proses pengerjaan.
