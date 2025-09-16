# Pemenuhan Retur API Specification

## Integrasi Sistem Farmasi Balimed dengan Distributor PT. PBF

### Database Structure Analysis

**Master Table**: `m_farmasi_receiving_retur_d`
**Detail Table**: `t_farmasi_receiving_retur_d`

### Master Receiving Retur Fields

```php
// Model: ReceivingReturD (m_farmasi_receiving_retur_d)
protected $fillable = [
    'tanggal',              // Date - Receiving date for return fulfillment
    'nomor',                // String - Receiving retur number (auto-generated)
    'nomor_faktur',         // String - Invoice number for replacement goods
    'distributor_id',       // Integer - Foreign key to m_farmasi_distributor
    'disc',                 // Decimal - Discount percentage for replacement
    'ppn',                  // Decimal - Tax percentage (PPN)
    'carabayar_id',         // Integer - Payment method (usually FREE for replacement)
    'tanggal_jt',           // Date - Due date (usually N/A for replacement)
    'tanggal_faktur',       // Date - Invoice date for replacement
    'biaya_lain',           // Decimal - Additional charges
    'kepada',               // String - Delivery recipient name
    'telp',                 // String - Recipient phone number
    'alamat',               // Text - Delivery address
    'note',                 // Text - Additional notes
    'file_pdf',             // String - Uploaded invoice/delivery note file path
    'created_by',           // String - User ID who created
    'updated_by',           // String - User ID who updated
    'subunit_owner_id',     // Integer - Hospital subunit owner
    'jdoc_kode',            // String - Document journal code
    'kode_doc',             // String - Document code
    'status_proof_kode',    // String - Approval status code
    'status_kode',          // String - Receiving status code
    'total_faktur',         // Decimal - Total replacement value
    'disc_rupiah',          // Decimal - Discount amount in rupiah
];
```

### Detail Receiving Retur Fields

```php
// Model: TReceivingReturD (t_farmasi_receiving_retur_d)
protected $fillable = [
    'receivingreturd_id',   // Integer - Foreign key to m_farmasi_receiving_retur_d
    'barang_id',            // Integer - Foreign key to m_farmasi_barang
    'satuan_id',            // Integer - Unit type ID
    'qty_ff',               // Integer - Quantity fulfilled/received
    'harga_hna',            // Decimal - Unit price (HNA = Harga Netto Apotek)
    'no_batch',             // String - Batch number of replacement items
    'ed',                   // Date - Expiry date of replacement items
    'returd_id',            // Integer - Reference to original return (m_farmasi_retur_d)
    'created_by',           // String - User ID
    'updated_by',           // String - User ID
];
```

### Fulfillment Types

```php
'REPLACEMENT'       => 'Penggantian Barang',
'EXCHANGE'          => 'Tukar dengan Barang Lain',
'UPGRADE'           => 'Upgrade ke Produk yang Lebih Baik',
'PARTIAL_REPLACEMENT' => 'Penggantian Sebagian',
'STORE_CREDIT'      => 'Store Credit (Future Use)',
'COMBINED'          => 'Kombinasi Multiple Actions'
```

### API Endpoints

#### 1. Receive Replacement Goods Notification from Distributor

**Endpoint**: `POST /api/balimed/return-fulfillments`
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
        "nomor_receiving_retur": "RRD/24/12/0001/DENPSR",
        "fulfillment_type": "REPLACEMENT",
        "tanggal": "2024-12-25",
        "tanggal_faktur": "2024-12-25",
        "nomor_faktur": "FK/REPLACEMENT/DIST001/2024/001",
        "distributor_id": 1,
        "distributor_code": "DIST001",
        "distributor_name": "PT. PBF Medika",
        "subunit_owner_id": 1,
        "subunit_code": "FARM_IGD",
        "return_reference": "RTN/24/12/0001/DENPSR",
        "distributor_return_id": "DRTN-2024-001",
        "credit_note_reference": "CN/DIST001/24/001-REV1",
        "carabayar_id": 5,
        "cara_bayar": "FREE_REPLACEMENT",
        "disc": 0.0,
        "ppn": 0.0,
        "biaya_lain": 0.0,
        "total_faktur": 11875.0,
        "disc_rupiah": 0.0,
        "kepada": "Apt. Made Sari",
        "telp": "0361-224466",
        "alamat": "Jl. Mahendradatta No.57, Denpasar",
        "note": "Penggantian barang untuk retur RTN/24/12/0001/DENPSR sesuai kesepakatan",
        "delivery_date": "2024-12-25T14:00:00+07:00",
        "driver_name": "Wayan Bagus",
        "driver_phone": "0812-3456-7890",
        "vehicle_number": "DK 1234 AB",
        "fulfillment_method": "DIRECT_DELIVERY",
        "estimated_arrival": "2024-12-25T14:30:00+07:00"
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
            "qty_ff": 5,
            "harga_hna": 2500.0,
            "no_batch": "PCM241201",
            "ed": "2027-12-01",
            "sub_total": 11875.0,
            "replacement_reason": "Penggantian untuk barang rusak",
            "original_return_line": 1,
            "original_batch": "PCM240801",
            "original_ed": "2026-08-01",
            "quality_status": "NEW_STOCK",
            "storage_requirement": "ROOM_TEMPERATURE",
            "keterangan": "Replacement dengan batch baru, ED lebih panjang"
        }
    ],
    "quality_assurance": {
        "qa_certificate": "QA/DIST001/2024/456",
        "qa_date": "2024-12-24",
        "qa_officer": "Dr. Kadek Suarta",
        "temperature_during_transport": "20-25°C",
        "packaging_condition": "SEALED_ORIGINAL",
        "storage_conditions_met": true,
        "special_handling": []
    },
    "logistics_info": {
        "pickup_date": "2024-12-25T10:00:00+07:00",
        "dispatch_time": "2024-12-25T11:00:00+07:00",
        "estimated_delivery": "2024-12-25T14:30:00+07:00",
        "transport_conditions": {
            "temperature_controlled": false,
            "cold_chain_required": false,
            "special_handling": []
        },
        "tracking_number": "TRK/DIST001/241225/001"
    },
    "attachments": [
        {
            "type": "REPLACEMENT_INVOICE",
            "filename": "replacement_invoice_001.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "description": "Invoice untuk barang penggantian"
        },
        {
            "type": "QA_CERTIFICATE",
            "filename": "qa_cert_PCM241201.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "description": "Quality assurance certificate"
        },
        {
            "type": "DELIVERY_NOTE",
            "filename": "delivery_note_replacement.pdf",
            "file_base64": "JVBERi0xLjQKJeLjz9MKMSAwIG9iago8PAovVHlwZSAvQ2F0YWxvZwo...",
            "description": "Surat jalan barang penggantian"
        }
    ]
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Replacement delivery notification berhasil diterima",
    "data": {
        "nomor_receiving_retur": "RRD/24/12/0001/DENPSR",
        "fulfillment_id": "RFUL-2024-001",
        "status": "DELIVERY_SCHEDULED",
        "scheduled_delivery": "2024-12-25T14:30:00+07:00",
        "assigned_receiver": "Apt. Made Sari",
        "receiving_location": "Gudang Farmasi IGD",
        "tracking_number": "TRK/DIST001/241225/001",
        "estimated_verification_time": "30 menit setelah barang tiba"
    }
}
```

#### 2. Send Replacement Goods Confirmation to Distributor

**Endpoint**: `POST /api/distributor/return-fulfillments/{fulfillment_id}/confirmation`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "fulfillment_id": "RFUL-2024-001",
    "nomor_receiving_retur": "RRD/24/12/0001/DENPSR",
    "tracking_number": "TRK/DIST001/241225/001",
    "receiving_date": "2024-12-25T14:45:00+07:00",
    "receiving_status": "FULLY_RECEIVED",
    "received_by": "APT001",
    "received_by_name": "Apt. Made Sari",
    "receiving_location": "Gudang Farmasi IGD",
    "note": "Semua replacement items diterima dengan kondisi baik",
    "received_items": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "no_batch": "PCM241201",
            "ed": "2027-12-01",
            "qty_delivered": 5,
            "qty_received": 5,
            "qty_accepted": 5,
            "qty_rejected": 0,
            "condition": "EXCELLENT",
            "packaging_condition": "INTACT",
            "expiry_check": "PASSED",
            "quality_check": "PASSED",
            "storage_location": "RAK-A-01",
            "acceptance_note": "Kondisi sangat baik, batch baru dengan ED panjang",
            "rejection_reason": null
        }
    ],
    "quality_verification": {
        "temperature_on_arrival": "22°C",
        "packaging_integrity": "100%",
        "visual_inspection": "PASSED",
        "documentation_complete": true,
        "qa_approval": "APPROVED",
        "verified_by": "APT001",
        "verification_time": "2024-12-25T15:00:00+07:00"
    },
    "stock_update": {
        "stock_added": 5,
        "new_stock_total": 95,
        "stock_location": "RAK-A-01",
        "batch_registered": true,
        "fifo_position": "NEWEST",
        "cost_update": "COMPLETED"
    },
    "return_closure": {
        "original_return": "RTN/24/12/0001/DENPSR",
        "return_line_completed": [1],
        "return_status": "FULFILLED",
        "closure_date": "2024-12-25T15:15:00+07:00",
        "final_notes": "Return case berhasil diselesaikan dengan replacement"
    }
}
```

**Success Response (200):**

```json
{
    "status": "success",
    "message": "Konfirmasi replacement berhasil diproses",
    "data": {
        "fulfillment_id": "RFUL-2024-001",
        "status": "COMPLETED",
        "completion_date": "2024-12-25T15:20:00+07:00",
        "return_case_status": "CLOSED",
        "customer_satisfaction": "FULFILLED",
        "next_actions": []
    }
}
```

#### 3. Handle Partial or Rejected Replacement

**Endpoint**: `POST /api/distributor/return-fulfillments/{fulfillment_id}/partial-rejection`
**Direction**: Balimed → Distributor

**Request Payload:**

```json
{
    "fulfillment_id": "RFUL-2024-001",
    "nomor_receiving_retur": "RRD/24/12/0001/DENPSR",
    "receiving_date": "2024-12-25T14:45:00+07:00",
    "receiving_status": "PARTIAL_REJECTED",
    "received_by": "APT001",
    "received_by_name": "Apt. Made Sari",
    "note": "Ada ketidaksesuaian pada replacement items",
    "received_items": [
        {
            "line_number": 1,
            "kode_barang": "BRG001",
            "no_batch": "PCM241201",
            "ed": "2027-12-01",
            "qty_delivered": 5,
            "qty_received": 5,
            "qty_accepted": 3,
            "qty_rejected": 2,
            "condition": "PARTIAL_DAMAGED",
            "packaging_condition": "SLIGHTLY_DAMAGED",
            "expiry_check": "PASSED",
            "quality_check": "FAILED",
            "rejection_reason": "2 strip kemasan sedikit rusak saat pengiriman",
            "acceptance_note": "3 strip diterima dengan kondisi baik",
            "storage_location": "RAK-A-01",
            "rejection_details": [
                {
                    "qty_rejected": 2,
                    "reason": "PACKAGING_DAMAGE",
                    "description": "Kemasan strip sedikit rusak di bagian seal",
                    "severity": "MINOR",
                    "action_requested": "REPLACEMENT"
                }
            ]
        }
    ],
    "discrepancy_report": {
        "total_items_expected": 5,
        "total_items_accepted": 3,
        "total_items_rejected": 2,
        "rejection_rate": 40.0,
        "impact_assessment": "MINOR",
        "resolution_request": {
            "action_needed": "SEND_REPLACEMENT_FOR_REJECTED",
            "quantity_needed": 2,
            "urgency": "NORMAL",
            "special_instructions": "Pastikan kemasan dalam kondisi sempurna"
        }
    },
    "photos_evidence": [
        {
            "type": "DAMAGED_PACKAGING",
            "filename": "damaged_replacement_001.jpg",
            "file_base64": "/9j/4AAQSkZJRgABAQEAeAB4AAD/2wBDAAEBAQEBAQEBAQEBAQEBAQEBAQEBAQ...",
            "description": "Foto kemasan yang rusak pada replacement items"
        }
    ]
}
```

#### 4. Track Fulfillment Status

**Endpoint**: `GET /api/balimed/return-fulfillments/{fulfillment_id}/status`
**Direction**: Balimed internal use

**Response:**

```json
{
    "status": "success",
    "data": {
        "fulfillment_id": "RFUL-2024-001",
        "current_status": "COMPLETED",
        "return_reference": "RTN/24/12/0001/DENPSR",
        "distributor_return_id": "DRTN-2024-001",
        "fulfillment_timeline": [
            {
                "status": "PLANNED",
                "timestamp": "2024-12-23T16:00:00+07:00",
                "note": "Replacement planned by distributor"
            },
            {
                "status": "PREPARED",
                "timestamp": "2024-12-24T10:00:00+07:00",
                "note": "Items prepared and QA checked"
            },
            {
                "status": "DISPATCHED",
                "timestamp": "2024-12-25T11:00:00+07:00",
                "note": "Items dispatched for delivery"
            },
            {
                "status": "IN_TRANSIT",
                "timestamp": "2024-12-25T12:00:00+07:00",
                "note": "In transit to hospital"
            },
            {
                "status": "DELIVERED",
                "timestamp": "2024-12-25T14:45:00+07:00",
                "note": "Delivered to hospital"
            },
            {
                "status": "RECEIVED",
                "timestamp": "2024-12-25T15:00:00+07:00",
                "note": "Received and verified by pharmacist"
            },
            {
                "status": "COMPLETED",
                "timestamp": "2024-12-25T15:20:00+07:00",
                "note": "Fulfillment completed successfully"
            }
        ],
        "fulfillment_summary": {
            "total_items_planned": 5,
            "total_items_delivered": 5,
            "total_items_accepted": 5,
            "fulfillment_rate": 100.0,
            "quality_score": 100.0,
            "delivery_time": "On Time",
            "customer_satisfaction": "Excellent"
        },
        "return_closure": {
            "original_return_closed": true,
            "all_items_fulfilled": true,
            "case_closure_date": "2024-12-25T15:20:00+07:00",
            "final_status": "SUCCESSFULLY_RESOLVED"
        }
    }
}
```

#### 5. Handle Exchange (Different Item) Fulfillment

**Endpoint**: `POST /api/balimed/return-fulfillments/exchange`
**Direction**: Distributor → Balimed

**Request Payload:**

```json
{
    "header": {
        "nomor_receiving_retur": "RRD/24/12/0002/DENPSR",
        "fulfillment_type": "EXCHANGE",
        "tanggal": "2024-12-26",
        "return_reference": "RTN/24/12/0002/DENPSR",
        "distributor_return_id": "DRTN-2024-002",
        "exchange_reason": "PRODUCT_UPGRADE",
        "note": "Tukar dengan produk yang lebih baik sesuai permintaan"
    },
    "exchange_details": [
        {
            "line_number": 1,
            "original_item": {
                "barang_id": 125,
                "kode_barang": "BRG003",
                "nama_barang": "Amoxicillin 250mg",
                "qty_returned": 20,
                "harga_hna": 2000.0,
                "total_value": 40000.0
            },
            "exchange_item": {
                "barang_id": 126,
                "kode_barang": "BRG004",
                "nama_barang": "Amoxicillin 500mg",
                "qty_given": 10,
                "harga_hna": 3500.0,
                "total_value": 35000.0,
                "no_batch": "AMX500-241201",
                "ed": "2027-12-01"
            },
            "value_adjustment": {
                "original_value": 40000.0,
                "exchange_value": 35000.0,
                "difference": -5000.0,
                "adjustment_type": "CREDIT_BALANCE",
                "credit_note_needed": true
            }
        }
    ]
}
```

### Status Codes

#### Fulfillment Status

-   `PLANNED` - Direncanakan
-   `PREPARED` - Disiapkan
-   `DISPATCHED` - Dikirim
-   `IN_TRANSIT` - Dalam Perjalanan
-   `DELIVERED` - Sampai di Tujuan
-   `RECEIVED` - Diterima
-   `VERIFIED` - Diverifikasi
-   `COMPLETED` - Selesai
-   `PARTIAL_FULFILLED` - Sebagian Terpenuhi
-   `REJECTED` - Ditolak
-   `CANCELLED` - Dibatalkan

#### Quality Status

-   `EXCELLENT` - Sangat Baik
-   `GOOD` - Baik
-   `ACCEPTABLE` - Dapat Diterima
-   `MINOR_ISSUES` - Ada Masalah Kecil
-   `MAJOR_ISSUES` - Ada Masalah Besar
-   `REJECTED` - Ditolak

### Business Rules

1. **Receiving Retur Number Format**: `RRD/YY/MM/NNNN/LOCATION`

2. **Validation Rules**:

    - `nomor` required, unique
    - `return_reference` required, must exist and be approved
    - `fulfillment_type` required, must be valid type
    - `qty_ff` required, integer, min: 1
    - `no_batch` required for pharmaceutical items
    - `ed` required, must be future date and longer than original

3. **Business Logic**:

    - Fulfillment hanya dapat dibuat untuk approved returns
    - Replacement items harus memiliki spesifikasi yang sama atau lebih baik
    - Batch baru harus memiliki expiry date yang lebih panjang
    - Stock otomatis bertambah setelah confirmation
    - Original return case otomatis closed setelah fulfillment completed

4. **Quality Requirements**:
    - Replacement items harus dalam kondisi perfect
    - Batch tracking mandatory untuk pharmaceutical items
    - Temperature-controlled items need special handling
    - QA certificate required for certain drug categories

### Integration Points

1. **With Return System**:

    - Link to original return request
    - Update return status to fulfilled
    - Close return case upon completion

2. **With Stock Management**:

    - Add replacement stock to inventory
    - Update batch information
    - Maintain FIFO/FEFO order
    - Update cost calculations

3. **With Quality Assurance**:
    - Verify QA certificates
    - Temperature monitoring for cold chain
    - Expiry date validation
    - Packaging integrity checks

### Fulfillment Workflow

1. **Distributor Plans Fulfillment**:

    - Review approved return
    - Plan replacement or exchange
    - Prepare items with QA check
    - Schedule delivery

2. **Delivery Notification**:

    - Send delivery notification via API
    - Include tracking and timing
    - Provide QA documentation

3. **Hospital Receives**:

    - Physical receipt of items
    - Quality verification
    - Stock registration
    - Confirmation to distributor

4. **Case Closure**:
    - Update return status
    - Close fulfillment case
    - Update customer satisfaction metrics

### Error Handling

**Common Error Codes:**

-   `VALIDATION_ERROR` - Input validation failed
-   `RETURN_NOT_ELIGIBLE` - Return not eligible for fulfillment
-   `QUALITY_FAILED` - Quality check failed
-   `BATCH_INVALID` - Batch information invalid
-   `DELIVERY_FAILED` - Delivery could not be completed
-   `PARTIAL_FULFILLMENT` - Only partial fulfillment possible
-   `SYSTEM_ERROR` - Internal server error

### Performance Metrics

**KPIs to Track**:

-   Fulfillment completion rate
-   Average fulfillment time
-   Quality acceptance rate
-   Customer satisfaction score
-   Cost per fulfillment
-   Repeat return rate

### Special Handling Cases

1. **Cold Chain Products**:

    - Temperature monitoring required
    - Special packaging
    - Faster delivery timeline
    - Additional QA requirements

2. **Controlled Substances**:

    - Additional documentation
    - Authorized personnel only
    - Special tracking requirements
    - Regulatory compliance

3. **High-Value Items**:
    - Insurance requirements
    - Signature confirmation
    - Photo documentation
    - Enhanced security measures
