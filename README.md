# Reflections

### Commit 1 Reflection Notes

Pada method `handle_connection`, kita membuat sebuah instance baru dari `BufReader` yang dibungkus dengan _mutable reference_ ke _stream_. `BufReader` digunakan untuk menambahkan _buffering_ ke dalam proses pembacaan data dari _stream_.

Kemudian, kita membuat sebuah variabel `http_request` untuk mengumpulkan baris-baris dari _request_ yang dikirimkan oleh _browser_ ke server kita dalam sebuah vektor. Di sini, `BufReader` mengimplementasikan trait `std::io::BufRead` yang menyediakan _method_ `lines`. _Method_ `lines` mengembalikan sebuah iterator dari `Result<String, std::io::Error>` dengan membagi _stream_ data setiap kali ia menemukan _byte newline_. Untuk mendapatkan setiap String, kita memetakan dan membuka setiap _Result. Result bisa berupa \_error_ jika data tidak valid dalam UTF-8 atau jika terjadi masalah saat membaca dari _stream_. Sebuah program produksi seharusnya menangani _error_-_error_ ini dengan lebih elegan, tapi di sini kita memilih untuk menghentikan program dalam kasus _error_ untuk kesederhanaan.

_Browser_ memberi sinyal akhir dari sebuah HTTP _request_ dengan mengirimkan dua karakter _newline_ berturut-turut. Oleh karena itu, untuk mendapatkan satu _request_ dari _stream_, kita mengambil baris-baris sampai kita mendapatkan baris yang kosong. Setelah berhasil mengumpulkan baris-baris _request_ ke dalam vektor, kita mencetaknya menggunakan format debug yang rapi agar kita dapat melihat instruksi-instruksi yang dikirimkan oleh browser ke server kita.

### Commit 2 Reflection Notes

![Commit 2 screen capture](/assets/images/commit2.png)

Pada kode baru untuk _method_ `handle_connection`, terdapat beberapa penambahan agar dapat mengembalikan konten HTML kepada klien. Pertama, _server_ membaca isi dari _file_ `hello.html` menggunakan `fs::read_to_string`, yang kemudian disertakan sebagai _body_ dari HTTP _response_. Selanjutnya, `status_line` diatur menjadi `"HTTP/1.1 200 OK"`, dan _header_ `Content-Length` ditambahkan untuk menentukan ukuran _response body_ sehingga HTTP _response_ menjadi valid. Hal ini memungkinkan pengguna untuk melihat halaman HTML yang dihasilkan saat mengakses _server_ melalui _browser_. Namun, saat ini _server_ masih belum membedakan jenis-jenis _request_ yang berbeda sehingga akan selalu mengembalikan isi dari _file_ `hello.html`.

### Commit 3 Reflection Notes

![Commit 3 screen capture](/assets/images/commit3.png)

Pemisahan antara _response_ dilakukan berdasarkan HTTP _request_ yang diterima oleh _web server_. Jika _request_-nya adalah untuk path _root_ ("/"), _server_ akan mengembalikan isi dari _file_ `hello.html` dengan kode status 200 (OK). Sementara itu, jika permintaannya adalah untuk _path_ lain, _server_ akan mengembalikan _error_ 404 (NOT FOUND) beserta isi dari _file_ `404.html`.

_Refactor_ diperlukan untuk menghilangkan pengulangan dalam kode tersebut. Sebelum di-_refactor_, blok `if` dan `else` mengandung kode yang diulang untuk membaca _file_ dan menulis _response_. Dengan melakukan _refactor_, perbedaan antara kasus (`status_line` dan `filename`) dimasukkan ke dalam variabel, dan kode umum untuk membaca _file_ dan menulis _response_ ditempatkan di luar blok kondisional. Hal ini membuat kode menjadi lebih ringkas, lebih _maintainable_, dan dapat memastikan konsistensi dalam penanganan _response_.

### Commit 4 Reflection Notes

Saat ini, _server_ akan memproses setiap _request_ secara bergantian, yang berarti tidak akan memproses koneksi kedua sampai yang pertama selesai diproses. Jika _server_ menerima lebih banyak lagi _request_, eksekusi serial ini akan menjadi kurang optimal. Jika server menerima _request_ yang memakan waktu lama untuk diproses, _request_ berikutnya harus menunggu sampai _request_ yang lama selesai, bahkan jika _request_ yang baru dapat diproses dengan cepat.

Untuk mensimulasikan _request_ yang lambat dalam implementasi _server_ saat ini, kita memodifikasi method `handle_connection` untuk menangani _request_ ke `/sleep` dengan _response_ yang disimulasikan lambat, yang menyebabkan _server_ untuk _sleep_ selama lima detik sebelum memberi _response_. Kode sekarang menggunakan _statement_ `match` untuk menangani tiga kasus berdasarkan isi dari `request_line`.

### Commit 5 Reflection Notes

Dalam _chapter_ ini, kita belajar tentang implementasi _thread pool_ dalam Rust untuk meningkatkan _throughput_ _web server_ kita. _Thread pool_ adalah kumpulan _thread_ yang siap menangani tugas, dengan setiap tugas diberikan kepada satu _thread_ dalam _pool_ untuk diproses. Dengan membatasi jumlah _thread_ dalam _pool_, kita dapat melindungi _server_ dari serangan _Denial of Service_ (DoS) dan meningkatkan kinerja secara keseluruhan. Kita juga menggunakan konsep _channel_ untuk mengirimkan tugas dari _pool_ ke _thread_ secara aman. Dengan cara ini, _server_ dapat mengelola banyak koneksi secara bersamaan tanpa membebani sistem dengan membuat _thread_ baru untuk setiap koneksi, sehingga meningkatkan responsivitas server secara keseluruhan.

Pada _file_ `lib.rs`, kita mulai dengan mendefinisikan struktur `ThreadPool` yang akan menjadi inti dari implementasi _thread pool_ kita. Kita menggunakan _vector_ `workers` untuk menyimpan _instance_ dari struktur `Worker`, yang mewakili setiap _thread_ dalam _pool_. Kemudian, kita menggunakan _channel_ untuk komunikasi antara _thread_ dalam _pool_ dan _executor_ (pemanggil tugas), di mana _sender_ disimpan dalam `ThreadPool` dan _receiver_ diberikan kepada setiap `Worker`. Struktur `Worker` memiliki _method_ `new` yang berperan dalam membuat _thread_ dan mengaitkannya dengan _receiver_ dari _channel_. Di dalam _thread_ masing-masing `Worker`, kita mengambil tugas dari _channel_ dan menjalankannya secara bersamaan. Setiap kali _executor_ (melalui _method_ `execute`) mengirimkan tugas melalui _sender_, tugas itu akan diterima oleh salah satu _worker_ yang tersedia dan dieksekusi. Dengan demikian, kita berhasil menciptakan _thread pool_ yang efisien untuk mengelola tugas dengan menghindari pembuatan _thread_ baru secara berlebihan.
