# **Praktikum 3: Jaringan Nirkabel dan Komunikasi HTTP**

## **3.1 Pengantar**

Pada Modul 1 dan Modul 2, kita telah mendalami cara mengontrol perangkat keras dasar dan mengambil keputusan secara lokal (*Edge Computing*). Kini, kita akan membuka potensi utama dari *Internet of Things* (IoT), yaitu "Internet" itu sendiri. Pada praktikum ini, mahasiswa akan mempelajari cara menghubungkan NodeMCU ESP8266 ke jaringan Wi-Fi lokal, lalu menjadikannya sebagai *Web Server* yang mampu menyajikan antarmuka halaman HTML, maupun sebagai *HTTP Client* yang secara aktif mengirim perintah ke perangkat (*server*) lain. Kemampuan ini menjadi landasan arsitektur pertukaran data cerdas antar perangkat.

## **3.2 Tujuan Pembelajaran**

* Mahasiswa memahami perbedaan mode jaringan pada ESP8266 (*Station, Access Point*, dan *Dual Mode*).  
* Mahasiswa mampu menggunakan NodeMCU ESP8266 sebagai **Web Server** untuk menyajikan antarmuka HTML tertanam (*embedded HTML*).  
* Mahasiswa mampu menggunakan NodeMCU ESP8266 sebagai **HTTP Client** untuk mengirimkan instruksi ke *server* lain.  
* Mahasiswa memahami implementasi arsitektur *Client-Server* menggunakan metode interaksi HTTP GET.

## **3.3 Teori Dasar**

### **3.3.1 Mode Jaringan ESP8266**

Cip ESP8266 dilengkapi dengan modul perangkat keras Wi-Fi internal (mendukung protokol 802.11 b/g/n). Konfigurasi Wi-Fi ini dapat dioperasikan dalam tiga mode berbeda, tergantung kebutuhan topologi IoT:

1. **Station (STA):** NodeMCU bertindak murni sebagai perangkat klien yang mencari dan menumpang koneksi ke *Router/Access Point* Wi-Fi eksternal (seperti terhubung ke Wi-Fi kampus atau *Tethering* HP). Ini adalah mode paling umum di IoT.  
2. **Access Point (AP):** NodeMCU memancarkan sinyal SSID sendiri dan bertindak sebagai *router* mandiri, sehingga HP atau laptop dapat terhubung ke NodeMCU meskipun tidak ada jaringan internet di sekitar.  
3. **Dual Mode (STA + AP):** NodeMCU bertindak sebagai pemancar mandiri sekaligus sebagai klien jaringan luar secara bersamaan.

### **3.3.2 Arsitektur Client-Server dan Komunikasi HTTP**

Dalam dunia web, komunikasi data dikendalikan oleh protokol **HTTP (HyperText Transfer Protocol)**. Komunikasi ini berjalan dengan metode **Request** (Permintaan) dan **Response** (Tanggapan).

* **Server:** Perangkat penyedia layanan. Ia mendengarkan (*listen*) secara pasif pada *port* 80. Ketika ada permintaan masuk, ia memprosesnya lalu membalas dengan dokumen HTML atau status *text*.  
* **Client:** Perangkat peminta layanan. Ia bertugas proaktif menghubungi *server* pada alamat IP (*IP Address*) atau tautan tertentu.

**HTTP GET** adalah metode komunikasi HTTP paling dasar, yang digunakan *Client* untuk meminta data dari server. Di ekosistem IoT, metode GET juga secara kreatif dimanfaatkan sebagai *trigger* (pemicu) aksi mekanis. Caranya adalah dengan memprogram *Client* untuk mengakses *URL/Endpoint* khusus (*API*) seperti `http://192.168.1.10/relay/on`, yang akan diterjemahkan oleh *Server* sebagai perintah eksekusi untuk menghidupkan perangkat keras.

## **3.4 Persiapan Praktikum**

### **3.4.1 Kebutuhan Perangkat Keras**

Pastikan komponen berikut tersedia di meja Anda:
* *Board* NodeMCU ESP8266 (1 buah)  
* Kabel Micro USB (1 buah)  
* *Breadboard* (1 buah)  
* Sensor Suhu & Kelembapan (DHT11 atau DHT22) (1 buah)  
* Sensor LDR (1 buah)  
* Resistor 10k Ohm (1 buah, untuk pembagi tegangan LDR)  
* Modul Relay 1-Channel 5V (1 buah)  
* Kabel *Jumper Male-to-Male* dan *Male-to-Female* (secukupnya)

### **3.4.2 Persiapan Perangkat Lunak & Jaringan**

1. Pada Praktikum Bab ini, seluruh NodeMCU membutuhkan koneksi ke jaringan lokal Wi-Fi yang sama (satu *subnet*).
2. Sangat disarankan Anda menggunakan fitur *Tethering/Mobile Hotspot* dari ponsel Anda untuk memperlancar praktikum, mengingat jaringan Wi-Fi kampus seringkali menerapkan isolasi klien (*AP Isolation*) yang memblokir komunikasi antar NodeMCU.
3. Pastikan Anda mencatat nama **SSID** dan **Password** dari *Hotspot* Anda dengan akurat.

## **3.5 Praktikum**

### **3.5.1 Praktikum 1: NodeMCU sebagai *Web Server* (Menyajikan HTML)**

Skenario: Anda merakit NodeMCU sebagai server pintar. Modul ini membaca sensor DHT dan menampilkan suhunya di sebuah halaman web (UI), sekaligus menyediakan tombol di web tersebut untuk menghidupkan/mematikan alat via Modul Relay.

**Langkah Kerja Rangkaian:**
1. Hubungkan pin **DATA / OUT / S** DHT ke pin **D4 (GPIO 2)**. (Pastikan VCC DHT ke 3V3, GND ke GND). 
2. Hubungkan pin **IN / Signal** Relay ke pin **D6 (GPIO 12)**. (Pastikan VCC Relay ke VIN/VU NodeMCU, GND ke GND).

**Langkah Kerja Pemrograman:**
1. Buat berkas baru (*New Sketch*). Tuliskan kode program di bawah. **Jangan lupa mengganti variabel `ssid` dan `password`!**
2. *Catatan teknis:* Kita menggunakan teknik **Raw String Literal** C++ (`R"rawliteral(...)rawliteral"`) untuk menulis kode antarmuka HTML multibaris, sehingga kita tidak repot mengetikkan tanda kutip `""` di setiap baris HTML.

```cpp
#include <ESP8266WiFi.h>  
#include <ESP8266WebServer.h>  
#include <DHT.h>

const char* ssid = "NAMA_WIFI_ANDA";  
const char* password = "PASSWORD_WIFI_ANDA";

ESP8266WebServer server(80);

const byte dhtPin = 2;         
const byte relayPin = 12;      
DHT dht(dhtPin, DHT11);

const char index_html[] PROGMEM = R"rawliteral(  
<!DOCTYPE html>  
<html>  
<head>  
  <meta name="viewport" content="width=device-width, initial-scale=1">  
  <title>IoT Dashboard</title>  
  <style>  
    body { font-family: Arial; text-align: center; margin-top: 50px; }  
    button { padding: 15px 30px; font-size: 20px; border-radius: 8px; margin: 10px; cursor: pointer;}  
    .btn-on { background-color: #4CAF50; color: white; border: none; }  
    .btn-off { background-color: #f44336; color: white; border: none; }  
    .sensor-box { font-size: 24px; font-weight: bold; }  
  </style>  
</head>  
<body>  
  <h1>ESP8266 Web Server</h1>  
  <div class="sensor-box">  
    <p>Suhu Saat Ini: <strong>%TEMPERATURE%</strong> Celcius</p>  
  </div>  
  <h2>Kendali Relay</h2>  
  <a href="/relay/on"><button class="btn-on">ON</button></a>  
  <a href="/relay/off"><button class="btn-off">OFF</button></a>  
</body>  
</html>  
)rawliteral";

void handleRoot() {  
  String html = index_html; // Salin kerangka HTML ke variabel dinamis
  float t = dht.readTemperature();  
  
  // Mengganti teks placeholder %TEMPERATURE% dengan suhu nyata
  if (isnan(t)) {
    html.replace("%TEMPERATURE%", "--"); 
  } else {
    html.replace("%TEMPERATURE%", String(t)); 
  }
  
  server.send(200, "text/html", html);  
}

void handleRelayOn() {  
  digitalWrite(relayPin, HIGH); 
  server.sendHeader("Location", "/");   
  server.send(303);  
}

void handleRelayOff() {  
  digitalWrite(relayPin, LOW);   
  server.sendHeader("Location", "/");   
  server.send(303);  
}

void setup() {  
  Serial.begin(115200);  
  pinMode(relayPin, OUTPUT);  
  digitalWrite(relayPin, LOW);  
  dht.begin();  
    
  WiFi.mode(WIFI_STA);   
  WiFi.begin(ssid, password);  
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }  
  Serial.println("\nIP Address Server Anda: ");  
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);  
  server.on("/relay/on", handleRelayOn);  
  server.on("/relay/off", handleRelayOff);  
  server.begin();  
}

void loop() {  
  server.handleClient();  
}
```

3. **Upload** kode tersebut. Buka **Serial Monitor**, dan catat alamat IP yang didapat (contoh: `192.168.43.15`).
4. Hubungkan Laptop/HP Anda ke Wi-Fi yang sama, buka *browser* (Chrome/Safari), lalu ketikkan alamat IP tersebut. Klik tombol ON dan OFF di layar untuk mendengarkan *Relay* berbunyi *cetek*.

**Penjelasan Singkat Kode:**
* `ESP8266WebServer server(80);`: Menginisialisasi *library Server* agar terus mendengarkan panggilan di jalur lalu lintas standar web (*Port 80*).
* `html.replace("%TEMPERATURE%", String(t));`: Fitur manipulasi teks (*String*). Perintah ini secara cerdas akan menyapu dokumen HTML, mencari teks *placeholder* `%TEMPERATURE%`, dan menggantinya dengan variabel angka suhu yang didapat dari sensor aktual sebelum dikirim ke layar pengunjung. Inilah esensi dari halaman *web* dinamis.
* `server.on("/relay/on", handleRelayOn);`: Fungsi *Routing*. Jika ada klien mengunjungi tautan URL berakhiran `/relay/on`, server akan menjalankan aksi mengubah *state* Relay, lalu me- *redirect* layar pengunjung kembali ke halaman depan (`Location: /`) via kode `303`.
* `server.handleClient();`: Berada di dalam `loop()` untuk terus-menerus mengecek apakah ada permintaan pengunjung yang datang setiap milidetiknya.

### **3.5.2 Praktikum 2: NodeMCU sebagai *HTTP Client* (Meminta Layanan)**

Skenario: Anda sekarang memprogram NodeMCU kedua (atau meminta teman kelompok di sebelah Anda) untuk menjadi *Client*. Perangkat Klien ini dipasangi LDR. Jika LDR tertutup kegelapan, ia akan mengirim sinyal HTTP secara jarak jauh (via jaringan Wi-Fi) untuk menyalakan Relay di *Server* milik teman Anda (Praktikum 1). 

**Langkah Kerja Rangkaian:**
1. Rangkai sensor **LDR** dan **Resistor 10k Ohm** sebagai pembagi tegangan (seperti di Modul 2) lalu hubungkan jalur sinyalnya ke pin **A0**.

**Langkah Kerja Pemrograman:**
1. Buat berkas baru (*New Sketch*). 
2. Tuliskan kode program di bawah. **Ganti variabel `serverName` dengan IP Address NodeMCU Praktikum 1 milik teman sebelah Anda.** (Pastikan Anda menggunakan Wi-Fi yang sama persis).

```cpp
#include <ESP8266WiFi.h>  
#include <ESP8266HTTPClient.h>  
#include <WiFiClient.h>

const char* ssid = "NAMA_WIFI_ANDA";  
const char* password = "PASSWORD_WIFI_ANDA";

const char* serverName = "http://192.168.1.15/relay/on";   
const byte ldrPin = A0;

void setup() {  
  Serial.begin(115200);  
  WiFi.mode(WIFI_STA);  
  WiFi.begin(ssid, password);  
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }  
  Serial.println("\nClient Terhubung ke Wi-Fi!");  
}

void loop() {  
  int ldrValue = analogRead(ldrPin);  
    
  if ((WiFi.status() == WL_CONNECTED) && (ldrValue < 300)) {  
    WiFiClient client;  
    HTTPClient http;  
      
    http.begin(client, serverName);  
    int httpResponseCode = http.GET();  
      
    Serial.print("HTTP Response code: ");  
    Serial.println(httpResponseCode); 
      
    http.end();  
    delay(10000); 
  }  
  delay(2000);  
}
```

3. **Upload** kode tersebut, dan buka Serial Monitor. Saat LDR Anda di *Client* ditutup tangan hingga gelap, perhatikan *Relay* di NodeMCU *Server* milik teman Anda pasti akan menyala!

**Penjelasan Singkat Kode:**
* `WiFiClient client;` dan `HTTPClient http;`: Kelas pemanggil untuk memulai eksekusi permintaan *client* jarak jauh (mirip fungsi aplikasi *browser* di laptop).
* `http.begin(client, serverName);`: Menargetkan sasaran URL *server*.
* `int httpResponseCode = http.GET();`: Mengeksekusi ketukan pintu (*Request* HTTP GET). Jika Server merespons sukses, nilai variabel ini akan berisi angka standar jaringan seperti `200` (OK) atau `303` (Redirect). Jika URL rusak atau *Server offline*, variabel akan mereturn nilai negatif.
* `delay(10000);`: *Cooldown* panjang 10 detik ini ditambahkan agar *Client* tidak memborbardir (*flood/DDoS*) Server dengan permintaan `on` secara terus-menerus setiap detik ketika kondisinya sedang gelap.

## **3.6 Latihan**

Kerjakan soal berikut secara mandiri untuk menguji penalaran dan modifikasi jaringan.

*   **Latihan 1 (Pemahaman Jaringan):** Berdasarkan wawasan dari subbab 3.3.1, sebutkan secara logis mengapa kita menggunakan mode jaringan *Station (STA)* pada kedua praktikum di atas, dan bukan mode *Access Point (AP)*?
*   **Latihan 2 (Modifikasi Antarmuka & Telemetri):** Pada Praktikum 1, tampilan web saat ini baru menyajikan data suhu ruangan dan masih menggunakan dua tombol terpisah (ON dan OFF). Lakukan dua modifikasi berikut pada *Web Server*:
    1. **Menampilkan Kelembapan:** Tambahkan pembacaan kelembapan menggunakan fungsi `dht.readHumidity()`. Sisipkan placeholder `%HUMIDITY%` pada kerangka HTML di dalam tag `<div class="sensor-box">` (contoh: `<p>Kelembapan: <strong>%HUMIDITY%</strong> %</p>`), lalu perbarui fungsi `handleRoot()` menggunakan `html.replace("%HUMIDITY%", ...)` agar nilai kelembapan muncul di halaman *web*.
    2. **Tombol Dinamis Tunggal (*Toggle Button*):** Ubah antarmuka agar **hanya menggunakan 1 buah tombol tunggal** yang statusnya adaptif mengikuti kondisi *Relay* saat ini:
       * Jika Relay sedang **OFF**, tombol menampilkan tulisan **"NYALAKAN (ON)"** (misal: warna hijau/biru) dan mengarahkan aksi ke `/relay/on`.
       * Jika Relay sedang **ON**, tombol otomatis berubah menjadi tulisan **"MATIKAN (OFF)"** (warna merah) dan mengarahkan aksi ke `/relay/off`.  
       *(Petunjuk: Anda dapat menyematkan placeholder baru di HTML seperti `%RELAY_BUTTON%`, lalu di fungsi `handleRoot()`, periksa status pin relay menggunakan `digitalRead(relayPin)` sebelum mengganti placeholder tersebut dengan elemen tautan tombol yang sesuai).*  
    Tuliskan potongan baris kode HTML dan fungsi `handleRoot()` yang Anda ubah!
*   **Latihan 3 (Analisis Kode Client):** Pada Praktikum 2, *Serial Monitor Client* mencetak `HTTP Response code`. Analisislah apa yang akan dicetak oleh *Serial Monitor* Anda jika ternyata NodeMCU *Server* teman Anda mendadak kehilangan daya (*offline*)? (Silakan cabut daya Server dan lihat hasil erornya).
*   **Latihan 4 (Pengembangan Algoritma):** Anda ingin mendesain Klien Pendeteksi Suhu. Modifikasi *conditional block* `if` di Praktikum 2 sehingga *Client* mengirim permintaan (GET Request) **HANYA JIKA** sensor DHT membaca suhu melebihi 35 Celcius (bukan menggunakan LDR). Tulis baris algoritmanya.

## **3.7 Tugas: Pengembangan *Web Server* Cerdas dan Analisis Arsitektur**

Antarmuka (*User Interface*) IoT tidak boleh statis. Saat ini, *Web Server* kita tidak mampu memperbarui informasi suhunya secara otomatis ke layar pengunjung tanpa diperintah. Kita harus menyelesaikannya.

**Skenario Proyek (*Web Dashboard*):**
Anda akan bertindak sebagai *Full-Stack IoT Developer* untuk memutakhirkan kapabilitas *Web Server* pada Praktikum 1, sembari mengevaluasi limitasi arsitektur tradisional HTTP.

*   **Tujuan:** Mampu memanipulasi struktur kerangka HTML mentah yang tersimpan di memori NodeMCU untuk mencapai interaktivitas tampilan otomatis (*Auto-Refresh*).
*   **Tingkat Kesulitan:** Menengah
*   **Instruksi:**
    1. Lakukan modifikasi kode hanya pada sisi **NodeMCU Server (Praktikum 1)**.
    2. Modifikasi variabel Raw Literal `index_html`. Sisipkan perintah *Meta Refresh HTML* (sebuah Tag standar web kuno namun ampuh) persis di dalam cakupan *tag* `<head>` untuk memaksa *browser* memuat ulang halaman secara sekuensial setiap 5 detik.
    3. *Hint (Petunjuk):* Sintaks standar yang harus disisipkan adalah `<meta http-equiv="refresh" content="5">`.
    4. Setelah selesai, buka *browser*, dan diamati layarnya. Biarkan selama 20 detik tanpa menyentuh *mouse/keyboard*.
*   **Instruksi Analisis Arsitektur:**
    1. Setelah Auto-Refresh berjalan, cobalah tekan tombol *Relay ON / OFF* dari *browser*.
    2. Apa kelemahan visual/interaksi (*flickering*) yang Anda alami saat menggunakan metode *Meta Refresh* HTTP konvensional seperti ini untuk sistem yang mengklaim diri sebagai *Real-Time Dashboard*? Tulis evaluasi logis Anda di Laporan Praktikum (Anda bisa membahas masalah *bandwidth*, layar berkedip putih, atau potensi bentrokan jika Anda mengeklik tepat di detik ke-5).

*   **Kriteria Keberhasilan:**
    - Nilai suhu di *browser* otomatis berganti seiring suhu ruangan tanpa perlu mengeklik panah *refresh browser* secara manual.
    - Jawaban analisis mengenai kelemahan protokol HTTP GET statis tertuang jelas di dokumentasi tugas.

> **[!IMPORTANT]**
> **Instruksi Pengumpulan Tugas Praktikum**
> Seluruh pengerjaan praktikum dan tugas (meliputi *source code* `.ino` Web Server termodifikasi, tangkapan layar/rekaman UI *browser* yang membuktikan *Auto-Refresh*, serta laporan analisis arsitektur) harus dikumpulkan dengan cara melakukan **commit dan push ke repositori GitHub pribadi Anda masing-masing**.
> 
> Tautkan/kumpulkan *URL repositori GitHub* Anda pada sistem manajemen pembelajaran (LMS) kampus sebagai bukti penyelesaian bab ini.

## **3.8 Rangkuman**

Pada Bab ini, kita telah berhasil membebaskan sensor dari batasan isolasi kabel mikrokontroler dengan memperkenalkan protokol Wi-Fi 802.11 pada cip ESP8266. Pemahaman mengenai NodeMCU sebagai *Web Server* memampukan perangkat IoT membangun antarmuka web kecilnya sendiri di memori *flash* dan bereaksi terhadap panggilan *browser* eksternal. Sebaliknya, peran sebagai *HTTP Client* membuka pintu luas ke dunia luar; memampukan sensor mengirim peringatan otomatis, menembak modul lain, hingga menyetor data langsung ke basis data berbasis Web (seperti RESTful API). Kendati arsitektur murni HTTP memiliki limitasi responsivitas (*real-time issues*), protokol lintas *platform* ini tetap menjadi fondasi universal terkuat untuk menghubungkan jutaan mesin cerdas di dunia.
