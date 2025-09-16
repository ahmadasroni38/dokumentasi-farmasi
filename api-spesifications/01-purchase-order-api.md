# Purchase Order API Specification

## Integrasi Sistem Farmasi Balimed dengan Distributor PT. PBF

### Database Structure Analysis

**Master Table**: `m_farmasi_po`
**Detail Table**: `t_farmasi_po`

### Master Purchase Order Fields

```php
// Model: PurchaseOrder (m_farmasi_po)
protected $fillable = [
    'tanggal',              // Date - PO creation date
    'tanggal_expired',      // Date - PO expiration date
    'nomor',                // String - PO number (auto-generated)
    'distributor_id',       // Integer - Foreign key to m_farmasi_distributor
    'disc',                 // Decimal - Discount percentage
    'ppn',                  // Decimal - Tax percentage (PPN)
    'status_id',            // Integer - PO status (draft, sent, approved, etc)
    'created_by',           // String - User ID who created
    'updated_by',           // String - User ID who updated
    'subunit_owner_id',     // Integer - Hospital subunit owner
    'jdoc_kode',            // String - Document journal code
    'kode_doc',             // String - Document code
    'status_proof_kode',    // String - Approval status code
    'tempo_hari',           // Integer - Payment terms in days
    'note'                  // Text - Additional notes
];
```

### Detail Purchase Order Fields

```php
// Model: TPurchaseOrder (t_farmasi_po)
protected $fillable = [
    'po_id',            // Integer - Foreign key to m_farmasi_po
    'barang_id',        // Integer - Foreign key to m_farmasi_barang
    'qty_rq',           // Integer - Quantity requested
    'qty_ff',           // Integer - Quantity fulfilled
    'stok_saat_ini',    // Integer - Current stock
    'stokmin',          // Integer - Minimum stock level
    'stokmax',          // Integer - Maximum stock level
    'created_by',       // String - User ID
    'updated_by',       // String - User ID
    'satuan_kode',      // String - Unit code (strip, box, bottle, etc)
    'harga_hna',        // Decimal - Unit price (HNA = Harga Netto Apotek)
    'disc',             // Decimal - Item discount percentage
    'keterangan',       // Text - Item notes
    'sub_total',        // Decimal - Line total amount
];
```

### API Endpoints

#### 1. Send Purchase Order to Distributor

**Endpoint**: `POST /api/distributor/purchase-orders`
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
        "nomor_po": "PO/24/12/0001/DENPSR",
        "tanggal": "2024-12-15",
        "tanggal_expired": "2024-12-30",
        "distributor_id": 1,
        "distributor_kode": "DIST001",
        "subunit_owner_id": 1,
        "subunit_code": "FARM_IGD",
        "subunit_name": "Farmasi IGD",
        "tempo_hari": 30,
        "disc": 5.5,
        "ppn": 11.0,
        "note": "Urgent order untuk stok IGD",
        "created_by": "USR001",
        "created_by_name": "Apt. Made Sari",
        "created_date": "2024-12-15T08:30:00+07:00"
    },
    "details": [
        {
            "line_number": 1,
            "barang_id": 123,
            "kode_barang": "BRG001",
            "nama_barang": "Paracetamol 500mg",
            "nama_generik": "Parasetamol",
            "satuan_kode": "STRIP",
            "qty_rq": 100,
            "stok_saat_ini": 25,
            "stokmin": 50,
            "stokmax": 500,
            "harga_hna": 2500.0,
            "disc": 5.0,
            "sub_total": 237500.0,
            "keterangan": "Prioritas tinggi"
        },
        {
            "line_number": 2,
            "barang_id": 124,
            "kode_barang": "BRG002",
            "nama_barang": "Amoxicillin 500mg",
            "nama_generik": "Amoksisilin",
            "satuan_kode": "STRIP",
            "qty_rq": 50,
            "stok_saat_ini": 10,
            "stokmin": 25,
            "stokmax": 200,
            "harga_hna": 3500.0,
            "disc": 3.0,
            "sub_total": 169750.0,
            "keterangan": null
        }
    ],
    "summary": {
        "total_items": 2,
        "total_qty": 150,
        "subtotal": 407250.0,
        "discount_amount": 20362.5,
        "tax_amount": 42556.25,
        "total_amount": 429443.75
    }
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Purchase order berhasil diterima",
    "data": {
        "distributor_po_id": "DPO-2024-001",
        "balimed_po_number": "PO/24/12/0001/DENPSR",
        "status": "RECEIVED",
        "received_date": "2024-12-15T09:15:00+07:00",
        "estimated_process_time": "2-3 hari kerja",
        "contact_person": "Budi Santoso",
        "contact_phone": "0361-123456"
    }
}
```

**Error Response (400):**

```json
{
    "status": "error",
    "error_code": "VALIDATION_ERROR",
    "message": "Data PO tidak valid",
    "errors": [
        {
            "field": "header.distributor_id",
            "message": "Distributor ID tidak ditemukan"
        },
        {
            "field": "details[0].kode_barang",
            "message": "Kode barang BRG001 tidak terdaftar"
        }
    ],
    "timestamp": "2024-12-15T09:15:00+07:00"
}
```

#### 2. Receive PO Status Update from Distributor

**Endpoint**: `PATCH /api/balimed/purchase-orders/{nomor_po}/status`
**Direction**: Distributor → Balimed

**Request Payload:**

```json
{
    "distributor_po_id": "DPO-2024-001",
    "balimed_po_number": "PO/24/12/0001/DENPSR",
    "status": "APPROVED",
    "status_keterangan": "PO disetujui dengan modifikasi qty beberapa item",
    "process_date": "2024-12-16T10:30:00+07:00",
    "estimated_delivery": "2024-12-18T14:00:00+07:00",
    "contact_person": "Budi Santoso",
    "contact_phone": "0361-123456",
    "items_status": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "status": "APPROVED",
            "qty_approved": 100,
            "qty_available": 100,
            "harga_confirmed": 2500.0,
            "estimated_delivery": "2024-12-18T14:00:00+07:00",
            "keterangan": "Stok tersedia"
        },
        {
            "line_number": 2,
            "kode_barang": "BRG002",
            "status": "PARTIAL",
            "qty_approved": 30,
            "qty_available": 30,
            "harga_confirmed": 3500.0,
            "estimated_delivery": "2024-12-18T14:00:00+07:00",
            "keterangan": "Stok terbatas, sisa 30 strip"
        }
    ]
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Status PO berhasil diupdate",
    "data": {
        "nomor_po": "PO/24/12/0001/DENPSR",
        "status_updated": "APPROVED",
        "updated_at": "2024-12-16T10:35:00+07:00"
    }
}
```

#### 3. Get PO Details for Distributor

**Endpoint**: `GET /api/distributor/purchase-orders/{nomor_po}`
**Direction**: Distributor → Balimed

**Response:**

```json
{
    "status": "success",
    "data": {
        "header": {
            "id": 1,
            "nomor_po": "PO/24/12/0001/DENPSR",
            "tanggal": "2024-12-15",
            "tanggal_expired": "2024-12-30",
            "distributor_id": 1,
            "subunit_owner_id": 1,
            "tempo_hari": 30,
            "disc": 5.5,
            "ppn": 11.0,
            "status_id": 2,
            "status_name": "Terkirim ke Distributor",
            "note": "Urgent order untuk stok IGD"
        },
        "details": [
            {
                "id": 1,
                "line_number": 1,
                "barang_id": 123,
                "kode_barang": "BRG001",
                "nama_barang": "Paracetamol 500mg",
                "nama_generik": "Parasetamol",
                "satuan_kode": "STRIP",
                "qty_rq": 100,
                "qty_ff": 0,
                "harga_hna": 2500.0,
                "disc": 5.0,
                "sub_total": 237500.0
            }
        ]
    }
}
```

### Status Codes

#### PO Status (status_id)

-   `1` - DRAFT (Draft)
-   `2` - SENT (Terkirim ke Distributor)
-   `3` - APPROVED (Disetujui)
-   `4` - PARTIAL (Sebagian Disetujui)
-   `5` - REJECTED (Ditolak)
-   `6` - CANCELLED (Dibatalkan)

#### Approval Status (status_proof_kode)

-   `PENDING` - Menunggu Approval
-   `APPROVED` - Disetujui
-   `REJECTED` - Ditolak

### Business Rules

1. **PO Number Format**: `PO/YY/MM/NNNN/LOCATION`

    - YY: 2-digit year
    - MM: 2-digit month
    - NNNN: 4-digit sequential number
    - LOCATION: Hospital location code

2. **Validation Rules**:

    - `nomor` required, unique
    - `tanggal` required, cannot be future date
    - `tanggal_expired` required, must be after tanggal
    - `distributor_id` required, must exist
    - `tempo_hari` required, integer, min: 1, max: 90
    - `qty_rq` required, integer, min: 1
    - `harga_hna` required, decimal, min: 0

3. **Business Logic**:
    - PO dapat dibuat hanya untuk distributor yang aktif
    - Setiap item PO harus memiliki stok minimum yang belum terpenuhi
    - Total amount dihitung otomatis: (qty × harga) - discount + tax
    - Status workflow: DRAFT → SENT → APPROVED/REJECTED → (if approved) DELIVERED

### Error Handling

**Common Error Codes:**

-   `VALIDATION_ERROR` - Input validation failed
-   `NOT_FOUND` - PO not found
-   `UNAUTHORIZED` - Invalid credentials
-   `FORBIDDEN` - Access denied
-   `BUSINESS_RULE_VIOLATION` - Business logic violation
-   `SYSTEM_ERROR` - Internal server error

### Authentication & Security

-   **Authentication**: Bearer token
-   **Rate Limiting**: 100 requests per minute per API key
-   **HTTPS**: Required for all communications
-   **Request Signing**: HMAC-SHA256 for critical operations
-   **IP Whitelisting**: Configured for both systems

### Monitoring

**Required Logs:**

-   All API requests/responses with timestamps
-   Authentication attempts
-   Business rule violations
-   System errors with stack traces

**Health Check Endpoint:**

```
GET /api/health
Response: {"status": "healthy", "timestamp": "2024-12-15T10:30:00+07:00"}
```
