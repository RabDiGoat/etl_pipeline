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
  Hapus duplikasi dilakukan supaya memastikan bahwa setiap baris / data masing-masing memiliki transaksi yang unik (dalam artian 1 baris melambangkan 1 transaksi saja) menggunakan orders = orders.drop_duplicates().
  Hasil penghapusan duplikasi mengubah 130 data menjadi 120 data.
- Langkah 2: Isi missing values
  Setelah dilakukan penghapusan duplikasi, tahapan berikutnya adalah pengisian missing values. Pengisian ini dilakukan dengan tujuan semua data dapat terisi sehingga data transaksi yang sebenarnya ada namun belum lengkap tidak akan terhapus. pada tahapan berikutnya menggunakan kode berikut:
orders['customer_email'] = orders['customer_email'].fillna('unknown@placeholder.com') untuk mengisi email,
median_harga = orders['total_harga'].median() untuk menentukan median harga
orders['total_harga'] = orders['total_harga'].fillna(median_harga) untuk mengisi harga menggunakan median.
  Missing value berupa 20 email dan 5 harga yang kosong kini sudah terisi dengan format:
  Untuk email : unknown@placeholder.com.
  Untuk harga : (terisi dengan median harga).
- Langkah 3: Hapus harga yang bernilai negatif menggunakan orders = orders[orders['total_harga'] >= 0]
  Harga negatif akan menyebabkan data menjadi error, sehingga harus dihapus dan hanya ditampilkan harga yang bernilai positif. Hal ini akan mengubah banyaknya data yang sebelumnya 120 menjadi 115.
- Langkah 4: Standarkan format tanggal dan teks unik.
  Setelah semua data terisi dan bernilai logis, hal yang dilakukan selanjutnya adalah merapikan data supaya konisten.
  Untuk tanggal: Karena formatnya masih tercampur-campur, maka format tanggal ini akan diubah menjadi format yyyy-mm-dd mengikuti standar internasional.
  orders['tanggal_order'] = pd.to_datetime(orders['tanggal_order'], format='mixed')
  Untuk channel dan kota: Format penulisan akan menggunakan huruf kapital di huruf pertamanya saja, dan seterusnya menggunakan huruf kecil.
  orders['channel'] = orders['channel'].str.strip().str.lower().str.replace(' ', '_')
  orders['kota'] = orders['kota'].str.strip().str.title()
  
- Langkah 5: Buat kolom baru: bulan dan kategori
  Setelah semua data terisi dengan konsisten, selanjutnya dapat membuat tambahan kolom berupa bulan kategori suatu transaksi menggunakan kode berikut:
  orders['bulan'] = orders['tanggal_order'].dt.month_name()
orders['kategori_harga'] = np.where(
    orders['total_harga'] < 500000, 'kecil',
    np.where(orders['total_harga'] <= 2000000, 'sedang', 'besar')
)
  Hal ini dilakukan untuk mengetahui hal-hal berikut:
  Untuk bulan transaksi, hal ini dilakukan supaya dapat mengetahui pola transaksi per bulan yang mungkin terjadi sehingga mungkin dapat menemukan pola dalam suatu transaksi.
  Untuk kategori transaksi, hal ini dapat dilakukan untuk menetapkan standar besar/kecilnya suatu transaksi berdasarkan harga supaya dapat disampaikan dengan lebih jelas mengenai besarnya transaksi.
  
## 4. Load
- Tujuan: Supaya data transaksi bersih dapat disimpan dan dapat digunakan oleh Data Analyst maupun Data Scientist untuk penelitian selanjutnya mengenai apa yang harus dilakukan dari data yang sudah bersih tersebut.
- Format output: Dalam exercise ini, data disimpan dalam bentuk .csv. Ada 2 jenis laporan yang disimpan dari data transaksi ini, yaitu laporan data bersih dan summary (kesimpulan).
- 
## 5. Orchestration
- Tool: Apache Airflow
- Schedule: Proses ETL data akan berjalan otomatis setiap pukul 06:00 WIB setiap harinya
- DAG flow: Ketika AirFlow dijalankan, maka secara otmatis akan melakukan ETL dari sumber data yang telah ada. Data diextract, kemudian akan ditransformasi. Karena program ini berjalan otomatis, maka setelah transformasi, akan dicek terlebih dahulu apakah data sudah bersifat konsisten atau belum. Jika sudah, maka data dapat langsung diload dan dibuat laporan. Namun jika masih belum valid, maka perlu dilakukan proses ETL ulang dengan membenahi pipelinenya terlebih dahulu.

## 6. Error Handling
- Skenario 1: File tidak ditemukan
  Solusi:
  Apabila suatu file tidak ditemukan, coba untuk retry sebanyak 3 kali. Jika masih belum bisa, maka hentikan proses ETL untuk sementara, upload ulang file, dan lakukan proses ETL lagi.
- Skenario 2: Validasi data gagal
  Solusi:
  Jika terdapat kegagalan dalam validasi data (misalnya masih ditemukan duplikat, adanya missing values, format yang tidak konsisten), maka kemungkinan besar terdapat suatu kesalahan dalam proses ETL. Maka yang dapat dilakukan adalah coba hentikan sementara proses ETL, coba benahi apa yang salah dalam prosesnya, lalu coba jalankan lagi.

## 7. Monitoring
- Bagaimana cara tahu pipeline sukses?
  Suatu pipeline dikatakan sukses jika berhasil melalui semua langkah dalam cleaning data.
- Bagaimana cara tahu data berkualitas?
  Suatu data dapat dikatakan berkualitas apabila pipeline sukses melalui semua langkah dalam proses ETL serta hasil data menunjukkan kekonsistenan dan terisinya semua data tanpa duplikat.
