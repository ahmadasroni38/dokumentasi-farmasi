# Retur Distributor API Specification

## Integrasi Sistem Farmasi Balimed dengan Distributor PT. PBF

### Database Structure Analysis

**Master Table**: `m_farmasi_retur_d`
**Detail Table**: `t_farmasi_retur_d`

### Master Retur Distributor Fields

```php
// Model: ReturD (m_farmasi_retur_d)
protected $fillable = [
    'tanggal',              // Date - Return date
    'nomor',                // String - Return number (auto-generated)
    'subunit_id',           // Integer - Hospital subunit ID
    'distributor_id',       // Integer - Foreign key to m_farmasi_distributor
    'note',                 // Text - Return notes
    'diskon',               // Decimal - Return discount percentage
    'ppn',                  // Decimal - Tax percentage (PPN)
    'status_proof_kode',    // String - Approval status code
    'status_kode',          // String - Return status code
    'created_by',           // String - User ID who created
    'updated_by',           // String - User ID who updated
    'subunit_owner_id',     // Integer - Hospital subunit owner
    'jdoc_kode',            // String - Document journal code
    'kode_doc',             // String - Document code
    'nomor_faktur',         // String - Original invoice number reference
    'file_faktur',          // String - Uploaded file path
    'biaya_lain',           // Decimal - Additional charges
    'total_faktur',         // Decimal - Total return amount
    'disc_rupiah',          // Decimal - Discount amount in rupiah
];
```

### Detail Retur Distributor Fields

```php
// Model: TReturD (t_farmasi_retur_d)
protected $fillable = [
    'retur_id',         // Integer - Foreign key to m_farmasi_retur_d
    'barang_id',        // Integer - Foreign key to m_farmasi_barang
    'stock_c',          // Integer - Current stock before return
    'qty',              // Integer - Quantity to return
    'qty_ff',           // Integer - Quantity actually returned (fulfilled)
    'harga_hna',        // Decimal - Unit price (HNA = Harga Netto Apotek)
    'diskon',           // Decimal - Item discount percentage
    'satuan_kode',      // String - Unit code (strip, box, bottle, etc)
    'alasan_kode',      // String - Return reason code
    'status_aksi_kode', // String - Return action status code
    'batchno',          // String - Batch number
    'created_by',       // String - User ID
    'updated_by',       // String - User ID
    'expired_date',     // Date - Product expiry date
];
```

### Return Reason Codes (alasan_kode)

```php
// From MRef table with specific group_id for return reasons
'DAMAGED'   => 'Barang Rusak/Cacat',
'EXPIRED'   => 'Barang Kadaluarsa',
'EXCESS'    => 'Kelebihan Stock',
'WRONG'     => 'Barang Salah/Tidak Sesuai Order',
'RECALL'    => 'Product Recall dari BPOM',
'DEFECTIVE' => 'Defective/Tidak Berfungsi',
'NEAR_ED'   => 'Mendekati Expired Date',
'OTHER'     => 'Lainnya'
```

### Return Action Codes (status_aksi_kode)

```php
'REPLACEMENT' => 'Minta Penggantian',
'CREDIT_NOTE' => 'Credit Note',
'REFUND'      => 'Refund/Pengembalian Uang',
'EXCHANGE'    => 'Tukar dengan Item Lain'
```

### API Endpoints

#### 1. Send Return Request to Distributor

**Endpoint**: `POST /api/distributor/returns`
**Direction**: Balimed → Distributor
**Content-Type**: `application/json`

**Headers:**

```http
Authorization: Bearer {api_token}
Content-Type: application/json
X-Hospital-Code: BALIMED_DENPASAR
X-Request-ID: {unique_request_id}
```

**Request Payload:**

```json
{
    "header": {
        "nomor_retur": "RTN/24/12/0001/DENPSR",
        "tanggal": "2024-12-20",
        "subunit_id": 1,
        "subunit_code": "FARM_IGD",
        "subunit_name": "Farmasi IGD",
        "distributor_id": 1,
        "distributor_code": "DIST001",
        "nomor_faktur": "FK/DIST001/2024/001",
        "ro_reference": "RO/24/12/0001/DENPSR",
        "diskon": 5.5,
        "ppn": 11.0,
        "biaya_lain": 0.0,
        "total_faktur": 87250.0,
        "disc_rupiah": 4362.5,
        "note": "Retur barang rusak saat diterima",
        "created_by": "APT001",
        "created_by_name": "Apt. Made Sari",
        "created_date": "2024-12-20T09:00:00+07:00",
        "return_type": "DAMAGED_GOODS",
        "urgency_level": "NORMAL"
    },
    "details": [
        {
            "line_number": 1,
            "barang_id": 123,
            "kode_barang": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "nama_generik": "Parasetamol",
            "satuan_kode": "STRIP",
            "batchno": "PCM240801",
            "expired_date": "2026-08-01",
            "stock_c": 90,
            "qty": 5,
            "harga_hna": 2500.0,
            "diskon": 5.0,
            "sub_total": 11875.0,
            "alasan_kode": "DAMAGED",
            "alasan_keterangan": "Kemasan strip rusak saat diterima",
            "status_aksi_kode": "REPLACEMENT",
            "keterangan": "5 strip dengan kemasan rusak, mohon penggantian",
            "photo_evidence": [
                {
                    "filename": "damaged_pcm_001.jpg",
                    "file_base64": "/9j/4AAQSkZJRgABAQEAeAB4AAD/2wBDAAEBAQEBAQEBAQEBAQEBAQEBAQEBAQ...",
                    "description": "Foto kemasan yang rusak"
                }
            ]
        },
        {
            "line_number": 2,
            "barang_id": 124,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "nama_generik": "Amoksisilin",
            "satuan_kode": "STRIP",
            "batchno": "AMX240701",
            "expired_date": "2026-07-01",
            "stock_c": 25,
            "qty": 10,
            "harga_hna": 3500.0,
            "diskon": 3.0,
            "sub_total": 33950.0,
            "alasan_kode": "NEAR_ED",
            "alasan_keterangan": "Mendekati expired date, sisa 6 bulan",
            "status_aksi_kode": "CREDIT_NOTE",
            "keterangan": "Stock mendekati ED, mohon credit note"
        }
    ],
    "summary": {
        "total_items": 2,
        "total_qty": 15,
        "subtotal": 45825.0,
        "discount_amount": 2291.25,
        "taxable_amount": 43533.75,
        "tax_amount": 4789.71,
        "grand_total": 48323.46
    },
    "attachments": [
        {
            "type": "EVIDENCE_PHOTO",
            "filename": "retur_evidence_001.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "description": "Dokumentasi barang yang diretur"
        }
    ]
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Return request berhasil diterima",
    "data": {
        "distributor_return_id": "DRTN-2024-001",
        "balimed_return_number": "RTN/24/12/0001/DENPSR",
        "status": "RECEIVED",
        "received_date": "2024-12-20T09:15:00+07:00",
        "estimated_process_time": "2-3 hari kerja",
        "case_handler": "Budi Santoso",
        "case_handler_phone": "0361-123456",
        "case_handler_email": "budi@pbfmedika.com",
        "tracking_url": "https://pbfmedika.com/returns/DRTN-2024-001"
    }
}
```

#### 2. Receive Return Status Update from Distributor

**Endpoint**: `PATCH /api/balimed/returns/{nomor_retur}/status`
**Direction**: Distributor → Balimed

**Request Payload:**

```json
{
    "distributor_return_id": "DRTN-2024-001",
    "balimed_return_number": "RTN/24/12/0001/DENPSR",
    "status": "APPROVED",
    "status_keterangan": "Return request disetujui dengan penyesuaian",
    "process_date": "2024-12-21T14:00:00+07:00",
    "processed_by": "Budi Santoso",
    "processed_by_position": "Return Handler",
    "estimated_completion": "2024-12-23T17:00:00+07:00",
    "case_notes": "Semua item return disetujui, replacement akan dikirim bersamaan dengan credit note",
    "items_status": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "batchno": "PCM240801",
            "status": "APPROVED",
            "qty_approved": 5,
            "action_approved": "REPLACEMENT",
            "replacement_batch": "PCM241201",
            "replacement_ed": "2027-12-01",
            "estimated_replacement_date": "2024-12-23T14:00:00+07:00",
            "keterangan": "Penggantian disetujui dengan batch baru"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "batchno": "AMX240701",
            "status": "APPROVED",
            "qty_approved": 10,
            "action_approved": "CREDIT_NOTE",
            "credit_amount": 33950.0,
            "credit_note_number": "CN/DIST001/2024/001",
            "credit_note_date": "2024-12-21",
            "keterangan": "Credit note akan diterbitkan"
        }
    ],
    "financial_impact": {
        "total_return_value": 48323.46,
        "replacement_value": 11875.0,
        "credit_note_value": 33950.0,
        "net_financial_impact": 2498.46
    }
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Status return berhasil diupdate",
    "data": {
        "nomor_retur": "RTN/24/12/0001/DENPSR",
        "status_updated": "APPROVED",
        "updated_at": "2024-12-21T14:05:00+07:00"
    }
}
```

#### 3. Handle Return Rejection from Distributor

**Endpoint**: `PATCH /api/balimed/returns/{nomor_retur}/rejection`
**Direction**: Distributor → Balimed

**Request Payload:**

```json
{
    "distributor_return_id": "DRTN-2024-001",
    "balimed_return_number": "RTN/24/12/0001/DENPSR",
    "status": "REJECTED",
    "rejection_date": "2024-12-21T11:00:00+07:00",
    "rejected_by": "Budi Santoso",
    "rejection_reason": "INSUFFICIENT_EVIDENCE",
    "rejection_details": "Bukti kerusakan tidak memadai untuk beberapa item",
    "items_status": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "batchno": "PCM240801",
            "status": "REJECTED",
            "rejection_reason": "Foto evidence tidak jelas menunjukkan kerusakan",
            "required_action": "Mohon kirim foto yang lebih jelas atau video",
            "keterangan": "Perlu evidence tambahan"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "batchno": "AMX240701",
            "status": "APPROVED",
            "qty_approved": 10,
            "action_approved": "CREDIT_NOTE",
            "credit_amount": 33950.0,
            "keterangan": "Item ini disetujui untuk credit note"
        }
    ],
    "next_steps": ["Kirim evidence tambahan untuk item BRG001", "Submit ulang return request dengan dokumentasi yang lengkap", "Hubungi case handler untuk diskusi lebih lanjut"],
    "resubmission_deadline": "2024-12-28T17:00:00+07:00"
}
```

#### 4. Get Return Status for Hospital

**Endpoint**: `GET /api/balimed/returns/{nomor_retur}/status`
**Direction**: Balimed internal use

**Response:**

```json
{
    "status": "success",
    "data": {
        "nomor_retur": "RTN/24/12/0001/DENPSR",
        "current_status": "APPROVED",
        "distributor_return_id": "DRTN-2024-001",
        "status_history": [
            {
                "status": "CREATED",
                "timestamp": "2024-12-20T09:00:00+07:00",
                "user": "APT001",
                "note": "Return request dibuat"
            },
            {
                "status": "SENT_TO_DISTRIBUTOR",
                "timestamp": "2024-12-20T09:15:00+07:00",
                "user": "System",
                "note": "Dikirim ke distributor via API"
            },
            {
                "status": "RECEIVED_BY_DISTRIBUTOR",
                "timestamp": "2024-12-20T09:15:00+07:00",
                "user": "Distributor",
                "note": "Diterima oleh sistem distributor"
            },
            {
                "status": "UNDER_REVIEW",
                "timestamp": "2024-12-21T08:00:00+07:00",
                "user": "Distributor",
                "note": "Sedang direview oleh tim distributor"
            },
            {
                "status": "APPROVED",
                "timestamp": "2024-12-21T14:00:00+07:00",
                "user": "Distributor",
                "note": "Return request disetujui"
            }
        ],
        "return_summary": {
            "total_items": 2,
            "items_approved": 2,
            "items_rejected": 0,
            "total_value": 48323.46,
            "replacement_value": 11875.0,
            "credit_note_value": 33950.0
        },
        "next_actions": ["Tunggu pengiriman replacement items", "Terima credit note yang akan diterbitkan", "Update stock system setelah replacement diterima"]
    }
}
```

#### 5. Cancel Return Request

**Endpoint**: `DELETE /api/distributor/returns/{nomor_retur}`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "nomor_retur": "RTN/24/12/0001/DENPSR",
    "cancellation_reason": "ITEM_ALREADY_USED",
    "cancellation_note": "Item sudah terpakai untuk pasien, tidak jadi diretur",
    "cancelled_by": "APT001",
    "cancelled_by_name": "Apt. Made Sari",
    "cancellation_date": "2024-12-20T15:00:00+07:00"
}
```

### Status Codes

#### Return Status (status_kode)

-   `1` - DRAFT (Draft)
-   `2` - SENT (Terkirim ke Distributor)
-   `3` - RECEIVED (Diterima Distributor)
-   `4` - UNDER_REVIEW (Sedang Direview)
-   `5` - APPROVED (Disetujui)
-   `6` - PARTIAL_APPROVED (Sebagian Disetujui)
-   `7` - REJECTED (Ditolak)
-   `8` - COMPLETED (Selesai)
-   `9` - CANCELLED (Dibatalkan)

#### Approval Status (status_proof_kode)

-   `PENDING` - Menunggu Approval Internal
-   `APPROVED` - Disetujui Internal
-   `REJECTED` - Ditolak Internal

### Business Rules

1. **Return Number Format**: `RTN/YY/MM/NNNN/LOCATION`

2. **Validation Rules**:

    - `nomor` required, unique
    - `tanggal` required, cannot be future date
    - `distributor_id` required, must exist and be active
    - `qty` required, integer, min: 1, max: current_stock
    - `batchno` required for pharmaceutical items
    - `expired_date` required, must match stock records
    - `alasan_kode` required, must be valid reason code
    - `status_aksi_kode` required, must be valid action code

3. **Business Logic**:

    - Return hanya dapat dibuat untuk barang yang sudah diterima (ada di RO)
    - Qty return tidak boleh melebihi stock yang tersedia
    - Batch number dan expiry date harus sesuai dengan stock records
    - Return untuk barang expired hanya diperbolehkan dalam periode tertentu
    - Stock berkurang otomatis setelah return approved dan processed

4. **Stock Impact**:
    - Stock reduction terjadi setelah return physically collected
    - Update `m_farmasi_buku_stock` dan `m_farmasi_lokasi_barang`
    - Record transaction di `t_farmasi_stock_combined`
    - Update cost calculations in `hpp_barang`

### Integration Points

1. **With Receiving Order System**:

    - Validate item was actually received
    - Check batch and expiry date against RO records
    - Link return to original RO for traceability

2. **With Stock Management**:

    - Validate available stock before return
    - Update stock levels after return collection
    - Handle batch tracking and expiry management

3. **With Finance System**:
    - Process credit notes for accounting
    - Handle replacement item values
    - Update cost of goods sold calculations

### Return Process Workflow

1. **Hospital Creates Return**:

    - Pharmacist identifies items to return
    - Creates return request in system
    - Provides evidence (photos, documentation)
    - Submits for internal approval

2. **Internal Approval**:

    - Supervisor reviews return request
    - Validates reason and evidence
    - Approves or rejects internally

3. **Send to Distributor**:

    - System automatically sends to distributor via API
    - Distributor acknowledges receipt

4. **Distributor Processing**:

    - Distributor reviews request and evidence
    - Makes decision on each item
    - Sends status update back

5. **Resolution**:
    - For approved returns: Physical collection scheduled
    - For replacements: New delivery arranged
    - For credit notes: Financial adjustment processed

### Error Handling

**Common Error Codes:**

-   `VALIDATION_ERROR` - Input validation failed
-   `INSUFFICIENT_STOCK` - Not enough stock to return
-   `BATCH_MISMATCH` - Batch number doesn't match records
-   `EXPIRED_RETURN_PERIOD` - Return period has expired
-   `DUPLICATE_RETURN` - Item already in return process
-   `UNAUTHORIZED_RETURN` - Not authorized to return this item
-   `SYSTEM_ERROR` - Internal server error

### File Handling

**Evidence Files**:

-   Photos of damaged items (JPG, PNG)
-   Documentation (PDF)
-   Video evidence (MP4) for complex cases

**File Requirements**:

-   Maximum file size: 5MB per file
-   Total attachment limit: 25MB per return
-   Required for certain return reasons (DAMAGED, DEFECTIVE)

### Audit Trail

All return transactions must maintain complete audit trail:

-   Who created the return
-   Internal approval chain
-   Distributor decision rationale
-   Physical collection confirmation
-   Stock adjustment records
-   Financial impact tracking
