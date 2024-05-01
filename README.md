### Tutorial 9

# Refleksi

1. Perbedaan Unary, server streaming, dan bi-directional rpc 

    - Unary rpc 

    Metode ini melibatkan pengiriman permintaan dari klien dan menerima respons dari server, yang berarti klien mengirim satu permintaan dan menerima satu respons, data yang dikirim kecil dan hanya untuk 1 kesempatan atau tidak berkelanjutan. 

    Skenario yang sesuai : untuk autentikasi, pengiriman form, meminta informasi suatu entitas. 

    - Server streaming rpc 

    Dalam metode ini, klien mengirimkan permintaan ke server dan server mulai mengirimkan serangkaian respons kepada klien, yang berarti klien menerima sejumlah respons dari server yang mungkin berlangsung dalam rentang waktu yang lama tanpa membuat koneksi baru  (1 stream). Biasa nya untuk data yang besar. 

    Skenario yang sesuai : mengunduh file besar, melihat berita terkini 

    - Bi-directional streaming rpc 

    Metode ini melibatkan komunikasi dua arah antara klien dan server, di mana keduanya dapat mengirimkan dan menerima sejumlah pesan secara independen satu sama lain. Data intensif dan kontinu. 

    Skenario yang sesuai : untuk interaksi secara real time seperti aplikasi chat, game daring, dan aplikasi kolaboratif untuk membuat sesuatu 

2. Pada GRPC pada authentication dan authorization validasi dilakukan pada setiap potongan data, begitu pula untuk enkripsi data setiap potongan data harus dienkripsi terpisah, dan untuk stream request validasi dilakukan berkali-kali untuk 1 stream.

    - Authentication: dengan mekanisme autentikasi seperti token JWT (JSON Web Tokens) untuk memverifikasi identitas pengguna atau layanan yang terhubung. Dengan ini dapat memastikan bahwa hanya klien yang sah yang dapat mengakses layanan.

    - Authorization: logika otorisasi di dalam layanan gRPC dapat diimplementasikan untuk memastikan bahwa hanya aksi yang diizinkan yang dapat dilakukan oleh pengguna yang memiliki hak akses yang sesuai.

    - Enkripsi Data: dengan menggunakan protokol seperti Transport Layer Security (TLS). Ini akan melindungi data dari penyadapan dan memastikan kerahasiaan serta integritas data saat transit.

3. Tantangaan saat handling bi-directional streaming pada rust grpc adalah 
    - pada handling operasi asinkronus karena harus menghindari data races dan memastikan sinkronisasi data yang akurat karena network interruptions, reconnections, dan timeout dapat mempengaruhi data, selain itu data juga dipengaruhi oleh back pressure yaitu kondisi ketika stream yang membuat atau mengirim data lebih cepat daripada penerima data.
    - harus menghindari memory leaks atau isu performance yang harus dilakukan dengan membuat management resource yang baik.
    - Jika koneksi berlangsung lama dapat menyulitkan load balancing karena banyak koneksi yang tidak dapat diputus
    - Perlu menangani error terutama yang mengganggu stream pesan
    - concurrency juga menjadi tantangan

4. Kelebihan dan kelemahan menggunakan tokio_stream::wrappers::ReceiverStream untuk respon streaming pada grpc services
    + Kelebihan :
    - Memungkinkan untuk stream secara asinkronus yang dapat membuat lebih efisien dan mempercepat pengolahan data integrasi sehingga dapat responsif
    - Lebih fleksibel karena dapat digunakan dengan berbagai tipe receiver seperti mpsc::Receiver or oneshot::Receiver
    - API memudahkan integrasi data seperti yang disebutkan sebelumnya karena menyediakan map, filter, and for_each dan juga memudahkan melakukan manipulasi data.

    + Kekurangan :
    - Program menjadi lebih kompleks karena menggunakan stream asinkronus terutama bagi yang belum terbiasa dengan paradigma asinkronus.
    - Tingkat kesulitan mempelajari nya meningkat karena mempelajari konsep futures, streams, and Tokio's runtime butuh usaha lebih.
    - Stream asinkronus berpotensi menyebabkan overhead.

5. Code rust grpc untuk code reuse dan modularity, promoting maintainability and extensibility over time dapat dibuat dengan cara :
    - membuat kode dengan modul-modul yang meng-enkapsulasi fungsi fungsi
    - menggunakan protokol buffer
    - interface yang dibuat secara konsisten atau dengan pemisahan concern yang memungkinkan untuk extensibility dan pemeliharaan kode
    - dengan menggunakan dependensi injection yang sesuai dengan open close principle dengan penambahan seperti implementasi fitur baru tetapi tidak melakukan modifikasi kode yang sudah ada. Sehingga kode lebih modular
    - menggunakan trait untuk mendefinisikan interface dan abstract pada berbagai implementasi.Yang memungkinkan untuk code reuse dan extensibility.

6. Pada implementasi MyPayment langkah tambahan yang perlu dihandle untuk logika proses pembayaran yang lebih complex adalah:
    - Metode process_payment diubah menjadi server streaming untuk mengirimkan respon secara bertahap kepada klien agar lebih efisien.
    - Membuat validasi data dan exception handling untuk mencegah dari serangan sql injection, xss, dan lain-lain

7. Dampak dari penggunaan gRpc sebagai protocol komunikasi pada keseluruhan arsitektur dan design distributed systems, khususnya pada term interoperability dengan teknologi dan platform lain adalah
    - gRPC dapat menangani koneksi secara otomatis sehingga sangat berdampak yaitu mempermudah akses modul melalui http
    - gRPC secara otomatis menghubungkan pemanggilan metode yang ditentukan melalui file proto sehingga klien merasa seperti benar-benar interaksi secara langsung
    - memudahkan konektivitas dan operasi antar teknologi, platform, dan sistem terdistribusi
    - meningkatkan interoperability pada berbagai teknologi dan platform

8. HTTP/2 yang merupakan underlying protocol dari gRPC memiliki kelebihan dan kekurangan dibandingkan HTTP/1 dan websocket untuk REST API
    + Kelebihan :
    - HTTP/2 memungkinkan banyak request dan respon yang dikirim dan diterima bersamaan dalam 1 koneksi sehingga mengurangi latency dan meningkatkan efisiensi
    - HTTP/2 mengurangi overhead dan meningkatkan performa network
    - Memaksimalkan penggunaan bandwidth
    
    + Kekurangan :
    - Tidak semua server dan browser mendukung HTTP/2 
    - Menggunakan HTTP/2 dalam implementasi lebih kompleks dari pada menggunakan HTTP/1
    - HTPP/2 harus menjaga connection state pada client dan server sehingga menggunakan lebih banyak sumberdaya dan dapat menyebabkan overhead terutama untuk koneksi yang berdurasi panjang

    + dibandingkan dengan websocket walaupun HTTP/2 mendukung komunikasi 2 arah websocket lebih sesuai  untuk skenario komunikasi real-time dan full-duplex.

9. Model respon request dari REST API kontras dengan kemampuan streaming bi-directional gRPC dalam hal komunikasi real time dan responsif karena :
    - gRPC lebih cocok untuk komunikasi real time, full duplex, dan respon yang lebih cepat dibandingkan REST API karena hanya memerlukan 1 koneksi untuk semua request dan response, sedangkan REST API harus membuat koneksi baru untuk setiap request dan kemungkinan dapat menyebabkan overhead jika digunakan untuk komunikasi real time.
    - REST API sederhana karena metode HTTP standar dan statelessness, sedangkan gRPC lebih kompleks dalam implementasinya yang dapat mempengaruhi maintainability aplikasi 

10. Implikasi gRPC dengan pendekatan skema menggunakan protocol buffer dibandingkan dengan nature tanpa skema dari JSON pada REST API payload yang lebih fleksibel adalah :
    - Pendekatan gRPC dengan protocol buffer memungkinkan mendefinisikan skema dan tipe data yang ketat, sehingga menghindari dari kesalahan data dan meningkatkan keamanan data, sedangkan JSON dalam REST API lebih fleksibel tetapi berpotensi terjadi kesalahan data, kurangnya keamanan, dan kesalahan definisi skema
    - JSON memungkinkan interoperability dalam bahasa dan platform karena bersifat universal
    - gRPC lebih diuntungkan untuk skenario yang memprioritaskan strong typing dan kinerja yang kuat, sementara JSON dalam REST API sesuai untuk situasi yang mengutamakan fleksibilitas dan keterbacaan

    Sehingga pemilihan antara gRPC dengan Protocol Buffers dan JSON harus mempertimbangkan prioritas aplikasi dan kebutuhan akan ketatnya definisi skema dan keamanan data
