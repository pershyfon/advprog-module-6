## Reflection
### Commit 1: handle_connection method
Method `handle_connection` bertanggung jawab untuk membaca steam line TCP yang masuk menggunakan BufReader baris demi baris sampai menemukan baris kosong, yang menandakan akhir dari header HTTP Request. Kemudian mencetak baris-baris yang terkumpul sebagai Request.
### Commit 2: new code on handle_conenction
Berikut adalah fungsi dari kode baru. Setelah `handle_connection` membaca isi dari file `hello.html` menggunakan `fs::read_to_string()`, HTTP Request yang terbaca akan dikirimkan kembali ke klien menggunakan `write_all()` melalui stream line TCP. Permintaan yang berhasil akan menunjukkan `status_line 200 OK`.
![img.png](assets/images/img.png)
### Commit 3: refactor for validating request and selective responding
Saya me-refactor method `handle_connection` dengan conditional. Jika request klien adalah `GET / HTTP/1.1` yang menunjukkan request untuk path root ('/') maka akan diberikan response `200 OK` dan mengembalikan `hello.html`. Namun, jika selain path root, akan diberikan respons `404 NOT FOUND` dan mengembalikan `404.html`.
![img.png](assets/images/img2.png)
### Commit 4: slow request
Modifikasi pada method `handle_connection` ini membuat pembacaan baris HTTP Request menggunakan pola match untuk memeriksa jenis request. Jika request cocok dengan `GET / sleep HTTP/1.1` menunjukkan request untuk tidur selama 10 detik menggunakan `thread::sleep(Duration::from_secs(10))`. Setelah tidur 10 detik, status_line menjadi `HTTP/1.1 200 OK` dan mengembalikan `hello.html`. Untuk request jenis lain, masih sama seperti sebelumnya.
### Commit 5: threadpool
Cara kerja `ThreadPool`:
1. `ThreadPool` diinisialisasi dengan ukuran jumlah thread yang bekerja dalam pool. Saluran `mpsc::channel` akan menjadi komunikasi antara thread utama (yang mengirimkan pekerjaan) dan thread pekerja pada vektor. Untuk setiap pekerja, dibuat struktur Worker baru yang meneruskan ID dan clone dari ujung penerima saluran.
2. Setiap thread pekerja dibuat dengan sebuah penutup yang berjalan dalam loop tak terbatas. Di dalam loop, pekerja menunggu pekerjaan dikirimkan oleh thread utama melalui saluran dan ketika mendapat pekerjaan akan dieksekusi.
3. `Arc<Mutex<mpsc::Receiver<Job>>>` pada semua thread pekerja memungkinkan beberapa thread mengakses ujung penerima saluran dengan aman. `Mutex` memastikan bahwa hanya satu thread pekerja yang mendapat sebuah pekerjaan dalam satu waktu. Saluran `mpsc` memungkinkan beberapa thread untuk mengirimkan pekerjaan secara bersamaan sambil tetap memastikan bahwa setiap pekerjaan hanya diproses oleh 1 thread pekerja
4. Hal ini untuk memungkinkan berjalannya multi-threaded tanpa menghadapi isu concurrency
### Bonus: build instead of new
https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html menyarankan untuk menggunakan metode `build` daripada `new` saat menginisialisasi  ThreadPool karena `new` dapat menyebabkan kesalahan jika jumlah thread yang diberikan terlalu kecil. Untuk itu, metode `new` diganti dengan `build`, yang mengembalikan `Result`. Kemudian, ketika nilai tersebut dikembalikan ke pemanggil, itu dapat di-unwrapped untuk mendapatkan nilai dari hasil eksekusi.