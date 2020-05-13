Kelas: Big Data

Nama: Raden Bimo Rizki Prayogo

NRP: 0511740000139

# Big Data - Tugas 7

# Local big data Irish meter
## Business Understanding
Workflow ini digunakan untuk melakukan analisa time series terhadap penggunaan listrik lokal di Irlandia. 

Overview Workflow:

![picture](/img/overview.PNG)

## Data Understanding

Data yang digunakan adalah dataset CSV penggungaan listrik lokal di Irlandia. 

Dataset mengandung 1,226,830 baris yang berisi informasi tentang penggunaan listrik suatu pada suatu meteran pada suatu saat. Dataset mengandung 3 kolom yakni:

- meterID, yakni ID dari suatu meteran listrik
- enc_datetime, yakni waktu pembacaan meteran 
- reading, yakni listrik yang digunakan pada saat pembacaan meteran


![picture](/img/dataset.PNG)


## Data Preparation
### Membuat Spark Context
Pertama-tama sebuah context spark perlu disiapkan terlebih dahulu. Spark Context dibuat secara lokal dan menggunakan 2 executor (thread).

![picture](/img/spark_setup.PNG)

![picture](/img/spark_con.PNG)

### Membaca dataset

![picture](/img/read_spark1.PNG)

Isi metanode load data.

![picture](/img/read_spark2.PNG)

Dataset pada csv dibaca dengan file reader. Dataset tersebut dijadikan tabel pada suatu koneksi Hive terlebih dahulu di metanode Load Data. Table Hive yang dihasilkan dijadikan sebuah spark Dataframe dengan node Hive to Spark.

Dataframe yang dihasilkan:

![picture](/img/res_read.PNG)

### Mengektraksi time series

![picture](/img/extraction1.PNG)

Isi Extract date-time attributes:

![picture](/img/extraction2.PNG)


Di metanode Extract date-time attributes kita melakukan konversi timestamp menjadi tanggal dan waktu, kemudian mengektraksi tahun, bulan, minggu, hari pada minggu, weekend atau weekday dari tanggal yang dihasilkan, dan mengekstraksi jam dan segmen hari dari waktu yang dihasilkan.

Query ekstraksi tanggal dan waktu:

![picture](/img/extraction_date_time.PNG)

Hasil ekstraksi tanggal dan waktu:

![picture](/img/extraction_date_time_res.PNG)

Dari tanggal dan waktu yang dihasilkan pada query sebelumnya, kita mengektraksi tahun, bulan, minggu, hari pada minggu, weekend atau weekday dan jam dengan query berikut:

![picture](/img/extraction_from_date_time.PNG)

Hasil query:

![picture](/img/extraction_from_date_time_res.PNG)

Dengan jam yang dihasilkan kita mengektrasksi segmen hari menggunakan query berikut:

![picture](/img/extraction_from_time.PNG)

Hasil query:

![picture](/img/extraction_from_time_res.PNG)

### Mengaggregasi time series

![picture](/img/extraction1.PNG)

Isi metanode Aggregations and time series:

![picture](/img/extraction3_1.PNG)

![picture](/img/extraction3_2.PNG)

Di metanode Aggregations and time series, pertama-tama kita persist/cache dataframe yang dihasilkan di metanode sebelumnya ke memory untuk mempermudah dan mempercepat aggregasi data. Kemudian kita melakukan aggregasi berupa sum, untuk mendapatkan penggunaan listrik tiap meteran, dan average terhadap setiap fitur-fitur yang diektraksi sebelumnya. Hasil-hasil agreggasi di rename untuk mempermudah pembacaan fitur dan kemudian saling di-join untuk menghasilkan satu tabel/dataframe yang baru dengan hasil-hasil aggregasi.

Melakukan persitence ke memory:

![picture](/img/aggregation_persist.PNG)

Contoh aggregasi fitur:

![picture](/img/aggregation_example.PNG)

Melakukan group by berdasarkan fitur dan lakukan sum:

![picture](/img/aggregation_example_group_sum1.PNG)

![picture](/img/aggregation_example_group_sum2.PNG)

Melakukan group by berdasarkan fitur dan lakukan average:

![picture](/img/aggregation_example_group_avg1.PNG)

![picture](/img/aggregation_example_group_avg2.PNG)

Melakukan rename terhadap kolom agreggasi:

![picture](/img/aggregation_example_rename.PNG)

Melakukan join:

![picture](/img/aggregation_join.PNG)


Dataframe yang dihasilkan:

![picture](/img/aggregation_res.PNG)

Kita kemudian mendapatkan persentase pengunaan listrik tiap segmen dengan query berikut:

![picture](/img/aggregation_percent.PNG)

Persentase pengunaan listrik tiap segmen didapat dengan membagi segmen dengan fitur yang terbagi oleh segmen tersebut. Contohnya rata-rata  pengunaan segmen hari senin dibagi rata-rata pengunaan tiap minggu.

Hasil query:

![picture](/img/aggregation_percent_res.PNG)


## Modelling

![picture](/img/modelling1.PNG)

Isi metanode:

![picture](/img/modelling2.PNG)

Nilai pada Dataframe yang dihasilkan pada preparasi data pertama-tama di normalisasi menjadi nilai antara 0 sampai 1. Kemudian kita melakukan K-Means Clustering terhadap data yang sudah dinormalisasi menjadi 5 cluster. Selain itu, kita juga melakukan Principal Component Analysis (PCA) untuk melakukan dimensionality reduction. Penamaan PC yang dihasilkan dirurutkan berdasarkan eigenvalue yang terbesar ke terkecil, jadi PC0 memiliki eigenvalue yang lebih tinggi dibanding PC1. Hasil dari kedua node tersebut kemudian digabung dengan join, kemudian nilai-nilai yang dinormalisasi dikembalikan ke nilai awalnya dengan melakukan denormalisasi.

Konfigurasi PCA:

![picture](/img/modelling_conf1.PNG)

Konfigurasi K-Means Clustering

![picture](/img/modelling_conf2.PNG)


Hasil join:

![picture](/img/res_training.PNG)

Hasil denormalisasi

![picture](/img/res_training2.PNG)

## Evaluasi


![picture](/img/evaluation.PNG)

Data yang dihasilkan diwarnai berdasarkan dan digambar di suatu grafik dimana x adalah PC0 dan y adalah PC1. Dengan melihat grafik tersebut kita bisa melihat korelasi antara cluster-cluster yang dibentuk. PC0 dan PC1 dipilih karena keduanya adalah PC yang memiliki eigenvalue yang tertinggi sehingga memiliki bobot yang tertinggi untuk mengetahui korelasi antara titik-titik data.

Pewarnaan Cluster:

![picture](/img/evaluation_colour.PNG)


Grafik yang dihasilkan:

![picture](/img/evaluation_res.PNG)

Dapat disimpulkan bahwa cluster 3 berkorelasi tinggi dengan cluster 0 dan 4 karena saling berdekatan, berkorelasi rendah dengan cluster 2, dan sangat tidak berkorelasi dengan cluster 1 karena jauh di sumbu x yakni PC1 yang memiliki bobot yang lebih besar.


## Deployment 

![picture](/img/Deployment.PNG)

Dataframe yang dibentuk sebelumnya di deploy ke Hive dan HDFS dalam bentuk Paraquet.
