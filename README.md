# 🌐 Pemrograman Internet of Things (IoT)

Repositori ini berisi modul panduan praktikum, materi teori dasar, skema rangkaian elektronika, dan kode program untuk mata kuliah/praktikum **Pemrograman Internet of Things (IoT)** berbasis mikrokontroler **NodeMCU ESP8266**.

---

## 🛠 Kebutuhan Perangkat

### Perangkat Keras (Hardware)
* NodeMCU ESP8266 (ESP-12E / ESP-12F)
* Kabel Micro-USB (mendukung transfer data)
* *Breadboard* (papan proyek)
* Kabel *Jumper* (*Male-to-Male*, *Male-to-Female*)
* LED 5mm (Merah / Kuning / Hijau)
* Resistor (220 $\Omega$, 10k $\Omega$)
* *Push Button*
* Sensor Cahaya LDR (*Light Dependent Resistor*)
* Sensor Suhu & Kelembaban (DHT11 atau DHT22)
* Modul Relay 5V / 3.3V dengan isolasi *Optocoupler*

### Perangkat Lunak (Software)
* [Arduino IDE](https://www.arduino.cc/en/software) (versi 1.8.x atau 2.x)
* Driver USB-to-UART: **CP210x** atau **CH340** (sesuai varian *board*)
* Pustaka (*Libraries*) Arduino yang diperlukan:
  * `esp8266 by ESP8266 Community` (Board Manager)
  * `DHT sensor library` by Adafruit
  * `Adafruit Unified Sensor`
  * `ESPAsyncWebServer`
  * `ESPAsyncTCP`
  * `ArduinoJson` (v6.x)

---

## 📝 Instruksi Pengumpulan Tugas

Bagi mahasiswa yang mengikuti kegiatan praktikum:
1. Kerjakan latihan dan tugas yang tertera pada bagian akhir masing-masing modul (`Praktikum-X.md`).
2. Buat folder terpisah untuk kode program praktikum Anda (misal: `tugas_prak01/`, `tugas_prak02/`, dst.).
3. Dokumentasikan foto/video rangkaian fisik serta tangkapan layar *Serial Monitor*.
4. Lakukan **commit dan push** seluruh berkas pekerjaan beserta laporan praktikum tertulis (.PDF) ke repositori GitHub pribadi Anda.
5. Kumpulkan tautan (*URL*) repositori GitHub Anda ke LMS kampus sesuai tenggat waktu yang ditentukan.
