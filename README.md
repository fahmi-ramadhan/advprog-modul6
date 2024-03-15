# Reflections

### Commit 1 Reflection Notes

Pada method `handle_connection`, kita membuat sebuah instance baru dari `BufReader` yang dibungkus dengan _mutable reference_ ke _stream_. `BufReader` digunakan untuk menambahkan _buffering_ ke dalam proses pembacaan data dari _stream_.

Kemudian, kita membuat sebuah variabel `http_request` untuk mengumpulkan baris-baris dari _request_ yang dikirimkan oleh _browser_ ke server kita dalam sebuah vektor. Di sini, `BufReader` mengimplementasikan trait `std::io::BufRead` yang menyediakan _method_ `lines`. _Method_ `lines` mengembalikan sebuah iterator dari `Result<String, std::io::Error>` dengan membagi _stream_ data setiap kali ia menemukan _byte newline_. Untuk mendapatkan setiap String, kita memetakan dan membuka setiap _Result. Result bisa berupa \_error_ jika data tidak valid dalam UTF-8 atau jika terjadi masalah saat membaca dari _stream_. Sebuah program produksi seharusnya menangani _error_-_error_ ini dengan lebih elegan, tapi di sini kita memilih untuk menghentikan program dalam kasus _error_ untuk kesederhanaan.

_Browser_ memberi sinyal akhir dari sebuah HTTP _request_ dengan mengirimkan dua karakter _newline_ berturut-turut. Oleh karena itu, untuk mendapatkan satu _request_ dari _stream_, kita mengambil baris-baris sampai kita mendapatkan baris yang kosong. Setelah berhasil mengumpulkan baris-baris _request_ ke dalam vektor, kita mencetaknya menggunakan format debug yang rapi agar kita dapat melihat instruksi-instruksi yang dikirimkan oleh browser ke server kita.
