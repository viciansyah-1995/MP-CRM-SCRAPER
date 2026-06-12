# MP-CRM-SCRAPER

Marketplace Customer Data Scraper — A desktop application for extracting customer data from TikTok Shop and Shopee.

## ⚠️ Disclaimer

This tool is for **authorized internal use only**. Scraping from TikTok Shop and Shopee may violate their Terms of Service. Use at your own risk and ensure you have proper authorization.

## 📋 Overview

**MP-CRM-SCRAPER** helps sales and marketing teams extract customer data from two major Indonesian marketplaces:

- **TikTok Shop**
- **Shopee**

### Features

- ✅ Manual login to TikTok Shop and Shopee
- ✅ Extract customer info: name, phone, address, order details
- ✅ Local SQLite database storage
- ✅ Export to CSV and Excel
- ✅ Dashboard with customer insights
- ✅ Filter by date range, location, products
- ⏳ Scheduled scraping (future)
- ⏳ PostgreSQL support (future)

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| UI | Electron + React + Tailwind |
| Backend | Node.js |
| Browser Automation | Playwright |
| Database | SQLite (default) |

## 🚀 Getting Started

### Prerequisites

- Node.js 18+
- npm or yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/viciansyah-1995/MP-CRM-SCRAPER.git

# Navigate to project folder
cd MP-CRM-SCRAPER

# Install dependencies
npm install

# Run in development mode
npm run dev
```

### Build

```bash
npm run build
```

## 📖 Documentation

- [SPEC.md](./SPEC.md) — Full specification document
- [UI Mockups](./docs/UI_MOCKUP.md) — Sample UI wireframes

## 📁 Project Structure

```
MP-CRM-SCRAPER/
├── src/
│   ├── main/           # Electron main process
│   ├── renderer/       # React frontend
│   ├── scraper/       # TikTok & Shopee scraper modules
│   └── database/       # SQLite/DB handlers
├── SPEC.md             # Specification document
└── README.md           # This file
```

## ⚙️ Configuration

Create `.env` file in root directory:

```env
# Database
DATABASE_URL=file:./data.db

# App
NODE_ENV=development
```

## 📜 License

Internal use only. See LICENSE for details.

---

*For internal purposes only. Not for public distribution.*