### Reflection

#### 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

- Unary RPC adalah jenis RPC yang paling sederhana di mana _client_ mengirimkan satu _request_ ke _server_ dan mendapatkan satu _response_ kembali. Hal ini cocok untuk operasi sederhana di mana tidak ada interaksi lebih lanjut setelah _request_ awal, misalnya untuk melakukan _fetch_ satu _item_ dari _database_ dan _user authentication_.

- Server Streaming RPC adalah jenis RPC di mana _client_ mengirimkan _request_ ke _server_ dan mendapatkan _stream_ untuk membaca urutan pesan kembali. _Client_ membaca dari _stream_ yang dikembalikan sampai tidak ada pesan lagi. Hal ini cocok untuk skenarion di mana _server_ perlu mengirim beberapa _response_ sepanjang waktu, seperti mengirim set data besar atau pembaruan _real-time_, misalnya harga pasar saham dan pembaruan cuaca.

- Bi-directional Streaming RPC adalah jenis RPC di mana kedua belah pihak mengirimkan urutan pesan menggunakan _read-write stream_. Dua _stream_ tersebut beroperasi secara independen, jadi _client_ dan _server_ dapat membaca dan menulis dalam urutan apa pun yang mereka suka. Hal ini cocok untuk skenario di mana baik _client_ dan _server_ perlu mengirim serangkaian pesan satu sama lain, seperti dalam aplikasi chat atau _real-time multiplayer game_.

#### 2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

- Authentication: pastikan pengguna tanpa autentikasi tidak dapat terhubung ke server gRPC dengan menerapkan mekanisme autentikasi yang kuat untuk mencegah akses yang tidak sah misalnya dengan menggunakan JWT.

- Authorization: tentukan peran dan izin untuk berbagai kelompok pengguna, terapkan pemeriksaan otorisasi pada tingkat layanan atau bahkan pada suatu metode RPC individu.

- Data encryption: gunakan TLS (Transport Layer Security) untuk mengenkripsi data saat berpindah antara _client_ dan _server_ atau dengan mengenkripsi data di tingkat aplikasi sebelum mengirimnya melalui gRPC.

Penting untuk diingat bahwa gRPC memungkinkan penggunaan _streaming data_, yang berarti kita harus memastikan bahwa autentikasi, otorisasi, dan enkripsi dilakukan secara konsisten sepanjang siklus hidup streaming, bukan hanya pada awal _request_. Ini memastikan keamanan data sepanjang waktu, bahkan selama transmisi data yang berkelanjutan.

#### 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

Dalam menangani _bidirectional streaming_ di Rust gRPC, terutama untuk aplikasi _chat_, beberapa tantangan yang mungkin muncul antara lain pengelolaan koneksi yang harus tetap terbuka selama sesi _chat_, penanganan pesan yang dapat datang dalam urutan apa pun, penanganan kesalahan jaringan dan pemulihan koneksi, sinkronisasi pesan di semua klien, keamanan komunikasi, dan penanganan beban tinggi dari banyak pengguna dan _request_ secara simultan.

#### 4. What are the advantages and disadvantages of using the `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?

Menggunakan `tokio_stream::wrappers::ReceiverStream` dalam layanan gRPC Rust memiliki beberapa keuntungan dan kerugian. Keuntungannya adalah memungkinkan kita untuk dengan mudah mengubah `tokio::sync::mpsc::Receiver` menjadi `Stream` yang dapat digunakan untuk _streaming response_ dalam gRPC. Hal ini memungkinkan integrasi yang mudah dengan sintaks _async/await_ Rust dan penanganan _backpressure_ secara otomatis. Namun, `ReceiverStream` tidak memiliki penanganan kesalahan bawaan, yang berarti _stream_ akan berakhir jika terjadi kesalahan saat penerimaan pesan. `ReceiverStream` hanya mendukung komunikasi satu arah, memerlukan dua saluran terpisah untuk komunikasi dua arah. Selain itu, ukuran _buffer_ ditetapkan saat pembuatan dan tidak dapat diubah yang berpotensi membatasi fleksibilitas dalam beberapa kasus penggunaan.

#### 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

- Memisahkan definisi layanan dan implementasinya ke dalam modul yang berbeda. Misalnya, kode dalam `grpc_server.rs` dan `grpc_client.rs` dapat dibagi menjadi beberapa modul atau file berdasarkan fungsionalitasnya.

- Menggunakan _trait_ dan implementasi _trait_ untuk abstraksi dan polimorfisme yang memungkinkan kita untuk mengganti implementasi tanpa mengubah kode yang menggunakannya.

- Menggunakan tipe data generik dan fungsi generik untuk membuat kode yang dapat digunakan kembali.

#### 6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

Dalam implementasi `MyPaymentService`, beberapa langkah tambahan yang mungkin diperlukan adalah menambahkan validasi lebih lanjut pada data pembayaran, seperti verifikasi nomor kartu kredit atau pengecekan saldo sebelum melakukan transaksi dan mengimplementasikan mekanisme untuk menangani transaksi yang gagal, seperti pengulangan otomatis atau sistem antrian.

#### 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Adopsi gRPC sebagai protokol komunikasi memungkinkan komunikasi yang efisien dan cepat antara layanan melalui penggunaan HTTP/2 dan Protobuf yang dapat meningkatkan kinerja sistem secara keseluruhan. Selain itu, gRPC mendukung berbagai bahasa pemrograman, termasuk Java, C++, Python, Go, dan Rust, yang memungkinkan interoperabilitas yang baik dengan teknologi dan platform lain. Hal ini berarti bahwa layanan yang ditulis dalam bahasa yang berbeda dapat berkomunikasi satu sama lain dengan mudah.

#### 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2, protokol yang mendasari gRPC, memiliki beberapa keuntungan dibandingkan dengan HTTP/1.1 atau HTTP/1.1 dengan WebSocket untuk REST APIs karena HTTP/2 mendukung _multiplexing_ yang memungkinkan beberapa _request_ dan _response_ untuk berbagi koneksi yang sama sehingga mengurangi latensi dan meningkatkan efisiensi. Selain itu, HTTP/2 mendukung pengiriman prioritas dan _stream handling_ yang dapat membantu mengoptimalkan penggunaan jaringan. HTTP/2 juga mendukung _push server_ yang memungkinkan _server_ untuk mengirim data ke _client_ sebelum _client_ memintanya sehingga daat meningkatkan kinerja. Namun, karena HTTP/2 adalah protkol ang relatif baru, dukungan untuknya belum sepenuhnya ada di semua _platform_ atau teknologi (misalnya _web browser_). Selain itu, HTTP/2 mungkin lebih kompleks untuk di-_debug_ atau di-_inspect_ dibandingkan dengan HTTP/1.1 karena sifatnya yang _binary_, bukan teks. Meskipun HTTP/2 mendukung _streaming_, dukungan ini mungkin tidak sefleksibel atau sekuat WebSocket untuk kasus penggunaan tertentu, seperi komunikasi dua arah _real-time_.

#### 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

Pada model _request-response_ dari REST API, _client_ mengirimkan _request_ ke _server_ dan menunggu _response_. Hal ini berarti komunikasi hanya berlangsung satu arah pada satu waktu dan _client_ harus menunggu _response_ sebelum mengirimkan _request_ lain. Sementara itu, gRPC mendukung _bidirectional streaming_ di mana _client_ dan _server_ dapat mengirim dan menerima pesan secara simultan dalam satu koneksi. Hal ini memungkinkan komunikasi _real-time_ dan responsivitas yang lebih baik karena pesan dapat ditukar tanpa menunggu _response_ untuk setiap _request_.

#### 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

Skema yang didefinisikan dalam Protocol Buffers memungkinkan validasi data yang kuat dan otomatis serta serialisasi dan deserialisasi yang efisien. Hal ini juga memudahkan generasi kode otomatis untuk berbagai bahasa pemrograman. Skema ini memastikan bahwa struktur dan tipe data dari pesan yang ditukar antara _client_ dan _server_ jelas dan konsisten yang dapat mengurangi kesalahan. Namun, pendekatan ini juga memiliki beberapa kerugian. Misalnya, skema yang ketat dapat mengurangi fleksibilitas dan membuat perubahan atau evolusi API menjadi lebih sulit. Selain itu, meskipun Protocol Buffers lebih efisien dalam hal ukuran dan kecepatan dibandingkan dengan JSON, dukungan untuk itu mungkin tidak sebanyak JSON yang telah menjadi standar de facto untuk REST API dan didukung oleh hampir semua teknologi dan platform web modern.
