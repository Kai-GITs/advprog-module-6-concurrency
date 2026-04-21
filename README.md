# Module 6 Concurrency

Repository ini adalah implementasi tutorial web server Rust untuk Modul 6 Advanced Programming tentang concurrency.

## Reflections

### Commit 1 Reflection notes

1. Pada tahap ini saya melihat bahwa `TcpListener` hanya bertugas menerima koneksi, sedangkan detail isi request lebih cocok dipindahkan ke `handle_connection`.
- Pemisahan ini membuat alur program lebih jelas karena `main` hanya fokus pada accept loop.
- Saat nanti logic respons bertambah, saya tidak perlu menumpuk semua langkah di satu fungsi.

2. `BufReader` dan rangkaian iterator pada `lines`, `map`, `take_while`, dan `collect` membantu membaca request HTTP sampai baris kosong penutup header.
- Dari output console saya bisa melihat bahwa browser mengirim lebih dari sekadar baris `GET`, misalnya `Host`, `User-Agent`, dan header lain.
- Bagian ini penting karena saya jadi paham server menerima request dalam bentuk teks mentah yang harus diparse sendiri.

3. `handle_connection` juga menjadi titik yang tepat untuk menambah behavior berikutnya, seperti mengirim HTML, validasi path, dan simulasi request lambat.
- Sebelum fungsi ini ada, program hanya tahu bahwa koneksi datang, tetapi belum memahami apa isi permintaannya.
- Setelah fungsi ini ditambahkan, program mulai bergerak dari sekadar listener menjadi web server sederhana yang bisa dikembangkan bertahap.

### Commit 2 Reflection notes

1. Pada milestone ini saya mulai melihat bentuk respons HTTP yang lebih lengkap karena server tidak cukup hanya membuka koneksi, tetapi juga harus mengirim status line, header, dan body.
- `Content-Length` menjadi penting agar browser tahu berapa banyak byte yang harus dibaca untuk body respons.
- Tanpa format respons yang benar, browser tidak akan merender halaman dengan konsisten.

2. Menyimpan halaman pada file `hello.html` membuat isi respons lebih mudah dibaca dan diubah dibanding menulis seluruh HTML sebagai string panjang di Rust.
- Saya bisa mengganti pesan halaman tanpa mengacak logic jaringan di `handle_connection`.
- Pendekatan ini juga menjadi dasar untuk pemisahan resource lain seperti halaman `404` pada milestone berikutnya.

3. Saya sengaja mengganti pesan contoh menjadi pesan milik saya sendiri agar screenshot benar-benar menunjukkan hasil dari environment saya.
- Ini membantu membedakan hasil kerja saya dari contoh dosen di modul.
- Dari tahap ini saya semakin paham bahwa web server sederhana pun sebenarnya hanya mengirim teks terstruktur yang lalu ditafsirkan browser sebagai halaman HTML.

![Commit 2 screen capture](/assets/images/commit2.png)

### Commit 3 Reflection notes

1. Perubahan paling penting pada tahap ini adalah server tidak lagi selalu mengirim `hello.html`, tetapi mulai memilih respons berdasarkan `request_line`.
- Dengan begitu path root mendapatkan halaman sukses, sedangkan path lain mendapatkan halaman `404`.
- Ini membuat server lebih mendekati perilaku HTTP yang benar walaupun masih sangat sederhana.

2. Saya mengikuti versi refactor karena pemisahan `status_line` dan `filename` membuat logic percabangan jauh lebih ringkas.
- Kalau pembacaan file dan pembentukan response ditulis ulang di tiap cabang, akan ada duplikasi yang mudah melebar saat endpoint bertambah.
- Setelah direfactor, yang berbeda hanya keputusan responsnya, sedangkan bagian umum seperti `read_to_string`, `Content-Length`, dan `write_all` tetap satu kali ditulis.

3. Penambahan `404.html` juga membantu memisahkan isi halaman error dari logic server.
- Saya bisa menulis pesan error yang lebih jelas tanpa mencampur HTML ke dalam percabangan Rust.
- Dari milestone ini saya belajar bahwa refactor bukan sekadar merapikan tampilan kode, tetapi juga mengurangi duplikasi dan memudahkan pengembangan endpoint berikutnya.

![Commit 3 screen capture](/assets/images/commit3.png)

### Commit 4 Reflection notes

1. Route `/sleep` menunjukkan masalah utama server single-threaded dengan sangat jelas.
- Ketika satu request masuk ke cabang yang tidur 10 detik, thread utama berhenti melayani request lain selama periode itu.
- Artinya bottleneck bukan hanya pada proses render HTML, tetapi pada fakta bahwa seluruh server masih berbagi satu thread eksekusi.

2. Saya juga melihat bahwa menambahkan endpoint lambat tidak perlu logic yang kompleks untuk memperlihatkan efek blocking.
- Satu `thread::sleep(Duration::from_secs(10))` saja sudah cukup untuk mensimulasikan operasi mahal seperti query lambat atau I/O lambat.
- Dari sini saya paham kenapa throughput server cepat turun ketika ada beberapa request berat yang datang bersamaan.

3. Tahap ini menjadi motivasi yang kuat untuk pindah ke thread pool pada milestone berikutnya.
- Selama semua request diproses serial oleh thread utama, request cepat seperti `/` tetap akan ikut antre di belakang request lambat seperti `/sleep`.
- Jadi masalah yang harus diselesaikan berikutnya bukan isi halaman, tetapi arsitektur concurrency pada server.

### Commit 5 Reflection notes

1. Perubahan inti pada milestone ini adalah request tidak lagi langsung ditangani oleh thread utama, tetapi dikirim ke `ThreadPool`.
- Thread utama sekarang fokus menerima koneksi, sementara worker di pool yang mengeksekusi closure `handle_connection`.
- Dengan pembagian ini, request cepat tidak harus menunggu request lambat selama masih ada worker lain yang idle.

2. Saya jadi lebih paham kenapa implementasi `ThreadPool` butuh `mpsc`, `Arc`, dan `Mutex` sekaligus.
- `mpsc::Sender<Job>` dipakai untuk mengantrekan pekerjaan dari `execute`.
- `Arc<Mutex<mpsc::Receiver<Job>>>` dipakai agar beberapa worker bisa berbagi receiver yang sama secara aman di banyak thread.
- Kombinasi ini membuat worker bisa mengambil job satu per satu tanpa data race.

3. `Worker` dan alias `Job` membuat desain pool lebih mudah dipahami.
- `Job` menyederhanakan tipe closure yang panjang menjadi satu alias yang jelas.
- `Worker` menyimpan identitas thread dan loop penerima job sehingga implementasi pool tidak menumpuk semua detail pada satu struct.
- Setelah melihat hasilnya, saya memahami bahwa thread pool bukan sekadar banyak thread, tetapi mekanisme pembatasan concurrency yang lebih terkontrol.

### Commit Bonus Reflection notes

1. Saya menambahkan `build` sebagai alternatif dari `new` supaya pembuatan `ThreadPool` bisa mengembalikan `Result` alih-alih langsung panic.
- Dengan `build`, caller bisa menangani kasus `size == 0` secara eksplisit.
- Ini membuat API lebih fleksibel kalau nanti thread pool dipakai pada konteks yang membutuhkan error handling yang rapi.

2. Saya tetap mempertahankan `new`, tetapi sekarang `new` menjadi wrapper yang memanggil `build(...).unwrap()`.
- Pendekatan ini menjaga kompatibilitas dengan tutorial utama yang memakai `ThreadPool::new(4)`.
- Di sisi lain, bonus ini menunjukkan bahwa API panic-based dan result-based bisa hidup berdampingan tanpa mengubah pemakaian yang sudah ada.

3. Perbandingan paling jelas antara keduanya ada pada perilaku error.
- `new(0)` cocok untuk asumsi bahwa ukuran nol adalah programmer error yang fatal.
- `build(0)` lebih cocok untuk alur yang menerima input dinamis dan ingin mengembalikan error terstruktur.
- Saya menambahkan test sederhana untuk membuktikan perbedaan tersebut secara langsung.
