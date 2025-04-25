
## Penanganan File PDF dengan XSS di Dalamnya pada Backend Laravel

Pada aplikasi berbasis web, penting untuk menangani file yang diupload dengan hati-hati agar menghindari potensi serangan XSS (Cross-Site Scripting) atau skrip berbahaya yang mungkin terkandung di dalamnya, seperti pada file PDF. Dalam tutorial ini, kita akan membahas cara menangani file PDF yang mungkin mengandung skrip JavaScript berbahaya menggunakan backend Laravel.

### 1. Install Package `pdf-parser`

Untuk memulai, kita perlu menginstal **pdf-parser** yang akan digunakan untuk membaca dan memproses file PDF yang diupload. Paket ini memungkinkan kita untuk mengakses konten dalam file PDF dan memeriksa apakah ada elemen JavaScript yang dapat mengeksekusi kode berbahaya.

Instalasi dapat dilakukan melalui **Composer**:
```bash
composer require smalot/pdfparser
```

Paket ini akan membantu kita memeriksa isi file PDF secara efisien dan memberikan fleksibilitas dalam menangani file PDF.

### 2. Buat Fungsi Pengecekan JavaScript dalam PDF

Setelah berhasil menginstal paket `pdf-parser`, langkah selanjutnya adalah membuat fungsi yang akan memeriksa apakah file PDF yang diupload mengandung skrip JavaScript. Dalam konteks ini, kita akan mencari string `app.alert` atau pola skrip JavaScript lainnya dalam teks PDF.

Berikut adalah fungsi PHP untuk memeriksa apakah file PDF mengandung JavaScript:

```php
function checkForJavascriptInPdf($filePath)
{
    // Pastikan file ada
    if (!file_exists($filePath)) {
        throw new \Exception("File tidak ditemukan.");
    }

    // Inisialisasi parser PDF
    $parser = new \Smalot\PdfParser\Parser();

    try {
        // Parsing file PDF
        $pdf = $parser->parseFile($filePath);
        $text = $pdf->getText(); // Ambil teks dari PDF

        // Cek apakah ada skrip JavaScript dalam file PDF
        if (strpos($text, 'app.alert') !== false || preg_match('/\/JS\s*\(.*\)/s', $text)) {
            return false; // Jika ditemukan skrip JavaScript, kembalikan false
        }
        return true; // Tidak ada JavaScript, kembalikan true
    } catch (\Exception $e) {
        // Jika terjadi kesalahan dalam parsing, log error
        Log::error('Error parsing PDF: ' . $e->getMessage());
        return false; // Kembalikan false jika terjadi error
    }
}
```

Penjelasan fungsi:
- **Cek file ada**: Fungsi ini pertama-tama memastikan bahwa file yang dimaksud ada di lokasi yang diberikan.
- **Parsing PDF**: Menggunakan `pdf-parser`, file PDF akan diproses dan diubah menjadi teks.
- **Pencarian JavaScript**: Fungsi akan mencari apakah ada string atau pola JavaScript (`app.alert` atau `/JS`) di dalam teks PDF yang menunjukkan adanya kode JavaScript.
- **Menangani kesalahan**: Jika ada kesalahan dalam parsing PDF (misalnya file rusak), fungsi akan menangkap dan mencatatnya.

### 3. Cara Menggunakan Fungsi Pengecekan dalam Proses Upload

Setelah fungsi pengecekan siap, langkah selanjutnya adalah mengintegrasikannya dalam alur upload file di Laravel. Saat pengguna mengupload file PDF, kita akan menjalankan pengecekan untuk memastikan bahwa file tersebut tidak mengandung JavaScript berbahaya sebelum disimpan di server.

Berikut adalah contoh penerapan pengecekan file PDF dalam fungsi upload:

```php
public function upload(Request $request)
{
    // Validasi file PDF yang diupload
    $request->validate([
        'file' => 'required|mimes:pdf|max:10240', // Memastikan file adalah PDF dan maksimal 10MB
    ]);

    // Menyimpan file sementara di server
    $file = $request->file('file');
    $filePath = $file->getRealPath(); // Mendapatkan path sementara file

    // Memeriksa apakah file PDF mengandung JavaScript
    if (!checkForJavascriptInPdf($filePath)) {
        return response()->json(['error' => 'File PDF mengandung skrip JavaScript yang berbahaya'], 400);
    }

    // Jika file aman, lanjutkan proses upload
    $path = $file->storeAs('uploads', $file->getClientOriginalName());

    return response()->json(['success' => 'File berhasil diupload', 'path' => $path]);
}
```

Penjelasan kode:
- **Validasi file PDF**: Menggunakan validasi Laravel, hanya file dengan ekstensi `pdf` yang dapat diupload. Ukuran file juga dibatasi hingga 10MB.
- **Menyimpan file sementara**: Setelah file diterima, kita memperoleh path file sementara di server.
- **Pengecekan JavaScript**: Sebelum file disimpan secara permanen, kita memeriksa apakah file tersebut mengandung JavaScript berbahaya menggunakan fungsi `checkForJavascriptInPdf()`.
- **Proses upload**: Jika file aman, maka file akan disimpan di direktori `uploads` dengan nama asli file.

### 4. Penanganan Kesalahan dan Keamanan

Jika file PDF mengandung JavaScript berbahaya, aplikasi akan memberikan respon dengan kode status `400` dan pesan error yang menyatakan bahwa file tersebut mengandung skrip berbahaya. Hal ini mencegah eksekusi skrip yang dapat merusak aplikasi atau mencuri data pengguna.

Selain itu, fungsi pengecekan akan menangkap dan mencatat setiap kesalahan yang terjadi selama proses parsing file. Hal ini sangat penting untuk debugging dan memastikan sistem berjalan dengan baik.

---

Dengan mengikuti langkah-langkah di atas, Anda dapat memastikan bahwa file PDF yang diupload ke backend Laravel aman dari potensi ancaman JavaScript berbahaya (XSS). Teknik ini meningkatkan keamanan aplikasi Anda dengan memeriksa dan menangani file yang mengandung skrip berbahaya sebelum file tersebut diproses lebih lanjut.
