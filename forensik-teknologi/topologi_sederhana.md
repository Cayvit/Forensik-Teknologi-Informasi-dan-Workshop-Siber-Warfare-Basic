# Catatan Praktikum: Perubahan Alamat IP di Router

## Pendahuluan
Dalam jaringan komputer, **Router** berfungsi untuk menghubungkan dua atau lebih jaringan yang berbeda. Router akan meneruskan paket dari satu jaringan ke jaringan lain berdasarkan **alamat IP tujuan (Destination IP)**.  
Pada praktikum ini, kita akan melihat bagaimana **alamat IP berubah ketika melewati router** sesuai dengan interface router yang dilalui.

## Topologi Jaringan
Topologi yang digunakan adalah sebagai berikut:

- **PC 1**
  - IP: `192.168.1.2/24`
  - Default Gateway: `192.168.1.1`
- **Router 1 (R1)**
  - Interface ke PC1: `192.168.1.1`
  - Interface ke Router2: `192.168.2.1`
- **Router 2 (R2)**
  - Interface ke R1: `192.168.2.2`
  - Interface ke PC2: `192.168.3.1`
- **PC 2**
  - IP: `192.168.3.2/24`
  - Default Gateway: `192.168.3.1`

## Alur Perubahan Alamat IP

### 1. Dari PC1 ke R1
- **Source IP (Src IP):** `192.168.1.2` (alamat PC1)
- **Destination IP (Dest IP):** `192.168.3.2` (alamat PC2)
- **Keterangan:** PC1 mengirim paket menuju PC2. Paket keluar melalui default gateway (`192.168.1.1` yaitu interface R1).

### 2. R1 ke R2
- **Source IP (Src IP):** `192.168.2.1` (interface R1 ke R2)
- **Destination IP (Dest IP):** `192.168.3.2`
- **Keterangan:** Saat paket melewati Router1, source IP akan berubah menjadi IP milik interface R1 (`192.168.2.1`). Tujuan tetap PC2.

### 3. R2 ke PC2
- **Source IP (Src IP):** `192.168.3.1` (interface R2 ke PC2)
- **Destination IP (Dest IP):** `192.168.3.2`
- **Keterangan:** Router2 menerima paket dari R1 dan meneruskan ke PC2. Source IP berubah menjadi IP milik interface R2 (`192.168.3.1`) yang sejalur dengan PC2.

## Ringkasan Jalur Paket
| Tahap         | Source IP     | Destination IP | Keterangan                            |
|---------------|---------------|----------------|---------------------------------------|
| PC1 → R1      | 192.168.1.2   | 192.168.3.2    | PC1 kirim paket ke gateway (R1)       |
| R1 → R2       | 192.168.2.1   | 192.168.3.2    | Paket diteruskan ke R2                |
| R2 → PC2      | 192.168.3.1   | 192.168.3.2    | Paket sampai ke jaringan tujuan (PC2) |

## Kesimpulan
1. **Alamat IP pengirim (Source IP) berubah di setiap router** sesuai dengan interface yang digunakan.  
2. **Alamat IP tujuan (Destination IP) tetap**, karena tujuan paket tidak berubah.  
3. Mekanisme ini memastikan bahwa setiap jaringan bisa saling terhubung meskipun berbeda segmen.  
4. Default Gateway pada setiap PC harus benar, agar paket bisa diteruskan ke router yang tepat.  

Dengan memahami mekanisme perubahan IP ini, kita dapat lebih mudah menganalisis alur komunikasi data dalam jaringan yang menggunakan beberapa router.

---
