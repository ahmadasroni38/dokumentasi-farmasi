# Pemenuhan Retur API Specification

## Requirements untuk Programmer PT. PBF

### Yang Harus Disiapkan oleh PBF

API ini menjelaskan struktur data dan endpoint yang **WAJIB disiapkan oleh programmer PT. PBF** untuk mengirim replacement goods dan menerima konfirmasi dari Sistem Farmasi Balimed.

---

## Endpoints yang WAJIB Dibuat dan Dipanggil PBF

### 1. Kirim Notifikasi Replacement ke Balimed

**Endpoint yang WAJIB Dipanggil PBF**: `POST /api/balimed/return-fulfillments`
**Direction**: **PBF (mengirim)** → Balimed
**Trigger**: Setelah PBF mengirim replacement goods

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
        "nomor_replacement": "RPL/PBF/24/001",
        "tanggal_replacement": "2024-12-25",
        "retur_reference": "RTN/PBF/24/001",
        "nomor_retur_balimed": "RTN/24/12/0001/DENPSR",
        "credit_note_reference": "CN/PBF001/24/001",
        "jenis_replacement": "PRODUCT_REPLACEMENT",
        "distributor_code": "PBF_DIST_001",
        "distributor_name": "PT. PBF Medika",
        "hospital_code": "BALIMED_DENPASAR",
        "nomor_faktur_replacement": "FK/RPL/PBF001/2024/001",
        "tanggal_faktur": "2024-12-25",
        "metode_pengiriman": "DELIVERY",
        "biaya_pengiriman": 0.0,
        "penerima": "Apt. Made Sari",
        "telp_penerima": "0361-224466",
        "alamat_pengiriman": "Jl. Mahendradatta No.57, Denpasar",
        "catatan": "Replacement untuk retur barang damaged",
        "nama_driver": "Wayan Bagus",
        "telp_driver": "0812-3456789",
        "nomor_kendaraan": "DK 1234 AB",
        "estimasi_tiba": "2024-12-25T14:00:00+07:00",
        "created_by": "SYS_PBF",
        "created_date": "2024-12-25T09:00:00+07:00"
    },
    "detail_replacement": [
        {
            "line_number": 1,
            "retur_line_reference": 1,
            "kode_barang_original": "BRG001",
            "nama_barang_original": "Paracetamol 500mg",
            "kode_barang_replacement": "BRG001",
            "nama_barang_replacement": "Paracetamol 500mg",
            "satuan": "STRIP",
            "qty_retur": 5,
            "qty_replacement": 5,
            "harga_satuan": 2500.0,
            "disc_persen": 5.0,
            "subtotal": 11875.0,
            "no_batch_original": "PCM240801",
            "no_batch_replacement": "PCM241101",
            "expired_original": "2026-08-01",
            "expired_replacement": "2026-11-01",
            "alasan_replacement": "Same product dengan batch lebih baru",
            "kondisi_barang": "BARU",
            "jenis_replacement": "SAME_PRODUCT",
            "catatan_item": "Batch baru dengan expired date lebih panjang"
        },
        {
            "line_number": 2,
            "retur_line_reference": 2,
            "kode_barang_original": "BRG002",
            "nama_barang_original": "Amoxicillin 500mg",
            "kode_barang_replacement": "BRG002B",
            "nama_barang_replacement": "Amoxicillin 500mg - Brand B",
            "satuan": "STRIP",
            "qty_retur": 1,
            "qty_replacement": 1,
            "harga_satuan": 3500.0,
            "disc_persen": 5.0,
            "subtotal": 3325.0,
            "no_batch_original": "AMX240901",
            "no_batch_replacement": "AMX2B241001",
            "expired_original": "2026-09-01",
            "expired_replacement": "2027-10-01",
            "alasan_replacement": "Produk equivalent dengan kualitas lebih baik",
            "kondisi_barang": "BARU",
            "jenis_replacement": "EQUIVALENT_PRODUCT",
            "catatan_item": "Upgrade ke brand yang lebih baik"
        }
    ],
    "dokumen_pendukung": [
        {
            "jenis_dokumen": "DELIVERY_NOTE",
            "nama_file": "delivery_note_RPL_PBF_24_001.pdf",
            "url_download": "https://pbf.com/documents/delivery_RPL_PBF_24_001.pdf",
            "hash_file": "abc123def456",
            "ukuran_file": 256000
        },
        {
            "jenis_dokumen": "REPLACEMENT_CERTIFICATE",
            "nama_file": "replacement_cert.pdf",
            "url_download": "https://pbf.com/documents/cert_RPL_PBF_24_001.pdf",
            "hash_file": "def456ghi789",
            "ukuran_file": 128000
        }
    ]
}
```

**Response yang Akan Diterima PBF dari Balimed:**

```json
{
    "status": "success",
    "message": "Notifikasi replacement berhasil diterima",
    "replacement_id": "RPL/BALIMED/24/001",
    "nomor_replacement_pbf": "RPL/PBF/24/001",
    "status_replacement": "PENDING_RECEIPT",
    "estimasi_konfirmasi": "2024-12-25T17:00:00+07:00",
    "pic_penerima": "Apt. Made Sari",
    "received_at": "2024-12-25T09:05:00+07:00"
}
```

### 2. Terima Konfirmasi Replacement dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/return-fulfillments/{replacement_id}/confirmation`
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
    "replacement_id": "RPL/BALIMED/24/001",
    "nomor_replacement_pbf": "RPL/PBF/24/001",
    "status_konfirmasi": "CONFIRMED",
    "tanggal_konfirmasi": "2024-12-25T15:30:00+07:00",
    "penerima_balimed": "Apt. Made Sari",
    "catatan_penerimaan": "Semua replacement diterima dalam kondisi baik",
    "detail_konfirmasi": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "qty_diterima": 5,
            "qty_sesuai": true,
            "kondisi_diterima": "BAIK",
            "batch_sesuai": true,
            "expired_sesuai": true,
            "status_item": "ACCEPTED",
            "catatan": "Batch baru dengan kualitas baik"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002B",
            "qty_diterima": 1,
            "qty_sesuai": true,
            "kondisi_diterima": "BAIK",
            "batch_sesuai": true,
            "expired_sesuai": true,
            "status_item": "ACCEPTED",
            "catatan": "Upgrade produk diterima dengan baik"
        }
    ],
    "retur_case_status": "CLOSED",
    "satisfaction_rating": 5,
    "feedback": "Replacement process sangat memuaskan dan cepat"
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Konfirmasi replacement berhasil diproses",
    "replacement_id": "RPL/PBF/24/001",
    "status_replacement": "COMPLETED",
    "retur_case_status": "CLOSED",
    "processed_at": "2024-12-25T15:35:00+07:00",
    "next_action": "Case retur telah selesai"
}
```

### 3. Cek Status Replacement dari Balimed

**Endpoint yang WAJIB Dipanggil PBF**: `GET /api/balimed/return-fulfillments/{replacement_id}/status`
**Direction**: **PBF (query)** → Balimed
**Usage**: Untuk monitoring status replacement

**Headers yang WAJIB Dikirim PBF:**

```http
Authorization: Bearer {api_token}
X-Distributor-Code: PBF_DIST_001
X-Request-ID: {unique_request_id}
```

**Response yang Akan Diterima PBF:**

```json
{
    "status": "success",
    "replacement_id": "RPL/BALIMED/24/001",
    "nomor_replacement_pbf": "RPL/PBF/24/001",
    "current_status": "CONFIRMED",
    "status_history": [
        {
            "status": "PENDING_RECEIPT",
            "timestamp": "2024-12-25T09:05:00+07:00",
            "pic": "System"
        },
        {
            "status": "IN_TRANSIT",
            "timestamp": "2024-12-25T10:00:00+07:00",
            "pic": "Driver"
        },
        {
            "status": "DELIVERED",
            "timestamp": "2024-12-25T14:30:00+07:00",
            "pic": "Driver"
        },
        {
            "status": "CONFIRMED",
            "timestamp": "2024-12-25T15:30:00+07:00",
            "pic": "Apt. Made Sari"
        }
    ],
    "estimated_completion": "2024-12-25T17:00:00+07:00",
    "last_updated": "2024-12-25T15:30:00+07:00"
}
```

---

## Database yang Harus Disiapkan PBF

### 1. Table Master Replacement

```sql
CREATE TABLE pbf_replacements (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nomor_replacement VARCHAR(50) UNIQUE NOT NULL,
    return_id BIGINT REFERENCES pbf_returns(id),
    retur_reference VARCHAR(50) NOT NULL,
    nomor_retur_balimed VARCHAR(50) NOT NULL,
    credit_note_reference VARCHAR(50),
    tanggal_replacement DATE NOT NULL,
    jenis_replacement ENUM('PRODUCT_REPLACEMENT', 'CREDIT_COMPENSATION', 'UPGRADE_REPLACEMENT') NOT NULL,
    nomor_faktur_replacement VARCHAR(50),
    tanggal_faktur DATE,
    metode_pengiriman ENUM('DELIVERY', 'PICKUP', 'COURIER') NOT NULL,
    biaya_pengiriman DECIMAL(10,2) DEFAULT 0,
    status_replacement ENUM('PREPARED', 'SENT', 'DELIVERED', 'CONFIRMED', 'COMPLETED', 'REJECTED') DEFAULT 'PREPARED',
    penerima VARCHAR(100),
    alamat_pengiriman TEXT,
    nama_driver VARCHAR(100),
    nomor_kendaraan VARCHAR(20),
    estimasi_tiba TIMESTAMP,
    tanggal_aktual_tiba TIMESTAMP NULL,
    satisfaction_rating INT NULL,
    feedback TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 2. Table Detail Replacement

```sql
CREATE TABLE pbf_replacement_details (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    replacement_id BIGINT REFERENCES pbf_replacements(id),
    line_number INT NOT NULL,
    retur_line_reference INT NOT NULL,
    kode_barang_original VARCHAR(50) NOT NULL,
    nama_barang_original VARCHAR(200) NOT NULL,
    kode_barang_replacement VARCHAR(50) NOT NULL,
    nama_barang_replacement VARCHAR(200) NOT NULL,
    satuan VARCHAR(20) NOT NULL,
    qty_retur INT NOT NULL,
    qty_replacement INT NOT NULL,
    qty_diterima INT DEFAULT 0,
    harga_satuan DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(15,2) NOT NULL,
    no_batch_original VARCHAR(50),
    no_batch_replacement VARCHAR(50) NOT NULL,
    expired_original DATE,
    expired_replacement DATE NOT NULL,
    jenis_replacement ENUM('SAME_PRODUCT', 'EQUIVALENT_PRODUCT', 'UPGRADE_PRODUCT', 'SUBSTITUTE_PRODUCT') NOT NULL,
    kondisi_barang ENUM('BARU', 'REFURBISHED', 'DEMO') DEFAULT 'BARU',
    status_item ENUM('SENT', 'ACCEPTED', 'REJECTED', 'PARTIAL') DEFAULT 'SENT',
    alasan_replacement VARCHAR(500),
    catatan_item TEXT
);
```

### 3. Table Status Tracking

```sql
CREATE TABLE pbf_replacement_status_history (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    replacement_id BIGINT REFERENCES pbf_replacements(id),
    status_dari ENUM('PREPARED', 'SENT', 'DELIVERED', 'CONFIRMED', 'COMPLETED', 'REJECTED'),
    status_ke ENUM('PREPARED', 'SENT', 'DELIVERED', 'CONFIRMED', 'COMPLETED', 'REJECTED') NOT NULL,
    timestamp_perubahan TIMESTAMP NOT NULL,
    pic VARCHAR(100),
    catatan TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Table Dokumen Replacement

```sql
CREATE TABLE pbf_replacement_documents (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    replacement_id BIGINT REFERENCES pbf_replacements(id),
    jenis_dokumen ENUM('DELIVERY_NOTE', 'REPLACEMENT_CERTIFICATE', 'QUALITY_CERTIFICATE', 'INVOICE_REPLACEMENT') NOT NULL,
    nama_file VARCHAR(200) NOT NULL,
    url_download VARCHAR(500) NOT NULL,
    hash_file VARCHAR(100),
    ukuran_file BIGINT,
    uploaded_by VARCHAR(100),
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Business Rules yang Harus Diimplementasi PBF

### 1. Replacement Preparation
- **WAJIB** prepare replacement dalam 48 jam setelah retur disetujui
- **WAJIB** pastikan kualitas produk replacement lebih baik/sama
- **WAJIB** batch number dan expired date lebih baik dari yang diretur
- **WAJIB** dokumentasi lengkap untuk audit trail

### 2. Quality Assurance
- **WAJIB** QA check untuk semua replacement products
- **WAJIB** certificate of quality untuk pharmaceutical products
- **WAJIB** cold chain maintenance untuk temperature-sensitive products
- **WAJIB** packaging sesuai standar farmasi

### 3. Delivery Management
- **WAJIB** notifikasi delivery schedule ke Balimed H-1
- **WAJIB** real-time tracking untuk delivery status
- **WAJIB** proof of delivery dengan signature dan foto
- **WAJIB** escalation process jika delivery tertunda

### 4. Case Closure
- **WAJIB** konfirmasi case closure dengan customer satisfaction
- **WAJIB** dokumentasi lessons learned untuk improvement
- **WAJIB** update internal database untuk future reference

### 5. Error Handling
```json
{
    "status": "error",
    "error_code": "REPLACEMENT_QUALITY_ISSUE",
    "message": "Produk replacement tidak memenuhi standar kualitas",
    "details": {
        "product_code": "BRG001",
        "batch_number": "PCM241101",
        "issue": "Expired date kurang dari 12 bulan"
    },
    "timestamp": "2024-12-25T09:00:00+07:00"
}
```

---

## Workflow Replacement yang Harus Diimplementasi

### 1. Preparation Phase
```python
def prepare_replacement(approved_return):
    # Validate replacement criteria
    if not validate_replacement_eligibility(approved_return):
        return reject_replacement("Tidak memenuhi syarat replacement")

    # Select best replacement products
    replacement_products = select_optimal_replacement(approved_return.items)

    # Prepare QA documentation
    qa_docs = generate_qa_certificates(replacement_products)

    # Schedule delivery
    delivery_schedule = schedule_delivery(approved_return.address)

    return prepare_replacement_package(replacement_products, qa_docs, delivery_schedule)
```

### 2. Quality Control
- **WAJIB** visual inspection untuk semua products
- **WAJIB** batch testing untuk pharmaceutical compliance
- **WAJIB** packaging inspection sesuai standard
- **WAJIB** documentation review untuk completeness

### 3. Delivery Execution
- **WAJIB** pre-delivery customer notification
- **WAJIB** real-time GPS tracking untuk delivery vehicle
- **WAJIB** temperature monitoring untuk cold chain products
- **WAJIB** delivery confirmation dengan digital signature

---

## Integration Testing Checklist untuk PBF

### Phase 1: Replacement Process
- [ ] Implement replacement preparation workflow
- [ ] Implement quality assurance process
- [ ] Test notification system ke Balimed
- [ ] Test document generation dan management

### Phase 2: Delivery Management
- [ ] Implement delivery scheduling system
- [ ] Test real-time tracking integration
- [ ] Implement delivery confirmation process
- [ ] Test escalation procedures

### Phase 3: Case Closure
- [ ] Implement confirmation handling dari Balimed
- [ ] Test satisfaction rating dan feedback system
- [ ] Implement case closure automation
- [ ] Test reporting dan analytics

### Phase 4: Quality & Compliance
- [ ] Setup quality assurance procedures
- [ ] Implement pharmaceutical compliance checks
- [ ] Test cold chain monitoring integration
- [ ] Setup audit trail dan documentation

---

_API ini melengkapi siklus return management dengan replacement process yang berkualitas, memastikan customer satisfaction dan compliance dengan standar farmasi._