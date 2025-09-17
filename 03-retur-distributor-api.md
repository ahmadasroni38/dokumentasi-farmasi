# Retur Distributor API Specification

## Requirements untuk Programmer PT. PBF

### Yang Harus Disiapkan oleh PBF

API ini menjelaskan struktur data dan endpoint yang **WAJIB disiapkan oleh programmer PT. PBF** untuk menerima dan memproses request retur dari Sistem Farmasi Balimed.

---

## Endpoints yang WAJIB Dibuat dan Dipanggil PBF

### 1. Terima Request Retur dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/returns`
**Direction**: Balimed → **PBF (menerima)**
**Trigger**: Ketika Balimed membuat request retur

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
    "header": {
        "nomor_retur": "RTN/24/12/0001/DENPSR",
        "tanggal_retur": "2024-12-20",
        "po_reference": "PO/24/12/0001/DENPSR",
        "ro_reference": "RO/24/12/0001/DENPSR",
        "nomor_faktur_reference": "FK/PBF001/2024/001",
        "jenis_retur": "DAMAGED",
        "alasan_retur": "Barang rusak saat diterima",
        "subunit_code": "FARM_IGD",
        "subunit_name": "Farmasi IGD",
        "pic_retur": "Apt. Made Sari",
        "telp_pic": "0361-224466",
        "alamat_pickup": "Jl. Mahendradatta No.57, Denpasar",
        "catatan_retur": "Kemasan rusak, isi masih baik",
        "foto_bukti": [
            "bukti_retur_001.jpg",
            "bukti_retur_002.jpg"
        ],
        "created_by": "USR001",
        "created_by_name": "Apt. Made Sari",
        "created_date": "2024-12-20T09:00:00+07:00"
    },
    "detail_retur": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "satuan": "STRIP",
            "qty_retur": 5,
            "harga_satuan": 2500.0,
            "total_nilai": 12500.0,
            "no_batch": "PCM240801",
            "tanggal_expired": "2026-08-01",
            "alasan_item": "Kemasan rusak",
            "kondisi_barang": "DAMAGED",
            "foto_item": ["item_001.jpg"],
            "catatan_item": "Strip sobek, tablet masih utuh"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "satuan": "STRIP",
            "qty_retur": 2,
            "harga_satuan": 3500.0,
            "total_nilai": 7000.0,
            "no_batch": "AMX240901",
            "tanggal_expired": "2026-09-01",
            "alasan_item": "Expired date pendek",
            "kondisi_barang": "NEAR_EXPIRED",
            "foto_item": ["item_002.jpg"],
            "catatan_item": "Sisa 3 bulan expired"
        }
    ]
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Request retur berhasil diterima",
    "retur_id": "RTN/PBF/24/001",
    "nomor_retur_balimed": "RTN/24/12/0001/DENPSR",
    "status_retur": "RECEIVED",
    "estimasi_review": "2024-12-22T17:00:00+07:00",
    "pic_review": "Customer Service PBF",
    "telp_pic": "021-1234567",
    "next_action": "Menunggu review dan approval dari PBF",
    "received_at": "2024-12-20T09:05:00+07:00"
}
```

### 2. Kirim Update Status Retur ke Balimed

**Endpoint yang WAJIB Dipanggil PBF**: `PATCH /api/balimed/returns/{return_id}/status`
**Direction**: **PBF (mengirim)** → Balimed
**Trigger**: Setiap ada perubahan status retur

**Headers yang WAJIB Dikirim PBF:**

```http
Authorization: Bearer {api_token}
Content-Type: application/json
X-Distributor-Code: PBF_DIST_001
X-Request-ID: {unique_request_id}
```

**Payload yang WAJIB Dikirim PBF:**

```json
{
    "retur_id": "RTN/PBF/24/001",
    "nomor_retur_balimed": "RTN/24/12/0001/DENPSR",
    "status_baru": "APPROVED",
    "status_sebelumnya": "UNDER_REVIEW",
    "tanggal_update": "2024-12-21T14:00:00+07:00",
    "pic_approval": "Manager PBF",
    "alasan_keputusan": "Retur disetujui sesuai kontrak",
    "catatan_approval": "Pickup akan dilakukan dalam 2 hari kerja",
    "detail_approval": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "status_item": "APPROVED",
            "qty_disetujui": 5,
            "nilai_penggantian": 12500.0,
            "metode_penggantian": "CREDIT_NOTE",
            "catatan": "Disetujui penuh"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "status_item": "PARTIAL_APPROVED",
            "qty_disetujui": 1,
            "nilai_penggantian": 3500.0,
            "metode_penggantian": "REPLACEMENT",
            "catatan": "Hanya 1 strip yang memenuhi syarat retur"
        }
    ],
    "jadwal_pickup": {
        "tanggal_pickup": "2024-12-23",
        "waktu_pickup": "09:00-12:00",
        "pic_pickup": "Driver PBF",
        "telp_pickup": "0812-9876543",
        "instruksi_khusus": "Hubungi 30 menit sebelum pickup"
    }
}
```

### 3. Terima Pembatalan Retur dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `DELETE /api/distributor/returns/{return_id}`
**Direction**: Balimed → **PBF (menerima)**

**Headers yang WAJIB Diterima PBF:**

```http
Authorization: Bearer {api_token}
Content-Type: application/json
X-Hospital-Code: BALIMED_DENPASAR
X-Request-ID: {unique_request_id}
```

**Payload yang Akan Diterima PBF:**

```json
{
    "retur_id": "RTN/PBF/24/001",
    "nomor_retur_balimed": "RTN/24/12/0001/DENPSR",
    "alasan_batal": "Barang sudah terpakai semua",
    "catatan_batal": "Pasien urgent memerlukan obat",
    "dibatalkan_oleh": "Apt. Made Sari",
    "tanggal_batal": "2024-12-21T08:00:00+07:00"
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Pembatalan retur berhasil diproses",
    "retur_id": "RTN/PBF/24/001",
    "status_retur": "CANCELLED",
    "processed_at": "2024-12-21T08:05:00+07:00"
}
```

---

## Database yang Harus Disiapkan PBF

### 1. Table Master Retur

```sql
CREATE TABLE pbf_returns (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    retur_id VARCHAR(50) UNIQUE NOT NULL,
    nomor_retur_balimed VARCHAR(50) NOT NULL,
    tanggal_retur DATE NOT NULL,
    po_reference VARCHAR(50),
    ro_reference VARCHAR(50),
    nomor_faktur_reference VARCHAR(50),
    jenis_retur ENUM('DAMAGED', 'EXPIRED', 'NEAR_EXPIRED', 'EXCESS', 'WRONG_ITEM', 'QUALITY_ISSUE') NOT NULL,
    alasan_retur TEXT NOT NULL,
    status_retur ENUM('RECEIVED', 'UNDER_REVIEW', 'APPROVED', 'REJECTED', 'PARTIAL_APPROVED', 'PICKUP_SCHEDULED', 'COMPLETED', 'CANCELLED') DEFAULT 'RECEIVED',
    pic_retur VARCHAR(100),
    telp_pic VARCHAR(20),
    alamat_pickup TEXT,
    catatan_retur TEXT,
    total_nilai DECIMAL(15,2) DEFAULT 0,
    nilai_disetujui DECIMAL(15,2) DEFAULT 0,
    pic_approval VARCHAR(100),
    tanggal_approval TIMESTAMP NULL,
    alasan_keputusan TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 2. Table Detail Retur

```sql
CREATE TABLE pbf_return_details (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    return_id BIGINT REFERENCES pbf_returns(id),
    line_number INT NOT NULL,
    kode_barang VARCHAR(50) NOT NULL,
    nama_barang VARCHAR(200) NOT NULL,
    satuan VARCHAR(20) NOT NULL,
    qty_retur INT NOT NULL,
    qty_disetujui INT DEFAULT 0,
    harga_satuan DECIMAL(10,2) NOT NULL,
    total_nilai DECIMAL(15,2) NOT NULL,
    nilai_disetujui DECIMAL(15,2) DEFAULT 0,
    no_batch VARCHAR(50) NOT NULL,
    tanggal_expired DATE NOT NULL,
    alasan_item VARCHAR(200),
    kondisi_barang ENUM('GOOD', 'DAMAGED', 'EXPIRED', 'NEAR_EXPIRED', 'DEFECTIVE') NOT NULL,
    status_item ENUM('PENDING', 'APPROVED', 'REJECTED', 'PARTIAL_APPROVED') DEFAULT 'PENDING',
    metode_penggantian ENUM('CREDIT_NOTE', 'REPLACEMENT', 'REFUND') NULL,
    catatan_item TEXT
);
```

### 3. Table Foto Bukti

```sql
CREATE TABLE pbf_return_evidence (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    return_id BIGINT REFERENCES pbf_returns(id),
    return_detail_id BIGINT REFERENCES pbf_return_details(id) NULL,
    file_path VARCHAR(500) NOT NULL,
    file_name VARCHAR(200) NOT NULL,
    file_type ENUM('GENERAL', 'ITEM_SPECIFIC') NOT NULL,
    description TEXT,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Table Pickup Schedule

```sql
CREATE TABLE pbf_pickup_schedules (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    return_id BIGINT REFERENCES pbf_returns(id),
    tanggal_pickup DATE NOT NULL,
    waktu_mulai TIME NOT NULL,
    waktu_selesai TIME NOT NULL,
    pic_pickup VARCHAR(100),
    telp_pickup VARCHAR(20),
    status_pickup ENUM('SCHEDULED', 'IN_PROGRESS', 'COMPLETED', 'FAILED') DEFAULT 'SCHEDULED',
    instruksi_khusus TEXT,
    catatan_pickup TEXT,
    tanggal_aktual_pickup TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Business Rules yang Harus Diimplementasi PBF

### 1. Review Timeline
- **WAJIB** response awal dalam 2 jam setelah menerima request
- **WAJIB** approval/rejection dalam 48 jam kerja
- **WAJIB** notifikasi real-time untuk setiap perubahan status

### 2. Approval Criteria
- **WAJIB** validasi dengan database penjualan/pengiriman
- **WAJIB** cek kondisi barang sesuai foto bukti
- **WAJIB** validasi batch number dan expiry date
- **WAJIB** cek sesuai dengan ketentuan kontrak retur

### 3. Status Flow Management
```
RECEIVED → UNDER_REVIEW → APPROVED/REJECTED/PARTIAL_APPROVED → PICKUP_SCHEDULED → COMPLETED
                     ↓
                 CANCELLED (bisa dari status manapun)
```

### 4. Notification Requirements
- **WAJIB** kirim update status ke Balimed setiap perubahan
- **WAJIB** berikan estimasi waktu yang realistis
- **WAJIB** notifikasi 24 jam sebelum pickup

### 5. Error Handling
```json
{
    "status": "error",
    "error_code": "INVALID_RETURN_CRITERIA",
    "message": "Barang tidak memenuhi syarat retur sesuai kontrak",
    "details": {
        "item_code": "BRG001",
        "reason": "Sudah melewati batas waktu retur 30 hari"
    },
    "timestamp": "2024-12-20T09:00:00+07:00"
}
```

**Error Codes yang Harus Didukung:**
- `INVALID_RETURN_CRITERIA` - Tidak memenuhi syarat retur
- `BATCH_NOT_FOUND` - Batch number tidak ditemukan dalam database PBF
- `EXPIRED_RETURN_PERIOD` - Melewati batas waktu retur
- `INSUFFICIENT_DOCUMENTATION` - Dokumentasi tidak lengkap
- `DUPLICATE_RETURN_REQUEST` - Request retur duplikat

---

## Workflow Approval yang Harus Diimplementasi

### 1. Validasi Awal (Otomatis)
```python
def validate_return_request(return_data):
    # Cek apakah barang pernah dikirim ke Balimed
    if not validate_delivery_history(return_data.po_reference):
        return reject("Barang tidak ditemukan dalam riwayat pengiriman")

    # Cek batch number
    if not validate_batch(return_data.batch_number):
        return reject("Batch number tidak valid")

    # Cek batas waktu retur
    if is_return_period_expired(return_data.delivery_date):
        return reject("Melewati batas waktu retur")

    return approve_for_manual_review()
```

### 2. Review Manual (Customer Service)
- **WAJIB** review foto bukti
- **WAJIB** validasi alasan retur
- **WAJIB** cek dengan tim warehouse untuk kondisi barang

### 3. Approval Final (Manager)
- **WAJIB** approval untuk nilai diatas threshold tertentu
- **WAJIB** review kasus yang disputed
- **WAJIB** final decision untuk partial approval

---

## Integration Testing Checklist untuk PBF

### Phase 1: Basic Return Processing
- [ ] Implement endpoint untuk terima request retur
- [ ] Implement status update notification
- [ ] Test validation rules dan business logic
- [ ] Test error handling dan edge cases

### Phase 2: Approval Workflow
- [ ] Implement approval workflow automation
- [ ] Test notification system untuk setiap status change
- [ ] Implement cancellation handling

### Phase 3: Database & File Management
- [ ] Setup database tables sesuai spesifikasi
- [ ] Implement file upload handling untuk foto bukti
- [ ] Setup backup dan archival system

### Phase 4: Business Intelligence
- [ ] Implement dashboard untuk monitoring retur
- [ ] Setup reporting untuk analisis trend retur
- [ ] Implement early warning system untuk excessive returns

---

_API ini memungkinkan PBF untuk mengelola proses retur secara efisien dengan workflow yang jelas dan audit trail yang lengkap untuk compliance dan business intelligence._