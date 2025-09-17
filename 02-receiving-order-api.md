# Receiving Order API Specification

## Integrasi Sistem Farmasi Balimed dengan Distributor PT. PBF

### Database Structure Analysis

**Master Table**: `m_farmasi_ro`
**Detail Table**: `t_farmasi_ro`

### Master Receiving Order Fields

```php
// Model: ReceivingOrder (m_farmasi_ro)
protected $fillable = [
    'tanggal',                  // Date - RO receiving date
    'nomor_ro',                 // String - RO number (auto-generated)
    'nomor_faktur',             // String - Invoice number from distributor
    'distributor_id',           // Integer - Foreign key to m_farmasi_distributor
    'po_id',                    // Integer - Foreign key to m_farmasi_po
    'disc',                     // Decimal - Invoice discount percentage
    'ppn',                      // Decimal - Tax percentage (PPN)
    'ppn_nominal',              // Decimal - Tax amount in rupiah
    'carabayar_id',             // Integer - Payment method (CASH/CREDIT)
    'tanggal_jt',               // Date - Due date for payment
    'tempo_hari',               // Integer - Payment terms in days
    'tanggal_faktur',           // Date - Invoice date
    'biaya_lain',               // Decimal - Additional charges
    'kepada',                   // String - Delivery recipient name
    'telp',                     // String - Recipient phone number
    'alamat',                   // Text - Delivery address
    'note',                     // Text - Additional notes
    'file_pdf',                 // String - Uploaded invoice file path
    'created_by',               // String - User ID who created
    'updated_by',               // String - User ID who updated
    'subunit_owner_id',         // Integer - Hospital subunit owner
    'jdoc_kode',                // String - Document journal code
    'kode_doc',                 // String - Document code
    'status_proof_kode',        // String - Approval status code
    'status_kode',              // String - RO status code
    'total_faktur',             // Decimal - Total invoice amount
    'disc_rupiah',              // Decimal - Discount amount in rupiah
    'is_diserahkan_keuangan',   // Boolean - Submitted to finance dept
];
```

### Detail Receiving Order Fields

```php
// Model: TReceivingOrder (t_farmasi_ro)
protected $fillable = [
    'ro_id',                // Integer - Foreign key to m_farmasi_ro
    'barang_id',            // Integer - Foreign key to m_farmasi_barang
    'po_id',                // Integer - Foreign key to m_farmasi_po
    'qty_ff',               // Integer - Quantity fulfilled/received
    'harga_hna',            // Decimal - Unit price (HNA = Harga Netto Apotek)
    'satuan_id',            // Integer - Unit type ID
    'no_batch',             // String - Batch number
    'ed',                   // Date - Expiry date
    'disc',                 // Decimal - Item discount percentage
    'ppn',                  // Decimal - Item tax percentage
    'created_by',           // String - User ID
    'updated_by',           // String - User ID
    'qty_btach_terpakai',   // Integer - Quantity from this batch used
];
```

### API Endpoints

#### 1. Receive Delivery Notification from Distributor

**Endpoint**: `POST /api/balimed/receiving-orders`
**Direction**: Distributor → Balimed
**Content-Type**: `application/json`

**Headers:**

```http
Authorization: Bearer {api_token}
Content-Type: application/json
X-Distributor-Code: DIST001
X-Request-ID: {unique_request_id}
```

**Request Payload:**

```json
{
    "header": {
        "nomor_ro": "RO/24/12/0001/DENPSR",
        "nomor_faktur": "FK/DIST001/2024/001",
        "tanggal": "2024-12-18",
        "tanggal_faktur": "2024-12-18",
        "po_reference": "PO/24/12/0001/DENPSR",
        "distributor_id": 1,
        "distributor_code": "DIST001",
        "distributor_name": "PT. PBF Medika",
        "subunit_owner_id": 1,
        "carabayar_id": 2,
        "cara_bayar": "CREDIT",
        "tempo_hari": 30,
        "tanggal_jt": "2025-01-17",
        "disc": 5.5,
        "ppn": 11.0,
        "ppn_nominal": 42556.25,
        "disc_rupiah": 20362.5,
        "biaya_lain": 5000.0,
        "total_faktur": 434443.75,
        "kepada": "Apt. Made Sari",
        "telp": "0361-224466",
        "alamat": "Jl. Mahendradatta No.57, Denpasar",
        "note": "Pengiriman sesuai PO, semua item tersedia",
        "delivery_date": "2024-12-18T14:30:00+07:00",
        "driver_name": "Wayan Bagus",
        "driver_phone": "0812-3456-7890",
        "vehicle_number": "DK 1234 AB"
    },
    "details": [
        {
            "line_number": 1,
            "barang_id": 123,
            "kode_barang": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "nama_generik": "Parasetamol",
            "satuan_id": 1,
            "satuan_kode": "STRIP",
            "qty_ff": 100,
            "harga_hna": 2500.0,
            "disc": 5.0,
            "ppn": 11.0,
            "no_batch": "PCM240801",
            "ed": "2026-08-01",
            "sub_total": 237500.0,
            "keterangan": "Kondisi baik"
        },
        {
            "line_number": 2,
            "barang_id": 124,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "nama_generik": "Amoksisilin",
            "satuan_id": 1,
            "satuan_kode": "STRIP",
            "qty_ff": 30,
            "harga_hna": 3500.0,
            "disc": 3.0,
            "ppn": 11.0,
            "no_batch": "AMX240701",
            "ed": "2026-07-01",
            "sub_total": 101500.0,
            "keterangan": "Sesuai permintaan, partial delivery"
        }
    ],
    "summary": {
        "total_items": 2,
        "total_qty": 130,
        "subtotal": 339000.0,
        "discount_amount": 16950.0,
        "taxable_amount": 322050.0,
        "tax_amount": 35425.5,
        "other_charges": 5000.0,
        "grand_total": 362475.5
    },
    "attachments": [
        {
            "type": "INVOICE",
            "filename": "faktur_FK_DIST001_2024_001.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "file_size": 245760,
            "mime_type": "application/pdf"
        },
        {
            "type": "DELIVERY_NOTE",
            "filename": "surat_jalan_001.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "file_size": 156432,
            "mime_type": "application/pdf"
        }
    ]
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Delivery notification berhasil diterima",
    "data": {
        "nomor_ro": "RO/24/12/0001/DENPSR",
        "status": "PENDING_VERIFICATION",
        "received_date": "2024-12-18T14:45:00+07:00",
        "estimated_verification_time": "1-2 jam",
        "assigned_pharmacist": "Apt. Made Sari",
        "verification_location": "Gudang Farmasi IGD"
    }
}
```

#### 2. Send Receiving Confirmation to Distributor

**Endpoint**: `POST /api/distributor/receiving-orders/{nomor_ro}/confirmation`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "nomor_ro": "RO/24/12/0001/DENPSR",
    "nomor_faktur": "FK/DIST001/2024/001",
    "verification_date": "2024-12-18T16:00:00+07:00",
    "verification_status": "VERIFIED",
    "verified_by": "APT001",
    "verified_by_name": "Apt. Made Sari",
    "note": "Semua item diterima dengan kondisi baik, sesuai spesifikasi",
    "discrepancies": [],
    "verified_items": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "no_batch": "PCM240801",
            "ed": "2026-08-01",
            "qty_delivered": 100,
            "qty_received": 100,
            "qty_accepted": 100,
            "qty_rejected": 0,
            "condition": "GOOD",
            "rejection_reason": null,
            "storage_location": "RAK-A-01",
            "keterangan": "Diterima dengan kondisi baik"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "no_batch": "AMX240701",
            "ed": "2026-07-01",
            "qty_delivered": 30,
            "qty_received": 30,
            "qty_accepted": 30,
            "qty_rejected": 0,
            "condition": "GOOD",
            "rejection_reason": null,
            "storage_location": "RAK-B-02",
            "keterangan": "Diterima dengan kondisi baik"
        }
    ],
    "invoice_verification": {
        "invoice_status": "VERIFIED",
        "amount_verified": 362475.5,
        "amount_discrepancy": 0.0,
        "tax_verification": "CORRECT",
        "payment_approved": true,
        "finance_submission_date": "2024-12-18T16:30:00+07:00"
    }
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Konfirmasi penerimaan berhasil diproses",
    "data": {
        "nomor_ro": "RO/24/12/0001/DENPSR",
        "status": "COMPLETED",
        "completion_date": "2024-12-18T16:35:00+07:00",
        "next_action": "Invoice akan diproses untuk pembayaran"
    }
}
```

#### 3. Send Partial/Rejected Receiving Report

**Endpoint**: `POST /api/distributor/receiving-orders/{nomor_ro}/discrepancy`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "nomor_ro": "RO/24/12/0001/DENPSR",
    "nomor_faktur": "FK/DIST001/2024/001",
    "verification_date": "2024-12-18T16:00:00+07:00",
    "verification_status": "PARTIAL_RECEIVED",
    "verified_by": "APT001",
    "verified_by_name": "Apt. Made Sari",
    "note": "Ada ketidaksesuaian pada beberapa item",
    "discrepancies": [
        {
            "type": "QUANTITY_MISMATCH",
            "description": "Qty yang diterima kurang dari faktur",
            "severity": "MEDIUM"
        },
        {
            "type": "DAMAGED_GOODS",
            "description": "Ada kemasan yang rusak",
            "severity": "HIGH"
        }
    ],
    "verified_items": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "no_batch": "PCM240801",
            "ed": "2026-08-01",
            "qty_delivered": 100,
            "qty_received": 95,
            "qty_accepted": 90,
            "qty_rejected": 5,
            "condition": "PARTIAL_DAMAGED",
            "rejection_reason": "5 strip rusak kemasan",
            "storage_location": "RAK-A-01",
            "keterangan": "90 strip diterima, 5 strip ditolak karena kemasan rusak"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "no_batch": "AMX240701",
            "ed": "2026-07-01",
            "qty_delivered": 30,
            "qty_received": 25,
            "qty_rejected": 5,
            "qty_accepted": 25,
            "condition": "QUANTITY_SHORTAGE",
            "rejection_reason": "Qty kurang 5 strip dari faktur",
            "storage_location": "RAK-B-02",
            "keterangan": "Qty tidak sesuai faktur"
        }
    ],
    "invoice_verification": {
        "invoice_status": "DISPUTED",
        "amount_verified": 310350.0,
        "amount_discrepancy": 52125.5,
        "tax_verification": "NEEDS_ADJUSTMENT",
        "payment_approved": false,
        "dispute_items": ["BRG001 - 5 strip rejected", "BRG002 - 5 strip shortage"]
    }
}
```

#### 4. Get RO Status for Distributor

**Endpoint**: `GET /api/distributor/receiving-orders/{nomor_ro}/status`
**Direction**: Distributor → Balimed

**Response:**

```json
{
    "status": "success",
    "data": {
        "nomor_ro": "RO/24/12/0001/DENPSR",
        "nomor_faktur": "FK/DIST001/2024/001",
        "current_status": "VERIFIED",
        "status_history": [
            {
                "status": "RECEIVED",
                "timestamp": "2024-12-18T14:45:00+07:00",
                "user": "System",
                "note": "Delivery notification received"
            },
            {
                "status": "VERIFICATION_IN_PROGRESS",
                "timestamp": "2024-12-18T15:00:00+07:00",
                "user": "APT001",
                "note": "Verification started by Apt. Made Sari"
            },
            {
                "status": "VERIFIED",
                "timestamp": "2024-12-18T16:00:00+07:00",
                "user": "APT001",
                "note": "All items verified and accepted"
            }
        ],
        "verification_summary": {
            "total_items": 2,
            "items_accepted": 2,
            "items_rejected": 0,
            "total_amount": 362475.5,
            "amount_accepted": 362475.5,
            "amount_disputed": 0.0
        }
    }
}
```

### Status Codes

#### RO Status (status_kode)

-   `1` - RECEIVED (Diterima)
-   `2` - VERIFICATION_IN_PROGRESS (Sedang Verifikasi)
-   `3` - VERIFIED (Terverifikasi)
-   `4` - PARTIAL_RECEIVED (Sebagian Diterima)
-   `5` - DISPUTED (Ada Sengketa)
-   `6` - COMPLETED (Selesai)

#### Approval Status (status_proof_kode)

-   `PENDING` - Menunggu Approval
-   `APPROVED` - Disetujui
-   `REJECTED` - Ditolak

#### Item Condition

-   `GOOD` - Kondisi Baik
-   `DAMAGED` - Rusak
-   `EXPIRED` - Kadaluarsa
-   `PARTIAL_DAMAGED` - Sebagian Rusak
-   `QUANTITY_SHORTAGE` - Kurang Quantity

### Business Rules

1. **RO Number Format**: `RO/YY/MM/NNNN/LOCATION`

2. **Validation Rules**:

    - `nomor_ro` required, unique
    - `nomor_faktur` required, unique per distributor
    - `tanggal` required, cannot be future date
    - `po_id` required, must exist and be approved
    - `qty_ff` required, integer, min: 1
    - `no_batch` required for pharmaceutical items
    - `ed` required, must be future date
    - `harga_hna` required, decimal, min: 0

3. **Business Logic**:

    - RO hanya dapat dibuat berdasarkan PO yang sudah approved
    - Qty yang diterima tidak boleh melebihi qty yang di-approve di PO
    - Setiap item harus memiliki batch number dan expiry date
    - Invoice amount harus sesuai dengan perhitungan: (qty × harga) - discount + tax + biaya_lain
    - Stock otomatis terupdate setelah verification approved

4. **Stock Update Logic**:
    - Stock bertambah sesuai qty_accepted di `m_farmasi_buku_stock`
    - Update `m_farmasi_lokasi_barang` untuk stock per lokasi
    - Catat transaksi di `t_farmasi_stock_combined`

### Integration Points

1. **With Purchase Order System**:

    - Validate PO existence and status
    - Update PO fulfillment quantity
    - Link RO items to PO items

2. **With Stock Management**:

    - Update stock levels
    - Record batch information
    - Update minimum/maximum stock calculations

3. **With Finance System**:
    - Submit verified invoices for payment processing
    - Handle payment term calculations
    - Process disputed amounts

### Error Handling

**Common Error Codes:**

-   `VALIDATION_ERROR` - Input validation failed
-   `PO_NOT_FOUND` - Referenced PO not found
-   `PO_NOT_APPROVED` - PO not in approved status
-   `QUANTITY_EXCEEDED` - Received quantity exceeds PO quantity
-   `DUPLICATE_INVOICE` - Invoice number already exists
-   `EXPIRED_PRODUCT` - Product expiry date invalid
-   `SYSTEM_ERROR` - Internal server error

### File Handling

**Supported File Types:**

-   PDF (invoices, delivery notes)
-   JPG/PNG (delivery photos)
-   Excel (item details)

**File Size Limits:**

-   Individual file: 10MB
-   Total per transaction: 50MB

**File Storage:**

-   Files stored in `/storage/app/receiving_orders/{year}/{month}/`
-   Database stores file path and metadata
-   Automatic backup to cloud storage
