# ETL Pipeline Design: E-Commerce Orders

## 1. Overview
Pipeline ini memproses data transaksi e-commerce harian dalam sebuah toko. Dalam data transaksi tersebut, ditemukan mengenai rincian order_id (id order), product_id (id product), product_name (nama produk), kategori (kategori produk) quantity (banyaknya produk yg ditransaksikan), total_harga (harga transaksi yg dilakukan), tanggal_order (tanggal transaksi terjadi), kota terjadinya transaksi, channel (platform transaksi yang dilalui), status transaksi, dan customer_email.

Data transaksi ini masih memiliki format yang acak-acakan karena diambil dari berbagai sumber sehingga harus dibersihkan menggunakan ETL (Extract, Transform, & Load) supaya data kemudian langsung dapat dianalisis.

## 2. Extract
- Sumber: Data bersumber dari simulasi berupa file raw_orders.csv, sehingga perlu dibangkitkan serta dibaca menggunakan pandas.
- Format: Setelah diextract, data masih memiliki ketidakkonsistenan. Adapun ketidakkonsistenan yang ditemukan antara lain sebagai berikut:
 Terdapat 10 data yang terduplikat.
 20 customer email tidak ada.
 Ada 10 data yang memiliki harga yang tidak logis, dimana 5 di antaranya tidak terisi dan 5 lainnya memiliki harga negatif.
 Format tanggal yang masih tercampur.
 Masih terdapat penamaan channel yang berbeda meskipun isinya sama (contoh: MARKETPLACE vs marketplace).
 Penamaan kota yang juga berbeda meskipun isinya sama (contoh: Medan vs MEDAN vs medan).
- Volume: [jelaskan]

## 3. Transform
- Langkah 1: Hapus duplikasi
  Hapus duplikasi dilakukan supaya memastikan bahwa setiap baris / data masing-masing memiliki transaksi yang unik (dalam artian 1 baris melambangkan 1 transaksi saja) supaya dapat dilakukan tahap berikutnya tanpa ada kebingungan.
  Hasil penghapusan duplikasi mengubah 130 data menjadi 120 data.
- Langkah 2: Isi missing values
  Setelah dilakukan penghapusan duplikasi, tahapan berikutnya adalah pengisian missing values. Pengisian ini dilakukan dengan tujuan semua data dapat terisi sehingga data transaksi yang sebenarnya ada namun belum lengkap tidak akan terhapus. pada tahapan berikutnya. Missing value berupa 20 email dan 5 harga yang kosong kini sudah terisi dengan format:
  Untuk email : unknown@placeholder.com.
  Untuk harga : (terisi dengan median harga).
- Langkah 3: Hapus harga yang bernilai negatif
  Harga negatif akan menyebabkan data menjadi error, sehingga harus dihapus. Hal ini akan mengubah banyaknya data yang sebelumnya 120 menjadi 115.
- Langkah 4: Standarkan format tanggal dan teks unik.
  Setelah semua data terisi dan bernilai logis, hal yang dilakukan selanjutnya adalah merapikan data supaya konisten.
  Untuk tanggal: Karena formatnya masih tercampur-campur, maka format tanggal ini akan diubah menjadi format yyyy-mm-dd mengikuti standar internasional.
  Untuk channel dan kota: Format penulisan akan menggunakan huruf kapital di huruf pertamanya saja, dan seterusnya menggunakan huruf kecil.
- Langkah 5: Buat kolom baru: bulan dan kategori harga
  Setelah semua data terisi dengan konsisten, selanjutnya dapat membuat tambahan kolom berupa bulan kategori suatu transaksi. Hal ini dilakukan untuk mengetahui hal-hal berikut:
  Untuk bulan transaksi, hal ini dilakukan supaya dapat mengetahui pola transaksi per bulan yang mungkin terjadi sehingga mungkin dapat menemukan pola dalam suatu transaksi.
  Untuk kategori transaksi, hal ini dapat dilakukan untuk memilah dan menentukan besar/kecilnya suatu transaksi berdasarkan harga supaya dapat disampaikan dengan lebih jelas mengenai besarnya transaksi.
## 4. Load
- Tujuan: Supaya data transaksi bersih dapat disimpan dan dapat digunakan oleh Data Analyst maupun Data Scientist untuk penelitian selanjutnya mengenai apa yang harus dilakukan dari data yang sudah bersih tersebut.
- Format output: Dalam exercise ini, data disimpan dalam bentuk .csv. Ada 2 jenis laporan yang disimpan dari data transaksi ini, yaitu laporan data bersih dan summary (kesimpulan).


