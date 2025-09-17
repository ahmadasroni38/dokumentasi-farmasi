# Faktur Retur API Specification

## Requirements untuk Programmer PT. PBF

### Yang Harus Disiapkan oleh PBF

API ini menjelaskan struktur data dan endpoint yang **WAJIB disiapkan oleh programmer PT. PBF** untuk mengirim credit note dan menerima verifikasi dari Sistem Farmasi Balimed.

---

## Endpoints yang WAJIB Dibuat dan Dipanggil PBF

### 1. Kirim Credit Note ke Balimed

**Endpoint yang WAJIB Dipanggil PBF**: `POST /api/balimed/credit-notes`
**Direction**: **PBF (mengirim)** → Balimed
**Trigger**: Setelah retur disetujui dan credit note dibuat

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
        "nomor_credit_note": "CN/PBF001/24/001",
        "tanggal_credit_note": "2024-12-22",
        "retur_reference": "RTN/PBF/24/001",
        "nomor_retur_balimed": "RTN/24/12/0001/DENPSR",
        "nomor_faktur_original": "FK/PBF001/2024/001",
        "distributor_code": "PBF_DIST_001",
        "distributor_name": "PT. PBF Medika",
        "hospital_code": "BALIMED_DENPASAR",
        "jenis_credit": "RETURN_CREDIT",
        "alasan_credit": "Pengembalian barang sesuai retur yang disetujui",
        "total_credit": 16000.0,
        "ppn_credit": 1760.0,
        "net_credit": 17760.0,
        "metode_credit": "ADJUSTMENT",
        "tempo_hari": 30,
        "tanggal_jatuh_tempo": "2025-01-21",
        "pic_finance": "Finance Manager PBF",
        "telp_pic": "021-1234567",
        "catatan": "Credit note untuk retur barang damaged",
        "created_by": "SYS_PBF",
        "created_date": "2024-12-22T10:00:00+07:00"
    },
    "detail_credit": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "satuan": "STRIP",
            "qty_credit": 5,
            "harga_satuan": 2500.0,
            "disc_persen": 5.0,
            "harga_setelah_disc": 2375.0,
            "subtotal": 11875.0,
            "ppn_persen": 11.0,
            "ppn_nominal": 1306.25,
            "total_line": 13181.25,
            "no_batch": "PCM240801",
            "tanggal_expired": "2026-08-01",
            "alasan_credit": "Barang damaged",
            "referensi_retur_line": 1
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "satuan": "STRIP",
            "qty_credit": 1,
            "harga_satuan": 3500.0,
            "disc_persen": 5.0,
            "harga_setelah_disc": 3325.0,
            "subtotal": 3325.0,
            "ppn_persen": 11.0,
            "ppn_nominal": 365.75,
            "total_line": 3690.75,
            "no_batch": "AMX240901",
            "tanggal_expired": "2026-09-01",
            "alasan_credit": "Near expired partial credit",
            "referensi_retur_line": 2
        }
    ],
    "dokumen_pendukung": [
        {
            "jenis_dokumen": "CREDIT_NOTE_ORIGINAL",
            "nama_file": "CN_PBF001_24_001.pdf",
            "url_download": "https://pbf.com/documents/CN_PBF001_24_001.pdf",
            "hash_file": "a1b2c3d4e5f6",
            "ukuran_file": 524288
        },
        {
            "jenis_dokumen": "RETURN_EVIDENCE",
            "nama_file": "return_evidence.pdf",
            "url_download": "https://pbf.com/documents/return_evidence.pdf",
            "hash_file": "f6e5d4c3b2a1",
            "ukuran_file": 1048576
        }
    ]
}
```

**Response yang Akan Diterima PBF dari Balimed:**

```json
{
    "status": "success",
    "message": "Credit note berhasil diterima",
    "credit_note_id": "CN/BALIMED/24/001",
    "nomor_credit_pbf": "CN/PBF001/24/001",
    "status_verifikasi": "PENDING_VERIFICATION",
    "estimasi_verifikasi": "2024-12-24T17:00:00+07:00",
    "pic_verifikasi": "Finance Balimed",
    "received_at": "2024-12-22T10:05:00+07:00"
}
```

### 2. Terima Verifikasi Credit Note dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/credit-notes/{credit_note_id}/verification`
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
    "credit_note_id": "CN/BALIMED/24/001",
    "nomor_credit_pbf": "CN/PBF001/24/001",
    "status_verifikasi": "VERIFIED",
    "tanggal_verifikasi": "2024-12-23T14:00:00+07:00",
    "pic_verifikasi": "Apt. Finance Manager",
    "hasil_verifikasi": "APPROVED",
    "catatan_verifikasi": "Credit note sesuai dengan retur yang disetujui",
    "detail_verifikasi": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "status_line": "VERIFIED",
            "qty_verified": 5,
            "nilai_verified": 13181.25,
            "catatan_line": "Sesuai dengan retur"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "status_line": "VERIFIED",
            "qty_verified": 1,
            "nilai_verified": 3690.75,
            "catatan_line": "Partial quantity sesuai approval"
        }
    ],
    "total_verified": 16872.0,
    "adjustment_amount": 0.0,
    "final_credit_amount": 16872.0,
    "next_action": "Akan diproses ke finance untuk payment adjustment"
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Verifikasi credit note berhasil diterima",
    "credit_note_id": "CN/PBF001/24/001",
    "status_internal": "VERIFIED_BY_CUSTOMER",
    "processed_at": "2024-12-23T14:05:00+07:00",
    "next_action": "Credit akan diaplikasikan ke outstanding balance"
}
```

### 3. Terima Dispute Credit Note dari Balimed

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/credit-notes/{credit_note_id}/dispute`
**Direction**: Balimed → **PBF (menerima)**

**Payload yang Akan Diterima PBF jika Ada Dispute:**

```json
{
    "credit_note_id": "CN/BALIMED/24/001",
    "nomor_credit_pbf": "CN/PBF001/24/001",
    "status_dispute": "DISPUTED",
    "tanggal_dispute": "2024-12-23T14:00:00+07:00",
    "pic_dispute": "Finance Manager Balimed",
    "jenis_dispute": "CALCULATION_ERROR",
    "deskripsi_dispute": "Perhitungan PPN tidak sesuai",
    "detail_dispute": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "jenis_masalah": "PPN_CALCULATION",
            "nilai_pbf": 1306.25,
            "nilai_balimed": 1306.25,
            "selisih": 0.0,
            "catatan": "Actually this line is correct"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "jenis_masalah": "QUANTITY_MISMATCH",
            "qty_pbf": 1,
            "qty_balimed": 0,
            "selisih": 1,
            "catatan": "Item ini tidak disetujui untuk retur"
        }
    ],
    "dokumen_pendukung": [
        {
            "nama_file": "dispute_evidence.pdf",
            "url_download": "https://balimed.com/documents/dispute_evidence.pdf"
        }
    ]
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "message": "Dispute credit note berhasil diterima",
    "dispute_id": "DISP/PBF/24/001",
    "credit_note_id": "CN/PBF001/24/001",
    "status_penanganan": "UNDER_REVIEW",
    "estimasi_penyelesaian": "2024-12-25T17:00:00+07:00",
    "pic_penanganan": "Finance Manager PBF",
    "telp_pic": "021-1234567",
    "processed_at": "2024-12-23T14:10:00+07:00"
}
```

---

## Database yang Harus Disiapkan PBF

### 1. Table Master Credit Note

```sql
CREATE TABLE pbf_credit_notes (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nomor_credit_note VARCHAR(50) UNIQUE NOT NULL,
    return_id BIGINT REFERENCES pbf_returns(id),
    retur_reference VARCHAR(50) NOT NULL,
    nomor_retur_balimed VARCHAR(50) NOT NULL,
    nomor_faktur_original VARCHAR(50) NOT NULL,
    tanggal_credit_note DATE NOT NULL,
    jenis_credit ENUM('RETURN_CREDIT', 'DAMAGE_CREDIT', 'ADJUSTMENT_CREDIT') NOT NULL,
    alasan_credit TEXT NOT NULL,
    total_credit DECIMAL(15,2) NOT NULL,
    ppn_credit DECIMAL(15,2) NOT NULL,
    net_credit DECIMAL(15,2) NOT NULL,
    metode_credit ENUM('ADJUSTMENT', 'REFUND', 'CREDIT_BALANCE') NOT NULL,
    tempo_hari INT NOT NULL,
    tanggal_jatuh_tempo DATE NOT NULL,
    status_credit ENUM('DRAFT', 'SENT', 'VERIFIED', 'DISPUTED', 'RESOLVED', 'APPLIED') DEFAULT 'DRAFT',
    pic_finance VARCHAR(100),
    catatan TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 2. Table Detail Credit Note

```sql
CREATE TABLE pbf_credit_note_details (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    credit_note_id BIGINT REFERENCES pbf_credit_notes(id),
    line_number INT NOT NULL,
    kode_barang VARCHAR(50) NOT NULL,
    nama_barang VARCHAR(200) NOT NULL,
    satuan VARCHAR(20) NOT NULL,
    qty_credit INT NOT NULL,
    harga_satuan DECIMAL(10,2) NOT NULL,
    disc_persen DECIMAL(5,2) DEFAULT 0,
    harga_setelah_disc DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(15,2) NOT NULL,
    ppn_persen DECIMAL(5,2) NOT NULL,
    ppn_nominal DECIMAL(15,2) NOT NULL,
    total_line DECIMAL(15,2) NOT NULL,
    no_batch VARCHAR(50) NOT NULL,
    tanggal_expired DATE NOT NULL,
    alasan_credit VARCHAR(200),
    referensi_retur_line INT,
    status_verifikasi ENUM('PENDING', 'VERIFIED', 'DISPUTED') DEFAULT 'PENDING'
);
```

### 3. Table Verifikasi dan Dispute

```sql
CREATE TABLE pbf_credit_verifications (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    credit_note_id BIGINT REFERENCES pbf_credit_notes(id),
    status_verifikasi ENUM('VERIFIED', 'DISPUTED', 'PARTIAL_VERIFIED') NOT NULL,
    tanggal_verifikasi TIMESTAMP NOT NULL,
    pic_verifikasi VARCHAR(100),
    hasil_verifikasi ENUM('APPROVED', 'REJECTED', 'PARTIAL_APPROVED') NOT NULL,
    catatan_verifikasi TEXT,
    total_verified DECIMAL(15,2),
    adjustment_amount DECIMAL(15,2) DEFAULT 0,
    final_credit_amount DECIMAL(15,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Table Dokumen Pendukung

```sql
CREATE TABLE pbf_credit_documents (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    credit_note_id BIGINT REFERENCES pbf_credit_notes(id),
    jenis_dokumen ENUM('CREDIT_NOTE_ORIGINAL', 'RETURN_EVIDENCE', 'DISPUTE_EVIDENCE', 'VERIFICATION_DOC') NOT NULL,
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

### 1. Credit Note Generation
- **WAJIB** generate credit note dalam 24 jam setelah retur disetujui
- **WAJIB** perhitungan yang akurat termasuk PPN dan discount
- **WAJIB** reference ke original invoice dan retur
- **WAJIB** attachment dokumen original credit note

### 2. Verification Process
- **WAJIB** respond verifikasi/dispute dalam 48 jam kerja
- **WAJIB** provide detailed explanation untuk setiap adjustment
- **WAJIB** maintain audit trail untuk semua perubahan

### 3. Finance Integration
- **WAJIB** integrasi dengan sistem accounting PBF
- **WAJIB** automatic application ke customer balance
- **WAJIB** tracking outstanding credit balance

### 4. Document Management
- **WAJIB** digital signature untuk credit note
- **WAJIB** PDF format untuk semua dokumen
- **WAJIB** backup dan archival sesuai regulasi

### 5. Error Handling
```json
{
    "status": "error",
    "error_code": "CREDIT_CALCULATION_ERROR",
    "message": "Perhitungan credit note tidak valid",
    "details": {
        "line_number": 2,
        "expected_amount": 3690.75,
        "received_amount": 3500.00,
        "difference": 190.75
    },
    "timestamp": "2024-12-22T10:00:00+07:00"
}
```

---

## Integration Testing Checklist untuk PBF

### Phase 1: Credit Note Generation
- [ ] Implement automatic credit note generation dari approved returns
- [ ] Test calculation accuracy termasuk tax dan discount
- [ ] Test document generation dan digital signature
- [ ] Test integration dengan accounting system

### Phase 2: Verification Workflow
- [ ] Implement endpoint untuk receive verification
- [ ] Implement dispute handling mechanism
- [ ] Test notification system untuk status changes
- [ ] Test document exchange dan validation

### Phase 3: Finance Integration
- [ ] Test credit application ke customer balance
- [ ] Implement reporting untuk finance reconciliation
- [ ] Test integration dengan payment system
- [ ] Setup monitoring untuk overdue credits

---

_API ini memungkinkan PBF untuk mengelola credit note secara otomatis dengan verification process yang transparan dan audit trail yang lengkap untuk compliance finansial._