#  Topologi Router–Router–LAN (Cisco Packet Tracer)

Gambar acuan: PC1 ↔ R1 ↔ R2 ↔ PC2  
Jalur IP:
- LAN-1: **192.168.1.0/24**  (PC1 ↔ R1)
- Link Antar-Router: **192.168.2.0/24** (R1 ↔ R2)
- LAN-2: **192.168.3.0/24**  (R2 ↔ PC2)

Tujuan: PC1 (**192.168.1.2**) dapat berkomunikasi dengan PC2 (**192.168.3.2**) melalui R1 dan R2.

---

## 1) Persiapan
- **Aplikasi**: Cisco Packet Tracer 8.x
- **Perangkat** (opsi yang umum tersedia):
  - 2× Router (mis. **1841** atau **2811** atau **2911**)
  - 2× PC (PC-PT)
- **Kabel**:
  - **PC ↔ Router**: Copper **Straight-Through**
  - **Router ↔ Router**:
    - **Opsi A (Ethernet)**: Copper **Cross-Over**
    - **Opsi B (Serial)**: Serial **DCE/DTE** (butuh modul WIC-2T)

> Nama antarmuka pada router bisa `Fa0/0`, `Gi0/0`, atau `Se0/0/0` tergantung tipe perangkat. Gunakan nama yang tampil pada `show ip interface brief`.

---

## 2) Rancang Topologi di Packet Tracer
1. **Letakkan perangkat**: tarik 2 PC dan 2 Router ke kanvas.
2. **(Opsional – hanya bila pakai Serial)**:  
   - Klik router → **Physical** → **Power off**.  
   - Tambahkan modul **WIC-2T** (slot kosong) di **kedua router**.  
   - **Power on** kembali.
3. **Hubungkan kabel**:
   - PC1 ↔ R1: pilih port `PC FastEthernet0` ke `R1 Fa0/0` (atau `Gi0/0`) → **Straight-Through**.
   - **Opsi A (Ethernet)** R1 ↔ R2: `R1 Fa0/1` ↔ `R2 Fa0/0` → **Cross-Over**.
   - **Opsi B (Serial)** R1 ↔ R2: `R1 Se0/0/0` ↔ `R2 Se0/0/0` → **Serial** (pastikan salah satu ujung adalah **DCE**).
   - R2 ↔ PC2: `R2 Fa0/1` (atau `Gi0/1`) ke `PC2 FastEthernet0` → **Straight-Through**.

> Jika lampu port masih merah/oranye, tunggu beberapa detik atau pastikan antarmuka di-`no shutdown`.

---

## 3) Konfigurasi IP PC
### PC1 (Desktop → IP Configuration)
- IP Address: **192.168.1.2**
- Subnet Mask: **255.255.255.0**
- Default Gateway: **192.168.1.1**

### PC2
- IP Address: **192.168.3.2**
- Subnet Mask: **255.255.255.0**
- Default Gateway: **192.168.3.1**

> **Kenapa gateway penting?** Karena paket dari PC menuju jaringan lain harus dikirim ke router (alamat gateway).

---

## 4) Konfigurasi Router 1 (R1)
Masuk ke **CLI** R1:

> Gantilah `fa0/0`, `fa0/1` sesuai antarmuka sebenarnya di perangkatmu (bisa `gi0/0`, `gi0/1`).

```bash
enable
configure terminal

! Ke LAN-1 (ke PC1)
interface fa0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

! Link ke R2 (Opsi A: Ethernet)
interface fa0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

! --- ATAU ---

! Link ke R2 (Opsi B: Serial) - pastikan ujung DCE set clock rate
interface se0/0/0
 ip address 192.168.2.1 255.255.255.0
 clock rate 64000   ! hanya pada sisi DCE
 no shutdown
exit

! Routing statis: rute menuju LAN-2 via R2
ip route 192.168.3.0 255.255.255.0 192.168.2.2

end
write memory
