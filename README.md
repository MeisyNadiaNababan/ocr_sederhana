**NAMA LENGKAP : Meisy Nadia Nababan**
<br>**KELAS : 3F**
<br>**NIM : 2341760031**
<br>**JOBSHEET – APLIKASI OCR SEDERHANA  DENGAN  FLUTTER**

--------------------------------------------------------------------------------------------------------------------------------------

**4.1. Langkah 1: Buat Proyek Baru**

Buka terminal, lalu jalankan:

<p align="center">
  <img src="images/01.png" width="400">
</p>

**4.2. Langkah 2: Tambahkan Plugin**

Buka file pubspec.yaml, lalu tambahkan dependensi berikut di bawah bagian dependencies:
```dart
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  google_mlkit_text_recognition: ^0.15.0
  camera: ^0.11.0
  path_provider: ^2.1.3
  path: ^1.9.0
```

<p align="center">
  <img src="images/02.png" width="400">
</p>

Simpan file lalu jalankan 

<p align="center">
  <img src="images/03.jpg" width="400">
</p>

**4.3. Langkah 3: Tambahkan Izin Kamera (Android)**

Buka file: android/app/src/main/AndroidManifest.xml

Tambahkan baris berikut di dalam tag <manifest>, sebelum <application>:

```dart
<uses-permission android:name="android.permission.CAMERA" />
```

<p align="center">
  <img src="images/04.jpg" width="400">
</p>

**4.4. Langkah 4: Buat Struktur Folder**

Di dalam folder lib/, buat struktur berikut:

<p align="center">
  <img src="images/05.jpg" width="400">
</p>

5. KODE PROGRAM

5.1. File: lib/main.dart
```dart
    import 'package:flutter/material.dart';
    import 'screens/splash_screen.dart';

    void main () {
    runApp (const MyApp()) ;
    }

    class MyApp extends StatelessWidget {
    const MyApp ({super.key}) ;

    @override
    Widget build (BuildContext context) {
        return MaterialApp (
        title : 'OCR Sederhana',
        theme : ThemeData (primarySwatch: Colors.blue),
        home : const SplashScreen (),
        debugShowCheckedModeBanner : false,
        );
    }
    }
```
5.2. File: lib/screens/splash screen.dart
```dart
    import 'dart:async';
    import 'package:flutter/material.dart';
    import 'home_screen.dart';

    class SplashScreen extends StatefulWidget {
    const SplashScreen({super.key});

    @override
    State<SplashScreen> createState() => _SplashScreenState();
    }

    class _SplashScreenState extends State<SplashScreen> {
    @override
    void initState() {
        super.initState();
        Timer(const Duration(seconds: 2), () {
        Navigator.pushReplacement(
            context,
            MaterialPageRoute(builder: (_) => const HomeScreen()),
        );
        });
    }

    @override
    Widget build(BuildContext context) {
        return Scaffold(
        backgroundColor: Colors.blue,
        body: Center(
            child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: const [
                CircularProgressIndicator(color: Colors.white),
                SizedBox(height: 20),
                Text(
                'OCR Scanner',
                style: TextStyle(color: Colors.white, fontSize: 24),
                ),
            ],
            ),
        ),
        );
    }
    }
```
5.3. File: lib/screens/home screen.dart
```dart
    import 'package:flutter/material.dart';
    import 'scan_screen.dart';

    class HomeScreen extends StatelessWidget {
    const HomeScreen({super.key});

    @override
    Widget build(BuildContext context) {
        return Scaffold(
        appBar: AppBar(title: const Text('Menu Utama')),
        body: Center(
            child: ElevatedButton(
            onPressed: () {
                Navigator.push(
                context,
                MaterialPageRoute(builder: (_) => const ScanScreen()),
                );
            },
            child: const Text('Mulai Scan Teks'),
            ),
        ),
        );
    }
    }
```
5.4. File: lib/screens/scan screen.dart
```dart
    import 'dart:io';
    import 'package:flutter/material.dart';
    import 'package:camera/camera.dart';
    import 'package:google_mlkit_text_recognition/google_mlkit_text_recognition.dart';
    import 'package:path_provider/path_provider.dart';
    import 'result_screen.dart';

    late List<CameraDescription> cameras;

    class ScanScreen extends StatefulWidget {
    const ScanScreen({super.key});

    @override
    State<ScanScreen> createState() => _ScanScreenState();
    }

    class _ScanScreenState extends State<ScanScreen> {
    CameraController? _controller; // gunakan nullable agar aman
    late Future<void> _initializeControllerFuture;

    @override
    void initState() {
        super.initState();
        _initCamera();
    }

    /// Inisialisasi kamera
    void _initCamera() async {
        try {
        cameras = await availableCameras();
        _controller = CameraController(
            cameras.first,
            ResolutionPreset.medium,
        );

        _initializeControllerFuture = _controller!.initialize();
        await _initializeControllerFuture;

        if (mounted) {
            setState(() {});
        }
        } catch (e) {
        debugPrint('Error initializing camera: $e');
        }
    }

    @override
    void dispose() {
        _controller?.dispose();
        super.dispose();
    }

    /// Proses OCR dari file gambar
    Future<String> _ocrFromFile(File imageFile) async {
        final inputImage = InputImage.fromFile(imageFile);
        final textRecognizer = TextRecognizer(script: TextRecognitionScript.latin);
        final RecognizedText recognizedText =
            await textRecognizer.processImage(inputImage);
        textRecognizer.close();
        return recognizedText.text;
    }

    /// Ambil foto lalu pindah ke halaman hasil
    Future<void> _takePicture() async {
        if (_controller == null) return;

        try {
        await _initializeControllerFuture;

        if (!mounted) return;

        ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(
            content: Text('Memproses OCR, mohon tunggu...'),
            duration: Duration(seconds: 2),
            ),
        );

        final XFile image = await _controller!.takePicture();
        final ocrText = await _ocrFromFile(File(image.path));

        if (!mounted) return;
        Navigator.push(
            context,
            MaterialPageRoute(
            builder: (_) => ResultScreen(ocrText: ocrText),
            ),
        );
        } catch (e) {
        if (!mounted) return;
        ScaffoldMessenger.of(context)
            .showSnackBar(SnackBar(content: Text('Error: $e')));
        }
    }

    @override
    Widget build(BuildContext context) {
        // Jika controller belum siap, tampilkan loading
        if (_controller == null || !_controller!.value.isInitialized) {
        return const Scaffold(
            body: Center(
            child: CircularProgressIndicator(),
            ),
        );
        }

        return Scaffold(
        appBar: AppBar(
            title: const Text('Kamera OCR'),
            centerTitle: true,
            backgroundColor: Colors.deepPurple,
        ),
        body: Column(
            children: [
            Expanded(
                child: AspectRatio(
                aspectRatio: _controller!.value.aspectRatio,
                child: CameraPreview(_controller!),
                ),
            ),
            Padding(
                padding: const EdgeInsets.all(16.0),
                child: ElevatedButton.icon(
                style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.deepPurple,
                    padding:
                        const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
                    shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                    ),
                ),
                onPressed: _takePicture,
                icon: const Icon(Icons.camera_alt, color: Colors.white),
                label: const Text(
                    'Ambil Foto & Scan',
                    style: TextStyle(color: Colors.white),
                ),
                ),
            ),
            ],
        ),
        );
    }
    }
```
5.5. File: lib/screens/result screen.dart
```dart
    import 'package:flutter/material.dart';

    class ResultScreen extends StatelessWidget {
    final String ocrText;

    const ResultScreen({super.key, required this.ocrText});

    @override
    Widget build(BuildContext context) {
        return Scaffold(
        appBar: AppBar(title: const Text('Hasil OCR')),
        body: Padding(
            padding: const EdgeInsets.all(16.0),
            child: SingleChildScrollView(
            child: SelectableText(
                ocrText.isEmpty
                    ? 'Tidak ada teks ditemukan.'
                    : ocrText.replaceAll('\n', ' '),
                style: const TextStyle(fontSize: 18),
            ),
            ),
        ),
        );
    }
    }
```

--------------------------------------------------------------------------------------------------------------------------------------

**6. TUGAS PRAKTIKUM**

1. Jalankan aplikasi di emulator atau HP.
<p align="center">
  <img src="images/06.jpg" width="400">
</p>
2. Lakukan scan terhadap teks cetak (misal: buku, koran, atau layar HP).
<p align="center">
  <img src="images/07.jpg" width="400">
</p>
3. Amati hasil OCR yang muncul.
<p align="center">
  <img src="images/08.jpg" width="400">
</p>
4. Jawab pertanyaan berikut:

a. Apakah semua teks terbaca dengan akurat? Mengapa?

Tidak semua teks terbaca dengan akurat.
Hal ini karena akurasi hasil OCR (Optical Character Recognition) dipengaruhi oleh beberapa faktor, seperti:
- Kualitas gambar (buram, gelap, atau terlalu terang).
- Jenis dan ukuran font yang tidak jelas atau terlalu dekoratif.
- Adanya noise atau bayangan pada gambar.
- Tata letak teks yang tidak rapi (miring, terpotong, atau bertumpuk).

b. Apa kegunaan fitur OCR dalam kehidupan sehari-hari?

Fitur OCR berguna untuk mengubah teks dari gambar menjadi teks digital agar bisa disalin, diedit, atau dicari.
Beberapa kegunaan dalam kehidupan sehari-hari antara lain:
- Mempermudah input data dari dokumen cetak ke komputer.
- Membantu membaca dokumen fisik bagi tunanetra dengan teknologi text-to-speech.
- Memudahkan digitalisasi arsip, nota, dan kuitansi agar lebih mudah disimpan dan dicari kembali.

c. Sebutkan 2 contoh aplikasi nyata yang menggunakan OCR!

- Google Lens – dapat mengenali teks pada foto, papan nama, atau dokumen, lalu menyalinnya ke teks digital.
- Adobe Scan – aplikasi pemindai dokumen yang mengubah foto menjadi file PDF dengan teks yang bisa disalin atau dicari.