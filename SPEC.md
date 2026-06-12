# MP-CRM-SCRAPER Specification

## 1. Project Overview

### Project Name
**MP-CRM-SCRAPER** — Marketplace Customer Data Scraper

### Project Type
Internal tool / Desktop Application (for authorized use only)

### Tagline
Customer data enrichment tool for TikTok Shop and Shopee marketplace.

### Problem Statement
Tim sales dan marketing membutuhkan data customer dari marketplace (TikTok Shop, Shopee) untuk:
- Lead generation
- Customer profiling
- Targeted marketing
- CRM enrichment

Saat ini tidak ada cara easy untuk get data customer dari kedua platform secara efisien.

### Proposed Solution
Aplikasi desktop yang bisa scrape customer data dari TikTok Shop dan Shopee dengan authorized access (login manual / API resmi kalau ada).

### Scope Clarification — CRM Follow-up Flow
Untuk TikTok Shop, data customer yang sensitif dan siap follow-up diasumsikan diambil saat user membuka **detail order**. Pada layar detail order, sistem internal perlu mengakomodasi status **hide / unhide** agar tim CRM dapat menentukan data mana yang siap diproses untuk follow-up dan mana yang harus tetap disembunyikan.

Implikasi scope:
- scraping tidak cukup berhenti di order list; harus bisa masuk ke detail order
- sistem perlu menyimpan status visibilitas data customer per order/customer
- tim CRM hanya memproses data customer dengan status `unhide` / visible
- audit log perlu mencatat perubahan status hide/unhide untuk kebutuhan kontrol internal

---

## 2. Goals & Objectives

### Primary Goals
1. Enable tim untuk dapat customer list dari TikTok Shop dan Shopee
2. Simpan data ke database lokal (SQLite/PostgreSQL)
3. Export data ke format yang bisa di-import ke CRM (CSV, Excel)

### Secondary Goals
1. Schedule scraping untuk data terbaru
2. Dashboard untuk visualize customer insights
3. Alert untuk customer tertentu (berdasarkan keyword, lokasi, dll)

### Out of Scope (Untuk Saat Ini)
- Auto-purchase / checkout automation
- Real-time monitoring
- Multi-account management
- Public API integration (belum tersedia dari platform)

---

## 3. User Stories

| As a | I want to | So that |
|------|-----------|---------|
| Sales Person | Login ke TikTok/Shopee via app | Saya bisa akses data order customer |
| Sales Person | Ambil data customer (nama, no HP, alamat, produk yang dibeli) | Saya bisa buat leads list |
| Sales Person | Buka detail order TikTok untuk mengambil data customer yang lebih lengkap | Saya bisa memastikan data yang masuk memang berasal dari order detail |
| CRM Team | Menandai data customer sebagai hide atau unhide | Saya bisa mengontrol data mana yang siap difollow up |
| Sales Person | Filter data berdasarkan date range, produk, lokasi | Saya bisa target market yang tepat |
| Marketing | Export data ke CSV/Excel | Saya bisa import ke CRM saya |
| Admin | Lihat history scraping dan status | Saya bisa monitor penggunaan |
| Admin | Setup schedule scraping harian | Data selalu up-to-date |

---

## 4. Functional Requirements

### FR-01: Authentication
- **FR-01-01**: User bisa login dengan credentials TikTok Shop
- **FR-01-02**: User bisa login dengan credentials Shopee
- **FR-01-03**: Session persistence (tidak perlu login ulang terus-menerus)
- **FR-01-04**: Logout functionality

### FR-02: Data Extraction - TikTok Shop
- **FR-02-01**: Ambil data order dari TikTok Shop
- **FR-02-02**: Masuk ke halaman **detail order** untuk mengambil data customer yang lebih lengkap
- **FR-02-03**: Ambil customer info: nama, phone, email (kalau ada), address
- **FR-02-04**: Ambil produk yang dibeli, qty, price
- **FR-02-05**: Filter berdasarkan date range
- **FR-02-06**: Pagination support (ambil semua halaman)
- **FR-02-07**: Simpan status visibilitas data customer per order: `hide` / `unhide`
- **FR-02-08**: Hanya data dengan status `unhide` yang dianggap siap untuk follow-up CRM
- **FR-02-09**: Catat audit log saat status hide/unhide berubah

### FR-03: Data Extraction - Shopee
- **FR-03-01**: Ambil data order dari Shopee
- **FR-03-02**: Ambil customer info: nama, phone, address, shop location
- **FR-03-03**: Ambil produk yang dibeli, qty, price, variant
- **FR-03-04**: Filter berdasarkan date range
- **FR-03-05**: Pagination support

### FR-04: Data Storage
- **FR-04-01**: Simpan data ke SQLite (local)
- **FR-04-02**: Option untuk PostgreSQL (remote)
- **FR-04-03**: Table untuk: customers, orders, products, scrape_logs
- **FR-04-04**: Tambahkan penyimpanan status visibilitas customer/order: `hide` / `unhide`
- **FR-04-05**: Tambahkan audit metadata: di-hide oleh siapa, di-unhide oleh siapa, kapan berubah
- **FR-04-06**: Timestamp untuk setiap scrape operation

### FR-05: Data Export
- **FR-05-01**: Export ke CSV
- **FR-05-02**: Export ke Excel (.xlsx)
- **FR-05-03**: Custom field selection saat export
- **FR-05-04**: Filter sebelum export

### FR-06: Dashboard
- **FR-06-01**: Tampilkan total customer yang sudah discrape
- **FR-06-02**: Tampilkan total order
- **FR-06-03**: Tampilkan customer berdasarkan lokasi
- **FR-06-04**: Tampilkan top products

### FR-07: Schedule (Future)
- **FR-07-01**: Setup daily/weekly scrape schedule
- **FR-07-02**: Background scraping tanpa open app

---

## 5. Non-Functional Requirements

| Requirement | Description |
|-------------|-------------|
| **Performance** | Mampu scrape 1000+ order dalam 10 menit |
| **Reliability** | Auto-retry saat network fail, max 3x |
| **Security** | Credential di-encrypt storage |
| **Privacy** | Data customer di-hidden default, perlu authorization untuk lihat |
| **Error Handling** | Clear error message untuk user |
| **Logging** | Semua operasi di-log dengan timestamp |

---

## 6. Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    UI Layer (Desktop)                    │
│                   (Electron / Tauri)                     │
├─────────────────────────────────────────────────────────┤
│                  Business Logic Layer                    │
│    (Auth, Scraper Engine, Data Processor, Scheduler)     │
├─────────────────────────────────────────────────────────┤
│                     Data Layer                           │
│            (SQLite / PostgreSQL + File Storage)           │
├─────────────────────────────────────────────────────────┤
│                External Integration                       │
│          (TikTok Shop + Shopee via Browser)               │
└─────────────────────────────────────────────────────────┘
```

### Flowchart

```
User Login → Select Platform → Scrape Orders → Process Data → Store to DB → Export
```

---

## 7. Tech Stack

| Component | Technology | Notes |
|-----------|------------|-------|
| UI Framework | **Electron** or **Tauri** | Desktop app |
| Frontend | **React + Tailwind** | Modern UI |
| Backend | **Node.js** or **Python** | Scripting + API |
| Browser Automation | **Puppeteer** or **Playwright** | Untuk scrape via browser |
| Database | **SQLite** (default) / **PostgreSQL** | Local storage |
| State Management | Zustand / Redux | Untuk frontend |
| Build Tool | Vite / Webpack | Production build |

### Alternative Tech Stack (Python-based)

| Component | Technology | Notes |
|-----------|------------|-------|
| UI Framework | **Tkinter** or **PyQt** | Simpler, lighter |
| Backend | **Python** | Selenium / Playwright |
| Database | **SQLite** / **PostgreSQL** | Same |
| Scripts | **Scrapy** / **Selenium** | For scraping |

---

## 8. Data Model

### Table: customers

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| platform | TEXT | 'tiktok' or 'shopee' |
| platform_customer_id | TEXT | Customer ID from platform |
| name | TEXT | Customer name |
| phone | TEXT | Phone number |
| email | TEXT | Email (nullable) |
| address | TEXT | Full address |
| city | TEXT | City |
| province | TEXT | Province |
| created_at | DATETIME | First seen |
| updated_at | DATETIME | Last update |

### Table: orders

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| order_id | TEXT | Order ID from platform |
| customer_id | UUID | FK to customers |
| platform | TEXT | 'tiktok' or 'shopee' |
| total_amount | REAL | Total order amount |
| status | TEXT | Order status |
| order_date | DATETIME | When order placed |
| created_at | DATETIME | When scraped |

### Table: order_items

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| order_id | UUID | FK to orders |
| product_name | TEXT | Product name |
| product_sku | TEXT | SKU |
| quantity | INTEGER | Qty |
| price | REAL | Unit price |

### Table: scrape_logs

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| platform | TEXT | Platform scraped |
| start_time | DATETIME | When started |
| end_time | DATETIME | When finished |
| status | TEXT | 'success' / 'failed' |
| error_message | TEXT | If failed |
| records_count | INTEGER | Number of records |

---

## 9. UI/UX Concept

### Layout Overview

```
┌─────────────────────────────────────────────────────────┐
│ [Logo] MP-CRM-SCRAPER          [User: Admin] [Settings]  │
├──────────┬──────────────────────────────────────────────┤
│          │                                               │
│ Dashboard│          Main Content Area                   │
│ ─────────│                                               │
│ TikTok   │   [Content varies by section]                │
│ ─────────│                                               │
│ Shopee   │                                               │
│ ─────────│                                               │
│ Reports  │                                               │
│ ─────────│                                               │
│ Settings │                                               │
│          │                                               │
├──────────┴──────────────────────────────────────────────┤
│ Status: Ready | Last scrape: 2026-06-12 12:00 | v0.1   │
└─────────────────────────────────────────────────────────┘
```

### Screen 1: Dashboard

- Cards: Total Customer, Total Orders, Last Scrape Time
- Charts: Customer by Location (pie), Top Products (bar)
- Recent Activity table

### Screen 2: TikTok Shop Scraper

- Login status indicator
- Date range picker
- "Start Scrape" button (besar, prominent)
- Progress bar during scrape
- Results preview table
- Akses ke **detail order** dari daftar order
- Di layar/detail panel order, tampilkan status **hide / unhide** customer
- Hanya order/customer `unhide` yang masuk ke antrean follow-up CRM
- Export button

### Screen 3: Shopee Scraper

- Same layout as TikTok

### Screen 4: Reports / Export

- Filter options: platform, date range, fields
- Preview table
- Export to CSV / Excel buttons

### Screen 5: Settings

- Database configuration (SQLite/PostgreSQL)
- Schedule settings (future)
- About / Version info

---

## 10. Sample UI Mockup (Text)

### Dashboard Screen

```
┌─────────────────────────────────────────────────────────────┐
│  🛒 MP-CRM-SCRAPER                              [⚙️ Settings] │
├─────────────┬───────────────────────────────────────────────┤
│             │  📊 DASHBOARD                                  │
│  🏠 Dashboard│  ─────────────────────────────────────────────│
│             │                                                 │
│  📱 TikTok  │  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│             │  │ 👥 Customers│ │ 📦 Orders │ │ ⏰ Last  │       │
│  🛍️ Shopee  │  │   1,234   │ │   5,678  │ │ 2h ago  │       │
│             │  └──────────┘ └──────────┘ └──────────┘       │
│  📑 Reports │                                                 │
│             │  📍 Customer by Location    📦 Top Products   │
│  ⚙️ Settings│  ┌──────────────┐ ┌──────────────────┐       │
│             │  │ Jakarta 40%  │ │ Product A   123 │       │
│             │  │ Surabaya 20% │ │ Product B    98 │       │
│             │  │ Bandung 15% │ │ Product C    76 │       │
│             │  │ Others 25%   │ │ ...              │       │
│             │  └──────────────┘ └──────────────────┘       │
│             │                                                 │
├─────────────┴───────────────────────────────────────────────┤
│  ✅ Ready | v1.0.0                                          │
└─────────────────────────────────────────────────────────────┘
```

### TikTok Scraper Screen

```
┌─────────────────────────────────────────────────────────────┐
│  🛒 MP-CRM-SCRAPER                              [⚙️ Settings] │
├─────────────┬───────────────────────────────────────────────┤
│             │  📱 TIKTOK SHOP SCRAPER                       │
│  🏠 Dashboard│  ─────────────────────────────────────────────│
│             │                                                 │
│  📱 TikTok  │  ┌─────────────────────────────────────────┐  │
│  ⬤ Active   │  │ Status: ● Logged in as @my_tiktok_store   │  │
│             │  └─────────────────────────────────────────┘  │
│  🛍️ Shopee  │                                                 │
│             │  📅 Date Range                                │
│  📑 Reports │  From: [06/01/2026] To: [06/12/2026]         │
│             │                                                 │
│  ⚙️ Settings│  📋 Options                                    │
│             │  ☑ Include customer phone                      │
│             │  ☑ Include address                            │
│             │  ☑ Include order details                      │
│             │                                                 │
│             │            [ 🚀 START SCRAPE ]                 │
│             │                                                 │
│             │  Progress: ████████░░░░░░░ 45% (456/1000)      │
│             │                                                 │
│             │  ┌─────────────────────────────────────────┐  │
│             │  │ Preview (last 5)                        │  │
│             │  │ ─────────────────────────────────────────│  │
│             │  │ John D. | Order #123 | Rp 150.000        │  │
│             │  │ Jane S. | Order #124 | Rp 75.000        │  │
│             │  │ ...                                      │  │
│             │  └─────────────────────────────────────────┘  │
│             │                                                 │
│             │            [ 📥 EXPORT CSV ] [ 📥 EXPORT XLSX ]│
│             │                                                 │
├─────────────┴───────────────────────────────────────────────┤
│  ✅ Ready | Last scrape: 2026-06-12 11:45 | v1.0.0          │
└─────────────────────────────────────────────────────────────┘
```

---

## 11. Future Roadmap

| Phase | Feature | Status |
|-------|---------|--------|
| v1.0.0 | Basic scrape (TikTok + Shopee), SQLite, CSV export | Planned |
| v1.1.0 | Dashboard dengan charts, Excel export | Planned |
| v1.2.0 | PostgreSQL support, scheduled scraping | Future |
| v1.3.0 | Alert system (notifikasi customer baru), multi-account | Future |
| v2.0.0 | API mode (bukan GUI), webhook notifications | Future |

---

## 12. Disclaimer & Notes

1. **Use at your own risk** — Scraping bisa violate platform ToS
2. **Authorized use only** — Jangan gunakan untuk data yang bukan milikmu
3. **Respect rate limits** — Jangan spam request, bisa cause account suspension
4. **Data privacy** — Handling customer data harus sesuai regulasi (UU PDP Indonesia)
5. **No warranty** — Software provided as-is

---

## 13. Revision History

| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-06-12 | Initial specification |

---

*End of Specification*