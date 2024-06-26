## Kebutuhan

- NodeMCU
- Board extender
- Kabel USB
- Sensor Ultrasonik
- Relay
- Buzzer
- Kabel jumper
- Jack DC adapter
- Adaptor 9V
- Obeng kecil
- Laptop
- Bracket Sensor (?)

### Modul 1: Merancang Rangkaian NodeMCU

1. **Pasang NodeMCU ke Board Extender**
   - Pasangkan NodeMCU ke dalam board extender.

2. **Hubungkan Sensor Ultrasonik**
   - Sambungkan 4 kabel jumper ke masing-masing pinout pada sensor ultrasonik: VCC, TRIG, ECHO, dan GND.
   - Hubungkan pin VCC ke pin 5V pada board extender.
   - Hubungkan pin GND ke pin GND pada board extender.
   - Hubungkan pin TRIG ke pin D5 pada board extender.
   - Hubungkan pin ECHO ke pin D6 pada board extender.

3. **Hubungkan Relay**
   - Sambungkan 3 kabel jumper ke masing-masing pinout pada relay: VCC, GND, dan IN.
   - Hubungkan pin VCC ke pin 3V pada board extender.
   - Hubungkan pin GND ke pin GND pada board extender.
   - Hubungkan pin IN ke pin D7 pada board extender.

4. **Hubungkan Buzzer ke Relay**
   - Sambungkan kabel hitam dari buzzer ke pin NO pada relay dan kencangkan menggunakan obeng.
   - Sambungkan kabel merah dari buzzer ke kutub positif pada jack DC adapter.

5. **Hubungkan Kabel Negatif**
   - Pasangkan kabel tunggal ke kutub negatif pada jack DC adapter.
   - Hubungkan ujung lain dari kabel tunggal ke pin COM pada relay.

6. **Hubungkan Adaptor 9V**
   - Colokkan jack DC adapter ke konektor adaptor 9V.

7. **Berikan Daya ke NodeMCU**
   - Colokkan konektor adaptor 9V ke board extender untuk memberikan daya ke NodeMCU.

## Catatan

- Pastikan semua sambungan sudah aman dan benar sebelum menyalakan adaptor.
- Gunakan obeng kecil untuk mengencangkan sambungan kabel pada relay dan jack DC agar tidak mudah lepas.

---

### Modul 2: Pembuatan Kode Arduino

1. **Library Import**:
    - Impor library yang dibutuhkan untuk koneksi WiFi, MQTT, JSON, dan kontrol servo (meskipun servo tidak digunakan di sini).

    ```cpp
    #include <ESP8266WiFi.h>
    #include <PubSubClient.h>
    #include <ArduinoJson.h>
    ```

2. **Mendefinisikan Pin**:
    - Tentukan pin yang digunakan untuk sensor ultrasonik dan relay.

    ```cpp
    #define TRIG D5
    #define ECHO D6
    #define RELAY D7
    ```

3. **Deklarasi Variabel**:
    - Deklarasikan variabel global `condition`.

    ```cpp
    int condition;
    ```

4. **Detail Koneksi WiFi**:
    - Tentukan SSID dan password WiFi yang akan digunakan untuk koneksi.

    ```cpp
    const char* ssid = "Rexa";
    const char* password = "AryaFadly2014";
    ```

5. **Detail Koneksi MQTT**:
    - Tentukan alamat broker MQTT, username, password, dan port. Disini kita menggunakan broker MQTT yang bersifat publik, yaitu Broker Emqx.

    ```cpp
    const char* mqtt_server = "broker.emqx.io";
    const char* mqtt_username = "emqx";
    const char* mqtt_password = "public";
    const int mqtt_port =1883;
    ```

6. **Inisialisasi WiFi dan MQTT Client**:
    - Buat objek untuk WiFi dan MQTT client.

    ```cpp
    WiFiClient espClient;
    PubSubClient client(espClient);
    ```

7. **Fungsi Koneksi WiFi**:
    - Buat fungsi untuk menghubungkan ke WiFi.

    ```cpp
    void setup_wifi() {
      delay(10);
      Serial.print("\nConnecting to ");
      Serial.println(ssid);

      WiFi.mode(WIFI_STA);
      WiFi.begin(ssid, password);

      while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
      }
      randomSeed(micros());
      Serial.println("\nWiFi connected\nIP address: ");
      Serial.println(WiFi.localIP());
    }
    ```

8. **Fungsi Koneksi ke MQTT Broker**:
    - Buat fungsi untuk menghubungkan ke broker MQTT.

    ```cpp
    void reconnect() {
      while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        String clientId = "ESP8266Client-";   
        clientId += String(random(0xffff), HEX);
        if (client.connect(clientId.c_str(), mqtt_username, mqtt_password)) {
          Serial.println("connected");
        } else {
          Serial.print("failed, rc=");
          Serial.print(client.state());
          Serial.println(" try again in 5 seconds");
          delay(5000);
        }
      }
    }
    ```

9. **Fungsi Publish Pesan ke MQTT**:
    - Buat fungsi untuk mengirim pesan ke broker MQTT.

    ```cpp
    void publishMessage(const char* topic, String payload, boolean retained) {
      if (client.publish(topic, payload.c_str(), true))
        Serial.println("Message published [" + String(topic) + "]: " + payload);
    }
    ```

10. **Setup Fungsi**:
    - Konfigurasi pin, koneksi WiFi, dan pengaturan MQTT server.

    ```cpp
    void setup() {
      pinMode(TRIG, OUTPUT);
      pinMode(ECHO, INPUT);
      pinMode(RELAY, OUTPUT);
      digitalWrite(RELAY, LOW);
      Serial.begin(9600);
      while (!Serial) delay(1);
      setup_wifi();
      client.setServer(mqtt_server, mqtt_port);
    }
    ```

11. **Loop Fungsi**:
    - Periksa koneksi MQTT, baca data dari sensor ultrasonik, kirim data ke broker MQTT, dan kendalikan relay berdasarkan jarak.

    ```cpp
    void loop() {
      if (!client.connected()) reconnect();
      client.loop();

      digitalWrite(TRIG, LOW);
      delayMicroseconds(2);
      digitalWrite(TRIG, HIGH);
      delayMicroseconds(10);
      digitalWrite(TRIG, LOW);

      long duration = pulseIn(ECHO, HIGH);
      float distance = (duration * 0.0343) / 2;

      Serial.print("Jarak: ");
      Serial.print(distance);
      Serial.println(" cm");

      if (distance >= 30) {
        digitalWrite(RELAY, HIGH);
        condition = 1;
      } else {
        digitalWrite(RELAY, LOW);
        condition = 0;
      }

      Serial.print("Condition: ");
      Serial.println(condition);

      String msgStr = String(distance) + "," + String(condition);
      byte arrSize = msgStr.length() + 1;
      char msg[arrSize];

      Serial.print("PUBLISH DATA:");
      Serial.println(msgStr);
      msgStr.toCharArray(msg, arrSize);
      publishMessage("water_data", msg, true);
      msgStr = "";

      delay(1000);
    }
    ```

### Penjelasan Kode:

- **Inisialisasi WiFi dan MQTT**:
    - Inisialisasi dilakukan di fungsi `setup()`, mengatur koneksi WiFi, dan menyetel server MQTT.

- **Fungsi loop()**:
    - Fungsi ini berjalan terus-menerus, memeriksa koneksi MQTT, membaca data dari sensor ultrasonik, dan mengirim data tersebut ke broker MQTT.
    - Sensor ultrasonik digunakan untuk mengukur jarak, dan jika jarak >= 30 cm, relay diaktifkan dan kondisi diatur ke `1` (aman). Jika jarak < 30 cm, relay dimatikan dan kondisi diatur ke `0` (berbahaya).

#--------------------------------------------------------------------------------------

### Modul 3: Programming Block Aplikasi MQTT

1. **Inisialisasi Global Variable**:
    - Buat global variable `data` dan inisialisasi dengan `create empty list`. Ini digunakan untuk menyimpan data yang diterima dari broker MQTT.

    ```blocks
    initialize global data to create empty list
    ```

2. **Event Button Click**:
    - Buat event handler untuk ketika tombol diklik. Event ini akan mengecek apakah sudah terkoneksi dengan MQTT.
    - Jika terkoneksi, akan memutuskan koneksi. Jika tidak, akan mencoba menghubungkan.

    ```blocks
    when button.Click do
      if call UrsPahoMqttClient1.IsConnected
      then call UrsPahoMqttClient1.Disconnect
      else call UrsPahoMqttClient1.Connect CleanSession true
    ```

3. **Event MQTT Connection State Changed**:
    - Buat event handler untuk menangani perubahan status koneksi MQTT.
    - Jika terkoneksi, akan mengubah teks tombol menjadi "Disconnect", teks koneksi menjadi "Tekan untuk Memutus Koneksi", dan subscribe ke topik `water_data`.
    - Jika tidak terkoneksi, akan mengubah teks tombol menjadi "Connect" dan teks koneksi menjadi "Tekan untuk Terkoneksi dengan Sensor".

    ```blocks
    when UrsPahoMqttClient1.ConnectionStateChanged do
      if get UrsPahoMqttClient1.IsConnected
      then set button.Text to "Disconnect"
           set tekskoneksi.Text to "Tekan untuk Memutus Koneksi MQTT."
           call UrsPahoMqttClient1.Subscribe Topic "water_data" QoS 0
      else set button.Text to "Connect"
           set tekskoneksi.Text to "Tekan untuk Terkoneksi dengan MQTT."
    ```

4. **Event MQTT Message Received**:
    - Buat event handler untuk menangani pesan yang diterima dari broker MQTT.
    - Memeriksa apakah topik adalah `water_data`, jika ya, memproses pesan.
    - Pisahkan data yang diterima menggunakan `split text` dan simpan dalam global variable `data`.
    - Set label `jarak` untuk menampilkan jarak dalam cm dari elemen pertama dari list.
    - Jika kondisi aman (nilai pada data = `1`), set label `kondisi` menjadi "Safe". Jika tidak aman (nilai pada data = `0`), set label `kondisi` menjadi "Danger".

    ```blocks
    when UrsPahoMqttClient1.MessageReceived do
      if get Topic = "water_data"
      then set global data to split text get Message at ","
           set jarak.Text to join select list item list get global data index 1 " cm"
           if select list item list get global data index 2 = "1"
           then set kondisi.Text to "Safe"
           else set kondisi.Text to "Danger"
    ```

### Penjelasan Mengenai Koneksi Aplikasi dengan MQTT:

1. **Menghubungkan ke MQTT Broker**:
    - Pastikan MQTT broker yang digunakan bisa diakses oleh aplikasi. Pada contoh di gambar, digunakan broker publik `broker.emqx.io`.
    - Gunakan `UrsPahoMqttClient1.Connect` untuk mencoba menghubungkan ke broker MQTT.

2. **Memutuskan Koneksi dari MQTT Broker**:
    - Gunakan `UrsPahoMqttClient1.Disconnect` untuk memutuskan koneksi dari broker MQTT ketika tombol diklik dan sudah terkoneksi.

3. **Subscribing ke Topik**:
    - Setelah terhubung ke broker, aplikasi akan subscribe ke topik `water_data` untuk menerima data.

4. **Menerima Pesan**:
    - Ketika pesan diterima di topik `water_data`, event `MessageReceived` akan dipanggil dan memproses pesan tersebut.

5. **Menampilkan Data**:
    - Data yang diterima akan ditampilkan di label `jarak` dan `kondisi` sesuai dengan nilai yang diterima dari pesan MQTT.

### Jelasin di latar apa itu broker emqx
#--------------------------------------------------------------------------------------

### Modul 4: Ujicoba NodeMCU dan Aplikasi

1. **Hubungkan NodeMCU ke Laptop**
   - Sambungkan kabel USB ke NodeMCU.
   - Sambungkan ujung kabel USB yang lain ke port USB di laptop Anda.

2. **Unggah Program ke NodeMCU**
   - Buka Arduino IDE.
   - Muat program yang telah dibuat sebelumnya ke dalam Arduino IDE.
   - Pilih papan (board) dan port yang benar di Arduino IDE.
   - Klik tombol "Upload" untuk mengunggah program ke NodeMCU.

3. **Putuskan Koneksi NodeMCU dari Laptop**
   - Setelah proses unggah selesai, lepaskan kabel USB dari laptop.

4. **Nyalakan NodeMCU**
   - Colokkan kedua adaptor 9V ke stopkontak untuk memberikan daya ke NodeMCU.

5. **Buka Aplikasi Deteksi Banjir**
   - Buka aplikasi deteksi banjir yang telah dibuat sebelumnya.
   - Tekan tombol "Connect" untuk membuat koneksi.

6. **Uji Sensor Ultrasonik**
   - Periksa apakah nilai jarak sudah ditampilkan.
   - Uji sensor dengan menghalanginya menggunakan tangan atau benda lain untuk melihat perubahan jarak yang terukur.

7. **Akhiri Pengujian**
   - Setelah pengujian selesai, cabut kedua adaptor dari stopkontak untuk memutus aliran daya ke NodeMCU.

## Catatan

- Pastikan semua sambungan sudah aman sebelum menyalakan NodeMCU.
- Jika nilai jarak tidak ditampilkan, periksa kembali sambungan dan program yang diunggah.

---
