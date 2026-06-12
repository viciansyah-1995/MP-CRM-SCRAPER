# TikTok Detail Order Flow

Dokumen ini menjelaskan alur scraping TikTok Shop yang berfokus pada:
- order list
- detail order
- status hide / unhide
- antrean follow-up CRM

---

## 1. High-Level Flow

```mermaid
flowchart TD
    A[User login ke TikTok Shop] --> B[Masuk ke halaman Order List]
    B --> C[Pilih rentang tanggal / filter order]
    C --> D[Load daftar order]
    D --> E[Loop setiap order]
    E --> F[Buka Detail Order]
    F --> G[Ekstrak data customer]
    G --> H[Ekstrak detail produk dan order]
    H --> I[Validasi data]
    I --> J{Visibility status?}
    J -->|Hide| K[Simpan data, tandai non-follow-up]
    J -->|Unhide| L[Simpan data, masukkan ke antrean CRM]
    K --> M[Audit log]
    L --> M[Audit log]
    M --> N[Lanjut ke order berikutnya]
    N --> O{Masih ada order?}
    O -->|Yes| E
    O -->|No| P[Generate hasil scrape]
```

---

## 2. Detail Decision Flow — Hide / Unhide

```mermaid
flowchart LR
    A[Detail order terbuka] --> B[Customer data terlihat]
    B --> C[System cek visibility status existing]
    C --> D{Status saat ini}
    D -->|Hide| E[Customer disimpan tapi tidak masuk CRM queue]
    D -->|Unhide| F[Customer masuk CRM queue]
    E --> G[Opsional: user ubah ke unhide]
    F --> H[Opsional: user ubah ke hide]
    G --> I[Catat changed_by, changed_at, reason]
    H --> I
    I --> J[Update customer/order visibility_status]
```

---

## 3. CRM Follow-up Queue Flow

```mermaid
flowchart TD
    A[Data order TikTok selesai diproses] --> B{visibility_status = unhide?}
    B -->|No| C[Masuk database saja]
    B -->|Yes| D[Masuk CRM intake queue]
    D --> E[CRM team review]
    E --> F{Layak difollow-up?}
    F -->|No| G[Update status: hidden / archived]
    F -->|Yes| H[Assign ke PIC CRM]
    H --> I[Set follow-up status]
    I --> J[Track outcome follow-up]
```

---

## 4. Suggested Data Fields

### Order-level
- order_id
- platform = tiktok
- order_date
- order_status
- total_amount
- product_summary
- visibility_status (`hide` / `unhide`)
- visibility_reason
- visibility_changed_by
- visibility_changed_at

### Customer-level
- customer_name
- phone
- address
- city
- province
- last_order_id
- last_seen_at
- crm_followup_status
- assigned_to
- notes

---

## 5. Scope Notes

1. Scraping TikTok harus masuk sampai **detail order**, bukan hanya list.
2. Hide/unhide adalah bagian dari **business workflow**, bukan cuma tampilan UI.
3. CRM queue hanya menerima customer/order dengan status **unhide**.
4. Semua perubahan visibility harus masuk **audit trail**.

---

## 6. Suggested Future Enhancements

- multi-step approval sebelum unhide
- assignment otomatis ke tim CRM berdasarkan kota / produk
- duplicate detection lintas order customer yang sama
- CRM score untuk prioritas follow-up

---

*End of TikTok Detail Order Flow Document*