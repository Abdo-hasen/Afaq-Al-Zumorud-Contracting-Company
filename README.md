# 🔧 Afaq Al-Zumorud Contracting Co. — On-Site Field Services Platform

> A full-featured **on-site services** platform built with **Laravel 11**, connecting **clients** with **technicians** through **service orders**, scheduling, and operational workflows.
> The system includes a **mobile-facing REST API** and a **fully operational admin dashboard** for day-to-day management.

---

## ✨ Features

### 👤 Client (Mobile API)

- OTP (SMS) + email/password login, forgot password & reset link flow
- Browse services and place orders — **ASAP** or **scheduled date + time slot**
- Track order status and step timeline (accepted → on the way → on-site → done)
- Receive, approve or reject **price quotations** with line-item breakdown
- Invoice details (items, VAT, total, paid state) and payment method summary
- **Push notifications** (FCM) for key order events (accepted, quote ready, completed)
- Post-service **rating** — star score, and optional comment

### 🔨 Technician (Mobile API)

- **Online / offline** availability toggle — only receives jobs when active
- Home dashboard with **current active task**, quick actions, and map context
- Full task details: job photos, address, problem description, client info
- **Create quotations** with multiple line items (service, qty, unit price, total)
- My jobs list filtered by status (all / pending / in progress / completed)
- Rate the **client** after job completion — two-way feedback

### 🌐 Public 

- Services catalog, settings, and banners

### 🛠️ Admin Dashboard

- Full management: Users, Technicians, Services, Orders, Quotations, Settings
- **Block days**: recurring weekday rules and one-off dates per service
- Role-based staff permissions (Spatie)
- Multi-language support (Arabic / English)
- Real-time stats and order charts
- Firebase push notifications (custom & bulk)

---

## 🏗️ Tech Stack


| Layer       | Technology                                 |
| ----------- | ------------------------------------------ |
| Backend     | Laravel 11 (PHP 8.2)                       |
| Auth        | Laravel Sanctum + OTP (SMS)                |
| Admin UI    | Blade + Yajra DataTables                   |
| Permissions | Spatie Laravel Permission                  |
| Media       | Spatie Laravel MediaLibrary                |
| i18n        | Spatie Translatable + Laravel Localization |
| Payments    | Moyasar Payment Integration                |
| Push        | Firebase Cloud Messaging (FCM)             |
| Export      | Maatwebsite Excel + MisterSpelik PDF       |
| Monitoring  | Laravel Telescope                          |


---

## 📐 Architecture

```
├── app/
│   ├── Http/Controllers/     # Thin controllers (Admin + API V1)
│   ├── Core/
│   │   ├── Services/         # Business logic layer
│   │   ├── Datatables/       # Yajra DataTable definitions
│   │   ├── Enums/            # Type-safe constants
│   │   ├── Helpers/          # Firebase, media, OTP
│   │   └── Traits/           # InteractWithResponse, HasDatatableTrait
├── routes/
│   ├── admin.php             # Admin dashboard routes
│   └── apis/v1/              # auth · client · technician · public · notifications
└── resources/views/          # Blade templates
```

---

## 🧩 Technical Challenges & Solutions

### 1. Blocking weekdays vs blocking single dates

**Challenge:** Admin sets rules like "no bookings every Friday" or "close Dec 25." The mobile calendar needs a ready list of disabled days — and the server must enforce the same rules when an order is saved.

**Solution:** Store two rule types: **recurring weekday** and **specific date**. On request, expand weekday rules into real dates only within a configurable rolling window (e.g. next 2 months), merge in one-off dates, deduplicate, and return the sorted list. Order validation re-runs the same check so the UI and the API always agree.

---

### 2. Cash / POS handover — two-party OTP confirmation

**Challenge:** After a technician collects cash or processes a POS payment, the business needs proof the right amount was handed over — not just a verbal claim.

**Solution:** Generate a **short-lived, single-use OTP** tied to the order and amount. The technician receives it via SMS; the customer writes it back. The server verifies once, logs the confirmation, and marks the payment as handed over. Codes expire and are scoped to one order so they cannot be reused or guessed across jobs.

---

### 3. OTP behavior controlled by environment

**Challenge:** Production must send real random codes via SMS. Development and staging should use a fixed known code and skip SMS entirely, without touching the codebase.

**Solution:** Two env flags drive the behavior — `RANDOM_OTP` (true = random, false = fixed dev code) and `SEND_OTP_SMS` (true = send SMS, false = skip). A bypass phone-number list allows QA testers to always receive a predictable code. Neither bypass mechanism is active in production.

---

### 4. Bulk Firebase Notifications (Rate Limits, Delays & Jobs)

**Challenge:** Sending push notifications to a large audience from the admin panel can hit **FCM quotas**, **HTTP timeouts**, and **unreliable partial delivery** if the server tries to send synchronously in one request. Retries can also **duplicate** messages without careful design.

**Solution:** Treat bulk sends as **asynchronous work**: enqueue **dedicated queue jobs** (e.g. **one job per user/token batch**) instead of sending inside the web request. Between sends, apply a controlled **delay/backoff** so traffic stays under FCM limits and workers stay healthy. Optionally group tokens into **FCM multicast chunks** where the API allows, while still keeping **per-user failure isolation** and **retry**  in the queue layer.

---

## 📸 Screenshots

### 🖥️ Admin Dashboard

---

### 📱 Mobile App

---

## 👤 My Role

Built end-to-end as the **sole backend developer** — architecture, API design, admin dashboard, payment integration, and deployment.

---

## 📄 License

Private commercial project — source shared for **portfolio purposes only**. Not for redistribution or reuse.
