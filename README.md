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
