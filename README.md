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
