# Stock Tracking API Specification

## Requirements untuk Programmer PT. PBF

### Yang Harus Disiapkan oleh PBF

API ini menjelaskan endpoint yang **WAJIB disiapkan oleh programmer PT. PBF** untuk menyediakan informasi stok terbaru kepada Sistem Farmasi Balimed dalam real-time.

---

## API Endpoints yang WAJIB Dibuat PBF

### 1. Get Current Stock Levels

**Endpoint yang WAJIB Dibuat PBF**: `GET /api/distributor/stock`
**Direction**: Balimed → **PBF (menyediakan data)**
**Content-Type**: `application/json`

**Headers yang WAJIB Diterima dan Divalidasi PBF:**

```http
Authorization: Bearer {api_token}        # WAJIB validasi token
Content-Type: application/json
X-Hospital-Code: BALIMED_DENPASAR       # WAJIB untuk identifikasi rumah sakit
X-Request-ID: {unique_request_id}       # WAJIB untuk idempotency
```

**Query Parameters:**

```http
GET /api/distributor/stock?product_codes=BRG001,BRG002&location=DENPASAR&include_batch=true
```

**Parameters yang Harus Didukung:**
- `product_codes` (optional) - Comma-separated list of product codes
- `location` (optional) - Distribution location code
- `include_batch` (optional) - Include batch details (default: false)
- `updated_since` (optional) - ISO timestamp untuk incremental sync

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "timestamp": "2024-12-15T10:30:00+07:00",
    "location": "DENPASAR",
    "data": [
        {
            "product_code": "BRG001",
            "product_name": "Paracetamol 500mg",
            "category": "ANALGETIK",
            "manufacturer": "KIMIA FARMA",
            "unit_code": "TAB",
            "unit_name": "Tablet",
            "available_stock": 5000,
            "reserved_stock": 200,
            "minimum_stock": 100,
            "last_updated": "2024-12-15T10:25:00+07:00",
            "price_per_unit": 450.00,
            "currency": "IDR",
            "batches": [
                {
                    "batch_number": "BTH240101",
                    "expiry_date": "2026-01-01",
                    "quantity": 3000,
                    "manufacturing_date": "2024-01-01"
                },
                {
                    "batch_number": "BTH240201",
                    "expiry_date": "2026-02-01",
                    "quantity": 2000,
                    "manufacturing_date": "2024-02-01"
                }
            ]
        }
    ],
    "pagination": {
        "total_items": 1,
        "page": 1,
        "per_page": 100,
        "total_pages": 1
    }
}
```

### 2. Get Specific Product Stock

**Endpoint yang WAJIB Dibuat PBF**: `GET /api/distributor/stock/{product_code}`
**Direction**: Balimed → **PBF (menyediakan data)**

**Path Parameters:**
- `product_code` - Kode produk yang ingin dicek

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "timestamp": "2024-12-15T10:30:00+07:00",
    "data": {
        "product_code": "BRG001",
        "product_name": "Paracetamol 500mg",
        "category": "ANALGETIK",
        "manufacturer": "KIMIA FARMA",
        "unit_code": "TAB",
        "unit_name": "Tablet",
        "available_stock": 5000,
        "reserved_stock": 200,
        "minimum_stock": 100,
        "last_updated": "2024-12-15T10:25:00+07:00",
        "price_per_unit": 450.00,
        "currency": "IDR",
        "stock_history": [
            {
                "date": "2024-12-15",
                "opening_stock": 4800,
                "received": 500,
                "sold": 300,
                "closing_stock": 5000
            }
        ],
        "batches": [
            {
                "batch_number": "BTH240101",
                "expiry_date": "2026-01-01",
                "quantity": 3000,
                "manufacturing_date": "2024-01-01",
                "status": "AVAILABLE"
            }
        ]
    }
}
```

### 3. Stock Movement Notifications (Real-time Updates)

**Endpoint yang WAJIB Dipanggil PBF**: `POST /api/balimed/stock-updates`
**Direction**: **PBF (mengirim)** → Balimed
**Trigger**: Setiap ada perubahan stok di sistem PBF

**Payload yang Harus Dikirim PBF:**

```json
{
    "timestamp": "2024-12-15T10:30:00+07:00",
    "location": "DENPASAR",
    "updates": [
        {
            "product_code": "BRG001",
            "previous_stock": 5000,
            "current_stock": 4700,
            "movement_type": "SALE",
            "movement_quantity": -300,
            "reason": "Sold to Balimed PO/24/12/0001",
            "reference_number": "PO/24/12/0001/DENPSR",
            "updated_at": "2024-12-15T10:30:00+07:00"
        }
    ]
}
```

### 4. Stock Reservation

**Endpoint yang WAJIB Dibuat PBF**: `POST /api/distributor/stock/reserve`
**Direction**: Balimed → **PBF (memproses)**

**Request yang Akan Dikirim Balimed:**

```json
{
    "reservation_id": "RSV240001",
    "po_number": "PO/24/12/0001/DENPSR",
    "expiry_hours": 24,
    "items": [
        {
            "product_code": "BRG001",
            "quantity": 100,
            "preferred_batch": "BTH240101"
        }
    ]
}
```

**Response yang WAJIB Diberikan PBF:**

```json
{
    "status": "success",
    "reservation_id": "RSV240001",
    "expires_at": "2024-12-16T10:30:00+07:00",
    "reserved_items": [
        {
            "product_code": "BRG001",
            "requested_quantity": 100,
            "reserved_quantity": 100,
            "batch_number": "BTH240101",
            "expiry_date": "2026-01-01"
        }
    ]
}
```

---

## Business Rules yang Harus Diimplementasi PBF

### 1. Stock Accuracy Requirements
- **WAJIB** update stok real-time setiap ada perubahan
- **WAJIB** sinkronisasi dengan sistem warehouse PBF
- **WAJIB** validasi stok sebelum konfirmasi PO
- **WAJIB** FIFO/FEFO untuk batch management

### 2. Data Freshness
- Stock data maksimal 5 menit terlambat dari actual
- Batch information harus akurat dan up-to-date
- Price information harus sesuai kontrak terbaru

### 3. API Performance Requirements
- Response time < 2 detik untuk stock queries
- Support minimum 1000 requests/hour
- Implement proper caching untuk frequently accessed products

### 4. Error Handling
```json
{
    "status": "error",
    "error_code": "PRODUCT_NOT_FOUND",
    "message": "Produk dengan kode BRG999 tidak ditemukan",
    "timestamp": "2024-12-15T10:30:00+07:00"
}
```

**Error Codes yang Harus Didukung:**
- `PRODUCT_NOT_FOUND` - Produk tidak ditemukan
- `INSUFFICIENT_STOCK` - Stok tidak mencukupi
- `INVALID_LOCATION` - Lokasi tidak valid
- `AUTHORIZATION_FAILED` - Token tidak valid
- `RATE_LIMIT_EXCEEDED` - Request limit terlampaui

---

## Security Requirements

### 1. Authentication & Authorization
- **WAJIB** Bearer token validation
- **WAJIB** IP whitelisting untuk Balimed servers
- **WAJIB** rate limiting per API key
- **WAJIB** audit logging untuk semua stock queries

### 2. Data Protection
- **WAJIB** HTTPS untuk semua komunikasi
- **WAJIB** encrypt sensitive data (pricing)
- **WAJIB** mask competitor information

---

## Integration Testing Checklist

### Phase 1: Basic Stock API
- [ ] Implement GET /api/distributor/stock endpoint
- [ ] Implement GET /api/distributor/stock/{product_code}
- [ ] Test authentication and authorization
- [ ] Test error handling scenarios

### Phase 2: Real-time Updates
- [ ] Implement stock movement notifications
- [ ] Test real-time sync with Balimed
- [ ] Validate data accuracy and timing

### Phase 3: Advanced Features
- [ ] Implement stock reservation system
- [ ] Test batch management and FIFO/FEFO
- [ ] Performance testing and optimization

### Phase 4: Production Deployment
- [ ] Setup monitoring and alerting
- [ ] Configure rate limiting and security
- [ ] Train staff on new stock management workflow

---

_API ini memungkinkan Balimed untuk mengakses informasi stok PBF secara real-time, membuat keputusan pembelian yang lebih akurat, dan mengoptimalkan inventory management._