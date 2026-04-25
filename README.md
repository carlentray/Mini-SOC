Technical Documentation: Mini SOC Architecture & Cyber Threat Simulation
Dokumentasi ini merinci spesifikasi teknis, alur data, dan simulasi serangan pada sistem Security Operations Center (SOC) berbasis Wazuh.

### 1. Blueprint Arsitektur Sistem
Sistem mengadopsi arsitektur Manager-Agent yang berjalan secara cross-platform melalui virtualisasi jaringan internal.

Endpoint: Windows Host (Windows 10/11) sebagai target observasi.

Server Engine: Ubuntu 22.04 LTS via WSL2 yang menjalankan komponen Indexer, Manager, dan Dashboard.

Secure Tunnel: Protokol komunikasi TLS terenkripsi pada port 1514/TCP untuk menjamin integritas log saat ditransfer dari Host ke Manager.

### 2. Technical Data Pipeline
Alur pemrosesan log dilakukan secara asynchronous untuk memastikan performa sistem tetap optimal:

Ingestion: Wazuh Agent melakukan tailing pada file binari .evtx (Windows Event Logs).

Normalization: Wazuh Manager melakukan decoding dari log mentah menjadi format terstruktur (JSON).

Heuristic Analysis: Data dicocokkan dengan Security Configuration Assessment (SCA) dan ribuan ruleset XML.

Storage: Data yang telah diperkaya (Enriched Data) disimpan dalam sharded index pada Wazuh Indexer.

### 3. Konfigurasi Monitoring & SCA
Kami melakukan kustomisasi pada file ossec.conf untuk memperluas visibilitas deteksi pada area kritis:

Event Channel: Fokus pada saluran Security, System, dan Application.

FIM (File Integrity Monitoring): Pemantauan real-time terhadap perubahan file di direktori sensitif seperti C:\Windows\System32.

Policy Monitoring: Audit kebijakan password dan konfigurasi akun pengguna.

### 4. Skenario Simulasi Serangan (Security Testing)
Kami mensimulasikan 3 vektor serangan utama untuk menguji efektivitas deteksi sistem:

Defense Evasion (T1070): Eksekusi perintah Clear-EventLog untuk menghapus jejak digital. Sistem merespons dengan Alert Level 5 (Event ID 1102).

Network Brute Force (T1110): Upaya login massal pada network share menggunakan kredensial acak. Sistem mendeteksi anomali pada Event ID 4625.

Privilege Escalation (T1078): Injeksi akun baru ke dalam grup Built-in Administrators. Ini memicu Alert kritis Level 12 (Event ID 4732).

### 5. Forensic Analysis & MITRE Mapping
Setiap alert yang dihasilkan tidak hanya berupa notifikasi, tetapi mencakup:

Full Log Trace: Menyediakan bukti perintah asli yang diketikkan penyerang di terminal.

ATT&CK Framework: Pemetaan otomatis ke taktik Persistence, Privilege Escalation, dan Defense Evasion.

User Attribution: Identifikasi akun mana yang digunakan untuk mengeksekusi serangan.

### Keunggulan Proyek (Competitive Edge)
Apa yang membuat projek kelompok kami lebih baik daripada projek kelompok lain?
