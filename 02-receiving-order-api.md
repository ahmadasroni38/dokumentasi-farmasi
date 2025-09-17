# Receiving Order API Specification

## Requirements untuk Programmer PT. PBF

### Yang Harus Disiapkan oleh PBF

API ini menjelaskan struktur data dan endpoint yang **WAJIB disiapkan oleh programmer PT. PBF** untuk mengirim notifikasi pengiriman barang ke Sistem Farmasi Balimed.

---

## Endpoints yang WAJIB Dibuat dan Dipanggil PBF

### 1. Kirim Notifikasi Pengiriman ke Balimed

**Endpoint yang WAJIB Dipanggil PBF**: `POST /api/balimed/receiving-orders`
**Direction**: **PBF (mengirim)** → Balimed
**Trigger**: Setelah PBF melakukan pengiriman barang

**Headers yang WAJIB Dikirim PBF:**

```http
Authorization: Bearer {api_token}        # Token yang diberikan Balimed
Content-Type: application/json
X-Distributor-Code: PBF_DIST_001        # Kode distributor PBF
X-Request-ID: {unique_request_id}       # ID unik untuk tracking
```

**Payload yang WAJIB Dikirim PBF:**

```json
{
    "header": {
        "nomor_ro": "RO/PBF/24/12/0001",
        "nomor_faktur": "FK/PBF001/2024/001",
        "tanggal_pengiriman": "2024-12-18",
        "tanggal_faktur": "2024-12-18",
        "po_reference": "PO/24/12/0001/DENPSR",
        "distributor_code": "PBF_DIST_001",
        "distributor_name": "PT. PBF Medika",
        "cara_bayar": "CREDIT",
        "tempo_hari": 30,
        "tanggal_jatuh_tempo": "2025-01-17",
        "disc_persen": 5.5,
        "ppn_persen": 11.0,
        "ppn_nominal": 42556.25,
        "disc_rupiah": 20362.5,
        "biaya_pengiriman": 5000.0,
        "total_faktur": 434443.75,
        "penerima": "Apt. Made Sari",
        "telp_penerima": "0361-224466",
        "alamat_pengiriman": "Jl. Mahendradatta No.57, Denpasar",
        "catatan": "Pengiriman sesuai PO, semua item tersedia",
        "waktu_pengiriman": "2024-12-18T14:30:00+07:00",
        "nama_driver": "Wayan Bagus",
        "telp_driver": "0812-3456-7890",
        "nomor_kendaraan": "DK 1234 AB"
    },
    "detail_barang": [
        {
            "line_number": 1,
            "kode_barang_pbf": "PBF001",
            "kode_barang_balimed": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "nama_generik": "Parasetamol",
            "satuan": "STRIP",
            "qty_dikirim": 100,
            "harga_satuan": 2500.0,
            "disc_persen": 5.0,
            "ppn_persen": 11.0,
            "no_batch": "PCM240801",
            "tanggal_expired": "2026-08-01",
            "tanggal_produksi": "2024-01-01",
            "sub_total": 237500.0,
            "kondisi_barang": "BAIK",
            "catatan_item": "Kemasan lengkap"
        },
        {
            "line_number": 2,
            "kode_barang_pbf": "PBF002",
            "kode_barang_balimed": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "nama_generik": "Amoksisilin",
            "satuan": "STRIP",
            "qty_dikirim": 50,
            "harga_satuan": 3500.0,
            "disc_persen": 5.0,
            "ppn_persen": 11.0,
            "no_batch": "AMX240901",
            "tanggal_expired": "2026-09-01",
            "tanggal_produksi": "2024-02-01",
            "sub_total": 166250.0,
            "kondisi_barang": "BAIK",
            "catatan_item": "Dus original"
        }
    ]
}
```

**Response yang Akan Diterima PBF dari Balimed:**

```json
{
    "status": "success",
    "message": "Notifikasi pengiriman berhasil diterima",
    "ro_number": "RO/24/12/0001/DENPSR",
    "received_at": "2024-12-18T14:35:00+07:00",
    "next_action": "Tunggu konfirmasi penerimaan dari Balimed"
}
```

### 2. Terima Konfirmasi Penerimaan dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/receiving-orders/{ro_number}/confirmation`
**Direction**: Balimed → **PBF (menerima)**

**Headers yang WAJIB Diterima dan Divalidasi PBF:**

```http
Authorization: Bearer {api_token}
Content-Type: application/json
X-Hospital-Code: BALIMED_DENPASAR
X-Request-ID: {unique_request_id}
```

**Payload yang Akan Diterima PBF dari Balimed:**

```json
{
    "ro_number": "RO/PBF/24/12/0001",
    "konfirmasi_tanggal": "2024-12-18T16:00:00+07:00",
    "status_penerimaan": "CONFIRMED",
    "penerima_balimed": "Apt. Made Sari",
    "catatan_penerimaan": "Semua barang diterima sesuai faktur",
    "detail_konfirmasi": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "qty_diterima": 100,
            "qty_sesuai": true,
            "kondisi_diterima": "BAIK",
            "batch_sesuai": true,
            "catatan": "OK"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "qty_diterima": 50,
            "qty_sesuai": true,
            "kondisi_diterima": "BAIK",
            "batch_sesuai": true,
            "catatan": "OK"
        }
    ]
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Konfirmasi penerimaan berhasil diproses",
    "ro_number": "RO/PBF/24/12/0001",
    "status_pengiriman": "COMPLETED",
    "processed_at": "2024-12-18T16:05:00+07:00"
}
```

### 3. Terima Laporan Discrepancy dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/receiving-orders/{ro_number}/discrepancy`
**Direction**: Balimed → **PBF (menerima)**

**Payload yang Akan Diterima PBF jika Ada Masalah:**

```json
{
    "ro_number": "RO/PBF/24/12/0001",
    "laporan_tanggal": "2024-12-18T16:00:00+07:00",
    "status_discrepancy": "REPORTED",
    "pelapor": "Apt. Made Sari",
    "jenis_masalah": "QUANTITY_MISMATCH",
    "deskripsi_masalah": "Quantity beberapa item tidak sesuai",
    "detail_discrepancy": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "qty_terkirim": 100,
            "qty_diterima": 95,
            "selisih": -5,
            "jenis_masalah": "KURANG",
            "catatan": "Kemasan rusak 1 strip",
            "foto_bukti": ["evidence_001.jpg", "evidence_002.jpg"]
        }
    ]
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Laporan discrepancy berhasil diterima",
    "discrepancy_id": "DISC/PBF/24/001",
    "status_penanganan": "UNDER_REVIEW",
    "estimasi_penyelesaian": "2024-12-20T17:00:00+07:00",
    "pic_penanganan": "Customer Service PBF"
}
```

---

## Database yang Harus Disiapkan PBF

### 1. Table Pengiriman (Receiving Orders)

```sql
CREATE TABLE pbf_deliveries (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    ro_number VARCHAR(50) UNIQUE NOT NULL,
    nomor_faktur VARCHAR(50) NOT NULL,
    po_reference VARCHAR(50) NOT NULL,
    tanggal_pengiriman DATE NOT NULL,
    tanggal_faktur DATE NOT NULL,
    status_pengiriman ENUM('SENT', 'CONFIRMED', 'DISCREPANCY', 'COMPLETED') DEFAULT 'SENT',
    total_faktur DECIMAL(15,2) NOT NULL,
    penerima VARCHAR(100),
    alamat_pengiriman TEXT,
    nama_driver VARCHAR(100),
    nomor_kendaraan VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 2. Table Detail Pengiriman

```sql
CREATE TABLE pbf_delivery_details (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    delivery_id BIGINT REFERENCES pbf_deliveries(id),
    line_number INT NOT NULL,
    kode_barang_pbf VARCHAR(50) NOT NULL,
    kode_barang_balimed VARCHAR(50) NOT NULL,
    nama_barang VARCHAR(200) NOT NULL,
    qty_dikirim INT NOT NULL,
    qty_dikonfirmasi INT DEFAULT 0,
    harga_satuan DECIMAL(10,2) NOT NULL,
    no_batch VARCHAR(50) NOT NULL,
    tanggal_expired DATE NOT NULL,
    status_item ENUM('SENT', 'CONFIRMED', 'DISCREPANCY') DEFAULT 'SENT'
);
```

### 3. Table Discrepancy

```sql
CREATE TABLE pbf_discrepancies (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    discrepancy_id VARCHAR(50) UNIQUE NOT NULL,
    delivery_id BIGINT REFERENCES pbf_deliveries(id),
    jenis_masalah VARCHAR(50) NOT NULL,
    deskripsi_masalah TEXT,
    status_penanganan ENUM('UNDER_REVIEW', 'RESOLVED', 'ESCALATED') DEFAULT 'UNDER_REVIEW',
    pic_penanganan VARCHAR(100),
    tanggal_laporan TIMESTAMP NOT NULL,
    tanggal_penyelesaian TIMESTAMP NULL
);
```

---

## Business Rules yang Harus Diimplementasi PBF

### 1. Notification Timing
- **WAJIB** kirim notifikasi pengiriman maximum 1 jam setelah barang dikirim
- **WAJIB** update status dalam sistem PBF setelah terkirim
- **WAJIB** simpan bukti pengiriman (foto, TTD, dll)

### 2. Data Validation
- **WAJIB** validasi semua data sebelum kirim notifikasi
- **WAJIB** pastikan batch number dan expiry date akurat
- **WAJIB** validasi harga sesuai kontrak

### 3. Discrepancy Handling
- **WAJIB** respond discrepancy dalam 24 jam
- **WAJIB** provide action plan untuk penyelesaian
- **WAJIB** dokumentasi lengkap untuk audit trail

### 4. Error Handling
```json
{
    "status": "error",
    "error_code": "INVALID_PO_REFERENCE",
    "message": "Nomor PO tidak ditemukan atau sudah selesai",
    "timestamp": "2024-12-18T16:00:00+07:00"
}
```

---

## Testing Checklist untuk PBF

### Phase 1: Basic Integration
- [ ] Implement endpoint untuk kirim notifikasi pengiriman
- [ ] Implement endpoint untuk terima konfirmasi
- [ ] Test authentication dan data validation
- [ ] Test error handling scenarios

### Phase 2: Discrepancy Management
- [ ] Implement endpoint untuk terima laporan discrepancy
- [ ] Test workflow penanganan masalah
- [ ] Implement notification system internal PBF

### Phase 3: Database & Monitoring
- [ ] Setup database tables sesuai spesifikasi
- [ ] Implement audit logging
- [ ] Setup monitoring dan alerting

---

_API ini memungkinkan PBF untuk mengirim notifikasi pengiriman secara otomatis dan menerima feedback dari Balimed untuk memastikan supply chain yang smooth dan akurat._