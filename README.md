### Langkah-langkah Pembuatan Programming Block:

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
           set tekskoneksi.Text to "Tekan untuk Memutus Koneksi."
           call UrsPahoMqttClient1.Subscribe Topic "water_data" QoS 0
      else set button.Text to "Connect"
           set tekskoneksi.Text to "Tekan untuk Terkoneksi dengan Sensor."
    ```

4. **Event MQTT Message Received**:
    - Buat event handler untuk menangani pesan yang diterima dari broker MQTT.
    - Memeriksa apakah topik adalah `water_data`, jika ya, memproses pesan.
    - Pisahkan data yang diterima menggunakan `split text` dan simpan dalam global variable `data`.
    - Set label `jarak` untuk menampilkan jarak dalam cm dari elemen pertama dari list.
    - Jika kondisi aman (nilai kedua dari list adalah `0`), set label `kondisi` menjadi "Safe". Jika tidak aman (nilai kedua adalah `1`), set label `kondisi` menjadi "Danger".

    ```blocks
    when UrsPahoMqttClient1.MessageReceived do
      if get Topic = "water_data"
      then set global data to split text get Message at ","
           set jarak.Text to join select list item list get global data index 1 " cm"
           if select list item list get global data index 2 = "0"
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

Dengan langkah-langkah dan blok pemrograman di atas, aplikasi Anda akan dapat terhubung ke broker MQTT, menerima pesan, dan menampilkan data sesuai yang diharapkan.

## Jelasin di latar apa itu broker emqx
