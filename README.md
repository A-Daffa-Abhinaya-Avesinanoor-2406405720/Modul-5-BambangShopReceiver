# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. Pada tutorial ini, `RwLock<Vec<Notification>>` diperlukan karena daftar notification disimpan sebagai data bersama yang bisa diakses oleh banyak request atau thread secara bersamaan. Saat ada notification yang baru masuk, data perlu ditulis ke dalam `Vec`. Di saat yang sama, bisa saja ada proses lain yang sedang read seluruh isi notification untuk ditampilkan. Kalau akses seperti ini tidak sinkron, data bisa jadi saling bertabrakan. `RwLock` dipakai karena mengizinkan banyak proses read data secara bersamaan selama tidak ada proses yang sedang di-write. Ini sesuai dengan kasus receiver, karena biasanya isi notification akan lebih sering dibaca daripada diubah.

     Tidak memakai `Mutex` karena `Mutex` lock semua akses sekaligus, read maupun write. Kalau ada satu proses yang hanya ingin read daftar notification, proses read lain tetap harus menunggu. Untuk kasus ini, `Mutex` memang tetap aman, tetapi kurang efisien dibanding `RwLock`. Dengan `RwLock`, banyak read bisa berjalan bersama, dan hanya operasi write saja yang mendapat akses eksklusif.

2. Rust tidak mengizinkan kita memodifikasi isi variabel `static` dengan bebas seperti di Java karena Rust sangat ketat dalam menjaga memory safety dan mencegah data race. Jika data global bisa diubah dari mana saja tanpa aturan yang jelas, maka pada program yang berjalan paralel akan sangat mudah terjadi bug yang sulit dilacak. Karena itu, Rust memaksa untuk menggunakan mekanisme yang aman dan eksplisit seperti `Mutex`, `RwLock`, atau struktur sinkronisasi lain saat ingin membuat data global yang bisa berubah.

    `lazy_static` dipakai karena data seperti `Vec` dan `DashMap` tidak bisa langsung diinisialisasi sebagai `static` biasa dengan cara yang fleksibel seperti di Java. Rust hanya mengizinkan `static` biasa untuk nilai yang benar-benar aman diinisialisasi secara tetap. Sementara itu, `Vec`, `DashMap`, dan struktur sinkronisasi butuh proses inisialisasi runtime. Jadi, `lazy_static` membantu membuat variabel global yang baru dibuat saat pertama kali dipakai, tetapi tetap aman dan sesuai aturan Rust.

#### Reflection Subscriber-2

1. Saya belajar bahwa `lib.rs` berisi fondasi bersama untuk aplikasi ini. Ada `APP_CONFIG` untuk membaca configuration seperti URL instance receiver, URL publisher, dan nama instance. Saya juga belajar bahwa ada `REQWEST_CLIENT` yang dipakai sebagai HTTP client global, sehingga service tidak perlu membuat client baru terus-menerus. Selain itu, ada `Result`, `Error`, dan `compose_error_response()` yang membuat penanganan error menjadi lebih rapi dan konsisten. Dari situ saya jadi lebih paham bahwa file ini penting karena menjadi tempat utility yang dipakai oleh banyak bagian aplikasi.

2. Observer pattern memudahkan untuk menambahkan subscriber baru karena publisher tidak perlu tahu detail implementasi setiap receiver. Selama subscriber mendaftarkan dirinya ke product type tertentu, publisher hanya perlu menyimpan data subscriber lalu mengirim notification saat ada perubahan yang relevan. Artinya, menambah receiver baru cukup mudah: jalankan instance baru, lakukan subscribe, lalu instance itu langsung bisa ikut menerima update. Namun, kalau yang ditambah adalah lebih dari satu instance Main app, situasinya jadi tidak sesimpel itu. Setiap Main app bisa saja punya data subscriber yang berbeda-beda kalau tidak memakai penyimpanan bersama. Jadi, menambah banyak subscriber masih mudah, tetapi menambah banyak publisher atau Main app biasanya butuh sinkronisasi data yang lebih rapi, misalnya database bersama atau mekanisme koordinasi lain.

3. Untuk saat ini, saya belum membuat test sendiri, tetapi saya mencoba menggunakan Postman untuk membantu pengujian manual. Menurut saya, Postman sangat berguna karena memudahkan saya mengirim request subscribe, unsubscribe, receive, dan list tanpa harus menulis program tambahan. Dengan Postman, saya juga lebih mudah melihat response JSON dan memastikan apakah notification benar-benar masuk ke receiver yang benar. Saya belum menambahkan dokumentasi Postman collection, tetapi kalau itu dilakukan, menurut saya akan sangat membantu untuk kerja kelompok karena semua anggota bisa memakai request yang sama dengan lebih konsisten.
