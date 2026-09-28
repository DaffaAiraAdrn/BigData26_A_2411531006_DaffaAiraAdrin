# Kamus Data

## Dataset akhir `trips_bulanan`

Lokasi: `/content/lapisan_terkurasi/trips_bulanan`, partisi `bulan=<YYYY-MM>/borough_naik=<nilai>`, 5,804,567 baris, 21 kolom.

### Sumber data

| sumber | asal | diambil pada (UTC) | jumlah baris | keterangan |
|---|---|---|---|---|
| zona | https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv | 2026-09-27 18:12:16 | 265 | tabel dimensi 265 zona |
| tarif_referensi | /content/lapisan_mentah/operasional.db | 2026-09-27 18:12:25 | 6 | tabel dimensi dari basis data operasional |
| trip 2023-01 | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet | 2026-09-27 18:15:48 | 3,066,766 | Parquet bulanan TLC |
| cuaca 2023-01 | https://archive-api.open-meteo.com/v1/archive | 2026-09-27 18:15:48 | 744 | Open-Meteo hourly archive |
| udara 2023-01 | https://air-quality-api.open-meteo.com/v1/air-quality | 2026-09-27 18:17:21 | 744 | Open-Meteo air quality, PM2.5 & US AQI per jam |
| trip 2023-07 | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-07.parquet | 2026-09-27 18:16:18 | 2,907,108 | Parquet bulanan TLC |
| cuaca 2023-07 | https://archive-api.open-meteo.com/v1/archive | 2026-09-27 18:16:18 | 744 | Open-Meteo hourly archive |
| udara 2023-07 | https://air-quality-api.open-meteo.com/v1/air-quality | 2026-09-27 18:17:29 | 744 | Open-Meteo air quality, PM2.5 & US AQI per jam |

### Jejak jumlah baris per bulan

| bulan | baris mentah | baris tersimpan | persen tersisa |
|---|---|---|---|
| 2023-01 | 3,066,766 | 2,986,910 | 97.40% |
| 2023-07 | 2,907,108 | 2,817,657 | 96.92% |

Rincian jumlah baris pada setiap tahap penyaringan tersimpan di susut_data.csv (Tugas 5).

### Ringkasan kolom

| kolom | tipe | satuan | persen kosong |
|---|---|---|---|
| tpep_pickup_datetime | datetime64[us] | tanggal dan waktu, waktu lokal New York | 0.0% |
| tpep_dropoff_datetime | datetime64[us] | tanggal dan waktu, waktu lokal New York | 0.0% |
| passenger_count | float64 | orang | 0.0% |
| trip_distance | float64 | mil | 0.0% |
| PULocationID | int64 | kode zona taksi TLC | 0.0% |
| DOLocationID | int64 | kode zona taksi TLC | 0.0% |
| payment_type | int64 | kode kategori metode pembayaran | 0.0% |
| fare_amount | float64 | USD | 0.0% |
| tip_amount | float64 | USD | 0.0% |
| total_amount | float64 | USD | 0.0% |
| durasi_menit | float64 | menit | 0.0% |
| jam | int32 | jam dalam sehari (0 sampai 23) | 0.0% |
| nama_pembayaran | object | kategori teks | 2.374% |
| jam_mulai | datetime64[us] | awal jam, waktu lokal New York | 0.0% |
| suhu_c | float64 | derajat Celsius | 0.001% |
| hujan_mm | float64 | milimeter | 0.001% |
| tanggal | object | teks berformat YYYY-MM-DD | 0.0% |
| pm25_ugm3 | float64 | mikrogram per meter kubik | 0.001% |
| aqi_us | float64 | indeks US EPA 0 sampai 500, tanpa satuan | 0.001% |
| borough_naik | category (partisi) | kategori wilayah | 0.0% |
| bulan | category (partisi) | teks berformat YYYY-MM | 0.0% |

### Definisi kolom

#### tpep_pickup_datetime

- **Tipe:** datetime64[us]
- **Satuan:** tanggal dan waktu, waktu lokal New York
- **Definisi dan cara penghitungan:** Waktu ketika argometer taksi mulai dinyalakan, diambil apa adanya dari data TLC tanpa perubahan nilai.
- **Sumber:** Kolom tpep_pickup_datetime pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Tipe datanya diperiksa terhadap KONTRAK di validasi(). Kesesuaian dengan periode bulan berkas dilaporkan pada laporan_kualitas_<bulan>.csv dan dicek ulang pada Tugas 1, tetapi baris di luar periode tidak dibuang.
- **Penanganan nilai hilang:** Nilai kosong menyebabkan durasi_menit ikut kosong, sehingga baris tersebut terbuang oleh filter durasi 1 sampai 180 menit. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### tpep_dropoff_datetime

- **Tipe:** datetime64[us]
- **Satuan:** tanggal dan waktu, waktu lokal New York
- **Definisi dan cara penghitungan:** Waktu ketika argometer taksi dimatikan, diambil apa adanya dari data TLC tanpa perubahan nilai.
- **Sumber:** Kolom tpep_dropoff_datetime pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Harus lebih akhir dari tpep_pickup_datetime dengan selisih 1 sampai 180 menit, dan syarat ini ditegakkan melalui filter durasi_menit pada pipeline().
- **Penanganan nilai hilang:** Nilai kosong menyebabkan durasi_menit ikut kosong, sehingga baris tersebut terbuang oleh filter durasi 1 sampai 180 menit. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### passenger_count

- **Tipe:** float64
- **Satuan:** orang
- **Definisi dan cara penghitungan:** Jumlah penumpang yang dimasukkan secara manual oleh pengemudi. Nilai yang kosong pada data mentah sudah diganti dengan hasil imputasi.
- **Sumber:** Kolom passenger_count pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Kelengkapannya dilaporkan pada aturan kelengkapan di laporan_kualitas_<bulan>.csv, tanpa aturan rentang nilai dan tanpa pembuangan baris.
- **Penanganan nilai hilang:** Nilai kosong diisi median passenger_count pada jam penjemputan yang sama di bulan yang sama melalui groupby("jam").transform("median"), dan pengisian ini dilakukan sebelum penghapusan duplikat. Berbeda dengan K-7, pipeline() tidak membuat kolom penanda passenger_count_hilang, sehingga baris hasil imputasi tidak dapat dibedakan dari baris asli pada dataset akhir. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membentuk kolom jam, lalu mengisi nilai kosong dengan median per jam sebelum penghapusan duplikat.
  4. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  5. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  6. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### trip_distance

- **Tipe:** float64
- **Satuan:** mil
- **Definisi dan cara penghitungan:** Jarak perjalanan yang dilaporkan argometer taksi, diambil apa adanya dari data TLC tanpa perubahan nilai.
- **Sumber:** Kolom trip_distance pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Harus berada pada rentang 0,01 sampai 100 mil, dan baris di luar rentang tersebut dibuang oleh pipeline(). Tipe datanya juga diperiksa terhadap KONTRAK di validasi().
- **Penanganan nilai hilang:** Nilai kosong tidak lolos filter rentang, sehingga barisnya ikut terbuang. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### PULocationID

- **Tipe:** int64
- **Satuan:** kode zona taksi TLC
- **Definisi dan cara penghitungan:** Kode zona taksi tempat argometer dinyalakan, diambil apa adanya dari data TLC. Kolom ini menjadi kunci penghubung ke taxi_zone_lookup.csv untuk membentuk borough_naik.
- **Sumber:** Kolom PULocationID pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Keberadaan kodenya di taxi_zone_lookup.csv dilaporkan pada laporan_kualitas_<bulan>.csv tanpa pembuangan baris. Kolom ini juga termasuk KUNCI penentu duplikat.
- **Penanganan nilai hilang:** Tidak ada penanganan khusus. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### DOLocationID

- **Tipe:** int64
- **Satuan:** kode zona taksi TLC
- **Definisi dan cara penghitungan:** Kode zona taksi tempat argometer dimatikan, diambil apa adanya dari data TLC.
- **Sumber:** Kolom DOLocationID pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Tidak ada aturan validitas khusus. Kolom ini termasuk KUNCI penentu duplikat, dan penggabungannya dengan zona tujuan hanya dilakukan pada Latihan 3, bukan pada dataset akhir.
- **Penanganan nilai hilang:** Tidak ada penanganan khusus. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### payment_type

- **Tipe:** int64
- **Satuan:** kode kategori metode pembayaran
- **Definisi dan cara penghitungan:** Kode numerik metode pembayaran dari data TLC, diambil apa adanya. Arti kodenya diterjemahkan ke kolom nama_pembayaran melalui tabel tarif_referensi.
- **Sumber:** Kolom payment_type pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Kode 1 sampai 6 memiliki pasangan di tarif_referensi. Kode di luar rentang tersebut tidak dibuang, tetapi menghasilkan nama_pembayaran kosong.
- **Penanganan nilai hilang:** Tidak ada penanganan khusus. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### fare_amount

- **Tipe:** float64
- **Satuan:** USD
- **Definisi dan cara penghitungan:** Tarif berdasarkan waktu dan jarak yang dihitung argometer, diambil apa adanya dari data TLC tanpa perubahan nilai.
- **Sumber:** Kolom fare_amount pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Aturan total_amount >= fare_amount dilaporkan pada laporan_kualitas_<bulan>.csv tanpa pembuangan baris.
- **Penanganan nilai hilang:** Tidak ada penanganan khusus. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### tip_amount

- **Tipe:** float64
- **Satuan:** USD
- **Definisi dan cara penghitungan:** Jumlah tip yang tercatat otomatis untuk pembayaran kartu kredit, diambil apa adanya dari data TLC. Tip tunai tidak tercatat pada kolom ini.
- **Sumber:** Kolom tip_amount pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Aturan tip_amount >= 0 dan aturan pembayaran tunai tanpa tip tercatat dari Latihan 2 dilaporkan pada laporan_kualitas_<bulan>.csv tanpa pembuangan baris.
- **Penanganan nilai hilang:** Tidak ada penanganan khusus. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### total_amount

- **Tipe:** float64
- **Satuan:** USD
- **Definisi dan cara penghitungan:** Total yang ditagihkan kepada penumpang, termasuk tip kartu kredit tetapi tidak termasuk tip tunai, diambil apa adanya dari data TLC tanpa perubahan nilai.
- **Sumber:** Kolom total_amount pada berkas Parquet TLC Yellow Taxi bulan yang bersangkutan.
- **Aturan validitas:** Harus lebih dari 0, dan baris yang tidak memenuhi dibuang oleh pipeline(). Kolom ini juga termasuk KUNCI penentu duplikat dan tipe datanya diperiksa terhadap KONTRAK.
- **Penanganan nilai hilang:** Nilai kosong tidak lolos filter lebih dari 0, sehingga barisnya ikut terbuang. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membaca kolom ini dari berkas tersebut melalui pd.read_parquet(path_trip, columns=KOLOM).
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### durasi_menit

- **Tipe:** float64
- **Satuan:** menit
- **Definisi dan cara penghitungan:** Lama perjalanan yang dihitung dari selisih tpep_dropoff_datetime dan tpep_pickup_datetime dalam detik, lalu dibagi 60.
- **Sumber:** Kolom turunan dari tpep_pickup_datetime dan tpep_dropoff_datetime.
- **Aturan validitas:** Harus berada pada rentang 1 sampai 180 menit, dan baris di luar rentang tersebut dibuang oleh pipeline(). Tipe datanya juga diperiksa terhadap KONTRAK di validasi().
- **Penanganan nilai hilang:** Nilai kosong tidak lolos filter rentang, sehingga barisnya ikut terbuang. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() menghitung durasi_menit dari (tpep_dropoff_datetime - tpep_pickup_datetime).dt.total_seconds() / 60.
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### jam

- **Tipe:** int32
- **Satuan:** jam dalam sehari (0 sampai 23)
- **Definisi dan cara penghitungan:** Jam penjemputan yang diambil dari tpep_pickup_datetime.dt.hour. Kolom ini dipakai sebagai kelompok imputasi passenger_count.
- **Sumber:** Kolom turunan dari tpep_pickup_datetime.
- **Aturan validitas:** Tidak ada aturan validitas khusus karena nilainya selalu berada pada rentang 0 sampai 23.
- **Penanganan nilai hilang:** Tidak ada nilai kosong setelah filter durasi karena tpep_pickup_datetime sudah terisi. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membentuk kolom jam dari tpep_pickup_datetime.dt.hour.
  3. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### nama_pembayaran

- **Tipe:** object
- **Satuan:** kategori teks
- **Definisi dan cara penghitungan:** Nama metode pembayaran hasil penerjemahan payment_type melalui tabel tarif_referensi.
- **Sumber:** Kolom nama_pembayaran pada tabel tarif_referensi di basis data SQLite lapisan_mentah/operasional.db.
- **Aturan validitas:** Join memakai validate="many_to_one", sehingga payment_type pada tarif_referensi wajib unik. Tipe datanya diperiksa terhadap KONTRAK di validasi().
- **Penanganan nilai hilang:** Kode payment_type yang tidak ada di tarif_referensi menghasilkan nilai kosong, dan nilai kosong tersebut dibiarkan tanpa pengisian. Persentase nilai kosong pada dataset akhir sebesar 2,374%.
- **Jejak transformasi:**
  1. K-5: tarif_referensi dibuat, ditulis ke tabel SQLite dengan to_sql(), lalu dibaca ulang dengan pd.read_sql().
  2. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  3. K-10: pipeline() menggabungkan tarif_referensi melalui join left pada payment_type dengan validate="many_to_one".
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### jam_mulai

- **Tipe:** datetime64[us]
- **Satuan:** awal jam, waktu lokal New York
- **Definisi dan cara penghitungan:** tpep_pickup_datetime yang dibulatkan ke bawah ke awal jam melalui dt.floor("h"), sehingga penjemputan pukul 08.47 bernilai 08.00.
- **Sumber:** Kolom turunan dari tpep_pickup_datetime.
- **Aturan validitas:** Dipakai sebagai kunci join cuaca dan udara, dan kedua join tersebut memakai validate="many_to_one", sehingga jam_mulai pada data cuaca dan udara wajib unik.
- **Penanganan nilai hilang:** Tidak ada nilai kosong karena tpep_pickup_datetime sudah terisi. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  3. K-10: pipeline() membentuk jam_mulai dari tpep_pickup_datetime.dt.floor("h").
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### suhu_c

- **Tipe:** float64
- **Satuan:** derajat Celsius
- **Definisi dan cara penghitungan:** Suhu udara pada ketinggian 2 meter di lintang 40,7128 dan bujur -74,0060 untuk jam tersebut. Semua perjalanan pada jam yang sama mendapat nilai yang sama tanpa memperhatikan borough.
- **Sumber:** Variabel temperature_2m dari Open-Meteo Archive API (API_CUACA) dengan timezone America/New_York.
- **Aturan validitas:** Join memakai validate="many_to_one", sehingga setiap jam hanya memiliki satu nilai suhu.
- **Penanganan nilai hilang:** Jam yang tidak tersedia di respons API menghasilkan nilai kosong, dan nilai tersebut dibiarkan. Persentase nilai kosong pada dataset akhir sebesar 0,001%.
- **Jejak transformasi:**
  1. Latihan 6: siapkan_cuaca(bulan) memanggil ambil_cuaca() dari K-3 untuk rentang satu bulan, lalu mengganti nama time, temperature_2m, dan precipitation menjadi jam_mulai, suhu_c, dan hujan_mm.
  2. K-10: pipeline() menggabungkan data cuaca melalui join left pada jam_mulai dengan validate="many_to_one".
  3. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  4. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### hujan_mm

- **Tipe:** float64
- **Satuan:** milimeter
- **Definisi dan cara penghitungan:** Jumlah presipitasi pada lintang 40,7128 dan bujur -74,0060. Menurut dokumentasi Open-Meteo, nilainya merupakan jumlah satu jam sebelumnya dan mencakup hujan serta salju, sehingga nilai pada jam_mulai 08.00 mewakili pukul 07.00 sampai 08.00.
- **Sumber:** Variabel precipitation dari Open-Meteo Archive API (API_CUACA) dengan timezone America/New_York.
- **Aturan validitas:** Join memakai validate="many_to_one", sehingga setiap jam hanya memiliki satu nilai presipitasi.
- **Penanganan nilai hilang:** Nilai kosong dibiarkan pada dataset akhir. Penanda hujan pada K-9 dan studi kasus menganggap nilai kosong sebagai 0 melalui fillna(0). Persentase nilai kosong pada dataset akhir sebesar 0,001%.
- **Jejak transformasi:**
  1. Latihan 6: siapkan_cuaca(bulan) memanggil ambil_cuaca() dari K-3 untuk rentang satu bulan, lalu mengganti nama time, temperature_2m, dan precipitation menjadi jam_mulai, suhu_c, dan hujan_mm.
  2. K-10: pipeline() menggabungkan data cuaca melalui join left pada jam_mulai dengan validate="many_to_one".
  3. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  4. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### tanggal

- **Tipe:** object
- **Satuan:** teks berformat YYYY-MM-DD
- **Definisi dan cara penghitungan:** Tanggal penjemputan dari tpep_pickup_datetime.dt.date yang diubah menjadi teks.
- **Sumber:** Kolom turunan dari tpep_pickup_datetime.
- **Aturan validitas:** Tidak ada aturan validitas khusus.
- **Penanganan nilai hilang:** Tidak ada nilai kosong karena tpep_pickup_datetime sudah terisi. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: berkas yellow_tripdata_<bulan>.parquet diunduh dengan unduh_aman() ke lapisan_mentah/.
  2. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  3. Latihan 6: pipeline_inkremental() membentuk tanggal dari tpep_pickup_datetime.dt.date.astype(str).
  4. Latihan 6: pipeline_inkremental() menulis hasil satu bulan ke lapisan_terkurasi/trips_bulanan/bulan=<bulan>/ dengan partisi borough_naik melalui folder sementara dan os.replace().
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### pm25_ugm3

- **Tipe:** float64
- **Satuan:** mikrogram per meter kubik
- **Definisi dan cara penghitungan:** Konsentrasi partikel berdiameter kurang dari 2,5 mikrometer di dekat permukaan, pada lintang 40,7128 dan bujur -74,0060 untuk jam tersebut.
- **Sumber:** Variabel pm2_5 dari Open-Meteo Air Quality API (API_UDARA) dengan timezone America/New_York.
- **Aturan validitas:** Join memakai validate="many_to_one", dan tambah_udara() menghentikan proses apabila jumlah baris berubah setelah join.
- **Penanganan nilai hilang:** Jam tanpa nilai dari API menghasilkan nilai kosong, dan nilai tersebut dibiarkan. Persentase nilai kosong pada dataset akhir sebesar 0,001%.
- **Jejak transformasi:**
  1. Tugas 3: ambil_udara(bulan) meminta variabel pm2_5 dan us_aqi dari API_UDARA untuk rentang satu bulan, lalu menyimpan JSON mentahnya ke lapisan_mentah/udara_<bulan>.json melalui berkas .part dan os.replace().
  2. Tugas 3: tabel_udara(bulan) mengubah JSON menjadi DataFrame, mengganti nama pm2_5 menjadi pm25_ugm3 dan us_aqi menjadi aqi_us, serta mencetak jumlah jam, keunikan jam_mulai, dan jumlah jam tanpa AQI.
  3. Tugas 3: tambah_udara() menggabungkan data udara ke partisi bulan melalui join left pada jam_mulai dengan validate="many_to_one", memeriksa bahwa jumlah baris tidak berubah, lalu menulis ulang partisi secara atomik.

#### aqi_us

- **Tipe:** float64
- **Satuan:** indeks US EPA 0 sampai 500, tanpa satuan
- **Definisi dan cara penghitungan:** Indeks kualitas udara standar Amerika Serikat yang dihitung Open-Meteo dari beberapa polutan dengan mengambil indeks tertinggi, pada koordinat yang sama dengan pm25_ugm3.
- **Sumber:** Variabel us_aqi dari Open-Meteo Air Quality API (API_UDARA) dengan timezone America/New_York.
- **Aturan validitas:** Join memakai validate="many_to_one", dan tambah_udara() menghentikan proses apabila jumlah baris berubah setelah join.
- **Penanganan nilai hilang:** Jam tanpa nilai dari API menghasilkan nilai kosong, dan nilai tersebut dibiarkan. Jumlah jam tanpa AQI per bulan dicetak oleh tabel_udara(). Persentase nilai kosong pada dataset akhir sebesar 0,001%.
- **Jejak transformasi:**
  1. Tugas 3: ambil_udara(bulan) meminta variabel pm2_5 dan us_aqi dari API_UDARA untuk rentang satu bulan, lalu menyimpan JSON mentahnya ke lapisan_mentah/udara_<bulan>.json melalui berkas .part dan os.replace().
  2. Tugas 3: tabel_udara(bulan) mengubah JSON menjadi DataFrame, mengganti nama pm2_5 menjadi pm25_ugm3 dan us_aqi menjadi aqi_us, serta mencetak jumlah jam, keunikan jam_mulai, dan jumlah jam tanpa AQI.
  3. Tugas 3: tambah_udara() menggabungkan data udara ke partisi bulan melalui join left pada jam_mulai dengan validate="many_to_one", memeriksa bahwa jumlah baris tidak berubah, lalu menulis ulang partisi secara atomik.

#### borough_naik

- **Tipe:** category (partisi)
- **Satuan:** kategori wilayah
- **Definisi dan cara penghitungan:** Borough zona penjemputan hasil penggabungan PULocationID dengan kolom Borough pada taxi_zone_lookup.csv. Kolom ini juga menjadi partisi tingkat kedua di bawah bulan.
- **Sumber:** Kolom Borough pada berkas taxi_zone_lookup.csv yang diunduh dari URL_ZONA.
- **Aturan validitas:** Join memakai validate="many_to_one", sehingga LocationID pada tabel zona wajib unik. Tipe datanya diperiksa terhadap KONTRAK di validasi().
- **Penanganan nilai hilang:** Nilai kosong, yang muncul karena Borough 'N/A' dibaca pandas sebagai NaN atau karena PULocationID tidak ditemukan, diisi 'N/A' sebelum ditulis. Pengisian ini diperlukan karena partisi dengan nilai kosong membuat pd.read_parquet() gagal membaca direktori. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. K-2: taxi_zone_lookup.csv diunduh dengan unduh_aman() ke PATH_ZONA.
  2. K-10: pipeline() membuang duplikat pada KUNCI, lalu menyaring baris dengan trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0.
  3. K-10: pipeline() membaca ulang PATH_ZONA, menggabungkan kolom Borough melalui join left pada PULocationID = LocationID dengan validate="many_to_one", lalu mengganti namanya menjadi borough_naik.
  4. Latihan 6: pipeline_inkremental() mengisi nilai kosong dengan 'N/A', lalu memakai kolom ini sebagai partisi borough_naik=<nilai> di dalam setiap direktori bulan.
  5. Tugas 3: tambah_udara() membaca ulang partisi bulan, menambahkan kolom udara, lalu menulis ulang partisi tanpa mengubah nilai kolom ini.

#### bulan

- **Tipe:** category (partisi)
- **Satuan:** teks berformat YYYY-MM
- **Definisi dan cara penghitungan:** Bulan berkas sumber yang diproses, bukan bulan penjemputan. Nilainya diambil dari nama direktori partisi bulan=<bulan>, bukan dari isi berkas Parquet.
- **Sumber:** Argumen bulan pada pipeline_inkremental().
- **Aturan validitas:** Setiap bulan hanya memiliki satu direktori partisi, dan pipeline_inkremental() melewati bulan yang direktorinya sudah ada.
- **Penanganan nilai hilang:** Tidak ada nilai kosong karena setiap berkas berada di dalam satu direktori bulan. Persentase nilai kosong pada dataset akhir sebesar 0,0%.
- **Jejak transformasi:**
  1. Latihan 6: pipeline_inkremental(bulan) menulis hasil satu bulan ke direktori bulan=<bulan>.
  2. Tugas 1: pipeline_inkremental() dijalankan untuk setiap bulan pada BULAN_TUGAS.
  3. Tugas 4: kolom ini muncul ketika trips_bulanan dibaca dengan pq.read_table() dari nama direktori partisi.

## Tabel analitik studi kasus (`tabel_analitik.parquet`)

### Sumber data

| sumber | asal | diambil pada (UTC) | jumlah baris | keterangan |
|---|---|---|---|---|
| trip | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet | 2026-09-27 18:12:16 | 3,066,766 | Parquet Bulanan TLC |
| zona | https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv | 2026-09-27 18:12:16 | 265 | tabel dimensi 265 zona |
| cuaca | https://archive-api.open-meteo.com/v1/archive | 2026-09-27 18:12:16 | 744 | Open-Meteo hourly archive |
| tarif_referensi | /content/lapisan_mentah/operasional.db | 2026-09-27 18:12:25 | 6 | tabel dimensi dari basis data operasional |

### Jejak jumlah baris

| langkah | keterangan | jumlah |
|---|---|---|
| K-2 | baris trip mentah | 3,066,766 |
| K-7 | setelah hapus duplikat pada KUNCI | 3,066,765 |
| K-8 | setelah filter kelayakan (bersih) | 2,986,910 |
| P | perjalanan tanpa borough_naik, tidak ikut dikelompokkan | 340 |
| P | kelompok borough x jam sebelum ambang | 3,799 |
| P | kelompok dibuang karena kurang dari 30 perjalanan | 1,705 |
| P | perjalanan di dalam kelompok yang dibuang | 16,077 |
| P | baris tabel_analitik | 2,094 |
| P | perjalanan yang terwakili di tabel_analitik | 2,970,493 |

### Definisi kolom

#### borough_naik

- **Tipe:** object
- **Satuan:** tidak bersatuan (kategori wilayah)
- **Sumber:** Kolom Borough pada berkas taxi_zone_lookup.csv yang diunduh dari URL_ZONA dan disimpan di lapisan_mentah/taxi_zone_lookup.csv.
- **Cara penghitungan:** Nilai diambil apa adanya dari kolom Borough milik zona yang LocationID-nya sama dengan PULocationID perjalanan, tanpa perhitungan tambahan. Perjalanan yang borough_naik-nya kosong, misalnya karena nilai Borough 'N/A' dibaca pandas sebagai NaN, tidak ikut dikelompokkan karena groupby mengabaikan kunci kosong, dan jumlahnya dicatat pada bagian Jejak jumlah baris.
- **Jejak transformasi:**
  1. K-2: berkas zona diunduh dengan unduh_aman() lalu dibaca ke DataFrame zona melalui pd.read_csv().
  2. K-8: bersih digabung dengan zona melalui join left pada PULocationID = LocationID dengan validate="many_to_one", lalu kolom Borough diganti namanya menjadi borough_naik.
  3. P: borough_naik dipakai sebagai kunci pertama groupby bersama jam_mulai.

#### jam_mulai

- **Tipe:** datetime64[us]
- **Satuan:** jam (awal jam, waktu lokal New York)
- **Sumber:** Kolom tpep_pickup_datetime pada berkas yellow_tripdata_2023-01.parquet yang diunduh dari URL_TRIP dan disimpan di lapisan_mentah/.
- **Cara penghitungan:** Waktu penjemputan dibulatkan ke bawah ke awal jam melalui dt.floor("h"), misalnya 08.47 menjadi 08.00, sehingga satu baris mewakili perjalanan yang dijemput antara 08.00.00 dan 08.59.59. Waktu TLC tercatat sebagai waktu lokal New York tanpa informasi zona waktu, sedangkan data cuaca diminta dengan timezone America/New_York, sehingga kedua sumber memakai acuan jam yang sama.
- **Jejak transformasi:**
  1. K-2: tpep_pickup_datetime dibaca dari Parquet TLC melalui pd.read_parquet(PATH_TRIP, columns=KOLOM).
  2. K-8: bersih["jam_mulai"] dibentuk dari tpep_pickup_datetime.dt.floor("h") sebagai kunci join cuaca.
  3. P: jam_mulai dibentuk ulang dengan rumus yang sama melalui assign(), lalu dipakai sebagai kunci kedua groupby.

#### jumlah_perjalanan

- **Tipe:** int64
- **Satuan:** perjalanan
- **Sumber:** Baris DataFrame bersih, yaitu perjalanan TLC yang sudah melewati penghapusan duplikat dan filter kelayakan.
- **Cara penghitungan:** Jumlah baris bersih yang memiliki borough_naik dan jam_mulai yang sama, dihitung dengan agregasi size pada kolom total_amount. Setelah agregasi, kelompok yang berisi kurang dari 30 perjalanan dibuang dari tabel, sehingga setiap baris yang tersisa mewakili minimal 30 perjalanan.
- **Jejak transformasi:**
  1. K-2: data perjalanan dibaca dari Parquet TLC.
  2. K-7: baris duplikat dibuang melalui drop_duplicates(subset=KUNCI, keep="first").
  3. K-8: baris disaring dengan syarat trip_distance 0,01 sampai 100 mil, durasi_menit 1 sampai 180 menit, dan total_amount lebih dari 0, lalu hasilnya disimpan sebagai bersih.
  4. P: bersih dikelompokkan per borough_naik dan jam_mulai, dihitung size-nya, lalu kelompok dengan jumlah_perjalanan kurang dari 30 dibuang.

#### rata_tarif_per_mil

- **Tipe:** float64
- **Satuan:** USD per mil
- **Sumber:** Kolom total_amount dan trip_distance pada berkas Parquet TLC.
- **Cara penghitungan:** Untuk setiap perjalanan, total_amount dibagi trip_distance lalu dibulatkan tiga desimal. Nilai pada tabel adalah median tarif per mil seluruh perjalanan dalam kelompok borough dan jam tersebut, bukan rerata, meskipun nama kolomnya diawali rata. total_amount adalah total yang ditagihkan kepada penumpang, termasuk tip kartu kredit tetapi tidak termasuk tip tunai, sehingga komposisi metode pembayaran dalam satu jam ikut memengaruhi nilai ini. Batas bawah trip_distance 0,01 mil mencegah pembagian dengan nol, tetapi jarak yang sangat pendek tetap menghasilkan tarif per mil yang sangat besar, dan hal ini menjadi alasan median dipakai.
- **Jejak transformasi:**
  1. K-8: baris dengan trip_distance di luar 0,01 sampai 100 mil atau total_amount tidak lebih dari 0 dibuang.
  2. K-9: bersih["tarif_per_mil"] dihitung dari (total_amount / trip_distance).round(3).
  3. P: tarif_per_mil diagregasi dengan median per borough_naik dan jam_mulai menjadi rata_tarif_per_mil.

#### rata_kecepatan

- **Tipe:** float64
- **Satuan:** mil per jam
- **Sumber:** Kolom trip_distance, tpep_pickup_datetime, dan tpep_dropoff_datetime pada berkas Parquet TLC.
- **Cara penghitungan:** Untuk setiap perjalanan, trip_distance dibagi durasi perjalanan dalam satuan jam (durasi_menit / 60) lalu dibulatkan dua desimal. Nilai pada tabel adalah median kecepatan seluruh perjalanan dalam kelompok tersebut, bukan rerata. Kecepatan ini merupakan kecepatan rata-rata sepanjang satu perjalanan dan dicatat pada jam penjemputan, walaupun perjalanan dapat berlanjut ke jam berikutnya.
- **Jejak transformasi:**
  1. K-6: durasi_menit dihitung dari (tpep_dropoff_datetime - tpep_pickup_datetime).dt.total_seconds() / 60.
  2. K-8: baris dengan durasi_menit di luar 1 sampai 180 menit dibuang.
  3. K-9: bersih["kecepatan_mph"] dihitung dari (trip_distance / (durasi_menit / 60)).round(2).
  4. P: kecepatan_mph diagregasi dengan median per borough_naik dan jam_mulai menjadi rata_kecepatan.

#### suhu_c

- **Tipe:** float64
- **Satuan:** derajat Celsius
- **Sumber:** Variabel temperature_2m dari Open-Meteo Archive API (API_CUACA) pada lintang 40,7128 dan bujur -74,0060 dengan timezone America/New_York.
- **Cara penghitungan:** Suhu udara pada ketinggian 2 meter yang dilaporkan Open-Meteo untuk jam tersebut. Satu jam hanya memiliki satu nilai suhu untuk seluruh kota, sehingga semua perjalanan dalam satu kelompok membawa nilai yang identik dan agregasi first menghasilkan nilai jam itu sendiri. Nilai yang sama berlaku untuk semua borough pada jam yang sama, dan jam tanpa data cuaca bernilai kosong.
- **Jejak transformasi:**
  1. K-3: data cuaca diambil melalui ambil_cuaca("2023-01-01", "2023-01-31"), lalu temperature_2m diganti namanya menjadi suhu_c dan time menjadi jam_mulai.
  2. K-8: bersih digabung dengan cuaca melalui join left pada jam_mulai dengan validate="many_to_one".
  3. P: suhu_c diagregasi dengan first per borough_naik dan jam_mulai.

#### hujan

- **Tipe:** int8
- **Satuan:** penanda biner (0 berarti tidak hujan, 1 berarti hujan)
- **Sumber:** Variabel precipitation dari Open-Meteo Archive API pada koordinat dan zona waktu yang sama dengan suhu_c.
- **Cara penghitungan:** Bernilai 1 apabila hujan_mm pada jam tersebut lebih dari 0,1 mm dan 0 apabila tidak, dengan jam tanpa data cuaca dianggap 0 karena fillna(0). Menurut dokumentasi Open-Meteo, precipitation merupakan jumlah presipitasi satu jam sebelumnya dan mencakup hujan serta salju, sehingga penanda pada jam_mulai 08.00 berasal dari presipitasi pukul 07.00 sampai 08.00. Seluruh perjalanan dalam satu jam membawa nilai yang sama, sehingga agregasi max menghasilkan nilai jam itu sendiri.
- **Jejak transformasi:**
  1. K-3: precipitation diganti namanya menjadi hujan_mm.
  2. K-8: hujan_mm masuk ke bersih melalui join left cuaca pada jam_mulai.
  3. K-9: bersih["hujan"] dihitung dari (hujan_mm.fillna(0) > 0.1).astype("int8").
  4. P: hujan diagregasi dengan max per borough_naik dan jam_mulai.

