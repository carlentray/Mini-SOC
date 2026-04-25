# Technical Documentation: Integrated Mini SOC System with Wazuh
Dokumentasi ini merinci spesifikasi teknis, arsitektur *cyber defense*, dan metodologi simulasi serangan pada sistem *Security Operations Center* (SOC) berbasis **Wazuh** yang diimplementasikan dalam lingkungan *hybrid*.

---

## 1. Pengenalan Platform Wazuh & Ecosystem
Wazuh adalah platform keamanan *open-source* yang menggabungkan kemampuan **EDR (Endpoint Detection and Response)** dan **XDR (Extended Detection and Response)**. Dalam proyek ini, Wazuh berperan sebagai pusat komando yang mengintegrasikan pemantauan integritas file, deteksi *rootkit*, hingga audit konfigurasi keamanan. Platform ini dipilih karena kemampuannya dalam melakukan korelasi data secara kompleks yang tidak dapat dilakukan oleh perangkat lunak keamanan konvensional, memberikan perlindungan aktif yang melampaui kemampuan antivirus standar.

## 2. Blueprint Arsitektur Sistem (Hybrid Infrastructure)
Sistem ini mengadopsi model arsitektur **Distributed Manager-Agent** yang diabstraksikan melalui lapisan virtualisasi jaringan internal:
* **Target Endpoint:** Windows Host (Windows 10/11) yang dipasang Wazuh Agent sebagai sensor primer untuk pengumpulan data *low-level*.
* **Central Manager Engine:** Ubuntu 22.04 LTS yang berjalan di atas **WSL2 (Windows Subsystem for Linux)** bertindak sebagai pusat pemrosesan dan analisis data.
* **Communication Protocol:** Jalur komunikasi diamankan dengan enkripsi **TLS/SSL** melalui port `1514/TCP`. Hal ini memastikan bahwa data log yang ditransfer dari Windows ke Manager bersifat *tamper-proof* dan tidak dapat diintersepsi oleh pihak ketiga di dalam jaringan.

## 3. Technical Data Pipeline (Data Life Cycle)
Proses pengolahan data log diatur dalam alur kerja *asynchronous* untuk menjamin skalabilitas dan performa sistem tanpa membebani CPU host:
1.  **Event Collection:** Wazuh Agent secara *real-time* melakukan *tailing* pada file biner `.evtx` (Windows Event Logs).
2.  **Log Normalization:** Wazuh Manager menerima log mentah dan melakukan *decoding* menjadi format terstruktur (JSON) agar setiap variabel dapat dianalisis secara atomik.
3.  **Heuristic & Rule Engine:** Data dicocokkan dengan ribuan *ruleset* XML yang telah dikonfigurasi. Di sini, sistem menentukan tingkat bahaya (Alert Level) berdasarkan korelasi perilaku (*behavioral analysis*).
4.  **Indexing & Archiving:** Data yang telah diperkaya (Enriched Data) disimpan dalam *sharded index* pada Wazuh Indexer untuk kebutuhan investigasi cepat dan audit jangka panjang.

## 4. Konfigurasi Monitoring & SCA (Security Configuration Assessment)
Kami melakukan kustomisasi mendalam pada berkas `ossec.conf` untuk memperluas jangkauan deteksi pada vektor serangan yang sering diabaikan:
* **Event Channel Audit:** Pemantauan khusus pada saluran `Security`, `System`, dan `Application` untuk menangkap anomali pada *driver* maupun *background services*.
* **FIM (File Integrity Monitoring):** Mengaktifkan sensor *real-time* pada direktori sistem kritis (`C:\Windows\System32`). Setiap perubahan pada *checksum* file akan memicu alarm seketika.
* **SCA Policy:** Implementasi audit kebijakan keamanan otomatis untuk mendeteksi konfigurasi OS yang lemah, seperti penggunaan protokol yang sudah usang atau kebijakan *password complexity* yang tidak memadai.

## 5. Mekanisme Deteksi Intrusi (IDS/IPS Capability)
Sistem ini tidak hanya bersifat pasif, tetapi memiliki kemampuan deteksi berbasis tanda tangan (*signature*) dan pola perilaku (*behavioral pattern*):
* **Rootkit Detection:** Secara berkala memindai direktori sistem untuk mencari *hidden files*, *malicious hooks*, atau proses yang mencoba menyembunyikan diri dari *task manager*.
* **Anomalous Behavior:** Jika terdapat aktivitas yang tidak biasa (seperti proses sistem yang tiba-tiba menjalankan koneksi *outbound* ke IP asing), Wazuh akan langsung meningkatkan level waspada.
* **Log Correlation:** Mampu menghubungkan dua kejadian yang tampak tidak berhubungan (misal: login sukses di jam tidak wajar diikuti oleh eksekusi perintah administratif) sebagai satu rangkaian serangan terencana.

## 6. Skenario Simulasi Serangan (Security Testing)
Validasi sistem dilakukan melalui simulasi 3 vektor serangan nyata yang merepresentasikan ancaman siber modern:
* **Defense Evasion (T1070):** Simulasi penghapusan *digital evidence* melalui perintah `Clear-EventLog`. Wazuh menangkap upaya penghapusan jejak ini sebagai Alert Level 5.
* **Network Brute Force (T1110):** Simulasi serangan tebakan password massal pada *network share*. Sistem mendeteksi lonjakan kegagalan login (Event ID 4625) secara eksponensial dalam waktu singkat.
* **Privilege Escalation (T1078):** Simulasi penguasaan sistem dengan injeksi akun baru ke dalam grup *Administrators*. Aktivitas ini memicu **Alert Kritis Level 12**, kategori ancaman tertinggi dalam sistem kami.

## 7. Forensic Analysis & MITRE ATT&CK Mapping
Setiap insiden yang terdeteksi di Dashboard telah diperkaya dengan standar intelijen keamanan global:
* **Digital Evidence:** Menyediakan *Full Log Trace* yang mencakup instruksi perintah asli dari penyerang, memudahkan kebutuhan forensik pasca-insiden.
* **Tactic Alignment:** Pemetaan otomatis ke taktik MITRE ATT&CK (*Persistence, Privilege Escalation, Defense Evasion*). Ini membantu tim keamanan memahami "tujuan akhir" dari seorang penyerang.
* **User Attribution:** Identifikasi akun pengguna yang dikompromikan secara akurat, mencakup informasi *source IP* dan *process ID*.

## 8. Visualisasi Dashboard & Manajemen Insiden
Dashboard Wazuh berfungsi sebagai pusat observasi visual yang menyajikan data kompleks dalam format yang intuitif:
* **Real-Time Analytics:** Grafik tren serangan dan sebaran alert berdasarkan tingkat keparahan (Low, Medium, High, Critical).
* **Top Alerts Data:** Memberikan daftar entitas (akun atau proses) yang paling sering melakukan pelanggaran keamanan untuk kebutuhan *profiling*.
* **Response Workflow:** Memungkinkan analis untuk melakukan *drill-down* dari grafik global hingga ke baris log spesifik hanya dalam hitungan detik, mempercepat *Mean Time To Respond* (MTTR).

---


## Keunggulan Proyek (Competitive Edge)
Apa yang membuat projek kelompok kami lebih baik daripada projek kelompok lain?
| Aspek Keunggulan | Deskripsi Teknis | Nilai Tambah |
| :--- | :--- | :--- |
| **Pipeline Latency** | Optimasi *buffer* komunikasi dan *polling interval* antara Windows Host dan WSL2. | Deteksi *near real-time* (< 15 detik), meminimalisir *blind spot* waktu dibandingkan implementasi standar. |
| **Severity Precision** | Kustomisasi korelasi log (Event ID 4732 & 4720) untuk memicu **Alert Level 12** pada *Privilege Escalation*. | Memastikan insiden kritis mendapatkan prioritas penanganan tertinggi dan tidak tenggelam dalam *noise* log biasa. |
| **Network Integration** | Implementasi manajemen IP dinamis dan konfigurasi *interface* virtual pada *subnet* WSL2. | Stabilitas koneksi Agent-Manager tetap terjaga tanpa perlu konfigurasi ulang setiap kali sistem melakukan *reboot*. |
| **Intelligence Context** | Integrasi *native* Framework **MITRE ATT&CK** pada setiap kategori serangan yang disimulasikan. | Memberikan wawasan taktis (Tactic & Technique) bagi analis SOC untuk memahami pola serangan secara komprehensif. |
| **Forensic Capability** | Kemampuan ekstraksi *payload* log hingga ke level instruksi spesifik di *Command Line Interface*. | Menyediakan barang bukti digital (digital evidence) yang valid dan akurat untuk kebutuhan investigasi pasca-insiden. |
