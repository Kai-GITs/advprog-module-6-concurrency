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
