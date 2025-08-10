# Operator EXISTS

Dalam PostgreSQL, operator `EXISTS` digunakan untuk memeriksa apakah sebuah _subquery_ mengembalikan setidaknya satu baris data. Operator `EXISTS` akan menghasilkan nilai _true_ jika _subquery_ tersebut mengembalikan satu atau lebih baris dan _false_ jika tidak ada baris yang dikembalikan.

Berikut adalah sintaksis dari operator `EXISTS`:

```sql
EXISTS (subquery)
```

Untuk membalikkan hasil dari operator `EXISTS`, kita dapat menggunakan operator `NOT`:

```sql
NOT EXISTS (subquery)
```

Biasanya, kita akan menggunakan operator `EXISTS` di dalam klausa `WHERE` pada pernyataan `SELECT`, `UPDATE` atau `DELETE`.

## Operator EXISTS di dalam Pernyataan SELECT

Contoh berikut menggunakan operator `EXISTS` untuk mencari semua produk yang disimpan di setidaknya satu gudang:

```sql
SELECT product_name
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM warehouse w
    WHERE w.product_id = p.product_id
);
```

Keluaran:

```bash
       product_name
---------------------------
 Sony Xperia 1 VI
 Samsung Galaxy Z Fold 5
 Samsung Galaxy Tab S9
 Apple iPad Pro 12.9
 Samsung Galaxy Buds Pro 2
 Apple Watch Series 9
 LG OLED TV C3
 Sony Bravia XR A95K
 LG G3 OLED
 Sony HT-A7000 Soundbar
 Dell XPS 15
 HP Spectre x360
 Lenovo ThinkPad X1 Carbon
 Apple iMac 24"
```

Pada contoh ini, _subquery_ memeriksa apakah tabel `inventories` memiliki setidaknya satu baris dengan `product_id` yang sama dengan `product_id` di tabel `products`:

```sql
SELECT
  1
FROM
  inventories i
WHERE
  i.product_id = p.product_id
```

Jika _subquery_ mengembalikan satu atau lebih baris, operator `EXISTS` akan menghasilkan nilai _true_, sehingga _outer query_ akan memasukkan produk tersebut ke dalam himpunan hasil (_result set_).

## Operator EXISTS di dalam Pernyataan UPDATE

Pernyataan berikut akan menaikkan harga semua produk yang berada di gudang dengan `warehouse_id` bernilai 1 sebesar 5%:

```sql
UPDATE products p
SET
  price = price * 1.05
WHERE
  EXISTS (
    SELECT
      1
    FROM
      inventories i
    WHERE
      i.product_id = p.product_id
      AND i.warehouse_id = 1
  ) 
RETURNING *;
```

Keluaran:

```bash
 product_id |        product_name        |  price  | brand_id | category_id
------------+----------------------------+---------+----------+-------------
          1 | Samsung Galaxy S24         | 1049.99 |        1 |           1
          4 | Xiaomi Mi 14               |  839.99 |        4 |           2
          7 | Apple iPhone 15 Pro Max    | 1364.99 |        2 |           3
         10 | Apple AirPods Pro 3        |  262.49 |        2 |           5
         13 | Samsung Galaxy Watch 6     |  367.49 |        1 |           6
         16 | Samsung QN900C Neo QLED    | 3149.99 |        1 |           8
         19 | Bose SoundLink Max         |  419.99 |        7 |           9
         22 | Microsoft Surface Laptop 5 | 1364.99 |        9 |          11
         25 | Dell Inspiron 27           | 1049.99 |        7 |          12
```

Pada contoh ini, _subquery_ memeriksa apakah tabel `inventories` memiliki setidaknya satu baris dengan `product_id` yang sama seperti `product_id` di tabel `products` dan `warehouse_id` bernilai 1.

Jika _subquery_ menghasilkan setidaknya satu baris, operator `EXISTS` akan mengembalikan nilai _true_ dan pernyataan `UPDATE` akan memperbarui harga produk tersebut.

`SELECT 1` di dalam _subquery_ bersama operator `EXISTS` adalah praktik yang umum digunakan. Hal ini karena `EXISTS` hanya memeriksa keberadaan baris, bukan nilai yang dikembalikan. Jadi, angka `1` di sini hanyalah nilai _dummy_ yang digunakan untuk efisiensi dan kejelasan, tanpa memengaruhi hasil evaluasi. Angka `1` tersebut berfungsi sebagai _placeholder_ dan tidak memengaruhi logika _query_. Selain itu, penggunaan `SELECT 1` lebih efisien karena menghindari beban tambahan untuk mengambil data aktual dari tabel, mengingat tujuan `EXISTS` hanyalah memeriksa apakah ada baris yang memenuhi kondisi.

## Operator EXISTS di dalam Pernyataan DELETE

Pertama, tambahkan dua baris baru ke dalam tabel `products`:

```sql
INSERT INTO
  products (product_name, price, safety_stock, gross_weight, brand_id, category_id)
VALUES
  ('Samsung Galaxy S25', 999.99, 10, 0.36, 1, 1),
  ('Apple iPhone 17', 1299.99, 20, 0.41, 2, 1);
```

Kedua, hapus produk yang tidak disimpan di gudang mana pun:

```sql
DELETE FROM products p
WHERE
  NOT EXISTS (
    SELECT
      1
    FROM
      inventories i
    WHERE
      i.product_id = p.product_id
  ) 
RETURNING *;
```

Keluaran:

```bash
 product_id |    product_name    |  price  | brand_id | category_id
------------+--------------------+---------+----------+-------------
         26 | Samsung Galaxy S25 |  999.99 |        1 |           1
         27 | Apple iPhone 17    | 1299.99 |        2 |           1
```

Pada contoh ini, _subquery_ memeriksa apakah tabel `inventories` memiliki baris dengan `product_id` yang sama seperti di tabel `products`.

`NOT EXISTS` akan bernilai _true_ jika _subquery_ tidak mengembalikan baris apa pun, sehingga pernyataan `DELETE` akan menghapus produk tersebut dari tabel.

> "PostgreSQL EXISTS Operator." PostgreSQL Tutorial, 22 Jan. 2025, www.pgtutorial.com/postgresql-tutorial/postgresql-exists/. Accessed 10 Aug. 2025.