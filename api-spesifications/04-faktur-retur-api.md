# Faktur Retur API Specification

## Integrasi Sistem Farmasi Balimed dengan Distributor PT. PBF

### Database Structure Analysis

**Master Table**: `m_farmasi_retur_d_faktur`
**Detail Table**: `t_farmasi_faktur_retur_d`

### Master Faktur Retur Fields

```php
// Model: FakturReturD (m_farmasi_retur_d_faktur)
protected $fillable = [
    'tanggal',              // Date - Credit note date
    'nomor',                // String - Credit note number (auto-generated)
    'subunit_id',           // Integer - Hospital subunit ID
    'distributor_id',       // Integer - Foreign key to m_farmasi_distributor
    'note',                 // Text - Credit note notes
    'diskon',               // Decimal - Credit note discount percentage
    'ppn',                  // Decimal - Tax percentage (PPN)
    'status_proof_kode',    // String - Approval status code
    'created_by',           // String - User ID who created
    'updated_by',           // String - User ID who updated
    'subunit_owner_id',     // Integer - Hospital subunit owner
    'jdoc_kode',            // String - Document journal code
    'kode_doc',             // String - Document code
];
```

### Detail Faktur Retur Fields

```php
// Model: TFakturReturD (t_farmasi_faktur_retur_d)
protected $fillable = [
    'faktur_retur_id',  // Integer - Foreign key to m_farmasi_retur_d_faktur
    'barang_id',        // Integer - Foreign key to m_farmasi_barang
    'satuan_kode',      // String - Unit code (strip, box, bottle, etc)
    'qty',              // Integer - Quantity credited
    'harga_hna',        // Decimal - Unit price (HNA = Harga Netto Apotek)
    'diskon',           // Decimal - Item discount percentage
    'batchno',          // String - Batch number
    'created_by',       // String - User ID
    'updated_by',       // String - User ID
];
```

### Credit Note Types

```php
'RETURN_CREDIT'     => 'Credit Note untuk Retur',
'PRICE_ADJUSTMENT'  => 'Credit Note untuk Penyesuaian Harga',
'QUANTITY_ADJUSTMENT' => 'Credit Note untuk Penyesuaian Quantity',
'PROMOTIONAL_CREDIT' => 'Credit Note untuk Program Promosi',
'GOODWILL_CREDIT'   => 'Credit Note Goodwill',
'OTHER'             => 'Credit Note Lainnya'
```

### API Endpoints

#### 1. Receive Credit Note from Distributor

**Endpoint**: `POST /api/balimed/credit-notes`
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
        "nomor_credit_note": "CN/DIST001/24/001",
        "tanggal": "2024-12-22",
        "credit_note_type": "RETURN_CREDIT",
        "subunit_id": 1,
        "subunit_code": "FARM_IGD",
        "distributor_id": 1,
        "distributor_code": "DIST001",
        "distributor_name": "PT. PBF Medika",
        "return_reference": "RTN/24/12/0001/DENPSR",
        "distributor_return_id": "DRTN-2024-001",
        "original_invoice": "FK/DIST001/2024/001",
        "diskon": 5.5,
        "ppn": 11.0,
        "note": "Credit note untuk retur barang rusak sesuai RTN/24/12/0001/DENPSR",
        "issued_by": "Budi Santoso",
        "issued_by_position": "Finance Manager",
        "issued_date": "2024-12-22T10:00:00+07:00",
        "effective_date": "2024-12-22",
        "payment_impact": {
            "apply_to_current_invoice": false,
            "apply_to_next_payment": true,
            "create_refund": false
        }
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
            "qty": 5,
            "harga_hna": 2500.0,
            "diskon": 5.0,
            "sub_total": 11875.0,
            "credit_reason": "Barang rusak/cacat",
            "return_line_reference": 1,
            "keterangan": "Credit untuk 5 strip paracetamol yang rusak"
        },
        {
            "line_number": 2,
            "barang_id": 124,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "nama_generik": "Amoksisilin",
            "satuan_kode": "STRIP",
            "batchno": "AMX240701",
            "qty": 10,
            "harga_hna": 3500.0,
            "diskon": 3.0,
            "sub_total": 33950.0,
            "credit_reason": "Mendekati expired date",
            "return_line_reference": 2,
            "keterangan": "Credit untuk 10 strip amoxicillin mendekati ED"
        }
    ],
    "summary": {
        "total_items": 2,
        "total_qty": 15,
        "subtotal": 45825.0,
        "discount_amount": 2291.25,
        "taxable_amount": 43533.75,
        "tax_amount": 4789.71,
        "credit_amount": 48323.46
    },
    "accounting_info": {
        "gl_account": "4210001",
        "gl_description": "Sales Return",
        "cost_center": "CC_PHARMA",
        "posting_date": "2024-12-22",
        "fiscal_period": "2024-12",
        "reference_document": "RTN/24/12/0001/DENPSR"
    },
    "attachments": [
        {
            "type": "CREDIT_NOTE_DOCUMENT",
            "filename": "credit_note_CN_DIST001_24_001.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "description": "Original credit note document"
        }
    ]
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Credit note berhasil diterima dan diproses",
    "data": {
        "nomor_faktur_retur": "FR/24/12/0001/DENPSR",
        "credit_note_number": "CN/DIST001/24/001",
        "status": "RECEIVED",
        "received_date": "2024-12-22T10:15:00+07:00",
        "processing_status": "PENDING_VERIFICATION",
        "assigned_finance_staff": "Ni Made Dewi",
        "estimated_verification_time": "1-2 jam",
        "credit_amount": 48323.46,
        "next_actions": ["Verifikasi oleh tim finance", "Posting ke accounting system", "Adjustment pada payment terms"]
    }
}
```

#### 2. Send Credit Note Verification to Distributor

**Endpoint**: `POST /api/distributor/credit-notes/{credit_note_number}/verification`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "credit_note_number": "CN/DIST001/24/001",
    "nomor_faktur_retur": "FR/24/12/0001/DENPSR",
    "verification_date": "2024-12-22T14:00:00+07:00",
    "verification_status": "VERIFIED",
    "verified_by": "FIN001",
    "verified_by_name": "Ni Made Dewi",
    "verified_by_position": "Finance Staff",
    "verification_note": "Credit note telah diverifikasi dan sesuai dengan return request",
    "accounting_status": "POSTED",
    "posting_date": "2024-12-22T14:30:00+07:00",
    "posting_reference": "JV/2024/12/0145",
    "items_verification": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "batchno": "PCM240801",
            "qty_verified": 5,
            "amount_verified": 11875.0,
            "verification_result": "APPROVED",
            "keterangan": "Sesuai dengan return request dan stock records"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "batchno": "AMX240701",
            "qty_verified": 10,
            "amount_verified": 33950.0,
            "verification_result": "APPROVED",
            "keterangan": "Sesuai dengan return request dan stock records"
        }
    ],
    "financial_impact": {
        "accounts_payable_adjustment": -48323.46,
        "cost_of_goods_adjustment": -45825.0,
        "tax_adjustment": -4789.71,
        "net_impact": -48323.46
    },
    "payment_adjustment": {
        "apply_to_invoice": "FK/DIST001/2024/002",
        "new_payment_amount": 1234567.89,
        "adjustment_amount": -48323.46,
        "new_due_date": "2025-01-15"
    }
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Verifikasi credit note berhasil diterima",
    "data": {
        "credit_note_number": "CN/DIST001/24/001",
        "verification_status": "ACCEPTED",
        "processed_date": "2024-12-22T14:35:00+07:00",
        "payment_adjustment_applied": true,
        "next_invoice_impact": -48323.46
    }
}
```

#### 3. Handle Credit Note Disputes

**Endpoint**: `POST /api/distributor/credit-notes/{credit_note_number}/dispute`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "credit_note_number": "CN/DIST001/24/001",
    "nomor_faktur_retur": "FR/24/12/0001/DENPSR",
    "dispute_date": "2024-12-22T14:00:00+07:00",
    "disputed_by": "FIN001",
    "disputed_by_name": "Ni Made Dewi",
    "dispute_reason": "AMOUNT_MISMATCH",
    "dispute_details": "Jumlah credit tidak sesuai dengan return yang disetujui",
    "disputed_items": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "batchno": "PCM240801",
            "credit_note_qty": 5,
            "expected_qty": 5,
            "credit_note_amount": 11875.0,
            "expected_amount": 11875.0,
            "dispute_type": "NO_DISPUTE",
            "keterangan": "Sesuai"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "batchno": "AMX240701",
            "credit_note_qty": 10,
            "expected_qty": 8,
            "credit_note_amount": 33950.0,
            "expected_amount": 27160.0,
            "dispute_type": "QUANTITY_EXCESS",
            "keterangan": "Credit note qty lebih besar dari return yang disetujui"
        }
    ],
    "resolution_request": {
        "requested_action": "QUANTITY_ADJUSTMENT",
        "requested_credit_amount": 38035.0,
        "adjustment_amount": -10288.0,
        "supporting_documents": ["Return approval document", "Stock movement records"]
    },
    "contact_info": {
        "contact_person": "Ni Made Dewi",
        "contact_phone": "0361-224466",
        "contact_email": "finance@balimed.com",
        "preferred_contact_method": "email"
    }
}
```

#### 4. Receive Revised Credit Note

**Endpoint**: `PUT /api/balimed/credit-notes/{credit_note_number}`
**Direction**: Distributor → Balimed

**Request Payload:**

```json
{
    "credit_note_number": "CN/DIST001/24/001-REV1",
    "original_credit_note": "CN/DIST001/24/001",
    "revision_reason": "Dispute resolution - quantity adjustment",
    "revision_date": "2024-12-23T09:00:00+07:00",
    "header": {
        "tanggal": "2024-12-23",
        "credit_note_type": "RETURN_CREDIT",
        "subunit_id": 1,
        "distributor_id": 1,
        "return_reference": "RTN/24/12/0001/DENPSR",
        "note": "Credit note revisi untuk penyesuaian quantity sesuai dispute",
        "revised_by": "Budi Santoso",
        "revision_approved_by": "Manager Finance"
    },
    "details": [
        {
            "line_number": 1,
            "barang_id": 123,
            "kode_barang": "BRG001",
            "satuan_kode": "STRIP",
            "batchno": "PCM240801",
            "qty": 5,
            "harga_hna": 2500.0,
            "diskon": 5.0,
            "sub_total": 11875.0,
            "revision_note": "Tidak ada perubahan"
        },
        {
            "line_number": 2,
            "barang_id": 124,
            "kode_barang": "BRG002",
            "satuan_kode": "STRIP",
            "batchno": "AMX240701",
            "qty": 8,
            "harga_hna": 3500.0,
            "diskon": 3.0,
            "sub_total": 27160.0,
            "revision_note": "Quantity disesuaikan dari 10 menjadi 8 sesuai dispute"
        }
    ],
    "summary": {
        "total_items": 2,
        "total_qty": 13,
        "original_credit_amount": 48323.46,
        "revised_credit_amount": 38035.46,
        "adjustment_amount": -10288.0,
        "subtotal": 39035.0,
        "discount_amount": 1951.75,
        "taxable_amount": 37083.25,
        "tax_amount": 4079.16,
        "final_credit_amount": 38035.46
    }
}
```

#### 5. Get Credit Note History

**Endpoint**: `GET /api/balimed/credit-notes/{credit_note_number}/history`
**Direction**: Balimed internal use

**Response:**

```json
{
    "status": "success",
    "data": {
        "credit_note_number": "CN/DIST001/24/001",
        "current_version": "CN/DIST001/24/001-REV1",
        "status": "VERIFIED",
        "history": [
            {
                "version": "CN/DIST001/24/001",
                "action": "CREATED",
                "timestamp": "2024-12-22T10:00:00+07:00",
                "user": "Distributor System",
                "amount": 48323.46,
                "note": "Initial credit note issued"
            },
            {
                "version": "CN/DIST001/24/001",
                "action": "RECEIVED",
                "timestamp": "2024-12-22T10:15:00+07:00",
                "user": "System",
                "amount": 48323.46,
                "note": "Credit note received via API"
            },
            {
                "version": "CN/DIST001/24/001",
                "action": "DISPUTED",
                "timestamp": "2024-12-22T14:00:00+07:00",
                "user": "FIN001",
                "amount": 48323.46,
                "note": "Disputed due to quantity mismatch"
            },
            {
                "version": "CN/DIST001/24/001-REV1",
                "action": "REVISED",
                "timestamp": "2024-12-23T09:00:00+07:00",
                "user": "Distributor System",
                "amount": 38035.46,
                "note": "Revised to resolve dispute"
            },
            {
                "version": "CN/DIST001/24/001-REV1",
                "action": "VERIFIED",
                "timestamp": "2024-12-23T11:00:00+07:00",
                "user": "FIN001",
                "amount": 38035.46,
                "note": "Verified and approved for processing"
            }
        ],
        "financial_impact": {
            "final_credit_amount": 38035.46,
            "accounts_payable_adjustment": -38035.46,
            "applied_to_invoices": [
                {
                    "invoice_number": "FK/DIST001/2024/002",
                    "applied_amount": -38035.46,
                    "new_balance": 962536.54
                }
            ]
        }
    }
}
```

### Status Codes

#### Credit Note Status

-   `DRAFT` - Draft (belum dikirim)
-   `ISSUED` - Diterbitkan oleh distributor
-   `RECEIVED` - Diterima oleh sistem Balimed
-   `PENDING_VERIFICATION` - Menunggu verifikasi finance
-   `VERIFIED` - Terverifikasi dan disetujui
-   `DISPUTED` - Ada dispute/sengketa
-   `REVISED` - Sudah direvisi
-   `POSTED` - Sudah diposting ke accounting
-   `CANCELLED` - Dibatalkan

#### Approval Status (status_proof_kode)

-   `PENDING` - Menunggu Approval
-   `APPROVED` - Disetujui
-   `REJECTED` - Ditolak

### Business Rules

1. **Credit Note Number Format**: `CN/DIST_CODE/YY/NNN[-REVN]`

2. **Validation Rules**:

    - `nomor` required, unique per distributor
    - `tanggal` required, cannot be future date
    - `distributor_id` required, must exist and be active
    - `return_reference` required for return-based credit notes
    - `qty` required, integer, min: 1
    - `harga_hna` required, decimal, min: 0
    - Credit note amount must not exceed original invoice amount

3. **Business Logic**:

    - Credit note hanya dapat dibuat berdasarkan approved return
    - Amount dihitung otomatis: (qty × harga) - discount + tax
    - Credit note otomatis mengurangi accounts payable
    - Stock tidak terpengaruh (sudah dikurangi saat return)
    - Credit dapat diaplikasikan ke invoice berikutnya atau refund

4. **Financial Impact**:
    - Mengurangi accounts payable ke distributor
    - Mengurangi cost of goods sold
    - Adjustment tax calculation
    - Impact pada cash flow projection

### Integration Points

1. **With Return System**:

    - Validate return was approved
    - Link credit note to specific return items
    - Ensure credit amount matches approved return value

2. **With Accounting System**:

    - Auto-posting journal entries
    - Accounts payable adjustment
    - Tax recalculation
    - Cost center allocation

3. **With Payment System**:
    - Apply credit to future payments
    - Adjust payment terms and due dates
    - Handle refund processing if needed

### Credit Note Workflow

1. **Distributor Issues Credit Note**:

    - Based on approved return request
    - Calculate credit amount with proper tax
    - Send to Balimed via API

2. **Balimed Receives Credit Note**:

    - Validate against return records
    - Check amount calculations
    - Route to finance for verification

3. **Finance Verification**:

    - Compare with return approval
    - Verify amount calculations
    - Check accounting implications
    - Approve or dispute

4. **Dispute Resolution** (if needed):

    - Hospital raises dispute with details
    - Distributor reviews and responds
    - Revised credit note issued if agreed

5. **Financial Processing**:
    - Post journal entries
    - Adjust accounts payable
    - Apply to future payments or process refund

### Error Handling

**Common Error Codes:**

-   `VALIDATION_ERROR` - Input validation failed
-   `RETURN_NOT_FOUND` - Referenced return not found
-   `RETURN_NOT_APPROVED` - Return not approved for credit
-   `AMOUNT_MISMATCH` - Credit amount doesn't match return
-   `DUPLICATE_CREDIT_NOTE` - Credit note number already exists
-   `INVALID_TAX_CALCULATION` - Tax calculation incorrect
-   `ACCOUNTING_ERROR` - Posting to accounting failed
-   `SYSTEM_ERROR` - Internal server error

### Audit Requirements

All credit note transactions require complete audit trail:

-   Original return request reference
-   Approval chain for return
-   Credit note creation and approval
-   Any disputes and resolutions
-   Financial posting records
-   Payment application records

### Compliance & Regulations

1. **Tax Compliance**:

    - Proper PPN calculation and reporting
    - Credit note numbering sequence
    - Support for tax audit requirements

2. **Financial Reporting**:

    - Impact on revenue recognition
    - Accounts payable accuracy
    - Cash flow reporting

3. **Pharmaceutical Regulations**:
    - Traceability of returned items
    - Proper documentation for BPOM audit
    - Cost calculation for pricing compliance
