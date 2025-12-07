⸻

MeetHub: Peminjaman Ruang Meeting – Full Technical Documentation

Format: .md – Long-term, production-grade documentation

⸻

1. BUSINESS REQUIREMENT DOCUMENT (BRD)

1.1 Executive Summary

Organisasi membutuhkan sistem terpusat untuk mengelola peminjaman beberapa ruang meeting. Proses manual menyebabkan bentrok jadwal, ketidakpastian, dan kurangnya transparansi. Sistem ini menyediakan booking otomatis dengan validasi hari kerja, jam operasional 08:00–22:00, mitigasi overlap, dan alur approval dua peran (User & Admin).

1.2 Business Goals
	•	Eliminasi bentrok booking.
	•	Otomatisasi approval.
	•	Peningkatan efisiensi administrasi ruang meeting.
	•	Pelacakan aktivitas dan log audit.

1.3 Stakeholders
	•	Operational User: melakukan permintaan booking.
	•	Admin Operator: mengelola approval.
	•	IT Engineer: mengelola sistem.
	•	Management: pemilik laporan aktivitas penggunaan ruang.

1.4 Business Constraints
	•	Booking hanya valid pada hari kerja (Mon–Fri).
	•	Jam operasional 08:00–22:00.
	•	Resolusi waktu 1 jam penuh.
	•	Sistem harus mencegah race condition.

1.5 Business Risks
	•	Double-booking akibat race.
	•	User mengakali jam di luar range.
	•	Admin overload approval.
	•	Ambiguitas status booking jika logic tidak ketat.

⸻

2. PRODUCT REQUIREMENT DOCUMENT (PRD)

2.1 Product Scope

In-Scope: booking, room management, approval, validation, logs.
Out-of-Scope: payment, integrasi eksternal, kalender publik otomatis.

2.2 User Roles

User
	•	Membuat booking.
	•	Melihat status bookingnya.

Admin
	•	Melihat semua booking.
	•	Approve / Reject booking.

2.3 Core Features
	1.	Room Listing
	2.	Booking Creation
	3.	Time & Business Rule Validation
	4.	Approval Workflow
	5.	Booking History / Logs
	6.	Admin Dashboard

2.4 Product Success Metrics
	•	0% double-booking.
	•	<1% rejected submission akibat error sistem.
	•	Waktu approval admin menurun ≥50%.

⸻

3. SOFTWARE REQUIREMENT SPECIFICATION (SRS)

3.1 Functional Requirements

FR-01 User Authentication
	•	Sistem harus menyediakan login dan role (user/admin).

FR-02 Room Management
	•	Admin dapat membuat room, dengan atribut:
	•	name, type, capacity, facilities.

FR-03 Booking Creation
	•	User membuat booking dengan input:
	•	room_id, date, start_time, end_time.

FR-04 Business Validation
	•	Booking hanya untuk weekday.
	•	Jam start & end harus antara 08–22.
	•	end_time > start_time.
	•	Resolusi jam penuh.

FR-05 Conflict Detection
	•	Sistem wajib menolak booking jika terjadi overlap:

start_time < existing_end
AND
end_time > existing_start



FR-06 Approval Workflow
	•	Booking default status: pending.
	•	Admin dapat approve atau reject.
	•	Approved booking tidak dapat diubah user.

FR-07 Activity Logging
	•	Semua perubahan status dicatat.

3.2 Non-Functional Requirements (NFR)

NFR-01 Performance
	•	Query booking < 300ms.

NFR-02 Security
	•	Role-based access.
	•	Validation di FormRequest dan Service Layer.

NFR-03 Reliability
	•	Semua booking creation dilakukan via DB transaction + row locking.

NFR-04 Scalability
	•	Harus mampu menampung ≥500 booking/hari.

NFR-05 Availability
	•	Target uptime 99%.

⸻

4. SYSTEM ARCHITECTURE DESIGN

4.1 High-Level Architecture
	•	Frontend: Blade
	•	Backend: Laravel 12
	•	Database: MySQL 8
	•	Infra: VPS Linux + Nginx
	•	Cache: Redis (opsional)

Arsitektur Komponen

User → Controller → BookingService → Repository → DB
                             ↓
                          Validator
                             ↓
                          Event Log

4.2 Database Design (ERD)

Tables:
	1.	users
	2.	rooms
	3.	bookings
	4.	logs (opsional)

Relasi:
	•	users (1) —— (∞) bookings
	•	rooms (1) —— (∞) bookings

4.3 Booking Overlap Logic

Teknik: SQL conditional overlap

SELECT 1 FROM bookings
WHERE room_id = :room
AND status IN ('pending', 'approved')
AND start_time < :end
AND end_time > :start
LIMIT 1;

4.4 Transaction & Lock
	•	Gunakan DB::transaction().
	•	Lock row room menggunakan FOR UPDATE saat booking dibuat.

⸻

5. RESPONSIVE DESIGN & USER EXPERIENCE SPECIFICATION

5.1 Mobile-First Approach

Sistem dirancang dengan prioritas tinggi pada pengalaman pengguna mobile (smartphone), mengingat mayoritas user akan mengakses sistem melalui perangkat mobile untuk kemudahan dan fleksibilitas.

5.2 Responsive Breakpoints

Sistem harus responsif dan user-friendly di semua ukuran perangkat:

	•	Mobile (Priority: HIGH)
	•	Small: 320px - 480px (smartphone portrait)
	•	Medium: 481px - 768px (smartphone landscape, small tablet)
	•	Tablet (Priority: MEDIUM)
	•	769px - 1024px (tablet portrait & landscape)
	•	Desktop (Priority: MEDIUM)
	•	1025px - 1440px (laptop, desktop)
	•	Large Desktop (Priority: LOW)
	•	1441px+ (large monitors)

5.3 Mobile-First Design Principles

Touch-Friendly Interface
	•	Minimum touch target: 44x44px (Apple HIG) / 48x48px (Material Design).
	•	Spacing antar elemen interaktif minimum 8px.
	•	Button dan form input harus mudah di-tap tanpa zoom.

Simplified Navigation
	•	Hamburger menu untuk navigasi utama di mobile.
	•	Bottom navigation bar untuk akses cepat fitur utama.
	•	Breadcrumb minimal atau hidden di mobile.

Optimized Content Layout
	•	Single column layout untuk mobile.
	•	Card-based design untuk listing rooms dan bookings.
	•	Collapsible sections untuk detail informasi.
	•	Infinite scroll atau pagination yang mobile-friendly.

Form Optimization
	•	Input fields full-width di mobile.
	•	Native date/time picker untuk booking.
	•	Auto-focus dan keyboard type sesuai input (numeric, email, etc).
	•	Inline validation dengan feedback jelas.
	•	Sticky submit button di bottom screen.

Performance on Mobile
	•	Lazy loading untuk images dan content.
	•	Minimal JavaScript bundle size.
	•	Progressive Web App (PWA) ready (opsional).
	•	Offline capability untuk view booking history (opsional).

5.4 Responsive Components

Navigation
	•	Mobile: Hamburger menu + bottom nav
	•	Tablet: Sidebar collapsible
	•	Desktop: Full sidebar + top navigation

Room Listing
	•	Mobile: Vertical card stack, 1 column
	•	Tablet: Grid 2 columns
	•	Desktop: Grid 3-4 columns

Booking Form
	•	Mobile: Full-screen modal, step-by-step wizard
	•	Tablet: Modal centered, single form
	•	Desktop: Sidebar form atau modal

Booking Calendar/Schedule
	•	Mobile: List view dengan date filter
	•	Tablet: Week view
	•	Desktop: Month/week view dengan detail sidebar

Admin Dashboard
	•	Mobile: Stacked cards, swipeable tabs
	•	Tablet: 2-column layout
	•	Desktop: Multi-column dashboard dengan widgets

5.5 Typography & Readability

Font Sizing (Mobile-First)
	•	Body text: 16px (mobile) → 16-18px (desktop)
	•	Headings: Scalable dengan viewport units
	•	Line height: 1.5-1.6 untuk readability

Contrast & Accessibility
	•	WCAG AA compliance minimum.
	•	Color contrast ratio ≥ 4.5:1 untuk text.
	•	Support dark mode (opsional).

5.6 Testing Requirements

Device Testing
	•	Real device testing pada:
	•	iPhone (Safari iOS)
	•	Android (Chrome)
	•	iPad
	•	Desktop browsers (Chrome, Firefox, Safari, Edge)

Responsive Testing Tools
	•	Chrome DevTools responsive mode
	•	BrowserStack atau LambdaTest
	•	Lighthouse mobile score ≥ 90

User Experience Metrics
	•	First Contentful Paint (FCP) < 1.8s pada 3G
	•	Time to Interactive (TTI) < 3.5s pada 3G
	•	Cumulative Layout Shift (CLS) < 0.1

5.7 Implementation Guidelines

CSS Framework
	•	Tailwind CSS (recommended) dengan responsive utilities
	•	Atau Bootstrap 5 dengan mobile-first grid

Viewport Meta Tag

<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0">

Media Queries Strategy
	•	Mobile-first: base styles untuk mobile, media queries untuk larger screens
	•	Avoid fixed widths, use relative units (%, rem, em, vw)

Images & Assets
	•	Responsive images dengan srcset
	•	WebP format dengan fallback
	•	SVG untuk icons
	•	Lazy loading untuk images below fold

5.8 Success Criteria
	•	100% fitur accessible di mobile tanpa horizontal scroll
	•	Touch targets memenuhi minimum size guidelines
	•	Form completion rate di mobile ≥ 85%
	•	Mobile bounce rate < 40%
	•	Google Mobile-Friendly Test: Pass

⸻

6. SECURITY SPECIFICATION

6.1 Authentication
	•	Laravel Authentication (Fortify/Passport/Sanctum).

6.2 Authorization
	•	Policies:
	•	User hanya akses booking miliknya.
	•	Admin akses semua booking.

6.3 Data Validation

Double-layer:
	1.	FormRequest
	2.	BookingService validator

6.4 Input Sanitization
	•	Laravel escape default.
	•	No unsafe HTML input.

6.5 Logging & Audit
	•	Log perubahan status booking.
	•	Log login attempt.

6.6 Rate Limiting
	•	API booking: 30 attempts/min.

⸻

7. TESTING PLAN

7.1 Unit Testing
	•	Booking validation
	•	Overlap detection
	•	Business rules weekday/time

7.2 Feature Testing
	•	Booking creation
	•	Admin approval
	•	Conflict prevention

7.3 Load Testing
	•	500 booking sekaligus via artillery/JMeter.

7.4 Security Testing
	•	Role escalation attempt
	•	SQL injection
	•	Invalid time manipulation

7.5 UAT
	•	Admin & user mencoba alur realistis.

⸻

8. DEPLOYMENT SPECIFICATION

8.1 Environment
	•	OS: Ubuntu 22.04
	•	Web server: Nginx
	•	PHP: 8.2
	•	DB: MySQL 8+/PostgreSQL 14+

8.2 Deployment Flow
	1.	Push to GitHub main.
	2.	CI run: test + lint.
	3.	SSH deploy → pull → composer install → migrate.
	4.	Set up Supervisor (queue).
	5.	Restart services.

8.3 Backup Strategy
	•	DB backup harian.
	•	7-day retention.
	•	Offsite weekly backup.

⸻

9. MAINTENANCE & LONG-TERM OPERATION

9.1 Monitoring
	•	Log aktivitas via Laravel log + Sentry.
	•	DB slow query logging aktif.

9.2 Performance Optimization
	•	Index on room_id, start_time, end_time.
	•	Caching room list.

9.3 Bug Handling
	•	Set branch hotfix untuk perbaikan cepat.

9.4 Versioning
	•	Semantic Versioning (MAJOR.MINOR.PATCH).

9.5 Future Enhancements
	•	Integrasi kalender Google.
	•	Notifikasi email/WhatsApp.
	•	Export laporan penggunaan ruangan.

⸻

END OF DOCUMENT