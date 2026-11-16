# Hasil Solver Bug — Sesi #12
**Mata Kuliah:** Pemrograman Web Lanjut  
**Praktik:** Debugging dan Pengembangan Aplikasi Web  

Repository ini berisi solusi lengkap dan perbaikan bug untuk **Kasus 01**, **Kasus 02**, dan **Kasus 03**.

---

## Ringkasan Perbaikan Bug

### 📁 Kasus 01 — CRUD Manajemen Buku
1. **Routing (`routes/web.php`)**:
   - Memperbaiki route POST `books.simpan` menjadi `books.store` sesuai dengan route yang dipanggil di form view `create.blade.php`.
   - Mengubah method route destroy dari `Route::put` menjadi `Route::delete` karena form Blade menggunakan `@method('DELETE')`.
2. **Controller (`BookController.php`)**:
   - Memperbaiki variabel yang dikirim ke view `books.index` dari `compact('daftarBuku')` menjadi `compact('books')` agar sesuai dengan `@forelse ($books as $book)` di Blade.
   - Menambahkan validasi request (`title`, `author`, `published_year`) dan memanggil `Book::create($validated)` agar data buku tersimpan ke database.
3. **Hasil**: Halaman index dapat diakses normal, form penambahan tervalidasi dan tersimpan, serta tombol hapus berfungsi dengan benar.

---

### 📁 Kasus 02 — Relasi Buku dan Kategori
1. **Migration (`create_books_table.php`)**:
   - Menambahkan foreign key constraint: `$table->foreignId('category_id')->constrained('categories')->cascadeOnDelete();` agar integritas relasi terjamin di database.
2. **Model Relationship**:
   - `Book.php`: Memperbaiki relasi ke Kategori menjadi `belongsTo(Category::class)` (sebelumnya keliru `hasMany`).
   - `Category.php`: Memperbaiki relasi ke Buku menjadi `hasMany(Book::class)` (sebelumnya keliru `belongsTo`).
3. **Controller & Blade**:
   - `BookController.php`: Memperbaiki eager loading dari `Book::with('categories')` menjadi `Book::with('category')`.
   - `create.blade.php`: Memperbaiki nilai `<option value="{{ $category->id }}">` (sebelumnya bernilai string nama).
   - `index.blade.php`: Menampilkan nama kategori dengan `{{ $book->category->name ?? '-' }}` alih-alih menampilkan ID angka `category_id`.
4. **Hasil**: Data buku terhubung sempurna dengan kategori, pilihan kategori di form tersimpan dengan benar, dan tabel menampilkan nama kategori relasinya.

---

### 📁 Kasus 03 — Pencarian dan Filter Buku
1. **Routing (`routes/web.php`)**:
   - Menambahkan penamaan route `->name('books.index')`.
2. **Controller (`BookController.php`)**:
   - Memperbaiki nama input keyword dari `$request->search` menjadi `$request->input('keyword')` sesuai form Blade.
   - Memperbaiki nama input kategori dari `$request->kategori` menjadi `$request->input('category')` sesuai select Blade.
   - Mengubah query pencarian dari exact match `=` menjadi partial search `LIKE %keyword%` dan mengelompokkan `title` serta `author` dalam closure `where(function($q) ...)` agar operator `OR` tidak merusak filter `AND category_id`.
   - Memperbaiki pengiriman data ke Blade dari `['hasilBuku' => $books]` menjadi `compact('categories', 'books')`.
3. **Blade (`index.blade.php`)**:
   - Mengubah `value="{{ old('keyword') }}"` menjadi `value="{{ request('keyword') }}"` agar query pencarian tetap bertahan di input field saat filter/pencarian dijalankan.
   - Menjaga dropdown kategori tetap terpilih sesuai request filter yang aktif.
4. **Hasil**: Pencarian fleksibel berdasarkan judul maupun penulis, filter kategori berfungsi independen maupun bersamaan, dan query filter tetap tersimpan di form setelah submit.

---

## Panduan Pengujian

Untuk menguji setiap kasus pada proyek Laravel:
1. Salin folder `app`, `database`, `resources`, dan `routes` dari kasus terkait ke direktori proyek Laravel.
2. Konfigurasi database di file `.env`.
3. Jalankan migrasi dan seeder:
   ```bash
   php artisan optimize:clear
   php artisan migrate:fresh --seed
   php artisan serve
   ```
4. Akses melalui browser pada `http://127.0.0.1:8000/books`.
