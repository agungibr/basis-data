# Tugas Guided - Modul 6: Sistem Basis Data 

## A. Implementasi DDL & Constraints

### 1. Membuat database

``` sql
CREATE DATABASE cafe_senja;
USE cafe_senja;

SELECT DATABASE();
```

### 2. Membuat tabel `pelanggan`

``` sql
CREATE TABLE pelanggan (
    id_pelanggan CHAR(3) PRIMARY KEY,
    nama_pelanggan VARCHAR(50) NOT NULL,
    nomor_hp VARCHAR(15) NOT NULL UNIQUE
);
```

-   `PRIMARY KEY`: identitas unik setiap pelanggan.
-   `NOT NULL`: nama dan nomor HP wajib diisi.
-   `UNIQUE`: nomor HP tidak boleh sama.
-   `VARCHAR` digunakan untuk nomor HP agar angka nol di awal tidak
    hilang.

### 3. Membuat tabel `pembelian_bahan_baku`

``` sql
CREATE TABLE pembelian_bahan_baku (
    id_pembelian CHAR(3) PRIMARY KEY,
    nama_bahan VARCHAR(50) NOT NULL,
    jumlah_kg DECIMAL(5,2) NOT NULL,
    total_harga INT NOT NULL CHECK (total_harga > 0),
    tanggal_pembelian DATE NOT NULL
);
```

-   `DECIMAL(5,2)`: menyimpan jumlah bahan baku dengan dua angka
    desimal.
-   `CHECK (total_harga > 0)`: harga pembelian harus lebih besar dari
    nol.
-   `PRIMARY KEY` dan `NOT NULL` menjaga identitas serta kelengkapan
    data.

### 4. Membuat tabel `penjualan`

``` sql
CREATE TABLE penjualan (
    id_penjualan CHAR(5) PRIMARY KEY,
    id_pelanggan CHAR(3),
    total_bayar INT NOT NULL,
    tanggal_transaksi DATE NOT NULL,

    CONSTRAINT fk_penjualan_pelanggan
    FOREIGN KEY (id_pelanggan)
    REFERENCES pelanggan(id_pelanggan)
    ON DELETE SET NULL
    ON UPDATE CASCADE
);
```

-   `FOREIGN KEY`: menghubungkan penjualan dengan pelanggan melalui
    `id_pelanggan`.
-   `ON DELETE SET NULL`: jika pelanggan dihapus, relasi pada transaksi
    menjadi `NULL`, sehingga riwayat penjualan tetap ada.
-   `ON UPDATE CASCADE`: perubahan ID pelanggan ikut diperbarui pada
    tabel penjualan.

Relasi antartabel: satu pelanggan dapat memiliki banyak transaksi
penjualan (one-to-many).

``` mermaid
erDiagram
    pelanggan ||--o{ penjualan : "memiliki"
    pelanggan {
        CHAR id_pelanggan PK
        VARCHAR nama_pelanggan
        VARCHAR nomor_hp
        VARCHAR email
        ENUM gender
    }
    penjualan {
        CHAR id_penjualan PK
        CHAR id_pelanggan FK
        INT total_bayar
        DATE tanggal_transaksi
    }
```

------------------------------------------------------------------------

## B. Modifikasi & Inspeksi Struktur Tabel dengan DDL

### 1. Menambahkan kolom `email` dan `gender`

``` sql
ALTER TABLE pelanggan
ADD COLUMN email VARCHAR(50),
ADD COLUMN gender ENUM('P', 'W');
```

`email` menampung alamat email hingga 50 karakter. `gender` hanya
menerima nilai `P` (Pria) atau `W` (Wanita).

### 2. Menampilkan seluruh tabel

``` sql
SHOW TABLES;
```

Tabel yang diharapkan: `pelanggan`, `pembelian_bahan_baku`, dan
`penjualan`.

### 3. Menampilkan struktur tabel

``` sql
DESC pelanggan;
DESC pembelian_bahan_baku;
DESC penjualan;

SHOW CREATE TABLE penjualan;
```

Pastikan kolom `email` dan `gender` muncul pada tabel `pelanggan`.
Gunakan `SHOW CREATE TABLE` untuk memeriksa definisi foreign key.

------------------------------------------------------------------------

## C. Pengelolaan Data Transaksi dengan DML

### 1. INSERT - Memasukkan data awal

Masukkan data pelanggan terlebih dahulu sebelum data penjualan karena
`id_pelanggan` pada tabel penjualan merupakan foreign key.

**a. Data pelanggan**

``` sql
INSERT INTO pelanggan
(id_pelanggan, nama_pelanggan, nomor_hp, email, gender)
VALUES
('S01', 'Budi Santoso', '081234567890', 'budi@gmail.com', 'P'),
('S02', 'Siti Aminah', '089876543210', 'siti@gmail.com', 'W'),
('S03', 'Andi Pratama', '085678901234', 'andi@yahoo.com', 'P'),
('S04', 'Dewi Lestari', '087712345678', 'dewi@gmail.com', 'W');
```

**b. Data pembelian bahan baku**

``` sql
INSERT INTO pembelian_bahan_baku
(id_pembelian, nama_bahan, jumlah_kg, total_harga, tanggal_pembelian)
VALUES
('C01', 'Biji Kopi Arabika', 10.50, 1500000, '2026-10-30'),
('C02', 'Susu UHT', 20.00, 360000, '2026-10-30'),
('C03', 'Sirup Vanila', 5.00, 250000, '2026-10-30'),
('C04', 'Bubuk Matcha', 3.00, 450000, '2026-10-30');
```

**c. Data penjualan**

``` sql
INSERT INTO penjualan
(id_penjualan, id_pelanggan, total_bayar, tanggal_transaksi)
VALUES
('PJ001', 'S01', 45000, '2026-10-30'),
('PJ002', 'S02', 75000, '2026-10-30'),
('PJ003', 'S03', 30000, '2026-10-30'),
('PJ004', 'S04', 90000, '2026-10-30');
```

### 2. SELECT - Menampilkan seluruh data

``` sql
SELECT * FROM pelanggan;
SELECT * FROM pembelian_bahan_baku;
SELECT * FROM penjualan;
```

Gambaran data yang diharapkan:

**Tabel `pelanggan`**

| id_pelanggan | nama_pelanggan | nomor_hp      |
|--------------|----------------|---------------|
| S01          | Budi Santoso   | 081234567890  |
| S02          | Siti Aminah    | 089876543210  |
| S03          | Andi Pratama   | 085678901234  |
| S04          | Dewi Lestari   | 087712345678  |

**Tabel `pembelian_bahan_baku`**

| id_pembelian | nama_bahan        | jumlah_kg | total_harga |
|--------------|-------------------|-----------|-------------|
| C01          | Biji Kopi Arabika |     10.50 |   1.500.000 |
| C02          | Susu UHT          |     20.00 |     360.000 |
| C03          | Sirup Vanila      |      5.00 |     250.000 |
| C04          | Bubuk Matcha      |      3.00 |     450.000 |

**Tabel `penjualan`**

| id_penjualan | id_pelanggan | total_bayar |
|--------------|--------------|-------------|
| PJ001        | S01          |      45.000 |
| PJ002        | S02          |      75.000 |
| PJ003        | S03          |      30.000 |
| PJ004        | S04          |      90.000 |

### 3. UPDATE - Mengubah nomor HP pelanggan

Ubah nomor HP Budi Santoso (ID `S01`):

``` sql
UPDATE pelanggan
SET nomor_hp = '081111222333'
WHERE id_pelanggan = 'S01';
```

Verifikasi:

``` sql
SELECT * FROM pelanggan
WHERE id_pelanggan = 'S01';
```

Nomor HP `S01` seharusnya berubah menjadi `081111222333`.

### 4. DELETE - Menghapus transaksi penjualan

Hapus transaksi dengan ID `PJ004`:

``` sql
DELETE FROM penjualan
WHERE id_penjualan = 'PJ004';
```

Verifikasi:

``` sql
SELECT * FROM penjualan;
```

Transaksi `PJ004` seharusnya tidak lagi muncul, sedangkan `PJ001`,
`PJ002`, dan `PJ003` tetap tersedia.

------------------------------------------------------------------------

## D. Pengujian Transaksi dengan TCL

Bagian ini menguji `START TRANSACTION`, `SAVEPOINT`, `ROLLBACK`, dan
`COMMIT`. Tujuannya adalah membatalkan penghapusan data dengan rollback
ke savepoint, lalu menyimpan hasil transaksi.

### 1. Memulai transaksi

``` sql
START TRANSACTION;
```

Perubahan dalam transaksi belum disimpan secara permanen sampai `COMMIT`
dijalankan.

### 2. Menambahkan data pembelian bahan baku

``` sql
INSERT INTO pembelian_bahan_baku
(id_pembelian, nama_bahan, jumlah_kg, total_harga, tanggal_pembelian)
VALUES
('C05', 'Gula Aren', 15.00, 300000, '2026-10-31');
```

Periksa data:

``` sql
SELECT * FROM pembelian_bahan_baku;
```

Tabel seharusnya memiliki lima baris, yaitu `C01` sampai `C05`.

### 3. Membuat SAVEPOINT

``` sql
SAVEPOINT sp1;
```

Savepoint membuat titik acuan di dalam transaksi. Perubahan setelah
titik ini dapat dibatalkan tanpa membatalkan seluruh transaksi.

### 4. Menghapus seluruh data pembelian bahan baku

``` sql
DELETE FROM pembelian_bahan_baku;
```

Perintah ini menghapus seluruh baris data, tetapi tidak menghapus
struktur tabel.

``` sql
SELECT * FROM pembelian_bahan_baku;
```

Hasil yang diharapkan: tabel kosong (0 rows).

### 5. Melakukan ROLLBACK TO SAVEPOINT

``` sql
ROLLBACK TO SAVEPOINT sp1;
```

Perintah ini membatalkan perubahan setelah `SAVEPOINT sp1`. Penghapusan
dibatalkan sehingga data `C01` sampai `C05`, termasuk Gula Aren, kembali
tersedia.

Verifikasi:

``` sql
SELECT * FROM pembelian_bahan_baku;
```

| id_pembelian | nama_bahan        | jumlah_kg | total_harga |
|--------------|-------------------|-----------|-------------|
| C01          | Biji Kopi Arabika |     10.50 |   1.500.000 |
| C02          | Susu UHT          |     20.00 |     360.000 |
| C03          | Sirup Vanila      |      5.00 |     250.000 |
| C04          | Bubuk Matcha      |      3.00 |     450.000 |
| C05          | Gula Aren         |     15.00 |     300.000 |

### 6. Menyimpan transaksi secara permanen

``` sql
COMMIT;
```

`COMMIT` menyimpan perubahan transaksi secara permanen. Karena
penghapusan sudah dibatalkan dengan `ROLLBACK TO SAVEPOINT`, kelima data
pembelian tetap tersimpan.

### 7. Verifikasi akhir

``` sql
SELECT * FROM pembelian_bahan_baku;
```

Pastikan kelima data pembelian bahan baku masih tersedia, termasuk data
Gula Aren.

------------------------------------------------------------------------

## E. Urutan

| No | Kegiatan                 | Perintah                                        |
|-----|--------------------------|-------------------------------------------------|
| 1   | Membuat database         | `CREATE DATABASE`, `USE`                        |
| 2   | Membuat tabel pelanggan  | `CREATE TABLE pelanggan`                        |
| 3   | Membuat tabel bahan baku | `CREATE TABLE pembelian_bahan_baku`             |
| 4   | Membuat tabel penjualan  | `CREATE TABLE penjualan`                        |
| 5   | Modifikasi tabel         | `ALTER TABLE pelanggan`                         |
| 6   | Inspeksi tabel           | `SHOW TABLES` dan ketiga `DESC`                 |
| 7   | Insert pelanggan         | `INSERT INTO pelanggan`                         |
| 8   | Insert bahan baku        | `INSERT INTO pembelian_bahan_baku`              |
| 9   | Insert penjualan         | `INSERT INTO penjualan`                         |
| 10  | Menampilkan data         | Ketiga `SELECT *`                               |
| 11  | Update pelanggan         | `UPDATE pelanggan` dan hasil SELECT             |
| 12  | Delete penjualan         | `DELETE FROM penjualan` dan hasil SELECT        |
| 13  | Memulai transaksi        | `START TRANSACTION`                             |
| 14  | Menambahkan Gula Aren    | `INSERT INTO pembelian_bahan_baku`              |
| 15  | Membuat savepoint        | `SAVEPOINT sp1`                                 |
| 16  | Menghapus seluruh data   | `DELETE FROM pembelian_bahan_baku`              |
| 17  | Rollback                 | `ROLLBACK TO SAVEPOINT sp1`                     |
| 18  | Commit                   | `COMMIT`                                        |
| 19  | Verifikasi akhir         | `SELECT * FROM pembelian_bahan_baku`            |
