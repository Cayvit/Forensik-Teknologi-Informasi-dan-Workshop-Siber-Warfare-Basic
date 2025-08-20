# Catatan Pembelajaran: Network Protocol

## 1. Pengantar
- **Network Protocol** adalah sekumpulan aturan komunikasi agar perangkat dalam jaringan dapat saling bertukar data.
- Bersifat **modular**, artinya dibagi dalam beberapa lapisan dengan fungsi tertentu.
- Model acuan utama:
  - **OSI (Open System Interconnection)** → 7 lapisan (teoritis)
  - **TCP/IP** → 4 lapisan (implementasi nyata)

---

## 2. OSI Model (7 Layer)

| Layer        | Fungsi Utama | Contoh Protokol |
|--------------|-------------|-----------------|
| **Application** | Interaksi langsung dengan user | HTTP, FTP, SMTP, DNS |
| **Presentation** | Format data, enkripsi, kompresi | SSL/TLS |
| **Session** | Mengatur sesi komunikasi (establish, maintain, terminate) | NetBIOS, PPTP |
| **Transport** | Kontrol komunikasi end-to-end | TCP, UDP |
| **Network** | Routing dan addressing | IP, ICMP |
| **Data Link** | Pengaturan frame, MAC Address | Ethernet, ARP, PPP |
| **Physical** | Media fisik pengiriman data | Kabel, FO, RF |

---

## 3. TCP/IP Model (4 Layer)

| Layer             | Fungsi Utama | Contoh Protokol |
|-------------------|--------------|-----------------|
| **Application**   | Aplikasi & protokol layanan | HTTP, FTP, DNS, SMTP, IMAP, POP3, SSH, Telnet |
| **Transport**     | Kontrol koneksi & segmentasi | TCP, UDP |
| **Internet**      | Pengalamatan & routing | IP, ICMP |
| **Network Access**| Akses jaringan fisik | ARP, RARP, Ethernet, WiFi |

---

## 4. Proses Komunikasi Client ↔ Server
1. **Client** mengirim data → turun dari lapisan atas (Application) ke bawah (Physical).
2. Data berubah bentuk di tiap layer:
   - **Transport** → Segment (TCP) / Datagram (UDP)
   - **Network** → Packet (IP)
   - **Data Link** → Frame
   - **Physical** → Bitstream (1 & 0)
3. Data dikirim lewat **media komunikasi** (Kabel, Fiber Optic, RF).
4. **Server** menerima data → naik dari Physical hingga Application.

---

## 5. Media Komunikasi
- **Cable**: UTP, STP, Coaxial
- **Fiber Optic (FO)**
- **Radio Frequency (RF)** → Wireless

---

## 6. Kesimpulan
- **OSI** → Model teoritis untuk pembelajaran.
- **TCP/IP** → Model implementasi nyata di jaringan modern.
- Prinsip utama:  
  > **Data dienkapsulasi dari layer atas → bawah (saat dikirim)**  
  > **Data di-dekapsulasi dari layer bawah → atas (saat diterima)**

---
