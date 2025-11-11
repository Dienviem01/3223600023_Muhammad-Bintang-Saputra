# 🧠 Praktikum FreeRTOS Multi-Task dengan ESP32-S3

Repository ini berisi **program multitasking menggunakan FreeRTOS** pada **ESP32-S3**.  
Setiap komponen seperti **OLED**, **Servo**, **Stepper**, **Potensiometer**, **Rotary Encoder**, **Button**, **LED**, dan **Buzzer** dijalankan sebagai *task terpisah* di core 0 dan core 1.

Tujuannya adalah memahami **cara kerja multitasking dan pembagian tugas antar core** pada ESP32 menggunakan **FreeRTOS**.

---

## ⚙️ Konsep Dasar

ESP32-S3 memiliki **dua core prosesor** yang dapat bekerja secara paralel:

| Core | Fungsi Utama | Contoh Penggunaan |
|------|---------------|-------------------|
| **Core 0** | Menangani sistem dan proses latar belakang seperti WiFi/Bluetooth | OLED, Potensiometer, Button, LED, Buzzer |
| **Core 1** | Menjalankan program utama pengguna | Servo, Encoder, Stepper |

Dengan **FreeRTOS**, setiap komponen dapat dijalankan di task terpisah menggunakan fungsi `xTaskCreatePinnedToCore()` agar tidak saling mengganggu.

---

## 🧩 Struktur Program

### 🔸 Pembuatan Task
Setiap *task* memiliki struktur dasar seperti berikut:

```cpp
void TaskName(void *pvParameters) {
  for (;;) {
    // Kode yang dijalankan terus menerus
    vTaskDelay(pdMS_TO_TICKS(100));  // jeda tanpa menghentikan core
  }
}
```

### 🔸 Menjalankan Task di Core Tertentu

```cpp
xTaskCreatePinnedToCore(
  TaskName,          // Nama fungsi task
  "TaskName",        // Nama task untuk identifikasi
  4096,              // Ukuran stack (byte)
  NULL,              // Parameter (bisa NULL)
  1,                 // Prioritas task
  NULL,              // Handle task (bisa NULL)
  0                  // Core (0 atau 1)
);
```

Dengan begitu, kamu bisa mengatur task tertentu agar hanya berjalan di core 0 atau core 1.  
Misalnya:

```cpp
xTaskCreatePinnedToCore(TaskServo, "TaskServo", 4096, NULL, 1, NULL, 1);  // Core 1
xTaskCreatePinnedToCore(TaskOLED, "TaskOLED", 4096, NULL, 1, NULL, 0);    // Core 0
```

---

## 💡 Komponen dan Fungsi Task

| Komponen | Fungsi | Core | Deskripsi |
|-----------|---------|------|------------|
| **OLED** | Menampilkan status sistem | 0 | Menampilkan nilai sensor dan status task |
| **Potensiometer** | Membaca input analog | 0 | Mengontrol nilai servo atau kecepatan stepper |
| **Servo** | Menggerakkan aktuator | 1 | Mengikuti nilai potensiometer |
| **Stepper** | Mengatur arah dan langkah | 1 | Berputar sesuai mode task |
| **Rotary Encoder** | Input tambahan | 1 | Mengubah mode atau arah motor |
| **Button** | Kontrol manual | 0 | Start/stop task tertentu |
| **LED & Buzzer** | Indikator sistem | 0 | Memberi umpan balik visual dan audio |

---

## 🧠 Alur Eksekusi

1. **Setup awal**: inisialisasi komponen dan I2C/SPI/UART.  
2. **Task dibuat** dengan `xTaskCreatePinnedToCore()`.  
3. Setiap task berjalan independen di loop `for(;;)` masing-masing.  
4. Task tidak saling blok karena menggunakan `vTaskDelay()`.  
5. Data antar-task bisa dikirim via **queue** atau **global variable**.

---

## ⚙️ Contoh Alokasi Core & Prioritas

| Task | Core | Prioritas |
|-------|------|------------|
| TaskOLED | 0 | 1 |
| TaskPotensiometer | 0 | 1 |
| TaskButton | 0 | 2 |
| TaskLED | 0 | 1 |
| TaskBuzzer | 0 | 1 |
| TaskServo | 1 | 2 |
| TaskStepper | 1 | 1 |
| TaskEncoder | 1 | 2 |

---

## 🧾 Kesimpulan

Dengan memanfaatkan **FreeRTOS** pada **ESP32-S3**, setiap komponen dapat berjalan secara paralel tanpa mengganggu satu sama lain.  
Pendekatan ini penting untuk **sistem real-time**, **IoT**, dan **robotika**, di mana beberapa proses harus berjalan bersamaan dengan respons cepat.

---

📘 *Dibuat oleh M Bintang S — Praktikum FreeRTOS ESP32-S3*
