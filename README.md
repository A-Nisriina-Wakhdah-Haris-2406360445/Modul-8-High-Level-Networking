Nama: Nisriina Wakhdah Haris<br>
NPM: 2406360445<br>
Kelas: A<br>

<details>
<Summary><b>Reflection</b></Summary>

1. Berikut ini adalah perbedaan utama antara unary, server streaming, dan bi-directional streaming, yaitu:
- Unary RPC merupakan metode komunikasi di mana client mengirim satu request dan server mengembalikan satu response. Karakteristiknya adalah lebih sederhana dan tidak ada streaming. Unary streaming RPC cocok untuk operasi CRUD, login atau authentication, dan query data tunggal, seperti ambil profil user atau submit form
- Server streaming merupakan metode di mana client mengirimkan satu request dan server mengirim banyak response (stream). Karakteristiknya adalah:
    - Data dikirim secara bertahap
    - Efisien untuk data besar
    - Client hanya menerima stream<br>
Cocok untuk download data besar, live feed dari sever, dan query dataset besar, seperti mengambil daftar transaksi dalam jumlah besar
- Bi-directional adalah metode di mana client dan server sama-sama bisa mengirim banyak pesan dan komunikasi berjalan dua arah secara simultan. Karakteristiknya adalah:
    - Real-time communication
    - Asynchronous
    - Kedua pihak dapat mengirim data kapan saja<br>
Cocok untuk aplikasi chat, multiplayer game, real-time collaboration, dan live monitoring system, seperti sistem trading real-time dan chat antara user
<br>

2. Berikut ini adalah pertimbangan keamanan khususnya terkait authentication, authorization, dan data encryprtion dalam mengimplementasikan gRPC service pada Rust, yaitu:
- Authentication
    1. Bagaimana memastikan hanya client yang valid yang bisa mengakses service
    2. Bagaimana cara mencegah token dicuri atau disalahgunakan
    3. Potensi pencurian token atau replay attack

    - Pendekatan yang dapat dilakukan pada gRPC Rust:
    1. Menggunakan TLS mutual authentication (mTLS) untuk verifikasi dua arah (client & server)
    2. Menggunakan token-based authentication seperti JWT (JSON Web Token)
    3. Validasi metadata gRPC (biasanya token dikirim lewat header)

- Authorization
    1. Apakah setiap RPC method memiliki kontrol akses
    2. Risiko unauthorized access ke data atau fungsi sensitif
    3. Potensi privilege escalation
    4. Apakah sudah menerapkan prinsip least privilege dan pembatasan akses endpoint

    - Pendekatan yang dapat dilakukan pada gRPC Rust:
    1. Menerapkan Role-Based Access Control (RBAC)
    2. Menerapkan Attribute-Based Access Control (ABAC)
    3. Melakukan validasi hak akses di setiap method/service

- Data Encryption
    1. Apakah komunikasi sudah menggunakan TLS
    2. Risiko Man-in-the-Middle (MITM) jika tidak dienkripsi
    3. Apakah data sensitif terlindungi selama transmisi
    4. Validitas dan manajemen sertifikat

    - Pendekatan yang dapat dilakukan pada gRPC Rust:
    1. Menggunakan TLS (Transport Layer Security) untuk semua koneksi gRPC
    2. Menggunakan sertifikat yang valid dan trusted
    3. Menerapkan enkripsi tambahan untuk data sensitif (seperti end-to-end encryption)
<br>

3. Berikut ini adalah potential challenges atau masalah yang dapat terjadi pada Bidirectional Streaming di Rust gRPC, yaitu:
- Kompleksitas concurrency dan sinkronisasi: Hal ini terjadi karena sistem harus mengelola async/await, task, dan channel dengan benar. Selain itu, konsep borrow checker dan ownership pada Rust dapat membatasi desain jika tidak dirancang dengan hati-hati. Contoh masalah yang dapat terjadi adalah client dan server mengirim pesan secara bersamaan (full-duplex), sehingga berpotensi menimbulkan race condition atau data yang tidak konsisten.
- Penanganan flow control dan backpressure: Merupakan salah satu tantangan utama, ketika salah satu pihak mengirim pesan terlalu cepat, buffer dapat cepat penuh dan menyebabkan memory bloat atau bahkan crash. Oleh karena itu, diperlukan mekanisme pembatasan agar sistem tetap stabil.
- Error handling dan lifecycle stream: Tantangan yang muncul meliputi bagaimana menangani koneksi yang tiba-tiba terputus serta stream yang berhenti di tengah jalan. Selain itu, sulit untuk menentukan kapan sebuah stream benar-benar selesai atau berhasil diproses.
- Menjaga urutan dan konsistensi pesan: Masalah yang dapat muncul adalah pesan datang tidak berurutan (out-of-order), serta kemungkinan terjadinya duplikasi atau kehilangan pesan, yang dapat memengaruhi konsistensi data, terutama pada aplikasi seperti chat.
- Manajemen resource untuk banyak koneks: Jika terdapat banyak koneksi streaming yang aktif, maka penggunaan memori dan asynchronous task akan meningkat. Hal ini berisiko menyebabkan resource leak apabila stream tidak dikelola dan ditutup dengan benar.
- Keamanan selama stream berlangsung: Tantangan yang muncul antara lain bagaimana menangani token yang expired saat stream masih aktif, serta memastikan bahwa setiap pesan tetap memenuhi aspek keamanan (misalnya validasi autentikasi dan otorisasi secara berkelanjutan).
- Kesulitan dalam debugging dan testing: Karena streaming bersifat asynchronous dan real-time, bug yang muncul sering kali tidak deterministik (sulit direproduksi), sehingga proses debugging dan pengujian menjadi lebih kompleks dibandingkan metode unary RPC.

4. Berikut ini merupakan keuntungan penggunaan `tokio_stream::wrappers::ReceiverStream` pada gRPC Rust, yaitu:
- Integrasi dengan model async Tokio mudah karena ReceiverStream mengubah `tokio::sync::mpsc::Receiver` menjadi sebuah Stream, sehingga mudah digunakan bersama async/await dalam gRPC (misalnya dengan tonic).
- Sederhana untuk implementasi streaming karena developer cukup mengirim data melalui channel (Sender), lalu ReceiverStream otomatis menjadi sumber stream response ke client.
- Mendukung concurrency secara natural di mana Producer (pengirim data) dan consumer (stream ke client) dapat berjalan di task yang berbeda serta cocok untuk skenario real-time seperti chat.
- Cocok untuk data yang dihasilkan bertahap misalnya notifikasi, event stream, atau hasil proses yang muncul secara incremental.

Berikut ini merupakan kekurangannya, yaitu:
- Perlu kehati-hatian dalam mengelola channel karena jika Sender tidak ditutup dengan benar, stream bisa tidak akan pernah selesai (hanging stream).
- Risiko masalah backpressure, terjadi jika producer mengirim terlalu cepat dan consumer lambat, buffer channel bisa penuh sehingga berpotensi menyebabkan memory bloat.
- Overhead tambahan karena jika menggunakan channel berarti ada lapisan tambahan dibanding langsung menghasilkan stream, sehingga ada sedikit overhead pada performa.
- Error handling lebih kompleks karena error dapat terjadi di sisi producer, channel, atau stream sehingga perlu penanganan yang lebih hati-hati
- Sulit untuk debugging karena berbasis asynchronous dan melibatkan banyak task sehingga menyebabkan bug bisa sulit dilacak (non-deterministic).

5. Untuk meningkatkan code reuse, modularity, maintainability, dan extensibility, kode Rust gRPC dapat disusun dengan cara berikut:
- Memisahkan kode menjadi beberapa lapisan, seperti:
    - Transport layer (gRPC service / handler) untuk menangani request atau response
    - Business logic layer (service logic) yang berisi logika utama aplikasi
    - Data access layer (repository/database) yang digunakan untuk berinteraksi dengan database
- Menggunakan Trait (abstraction) seperti membuat trait untuk repository dan service logic sehingga memudahkan ketika mengganti implementasi dan sesuai dengan prinsip dependency injection
- Memisahkan kode ke dalam module (mod) untuk fitur kecil dan crate terpisah untuk komponen yang besar sehingga kode lebih terorganisir dan mudah dikembangkan
- Membuat fungsi umum atau helper function seperti validasi input dan error handling untuk menghindari duplikasi kode
- Menggunakan interceptor seperti tonic untuk authentication, logging, dan rate limiting agar tidak perlu menulis ulang kode di setiap service
- Menghindari penggunaan hardcode dependency, melainkan inject melalui contructor agar mudah di-testing dengan menggunakan mock dan lebih fleksibel

6. Untuk menangani logika pembayaran yang lebih kompleks, implementasi MyPaymentService perlu ditambahkan beberapa langkah berikut ini, yaitu:
- Validasi input: Sebelum memproses pembayaran, request harus divalidasi dengan mengecek nominal pembayaran harus valid (tidak negatif atau nol), data pengguna harus lengkap, dan metode pembayaran harus valid untuk mencegah error dan data tidak valid memasuki sistem
- Authentication and authorization: Program perlu memastikan bahwa request berasal dari user yang terautentikasi dan periksa apakah user memiliki hak untuk melakukan transaksi. Tujuannya adalah agar proses pembayaran dilakukan oleh user yang sesuai bukan orang lain yang tidak berwenang dan mencegah anonymous user
- Menambahkan error handling: Program harus dapat menangani berbagai kemungkinan error yang disebabkan oleh koneksi yang gagal, saldo tidak cukup, atau payment ditolak
- Memastikan bahwa request yang sama tidak diproses dua kali (idempotency handling)
- Menambahkan security untuk enkripsi dan melindungi data sensitif dengan menggunakan TLS dan sanitasi data
- Menambahkan fitur logging dan monitoring: Setiap transaksi yang berhasil atau gagal harus dicatat dan disimpan untuk audit dan debugging

7. Adopsi gRPC sebagai protokol komunikasi memberikan dampak yang signifikan terhdap arsitektur dan desain sistem terdistribusi, khususnya dalam hal interoperabilitas dengan teknlogi dan platform lain, yaitu:
- Interoperabilitas lintas bahasa: gRPC menggunakan protocol buffers (protobuf) sebagai format datanya sehingga service dapat ditulis dalam berbagai bahasa (seperti Rust, Java, Python, dll) namun komunikasinya tetap konsisten karena menggunakan kontrak `.proto` sehingga memudahkan integrasi antar layanan dengan teknologi yang berbeda.
- Kontrak API yang ketat: gRPC mengharuskan definisi service melalui file `.proto` sehingga API menjadi strict dan terdefinisi, mengurangi ambiguitas dibanding REST, serta lebih mudah dimantain dan minim error
- Performa tinggi dan efisiensi: gRPC menggunakan HTTP/2 dan binary serialization (protobuf) yang menyebabkan latency lebih rendah dan bandwidthnya lebih efisien, cocok untuk microservices dengan komunikasi intensif
- gRPC mendukung unary, server streaming, client streaming, dan bidirectional streaming sehingga dapat mendukung real-time communication

Akan tetapi, gRPC juga menimbulkan tantangan interoperabilitas dengan sistem non-gRPC sehingga sering memerlukan API gateaway atau layer translasi serta menambah kompleksitas dalam versioning, debugging, dan observability

8. Berikut ini adalah keuntungan penggunaan HTTP/2 sebagai protokol dasar gRPC, yaitu:
- Multiplexing (banyak request dalam satu koneksi): HTTP/2 memungkinkan banyak request dan response berjalan paralel dalam satu koneksi TCP sehingga dapat mengurangi latency dan tidak perlu membuka banyak koneksi seperti HTTP/1.1
- Binary farming: HTTP/2 menggunakan format biner sehingga lebih cepat diproses oleh mesain dan hemat bandwidth
- Header compression: Header dikompresi menggunakan HPACK sehingga dapat mengurangi ukuran data dan lebih efisien untuk komunikasi berulang
- Server push: server dapat mengirim data tanpa menunggu request tambahan
- Mendukung streaming: Cocok untk gRPC (unary, streaming, bidirectional)

Selain keuntungan, terdapat juga kekurangannya, yaitu:
- Tidak human-readable: Format biner sulit dibaca manusia sehinggg debugging lebih sulit dibanding JSON/HTTP biasa
- Tidak Didukung Secara Native oleh Semua Client: Browser butuh gRPC-Web dan tidak semua tools mendukung penuh HTTP/2
- Kompleksitas Implementasi: HTTP/2 lebih kompleks dibanding HTTP/1.1 sehingga membutuhkan konfigurasi dan tooling tambahan

9. Secara fundamental, model request-response pada REST API berbeda dengan bidirectional streaming pada gRPC, terutama dalam hal real-time communication, berikut ini adalah perbedaannya:
- Pola komunikasi:
    1. REST (Request-Response): Client mengirim request -> server memberi satu response -> selesai, bersifat stateless, dan satu arah per interaksi, sehingga tidak ada komunikasi berkelanjutan dan untuk update real-time memerlukan request berulang
    2. gRPC (Bidirectional Streaming): Client dan server dapat saling mengirim pesan secara terus-menerus, bersifat full-duplex (dua arah simultan) sehingga komunikasi dapat berlangsung secara kontinu tanpa harus membuka koneksi baru
- Real-time communication:
    1. REST: Tidak native untuk real-time, harus menggunakan polling, long polling, atau tambahan seperti WebSocket. Dampaknya adalah latency lebih tinggi dan tidak efisien (memerlukan banyak request berulang)
    2. gRPC: Native mendukung real-time dan server bisa langsung push data ke client. Dampaknya adalah latency rendah dan lebih efisien untuk event-driven system
- Responsiveness
    1. REST: Respons bergantung pada request dari client dan tidak responsif terhadap perubahan jika client tidak request ulang, contohnya: Client harus refresh untuk melihat update
    2. gRPC: Sangat responsif karena server bisa langsung kirim update dan client juga bisa kirim data kapan saja, contohnya: Chat langsung muncul tanpa refresh
- Kompleksitas
    1. REST: Lebih sederhana, mudah diimplementasikan dan di-debug
    2. gRPC: Lebih kompleks karena merupakan asynchronous programming dan memerlukan stream management
- Efisiensi komunikasi:
    1. REST: Banyak overhead karena harus buka koneksi dan mengirim header berulang
    2. gRPC: Hanya memerlukan satu koneksi (HTTP/2) + streaming dan lebih hemat bandwidth serta resource

10. Pendekatan schema-based pada gRPC (menggunakan Protobuf) memiliki implikasi yang cukup berbeda dibandingkan pendekatan schema-less pada JSON dalam REST API, terutama dalam hal validasi, fleksibilitas, performa, dan evolusi sistem, yaitu:
- Validasi dan konsistensi data:
    1. gRPC: Struktur data harus didefinisikan secara eksplisit di file `.proto` dan tipe data bersifat strongly typed, sehingga data lebih konsisten dan terjamin valid serta error bisa terdeteksi lebih awal (compile-time / contract level)
    2. REST:  Karena tidak ada schema yang wajib dan struktur data fleksibel dan bebas, maka ia lebih fleksibel, tetapi rentan terhadap ketidak konsistenan data dan kesalahan format
- Fleksibilitas
    1. gRPC: Pada gRPC, perubahan harus mengikuti schema `.proto`, sehingga lebih kaku dan memerlukan manajemen versioning yang baik
    2. REST: Mudah menambah atau mengubah field, maka ia lebih fleksibel untuk perubahan cepat dan cocok untuk prototyping
- Performa dan efisiensi:
    1. gRPC: Menggunakan format biner sehingga ukuran payload lebih kecil, parsing lebih cepat, dan lebih efisien untuk sistem besar
    2. REST: Menggunakan format text (human-readable), maka ukurannya lebih besar dan parsing lebih lambat
- Debugging dan readability
    1. gRPC: Karena datanya berbentuk biner, maka sulit dibaca manusia dan membutuhkan alat khusus untuk debugging
    2. REST: Format datanya merupakan text sehingga dapat dibaca manusia dan mempermudah debugging serta testing<br>

Pendekatan schema-based pada gRPC dengan Protocol Buffers memberikan keunggulan dalam hal konsistensi, validasi, dan performa karena menggunakan tipe data yang terdefinisi dengan jelas dan format biner yang efisien. Akan tetapi, pendekatan ini lebih kaku dan memerlukan manajemen versioning yang lebih kompleks. Di sisi lain, JSON pada REST API lebih fleksibel dan mudah digunakan serta didukung secara luas, namun rentan terhadap inkonsistensi data dan memiliki performa yang lebih rendah.
</details>