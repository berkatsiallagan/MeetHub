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
	•	Memberikan rating dan feedback untuk ruangan yang sudah digunakan (sekali per ruangan).
	•	Membalas komentar user lain pada ruangan yang sama.

Admin
	•	Melihat semua booking.
	•	Approve / Reject booking.
	•	Mengelola ruangan yang menjadi tanggung jawabnya (PIC).
	•	Satu admin dapat mengelola banyak ruangan.
	•	Memiliki nomor WhatsApp untuk komunikasi dengan user.
	•	Dapat memfilter kata-kata tidak sopan pada komentar.
	•	Mengajukan penghapusan komentar brutal kepada developer (tidak dapat menghapus langsung).

2.3 Core Features
	1.	Room Listing
	2.	Booking Creation
	3.	Time & Business Rule Validation
	4.	Approval Workflow
	5.	Booking History / Logs
	6.	Admin Dashboard
	7.	Multi-Admin Management dengan PIC per Ruangan
	8.	Room Gallery (Multiple Photos)
	9.	Room Facilities & Rules Management
	10.	Rating & Review System
	11.	Comment & Feedback System dengan Reply Thread

2.4 Product Success Metrics
	•	0% double-booking.
	•	<1% rejected submission akibat error sistem.
	•	Waktu approval admin menurun ≥50%.

⸻

3. SOFTWARE REQUIREMENT SPECIFICATION (SRS)

3.1 Functional Requirements

FR-01 User Authentication & Authorization

Priority: HIGH

Authentication Methods (Prioritas berurutan):
	1.	Login with Google (OAuth 2.0) - PRIORITY TINGGI
	•	User dapat login menggunakan akun Google mereka
	•	Otomatis membuat akun baru jika belum terdaftar (auto-registration)
	•	Mengambil data: email, name, profile picture dari Google
	•	Default role: user (admin assignment manual via database/seeder)
	2.	Traditional Registration & Login - PRIORITY MEDIUM
	•	Fallback option jika user tidak ingin menggunakan Google
	•	Registration form: name, email, password, password_confirmation
	•	Login form: email, password
	•	Email verification (opsional, recommended)

Role Management:
	•	Sistem memiliki 2 role: user dan admin
	•	Default role untuk registrasi baru: user
	•	Admin role di-assign manual oleh super admin atau via database seeder

Session & Security:
	•	Session-based authentication menggunakan Laravel Sanctum/Fortify
	•	Remember me functionality
	•	Logout dari semua device (opsional)
	•	Password reset via email (untuk traditional login)

FR-02 Room Management
	•	Admin dapat membuat room, dengan atribut:
	•	name, type, capacity
	•	status (enum: available, maintenance, unavailable, reserved)
	•	status_note (optional, keterangan status)
	•	Multiple photos (gallery)
	•	Facilities (JSON/relational)
	•	SOP (Standard Operating Procedure)
	•	Aturan penggunaan
	•	Denda dan pelanggaran
	•	Admin PIC (Person In Charge) - relasi many-to-many dengan admin
	•	Average rating (calculated field)

FR-02a Room Status Management
	•	Setiap ruangan memiliki status yang menunjukkan ketersediaan:
	•	available: Ruangan tersedia untuk dibooking
	•	maintenance: Ruangan sedang dalam perbaikan/maintenance, tidak bisa dibooking
	•	unavailable: Ruangan tidak tersedia untuk sementara (alasan lain)
	•	reserved: Ruangan direservasi untuk keperluan khusus
	•	Admin PIC dapat mengubah status ruangan kapan saja
	•	Saat status bukan "available", ruangan tidak muncul di daftar booking
	•	User dapat melihat semua ruangan dengan status indicator yang jelas
	•	Status note (opsional) memberikan informasi detail, contoh:
	•	"Sedang perbaikan AC, estimasi selesai 15 Desember 2025"
	•	"Direservasi untuk acara perusahaan"
	•	"Renovasi ruangan, tidak tersedia hingga pemberitahuan lebih lanjut"
	•	Sistem mencatat history perubahan status untuk audit trail
	•	Notifikasi otomatis ke user yang memiliki booking pending jika ruangan berubah status

FR-03 Booking Creation
	•	User membuat booking dengan input:
	•	room_id, date, start_time, end_time
	•	purpose (tujuan peminjaman) - dropdown dengan opsi custom input
	•	participant_count (jumlah orang) - numeric input
	•	notes (optional) - textarea untuk catatan tambahan
	•	Sistem hanya menampilkan ruangan dengan status "available" untuk booking
	•	Ruangan dengan status lain (maintenance, unavailable, reserved) tidak dapat dibooking

FR-04 Business Validation
	•	Booking hanya untuk weekday.
	•	Jam start & end harus antara 08–22.
	•	end_time > start_time.
	•	Resolusi jam penuh.
	•	Participant count harus > 0 dan ≤ room capacity.
	•	Real-time validation: alert user jika jumlah orang melebihi kapasitas ruangan.
	•	Purpose wajib diisi (dari dropdown atau custom input).

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

FR-08 Multi-Admin Management
	•	Sistem mendukung banyak admin.
	•	Setiap admin memiliki nomor WhatsApp untuk komunikasi.
	•	Admin dapat menjadi PIC untuk banyak ruangan.
	•	Relasi many-to-many antara admin dan rooms.
	•	User dapat melihat admin PIC dan nomor WhatsApp untuk komunikasi.

FR-09 Room Gallery Management
	•	Setiap ruangan dapat memiliki multiple photos.
	•	Minimum 1 foto, maksimum 10 foto (configurable).
	•	Foto pertama menjadi thumbnail utama.
	•	Admin PIC dapat menambah/menghapus foto ruangan.
	•	Format: JPG, PNG, WebP.
	•	Max size per foto: 2MB.

FR-10 Room Information Management
	•	Setiap ruangan memiliki:
	1.	Facilities: daftar fasilitas (AC, Projector, Whiteboard, dll)
	2.	SOP: Standard Operating Procedure (rich text)
	3.	Aturan: Rules penggunaan ruangan (rich text)
	4.	Denda dan Pelanggaran: ketentuan denda (rich text)

FR-11 Rating System
	•	User dapat memberikan rating setelah menggunakan ruangan.
	•	Rating: skala 1-10 dengan 1 desimal (contoh: 9.6/10).
	•	Visualisasi: bintang (5 bintang, dengan half-star support).
	•	Konversi: rating 10 = 5 bintang, rating 5 = 2.5 bintang.
	•	User hanya dapat memberikan rating 1x per ruangan (lifetime).
	•	Rating hanya dapat diberikan setelah booking approved dan waktu booking telah lewat.
	•	Average rating ruangan dihitung otomatis.
	•	Rating tidak dapat diedit atau dihapus oleh user.

FR-12 Feedback/Comment System
	•	User dapat memberikan feedback setelah menggunakan ruangan.
	•	User hanya dapat membuat 1 komentar utama per ruangan (lifetime).
	•	Komentar dapat diberikan setelah booking approved dan waktu booking telah lewat.
	•	User dapat membalas komentar user lain (nested replies).
	•	User dapat membalas reply dalam thread yang sama (unlimited depth).
	•	Sistem mencegah spam dengan batasan 1 komentar utama per user per ruangan.
	•	Komentar memiliki timestamp dan user info.
	•	Admin tidak dapat menghapus komentar/rating secara langsung.
	•	Admin dapat memfilter kata-kata tidak sopan (profanity filter).
	•	Admin dapat mengajukan penghapusan komentar brutal kepada developer.
	•	Komentar yang diajukan untuk dihapus akan di-flag untuk review developer.

FR-13 Booking Purpose Management
	•	System menyediakan dropdown purpose yang dapat dikonfigurasi.
	•	Admin/Developer dapat menambah, edit, hapus purpose options.
	•	Purpose options diurutkan berdasarkan frekuensi penggunaan (most used first).
	•	User dapat memilih dari dropdown atau input custom purpose.
	•	System tracking purpose usage untuk analytics dan auto-populate dropdown.
	•	Purpose examples: "Meeting Tim", "Presentasi Client", "Training", "Interview", "Workshop", dll.

FR-14 Participant Count Validation
	•	Setiap ruangan memiliki max_capacity (jumlah maksimal orang).
	•	User input participant_count saat booking.
	•	Real-time validation: jika participant_count > room.max_capacity:
	•	Show alert: "Jumlah peserta ({count}) melebihi kapasitas ruangan ({capacity}). Silakan pilih ruangan lain."
	•	Disable submit button.
	•	Suggest alternative rooms dengan kapasitas yang sesuai.
	•	Visual indicator: red border pada input + warning icon.
	•	Validation dilakukan client-side (instant feedback) dan server-side (security).

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
	•	id, name, email, email_verified_at, password (nullable untuk Google login), 
	•	google_id (nullable), avatar (nullable), role (enum: user/admin), 
	•	whatsapp_number (nullable, untuk admin), 
	•	remember_token, created_at, updated_at
	
	2.	rooms
	•	id, name, type, max_capacity (integer), 
	•	status (enum: 'available', 'maintenance', 'unavailable', 'reserved'), 
	•	status_note (text, nullable),
	•	sop (text), rules (text), penalties (text),
	•	average_rating (decimal 2,1), total_ratings (integer),
	•	created_at, updated_at
	
	3.	booking_purposes
	•	id, purpose_name, usage_count (integer), 
	•	is_active (boolean), created_at, updated_at
	
	4.	room_photos
	•	id, room_id, photo_path, order (integer), 
	•	is_primary (boolean), created_at, updated_at
	
	5.	room_facilities
	•	id, room_id, facility_name, 
	•	created_at, updated_at
	
	6.	room_admins (pivot table)
	•	id, room_id, admin_id (user_id), 
	•	assigned_at, created_at, updated_at
	
	7.	bookings
	•	id, user_id, room_id, date, start_time, end_time, 
	•	purpose (string), participant_count (integer),
	•	status, notes, created_at, updated_at
	
	8.	ratings
	•	id, user_id, room_id, booking_id, 
	•	rating (decimal 2,1), 
	•	created_at, updated_at
	•	UNIQUE constraint: (user_id, room_id)
	
	9.	comments
	•	id, user_id, room_id, booking_id, 
	•	parent_comment_id (nullable, untuk reply), 
	•	comment (text), 
	•	is_flagged (boolean, untuk admin flag), 
	•	flagged_reason (text, nullable),
	•	created_at, updated_at
	•	UNIQUE constraint untuk root comment: (user_id, room_id) WHERE parent_comment_id IS NULL
	
	10.	logs (opsional)

Relasi:
	•	users (1) —— (∞) bookings
	•	rooms (1) —— (∞) bookings
	•	rooms (1) —— (∞) room_photos
	•	rooms (1) —— (∞) room_facilities
	•	rooms (∞) —— (∞) users (admins) via room_admins
	•	users (1) —— (∞) ratings
	•	rooms (1) —— (∞) ratings
	•	bookings (1) —— (1) rating (optional)
	•	users (1) —— (∞) comments
	•	rooms (1) —— (∞) comments
	•	comments (1) —— (∞) comments (self-referential, untuk replies)
	•	bookings (1) —— (1) comment (optional, untuk root comment)

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

Implementation Stack:
	•	Laravel Sanctum - untuk session-based authentication
	•	Laravel Socialite - untuk Google OAuth integration
	•	Laravel Fortify (opsional) - untuk traditional login/register features

Google OAuth Flow:
	1.	User klik "Login with Google"
	2.	Redirect ke Google OAuth consent screen
	3.	User authorize aplikasi
	4.	Google redirect kembali dengan authorization code
	5.	Backend exchange code untuk access token
	6.	Retrieve user info dari Google API
	7.	Check apakah email sudah terdaftar:
	•	Jika ya: login user tersebut
	•	Jika tidak: buat user baru dengan data dari Google
	8.	Create session dan redirect ke dashboard

Traditional Login Flow:
	1.	User input email & password
	2.	Validate credentials
	3.	Check email verification status (jika diaktifkan)
	4.	Create session dan redirect ke dashboard

Security Measures:
	•	Google OAuth: validate state parameter untuk CSRF protection
	•	Password: minimum 8 karakter, hashed dengan bcrypt
	•	Rate limiting: 5 login attempts per minute per IP
	•	Session timeout: 2 jam inactivity (configurable)

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
	•	Notifikasi email/WhatsApp otomatis.
	•	Export laporan penggunaan ruangan.
	•	Social login tambahan (Microsoft, GitHub).
	•	Two-factor authentication (2FA).
	•	Single Sign-On (SSO) untuk enterprise.
	•	AI-powered profanity filter untuk komentar.
	•	Analytics dashboard untuk rating dan feedback trends.
	•	WhatsApp Business API integration untuk komunikasi langsung.
	•	Push notification untuk mobile app.

⸻

10. RATING & FEEDBACK SYSTEM SPECIFICATION

10.1 Rating System Design

Rating Scale & Conversion
	•	Input: 1.0 - 10.0 (dengan 1 desimal)
	•	Display: Bintang (★) 0.5 - 5.0
	•	Konversi: rating_bintang = rating_angka / 2
	•	Contoh: 9.6/10 = 4.8★, 7.0/10 = 3.5★

Rating Rules
	•	User hanya dapat rating 1x per ruangan (lifetime restriction).
	•	Rating hanya dapat diberikan setelah:
	1.	Booking status = approved
	2.	Waktu booking telah selesai (end_time < NOW())
	•	Rating tidak dapat diedit atau dihapus oleh user.
	•	Admin tidak dapat menghapus rating.

Rating Calculation
	•	Average rating dihitung real-time atau via scheduled job.
	•	Formula: AVG(rating) dari semua ratings untuk room tersebut.
	•	Disimpan di rooms.average_rating untuk performa.
	•	Total ratings disimpan di rooms.total_ratings.

Rating Display
	•	List view: tampilkan average rating + total ratings (contoh: 4.5★ (23 reviews))
	•	Detail view: tampilkan average + distribusi rating (bar chart)
	•	User profile: tampilkan history ratings yang diberikan user

10.2 Comment/Feedback System Design

Comment Structure
	•	Root Comment: komentar utama user untuk ruangan
	•	Reply: balasan terhadap root comment atau reply lain
	•	Thread: struktur nested unlimited depth

Comment Rules
	•	User hanya dapat membuat 1 root comment per ruangan (lifetime).
	•	User dapat membalas comment lain unlimited (dalam thread yang sama).
	•	Comment hanya dapat dibuat setelah:
	1.	Booking status = approved
	2.	Waktu booking telah selesai
	•	Comment tidak dapat diedit atau dihapus oleh user.
	•	Comment dapat di-flag oleh admin untuk review.

Comment Moderation
	•	Profanity Filter:
	•	Auto-detect kata-kata tidak sopan (configurable word list).
	•	Replace dengan asterisk (contoh: "b***k").
	•	Admin dapat update filter word list.
	•	Flag System:
	•	Admin dapat flag comment sebagai "brutal/inappropriate".
	•	Flagged comment tetap visible tapi marked untuk developer review.
	•	Developer dapat menghapus comment yang di-flag.
	•	Flagged comment history disimpan untuk audit.

Comment Display
	•	Chronological order (newest first) untuk root comments.
	•	Nested display untuk replies (indented).
	•	Show user name, avatar, timestamp.
	•	Show "Edited" indicator jika ada (future feature).
	•	Pagination: 10 root comments per page.
	•	Load more untuk replies (collapse/expand).

10.3 User Eligibility Check

Validation Logic

// Pseudo-code
function canUserRateOrComment(user_id, room_id) {
    // Check if user has completed booking
    completed_booking = Booking::where('user_id', user_id)
        ->where('room_id', room_id)
        ->where('status', 'approved')
        ->where('end_time', '<', NOW())
        ->exists();
    
    if (!completed_booking) {
        return false;
    }
    
    // Check if already rated/commented
    already_rated = Rating::where('user_id', user_id)
        ->where('room_id', room_id)
        ->exists();
    
    already_commented = Comment::where('user_id', user_id)
        ->where('room_id', room_id)
        ->whereNull('parent_comment_id')
        ->exists();
    
    return [
        'can_rate' => !already_rated,
        'can_comment' => !already_commented,
        'can_reply' => true // always can reply
    ];
}

10.4 Admin Communication System

WhatsApp Integration
	•	Admin profile memiliki field whatsapp_number.
	•	Format: +62xxx (international format).
	•	Validation: regex untuk format nomor Indonesia.
	•	Display: click-to-chat link (wa.me/{number}).

Admin Contact Display
	•	Room detail page: tampilkan PIC admin dengan WhatsApp button.
	•	Booking detail: tampilkan admin PIC untuk komunikasi.
	•	Pre-filled message template:
	"Halo Admin {name}, saya ingin bertanya tentang ruangan {room_name}..."

Admin Assignment
	•	Super admin dapat assign/unassign admin ke ruangan.
	•	Admin dapat melihat daftar ruangan yang menjadi tanggung jawabnya.
	•	Notification ke admin saat ada booking baru untuk ruangannya.

10.5 UI/UX Specifications

Rating Input Component
	•	Star rating input (clickable stars).
	•	Numeric input (1-10) dengan slider.
	•	Real-time preview konversi bintang.
	•	Confirmation dialog sebelum submit (tidak dapat diubah).

Rating Display Component
	•	Star visualization (★★★★☆ 4.5/5.0).
	•	Color coding: 
	•	5.0-4.0 = green
	•	3.9-3.0 = yellow
	•	2.9-1.0 = red
	•	Tooltip: "Based on X reviews"

Comment Input Component
	•	Textarea dengan character limit (500 chars).
	•	Real-time character counter.
	•	Preview mode.
	•	Reply button untuk nested comments.
	•	Cancel button untuk reply mode.

Comment Display Component
	•	Card-based layout.
	•	User avatar + name + timestamp.
	•	Reply button (if eligible).
	•	Flag button (admin only).
	•	Nested indentation untuk replies (max 5 levels visual).
	•	"Load more replies" untuk collapsed threads.

Admin Moderation Panel
	•	List flagged comments.
	•	Filter by room, date, severity.
	•	Quick actions: unflag, escalate to developer.
	•	Profanity filter word list management.
	•	Bulk actions untuk multiple flags.

10.6 Performance Considerations

Caching Strategy
	•	Cache average rating per room (TTL: 1 hour).
	•	Cache comment count per room.
	•	Invalidate cache on new rating/comment.

Database Indexing
	•	Index: (user_id, room_id) pada ratings table.
	•	Index: (user_id, room_id, parent_comment_id) pada comments table.
	•	Index: room_id pada ratings dan comments untuk aggregation.
	•	Index: is_flagged pada comments untuk admin panel.

Query Optimization
	•	Eager load user relation untuk comments.
	•	Paginate comments (10 per page).
	•	Lazy load nested replies (on-demand).
	•	Use DB aggregation untuk rating calculation.

10.7 Security Considerations

Input Validation
	•	Rating: numeric, range 1.0-10.0, 1 decimal.
	•	Comment: string, max 500 chars, XSS sanitization.
	•	WhatsApp: regex validation, international format.

Authorization
	•	Policy: user can only rate/comment if eligible.
	•	Policy: admin can only flag, not delete.
	•	Policy: developer role untuk delete flagged comments.

Rate Limiting
	•	Rating submission: 1 per user per room (lifetime).
	•	Comment submission: 1 root comment per user per room (lifetime).
	•	Reply submission: 10 per hour per user (prevent spam).
	•	Flag action: 20 per hour per admin.

Audit Trail
	•	Log semua rating submissions.
	•	Log semua comment submissions dan flags.
	•	Log admin flag actions dengan reason.
	•	Log developer delete actions.

⸻

11. MULTI-ADMIN & ROOM PIC SPECIFICATION

11.1 Admin Management

Admin Attributes
	•	Semua field dari user table.
	•	role = 'admin'.
	•	whatsapp_number (required untuk admin).
	•	Dapat mengelola multiple rooms.

Admin Assignment Flow
	1.	Super admin akses admin management panel.
	2.	Select admin dan room(s) untuk di-assign.
	3.	System create record di room_admins pivot table.
	4.	Admin menerima notification (email/in-app).
	5.	Admin dapat melihat assigned rooms di dashboard.

Admin Responsibilities
	•	Approve/reject bookings untuk assigned rooms.
	•	Update room information (photos, facilities, rules).
	•	Moderate comments (flag inappropriate content).
	•	Respond to user inquiries via WhatsApp.
	•	Monitor room usage dan ratings.

11.2 Room PIC Display

Room Detail Page
	•	Section: "Person In Charge"
	•	Display: Admin name, avatar, WhatsApp button.
	•	Multiple admins: list semua PIC dengan contact info.
	•	Click WhatsApp: open wa.me link dengan pre-filled message.

Booking Confirmation
	•	Show assigned admin untuk room tersebut.
	•	Provide contact info untuk komunikasi.
	•	Template message: "Untuk pertanyaan, hubungi Admin {name}"

Admin Dashboard
	•	Widget: "My Rooms" dengan list assigned rooms.
	•	Quick stats: pending bookings, average rating, total bookings.
	•	Quick actions: view bookings, manage room, view feedback.

11.3 Communication Flow

User → Admin Communication
	1.	User melihat room detail atau booking.
	2.	User klik WhatsApp button admin PIC.
	3.	WhatsApp web/app terbuka dengan pre-filled message.
	4.	User dan admin berkomunikasi di WhatsApp.
	5.	Admin dapat approve/reject booking via sistem.

Admin → User Communication
	•	Admin dapat melihat user contact (jika tersedia).
	•	Admin dapat mengirim update via WhatsApp.
	•	System dapat send notification ke user (future: WhatsApp API).

11.4 Authorization & Permissions

Admin Permissions
	•	Can view all bookings untuk assigned rooms.
	•	Can approve/reject bookings untuk assigned rooms.
	•	Can update room info untuk assigned rooms.
	•	Can flag comments untuk assigned rooms.
	•	Cannot delete ratings atau comments.
	•	Cannot assign/unassign self dari rooms.

Super Admin Permissions
	•	All admin permissions.
	•	Can assign/unassign admin ke rooms.
	•	Can create/update/delete rooms.
	•	Can view all rooms dan bookings.
	•	Can manage admin accounts.

Policy Implementation

// Pseudo-code
class RoomPolicy {
    public function update(User $user, Room $room) {
        return $user->role === 'admin' 
            && $room->admins->contains($user->id);
    }
    
    public function manageBookings(User $user, Room $room) {
        return $user->role === 'admin' 
            && $room->admins->contains($user->id);
    }
}

11.5 Scalability Considerations

Multiple Admins per Room
	•	Support unlimited admins per room.
	•	Notification ke semua assigned admins untuk booking baru.
	•	First-come-first-serve untuk approval (prevent conflict).
	•	Lock mechanism untuk prevent double approval.

Admin Workload Distribution
	•	Dashboard widget: pending bookings count per admin.
	•	Auto-assign algorithm (future): distribute berdasarkan workload.
	•	Admin dapat request unassign jika overload.

Performance
	•	Index room_admins.admin_id untuk fast lookup.
	•	Cache assigned rooms per admin.
	•	Eager load admins relation saat query rooms.

⸻

12. USER FEEDBACK & NOTIFICATION SYSTEM

12.1 Toast Notification System

Implementation Requirements
	•	Library: Alpine.js + Tailwind CSS atau Toast library (Toastify, Notyf)
	•	Position: Top-right corner (desktop), Top-center (mobile)
	•	Duration: 3-5 seconds (auto-dismiss), persistent untuk error critical
	•	Stack: Multiple notifications dapat muncul bersamaan (max 3 visible)
	•	Animation: Slide-in dari kanan/atas, fade-out saat dismiss

Notification Types & Use Cases

Success Notifications (Green)
	•	Login berhasil: "Selamat datang, {name}!"
	•	Logout berhasil: "Anda telah keluar dari sistem"
	•	Booking created: "Booking berhasil dibuat, menunggu approval admin"
	•	Booking approved: "Booking Anda telah disetujui!"
	•	Rating submitted: "Terima kasih atas rating Anda!"
	•	Comment posted: "Komentar berhasil dipublikasikan"
	•	Room created: "Ruangan berhasil ditambahkan"
	•	Room updated: "Informasi ruangan berhasil diperbarui"
	•	Photo uploaded: "Foto berhasil diunggah"
	•	Admin assigned: "Admin berhasil ditugaskan ke ruangan"

Error Notifications (Red)
	•	Login failed: "Email atau password salah"
	•	Booking conflict: "Ruangan sudah dibooking pada waktu tersebut"
	•	Validation error: "Mohon periksa kembali form Anda"
	•	Upload failed: "Gagal mengunggah foto, ukuran maksimal 2MB"
	•	Unauthorized: "Anda tidak memiliki akses untuk melakukan aksi ini"
	•	Network error: "Koneksi terputus, silakan coba lagi"
	•	Already rated: "Anda sudah memberikan rating untuk ruangan ini"
	•	Already commented: "Anda sudah memberikan komentar untuk ruangan ini"
	•	Booking time invalid: "Jam booking harus antara 08:00-22:00"
	•	Weekend booking: "Booking hanya tersedia untuk hari kerja (Senin-Jumat)"

Warning Notifications (Yellow/Orange)
	•	Session expiring: "Sesi Anda akan berakhir dalam 5 menit"
	•	Unsaved changes: "Anda memiliki perubahan yang belum disimpan"
	•	Booking pending: "Booking Anda masih menunggu approval"
	•	Incomplete profile: "Lengkapi profil Anda untuk pengalaman lebih baik"
	•	Photo limit: "Maksimal 10 foto per ruangan"

Info Notifications (Blue)
	•	Booking rejected: "Booking Anda ditolak. Alasan: {reason}"
	•	Comment flagged: "Komentar telah ditandai untuk review"
	•	Password reset sent: "Link reset password telah dikirim ke email Anda"
	•	Email verification sent: "Email verifikasi telah dikirim"
	•	Data loading: "Memuat data..."

12.2 Inline Validation Feedback

Form Field Validation
	•	Real-time validation saat user mengetik (debounced 500ms)
	•	Visual indicators:
	•	Valid: green border + checkmark icon
	•	Invalid: red border + error message below field
	•	Neutral: default border (gray)
	•	Error messages harus spesifik dan actionable

Example Error Messages:
	•	Email: "Format email tidak valid"
	•	Password: "Password minimal 8 karakter"
	•	Phone: "Format nomor WhatsApp: +62xxx"
	•	Date: "Pilih tanggal hari kerja (Senin-Jumat)"
	•	Time: "Jam harus antara 08:00-22:00"
	•	Rating: "Rating harus antara 1.0-10.0"

Button State Feedback
	•	Default: normal state
	•	Hover: subtle color change + cursor pointer
	•	Active/Clicked: loading spinner + disabled state
	•	Disabled: grayed out + cursor not-allowed
	•	Success: brief checkmark animation (optional)

12.3 Loading States & Skeleton Screens

Skeleton Screen Implementation

Purpose:
	•	Mengurangi perceived loading time
	•	Memberikan feedback visual bahwa konten sedang dimuat
	•	Meningkatkan user experience dengan menghindari blank screen

Components yang Memerlukan Skeleton:

1. Room List Skeleton
	•	Card layout dengan placeholder untuk:
	•	Image placeholder (shimmer effect)
	•	Title placeholder (2 lines)
	•	Rating placeholder (stars + text)
	•	Facility icons placeholder
	•	Button placeholder
	•	Show 6-8 skeleton cards saat loading

2. Room Detail Skeleton
	•	Gallery placeholder (large image + thumbnails)
	•	Title + rating placeholder
	•	Tabs placeholder (Fasilitas, SOP, Aturan, dll)
	•	Content area placeholder
	•	Admin PIC section placeholder
	•	Booking form placeholder

3. Booking List Skeleton
	•	Table rows dengan placeholder columns
	•	Status badge placeholder
	•	Action buttons placeholder
	•	Show 10 skeleton rows

4. Comment Section Skeleton
	•	Avatar + name placeholder
	•	Comment text placeholder (3-4 lines)
	•	Timestamp placeholder
	•	Reply button placeholder
	•	Show 5 skeleton comments

5. Dashboard Skeleton
	•	Widget cards placeholder
	•	Chart/graph placeholder
	•	Stats numbers placeholder
	•	Recent activity list placeholder

Skeleton Design Guidelines:
	•	Use shimmer/pulse animation (subtle, not distracting)
	•	Match actual content layout closely
	•	Use neutral gray colors (#E0E0E0, #F5F5F5)
	•	Animate from left to right (shimmer effect)
	•	Duration: show skeleton until data loaded (no minimum time)

12.4 Progress Indicators

Linear Progress Bar
	•	Use for: file uploads, multi-step forms
	•	Position: top of container atau below action button
	•	Show percentage: "Uploading... 45%"
	•	Color: primary brand color

Circular Spinner
	•	Use for: button actions, inline loading
	•	Size: small (16px) untuk buttons, medium (32px) untuk page loading
	•	Position: center of button atau content area
	•	Color: match button/context color

Full Page Loader
	•	Use for: initial page load, critical operations
	•	Overlay: semi-transparent backdrop
	•	Spinner: centered, medium-large size
	•	Optional text: "Memuat..." atau "Memproses..."
	•	Prevent user interaction saat loading

12.5 Empty States

Design Guidelines:
	•	Show meaningful illustration atau icon
	•	Clear message explaining why empty
	•	Call-to-action button (jika applicable)
	•	Helpful tips atau next steps

Empty State Scenarios:

No Rooms Available
	•	Icon: empty room illustration
	•	Message: "Belum ada ruangan tersedia"
	•	CTA: "Tambah Ruangan" (admin only)

No Bookings Yet
	•	Icon: calendar illustration
	•	Message: "Anda belum memiliki booking"
	•	CTA: "Lihat Ruangan Tersedia"

No Comments
	•	Icon: chat bubble illustration
	•	Message: "Belum ada komentar untuk ruangan ini"
	•	Subtext: "Jadilah yang pertama memberikan feedback!"

No Search Results
	•	Icon: magnifying glass
	•	Message: "Tidak ada hasil untuk '{query}'"
	•	Suggestion: "Coba kata kunci lain atau filter berbeda"

12.6 Confirmation Dialogs

Use Cases:
	•	Delete actions: "Yakin ingin menghapus ruangan ini?"
	•	Approve/Reject booking: "Konfirmasi approval booking?"
	•	Submit rating: "Rating tidak dapat diubah setelah dikirim. Lanjutkan?"
	•	Logout: "Yakin ingin keluar?"
	•	Cancel booking: "Yakin ingin membatalkan booking?"

Dialog Design:
	•	Modal overlay dengan backdrop
	•	Clear title + description
	•	Two buttons: Primary action + Cancel
	•	Destructive actions: red button
	•	Keyboard support: Enter (confirm), Esc (cancel)
	•	Focus trap: prevent interaction outside modal

12.7 Status Badges & Visual Indicators

Booking Status Badges:
	•	Pending: Yellow/Orange badge with clock icon
	•	Approved: Green badge with checkmark icon
	•	Rejected: Red badge with X icon
	•	Completed: Blue badge with flag icon
	•	Cancelled: Gray badge with minus icon

Room Availability Indicators:
	•	Available: Green dot
	•	Booked: Red dot
	•	Partially booked: Yellow dot

Rating Visual:
	•	Stars: filled (★) vs empty (☆)
	•	Color: gold/yellow for stars
	•	Show numeric value: "4.5/5.0"
	•	Show review count: "(23 reviews)"

12.8 Micro-interactions

Hover Effects:
	•	Cards: subtle shadow elevation
	•	Buttons: color darken + scale 1.02
	•	Links: underline appear
	•	Images: slight zoom (1.05)

Click Feedback:
	•	Buttons: scale down (0.98) on click
	•	Cards: brief highlight
	•	Checkboxes: checkmark animation
	•	Radio buttons: ripple effect

Focus States:
	•	Form inputs: blue outline ring
	•	Buttons: outline ring
	•	Links: outline ring
	•	Keyboard navigation: clear focus indicator

12.9 Accessibility Considerations

ARIA Labels:
	•	Loading states: aria-busy="true"
	•	Notifications: role="alert" untuk urgent, role="status" untuk info
	•	Buttons: aria-label untuk icon-only buttons
	•	Form errors: aria-describedby linking to error message

Screen Reader Support:
	•	Announce notifications via aria-live regions
	•	Announce loading states
	•	Announce form validation errors
	•	Skip to main content link

Keyboard Navigation:
	•	Tab order logical dan predictable
	•	Enter/Space untuk activate buttons
	•	Escape untuk close modals
	•	Arrow keys untuk navigate lists/menus

12.10 Implementation Checklist

Frontend Components:
	☐	Toast notification component
	☐	Skeleton loader components (room, booking, comment, dashboard)
	☐	Loading spinner component (button, inline, full-page)
	☐	Empty state component
	☐	Confirmation dialog component
	☐	Status badge component
	☐	Form validation feedback component

JavaScript/Alpine.js:
	☐	Toast notification service
	☐	Form validation utilities
	☐	Loading state management
	☐	Confirmation dialog service

CSS/Tailwind:
	☐	Skeleton shimmer animation
	☐	Hover/focus/active states
	☐	Transition utilities
	☐	Responsive breakpoints untuk notifications

Backend Integration:
	☐	Flash messages untuk redirect scenarios
	☐	JSON responses dengan success/error flags
	☐	Validation error formatting
	☐	Rate limiting error messages

12.11 Performance Considerations

Optimization:
	•	Debounce inline validation (500ms)
	•	Throttle scroll-triggered skeleton loading
	•	Lazy load images dengan skeleton placeholder
	•	Minimize notification DOM manipulation
	•	Use CSS animations over JavaScript when possible

Bundle Size:
	•	Use lightweight toast library (<5KB)
	•	Inline critical CSS untuk skeleton screens
	•	Defer non-critical notification scripts

12.12 Testing Requirements

Manual Testing:
	☐	Test semua success scenarios
	☐	Test semua error scenarios
	☐	Test notification stacking (multiple simultaneous)
	☐	Test skeleton screens pada slow 3G
	☐	Test keyboard navigation
	☐	Test screen reader announcements

Automated Testing:
	☐	Unit tests untuk notification service
	☐	Integration tests untuk form validation
	☐	E2E tests untuk critical user flows dengan feedback
	☐	Visual regression tests untuk skeleton screens

⸻

13. BOOKING FORM ENHANCEMENT SPECIFICATION

13.1 Purpose (Tujuan) Field

Field Design
	•	Type: Hybrid dropdown + custom input
	•	Label: "Tujuan Peminjaman *" (required field)
	•	Placeholder: "Pilih atau ketik tujuan..."

Dropdown Options
	•	Populated from booking_purposes table
	•	Sorted by usage_count DESC (most used first)
	•	Show top 10 most used purposes
	•	Last option: "Lainnya (ketik sendiri)..."

Custom Input Behavior
	•	When user selects "Lainnya", show text input field
	•	Text input placeholder: "Ketik tujuan peminjaman..."
	•	Max length: 100 characters
	•	Validation: required, min 3 characters
	•	Auto-save new purpose to database untuk future use
	•	Increment usage_count jika purpose sudah ada

Default Purpose Options (Seeder):
	1.	Meeting Tim Internal
	2.	Presentasi Client
	3.	Training/Workshop
	4.	Interview Kandidat
	5.	Brainstorming Session
	6.	Project Review
	7.	Rapat Koordinasi
	8.	Seminar/Webinar
	9.	Focus Group Discussion
	10.	Lainnya (ketik sendiri)...

13.2 Participant Count (Jumlah Orang) Field

Field Design
	•	Type: Numeric input dengan increment/decrement buttons
	•	Label: "Jumlah Peserta *" (required field)
	•	Min value: 1
	•	Max value: room.max_capacity
	•	Default value: 1
	•	Step: 1

Input Controls
	•	Text input (center-aligned number)
	•	Minus button (-) on left
	•	Plus button (+) on right
	•	Keyboard input allowed (numeric only)
	•	Scroll wheel support (optional)

Visual Feedback
	•	Show room capacity below input: "Kapasitas ruangan: {max_capacity} orang"
	•	Color coding:
	•	Green: participant_count ≤ max_capacity
	•	Red: participant_count > max_capacity
	•	Progress bar visual (optional): filled based on percentage

13.3 Real-Time Capacity Validation

Client-Side Validation (Instant Feedback)

Validation Trigger:
	•	On input change (keyup, button click)
	•	On room selection change
	•	Debounced 300ms untuk typing

Validation Logic:

// Pseudo-code
function validateParticipantCount() {
    const count = parseInt(participantCountInput.value);
    const capacity = selectedRoom.max_capacity;
    
    if (count > capacity) {
        // Show error state
        showError(`Jumlah peserta (${count}) melebihi kapasitas ruangan (${capacity} orang)`);
        disableSubmitButton();
        showAlternativeRooms(count);
        
        // Visual feedback
        participantCountInput.classList.add('border-red-500');
        capacityIndicator.classList.add('text-red-500');
        
        return false;
    } else {
        // Show success state
        clearError();
        enableSubmitButton();
        hideAlternativeRooms();
        
        // Visual feedback
        participantCountInput.classList.remove('border-red-500');
        participantCountInput.classList.add('border-green-500');
        capacityIndicator.classList.remove('text-red-500');
        capacityIndicator.classList.add('text-green-500');
        
        return true;
    }
}

Error Alert Design:
	•	Position: Below participant count input
	•	Icon: Warning triangle (⚠️)
	•	Color: Red background (#FEE2E2), red text (#DC2626)
	•	Message: "Jumlah peserta ({count}) melebihi kapasitas ruangan ({capacity} orang). Silakan pilih ruangan lain."
	•	Animation: Slide down + shake effect

Alternative Room Suggestions:
	•	Show when participant_count > current room capacity
	•	Display: Collapsible section below error message
	•	Title: "Ruangan alternatif yang sesuai:"
	•	List: 3-5 rooms dengan capacity ≥ participant_count
	•	Each item shows:
	•	Room name
	•	Capacity
	•	Availability status
	•	"Pilih Ruangan" button
	•	Sorted by: capacity ASC (closest match first)

13.4 Server-Side Validation

Validation Rules (Laravel FormRequest):

public function rules()
{
    return [
        'room_id' => 'required|exists:rooms,id',
        'date' => 'required|date|after_or_equal:today',
        'start_time' => 'required|date_format:H:i',
        'end_time' => 'required|date_format:H:i|after:start_time',
        'purpose' => 'required|string|min:3|max:100',
        'participant_count' => [
            'required',
            'integer',
            'min:1',
            function ($attribute, $value, $fail) {
                $room = Room::find($this->room_id);
                if ($value > $room->max_capacity) {
                    $fail("Jumlah peserta ($value) melebihi kapasitas ruangan ({$room->max_capacity} orang).");
                }
            },
        ],
        'notes' => 'nullable|string|max:500',
    ];
}

Error Response:
	•	Return JSON dengan validation errors
	•	Frontend display errors via toast notification
	•	Highlight invalid fields dengan red border

13.5 Booking Form Complete Layout

Form Structure (Step-by-Step):

Step 1: Pilih Ruangan
	•	Room selection (cards atau dropdown)
	•	Show: photo, name, capacity, rating, facilities
	•	Filter: by capacity, type, availability

Step 2: Pilih Waktu
	•	Date picker (weekdays only)
	•	Start time dropdown (08:00-21:00)
	•	End time dropdown (09:00-22:00)
	•	Duration display: "Durasi: X jam"

Step 3: Detail Peminjaman
	•	Purpose dropdown + custom input
	•	Participant count input dengan validation
	•	Notes textarea (optional)

Step 4: Review & Submit
	•	Summary semua input
	•	Room details
	•	Time details
	•	Purpose & participant count
	•	Admin PIC contact info
	•	Terms & conditions checkbox
	•	Submit button

Mobile Layout:
	•	Full-screen modal
	•	Progress indicator (1/4, 2/4, 3/4, 4/4)
	•	Back/Next buttons
	•	Sticky submit button

Desktop Layout:
	•	Sidebar form atau centered modal
	•	All steps visible (accordion style)
	•	Inline validation
	•	Submit button at bottom

13.6 Purpose Analytics & Management

Admin Dashboard - Purpose Analytics:
	•	Widget: "Top 10 Tujuan Peminjaman"
	•	Display: Bar chart atau table
	•	Columns: Purpose name, Usage count, Percentage
	•	Actions: Edit, Deactivate, Delete
	•	Filter: by date range, room

Purpose Management Interface:
	•	List all purposes dengan usage statistics
	•	Add new purpose manually
	•	Edit purpose name
	•	Deactivate purpose (hide from dropdown, keep data)
	•	Delete purpose (only if usage_count = 0)
	•	Merge duplicate purposes

Auto-Cleanup:
	•	Scheduled job: merge similar purposes (fuzzy matching)
	•	Archive purposes dengan usage_count = 0 after 6 months
	•	Notify admin untuk review dan approve merge

13.7 Capacity Planning Features

Room Capacity Utilization Report:
	•	Show average participant_count per room
	•	Identify underutilized rooms (avg < 50% capacity)
	•	Identify overbooked attempts (validation failures)
	•	Suggest room reallocation

Capacity Alerts:
	•	Notify admin jika banyak booking ditolak karena capacity
	•	Suggest adding more rooms atau increasing capacity
	•	Show peak usage times dan participant counts

13.8 UI/UX Enhancements

Smart Room Suggestions:
	•	When user inputs participant_count first, filter rooms automatically
	•	Show only rooms dengan capacity ≥ participant_count
	•	Sort by: capacity ASC (best fit first)

Capacity Indicator Visual:
	•	Progress bar showing capacity usage
	•	Color gradient: green → yellow → red
	•	Icon: person icons (👤) filled based on count
	•	Example: "👤👤👤👤👥 (8/10 orang)"

Quick Booking Presets:
	•	"Meeting Kecil (2-5 orang)"
	•	"Meeting Sedang (6-10 orang)"
	•	"Meeting Besar (11-20 orang)"
	•	"Seminar (20+ orang)"
	•	Auto-filter rooms based on preset

13.9 Accessibility & Validation

Form Accessibility:
	•	Label for all inputs dengan proper association
	•	Required field indicators (*)
	•	Error messages dengan aria-describedby
	•	Focus management untuk multi-step form
	•	Keyboard navigation support

Validation Messages:
	•	Purpose required: "Tujuan peminjaman wajib diisi"
	•	Purpose too short: "Tujuan minimal 3 karakter"
	•	Participant count required: "Jumlah peserta wajib diisi"
	•	Participant count min: "Minimal 1 peserta"
	•	Participant count exceeds: "Jumlah peserta melebihi kapasitas ruangan"
	•	Participant count invalid: "Jumlah peserta harus berupa angka"

13.10 Database Schema Updates

booking_purposes Table:

CREATE TABLE booking_purposes (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    purpose_name VARCHAR(100) NOT NULL,
    usage_count INT UNSIGNED DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    INDEX idx_usage_count (usage_count DESC),
    INDEX idx_is_active (is_active)
);

bookings Table Updates:

ALTER TABLE bookings 
ADD COLUMN purpose VARCHAR(100) NOT NULL AFTER end_time,
ADD COLUMN participant_count INT UNSIGNED NOT NULL AFTER purpose,
ADD INDEX idx_purpose (purpose),
ADD INDEX idx_participant_count (participant_count);

rooms Table Updates:

ALTER TABLE rooms 
CHANGE COLUMN capacity max_capacity INT UNSIGNED NOT NULL;

13.11 Implementation Checklist

Backend:
	☐	Create booking_purposes migration
	☐	Update bookings migration (add purpose, participant_count)
	☐	Update rooms migration (rename capacity to max_capacity)
	☐	Create BookingPurpose model
	☐	Update Booking model (add fillable fields)
	☐	Update BookingFormRequest validation
	☐	Create PurposeService untuk auto-increment usage_count
	☐	Create API endpoint untuk alternative room suggestions
	☐	Seed default booking purposes

Frontend:
	☐	Create purpose dropdown component
	☐	Create custom purpose input component
	☐	Create participant count input component (with +/- buttons)
	☐	Implement real-time capacity validation
	☐	Create capacity alert component
	☐	Create alternative rooms suggestion component
	☐	Add capacity indicator visual
	☐	Update booking form layout (multi-step)
	☐	Add inline validation feedback

Admin Panel:
	☐	Create purpose management interface
	☐	Create purpose analytics dashboard
	☐	Create capacity utilization report
	☐	Add purpose edit/delete functionality

Testing:
	☐	Test purpose dropdown population
	☐	Test custom purpose input dan auto-save
	☐	Test participant count validation (client & server)
	☐	Test alternative room suggestions
	☐	Test capacity alerts
	☐	Test purpose usage_count increment
	☐	Test edge cases (0 participants, negative, exceeds capacity)

⸻

14. ROOM STATUS MANAGEMENT SPECIFICATION

14.1 Room Status Overview

Status Types & Definitions

1. Available (Tersedia)
	•	Color: Green
	•	Icon: ✓ (checkmark)
	•	Description: Ruangan tersedia dan dapat dibooking
	•	Behavior: Muncul di daftar booking, dapat dipilih user
	•	Default status untuk ruangan baru

2. Maintenance (Dalam Perbaikan)
	•	Color: Orange/Yellow
	•	Icon: 🔧 (wrench) atau ⚠️ (warning)
	•	Description: Ruangan sedang dalam perbaikan atau maintenance
	•	Behavior: Tidak muncul di daftar booking, visible di room list dengan indicator
	•	Requires: status_note (recommended) untuk informasi detail

3. Unavailable (Tidak Tersedia)
	•	Color: Red
	•	Icon: ✗ (cross) atau 🚫 (prohibited)
	•	Description: Ruangan tidak tersedia untuk sementara (alasan umum)
	•	Behavior: Tidak muncul di daftar booking, visible di room list dengan indicator
	•	Requires: status_note (recommended) untuk alasan

4. Reserved (Direservasi)
	•	Color: Blue
	•	Icon: 🔒 (lock) atau ⭐ (star)
	•	Description: Ruangan direservasi untuk keperluan khusus/VIP
	•	Behavior: Tidak muncul di daftar booking publik, hanya admin dapat booking
	•	Requires: status_note (recommended) untuk keperluan apa

14.2 Status Management Interface

Admin Room Status Panel

Location: Admin Dashboard → Rooms → [Room Detail] → Status Tab

Components:
	1.	Current Status Display
	•	Large status badge dengan color coding
	•	Status name dan icon
	•	Last updated timestamp
	•	Updated by (admin name)
	
	2.	Status Change Form
	•	Dropdown: Select new status
	•	Textarea: Status note (required jika status bukan "available")
	•	Character limit: 500 characters
	•	Placeholder examples:
		○	"Perbaikan AC, estimasi selesai 15 Des 2025"
		○	"Renovasi lantai dan cat dinding"
		○	"Direservasi untuk rapat direksi"
	•	Checkbox: "Notify affected users" (jika ada pending bookings)
	•	Submit button: "Update Status"
	
	3.	Status History Log
	•	Table showing all status changes
	•	Columns: Date, Old Status, New Status, Note, Changed By
	•	Pagination: 10 records per page
	•	Export: CSV/PDF untuk audit

Quick Status Toggle:
	•	Room list view: quick toggle button
	•	One-click switch: Available ↔ Maintenance
	•	Requires confirmation dialog
	•	Auto-fill common status notes (templates)

14.3 User-Facing Status Display

Room List View

Status Indicator:
	•	Badge position: Top-right corner of room card
	•	Badge design: Rounded, semi-transparent overlay
	•	Text: Status name (e.g., "Tersedia", "Maintenance")
	•	Icon: Status icon
	•	Tooltip: Show status_note on hover

Visual Treatment:
	•	Available rooms: Normal display, full color
	•	Non-available rooms: 
	•	Slightly grayed out (opacity 0.7)
	•	"Tidak Tersedia" overlay
	•	Click to view details (read-only)
	•	No booking button

Filter Options:
	•	"Tampilkan semua ruangan" (default)
	•	"Hanya ruangan tersedia" (recommended for booking)
	•	Filter by status: Available, Maintenance, Unavailable, Reserved

Room Detail View

Status Section:
	•	Prominent status banner at top
	•	Color-coded background
	•	Large icon + status name
	•	Status note displayed clearly
	•	Last updated info

Booking Form Behavior:
	•	Available: Show booking form normally
	•	Non-available: 
	•	Hide booking form
	•	Show message: "Ruangan ini sedang tidak tersedia untuk booking"
	•	Show status note
	•	Show alternative available rooms
	•	Show admin PIC contact untuk inquiry

14.4 Booking Validation with Room Status

Server-Side Validation

Validation Rules:

public function rules()
{
    return [
        'room_id' => [
            'required',
            'exists:rooms,id',
            function ($attribute, $value, $fail) {
                $room = Room::find($value);
                if ($room->status !== 'available') {
                    $fail("Ruangan ini sedang {$room->status} dan tidak dapat dibooking.");
                }
            },
        ],
        // ... other rules
    ];
}

Booking Creation Flow:
	1.	User selects room
	2.	System checks room status
	3.	If status !== 'available':
	•	Show error message
	•	Suggest alternative rooms
	•	Prevent booking submission
	4.	If status === 'available':
	•	Proceed with normal booking flow

Client-Side Validation:
	•	Real-time check saat user pilih ruangan
	•	Disable booking form jika status bukan "available"
	•	Show alert dengan status note
	•	Auto-suggest alternative rooms

14.5 Status Change Impact on Existing Bookings

Pending Bookings Handling

Scenario: Admin changes room status to non-available

Automatic Actions:
	1.	Identify all pending bookings untuk room tersebut
	2.	Send notification ke affected users:
	•	Email notification
	•	In-app notification
	•	Toast notification (jika user online)
	3.	Notification content:
	•	"Ruangan {room_name} untuk booking Anda pada {date} {time} sedang {status}"
	•	Status note dari admin
	•	Suggest alternative rooms
	•	Contact admin PIC untuk assistance
	4.	Admin options:
	•	Auto-reject pending bookings (with reason)
	•	Keep pending untuk review manual
	•	Suggest alternative rooms to users

Admin Decision Dialog:

When changing status to non-available:
	•	Show warning: "Ada {count} booking pending untuk ruangan ini"
	•	Options:
	•	"Reject semua pending bookings" (with notification)
	•	"Biarkan pending untuk review manual"
	•	"Pindahkan ke ruangan alternatif" (suggest rooms)
	•	Textarea: Reason/note untuk users
	•	Checkbox: "Send notification to affected users"

Approved Bookings Handling:
	•	Approved bookings tetap valid (tidak auto-cancel)
	•	Admin harus manual review dan contact user
	•	System show warning: "Ada {count} approved bookings di masa depan"
	•	Suggest admin untuk reschedule atau cancel dengan kompensasi

14.6 Status Templates & Quick Actions

Pre-defined Status Templates

Admin dapat save dan reuse status notes:

Template Examples:
	1.	"Perbaikan AC - Estimasi selesai [DATE]"
	2.	"Renovasi ruangan - Tidak tersedia hingga pemberitahuan lebih lanjut"
	3.	"Direservasi untuk acara perusahaan"
	4.	"Maintenance rutin - Kembali tersedia besok"
	5.	"Perbaikan proyektor dan sound system"
	6.	"Pembersihan mendalam (deep cleaning)"

Template Management:
	•	Admin dapat create, edit, delete templates
	•	Templates stored per admin atau global
	•	Quick select dari dropdown saat update status
	•	Auto-fill status_note field

Scheduled Status Changes (Future Enhancement):
	•	Admin dapat schedule status change
	•	Example: "Set to maintenance dari 10 Des - 15 Des"
	•	Auto-revert to available setelah periode
	•	Notification reminder ke admin sebelum auto-revert

14.7 Status Analytics & Reporting

Admin Dashboard Widgets

1. Room Availability Overview
	•	Pie chart: Distribution of room statuses
	•	Available: X rooms (X%)
	•	Maintenance: X rooms (X%)
	•	Unavailable: X rooms (X%)
	•	Reserved: X rooms (X%)

2. Room Downtime Report
	•	Table: Rooms dengan longest downtime
	•	Columns: Room, Status, Days Unavailable, Last Status Change
	•	Alert: Rooms unavailable > 7 days (highlight red)

3. Status Change History
	•	Timeline view: Recent status changes
	•	Filter by: Room, Status, Date range, Admin
	•	Export untuk audit

4. Impact Analysis
	•	Rejected bookings due to status: Count
	•	User inquiries about unavailable rooms: Count
	•	Average downtime per room: Days

14.8 User Communication & Transparency

Status Information Display

Room List Page:
	•	Clear status badges
	•	Tooltip dengan status note
	•	Filter untuk show/hide unavailable rooms
	•	Count: "X dari Y ruangan tersedia"

Room Detail Page:
	•	Prominent status banner
	•	Detailed status note
	•	Estimated availability (jika ada)
	•	Admin PIC contact untuk inquiry
	•	Alternative rooms suggestion

Booking Page:
	•	Only show available rooms (default)
	•	Option: "Lihat semua ruangan termasuk yang tidak tersedia"
	•	Real-time status check sebelum submit

Notification System:
	•	Email: Status change affecting user's booking
	•	In-app: Real-time notification
	•	SMS/WhatsApp (future): Critical status changes

14.9 Database Schema for Status Management

rooms Table Updates:

ALTER TABLE rooms 
ADD COLUMN status ENUM('available', 'maintenance', 'unavailable', 'reserved') 
    DEFAULT 'available' NOT NULL AFTER max_capacity,
ADD COLUMN status_note TEXT NULL AFTER status,
ADD COLUMN status_updated_at TIMESTAMP NULL AFTER status_note,
ADD COLUMN status_updated_by BIGINT UNSIGNED NULL AFTER status_updated_at,
ADD INDEX idx_status (status),
ADD FOREIGN KEY (status_updated_by) REFERENCES users(id) ON DELETE SET NULL;

room_status_history Table (New):

CREATE TABLE room_status_history (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    room_id BIGINT UNSIGNED NOT NULL,
    old_status ENUM('available', 'maintenance', 'unavailable', 'reserved') NOT NULL,
    new_status ENUM('available', 'maintenance', 'unavailable', 'reserved') NOT NULL,
    status_note TEXT NULL,
    changed_by BIGINT UNSIGNED NOT NULL,
    affected_bookings_count INT UNSIGNED DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    FOREIGN KEY (room_id) REFERENCES rooms(id) ON DELETE CASCADE,
    FOREIGN KEY (changed_by) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_room_id (room_id),
    INDEX idx_created_at (created_at)
);

status_note_templates Table (New):

CREATE TABLE status_note_templates (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    template_name VARCHAR(100) NOT NULL,
    template_content TEXT NOT NULL,
    status_type ENUM('available', 'maintenance', 'unavailable', 'reserved') NOT NULL,
    is_global BOOLEAN DEFAULT FALSE,
    created_by BIGINT UNSIGNED NULL,
    usage_count INT UNSIGNED DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_status_type (status_type),
    INDEX idx_is_global (is_global)
);

14.10 API Endpoints for Status Management

Admin Endpoints:

1. Update Room Status
	•	POST /admin/rooms/{id}/status
	•	Body: { status, status_note, notify_users }
	•	Response: Updated room + affected bookings count

2. Get Status History
	•	GET /admin/rooms/{id}/status-history
	•	Query: ?page=1&per_page=10
	•	Response: Paginated status history

3. Get Status Templates
	•	GET /admin/status-templates
	•	Query: ?status_type=maintenance
	•	Response: List of templates

4. Create Status Template
	•	POST /admin/status-templates
	•	Body: { template_name, template_content, status_type, is_global }
	•	Response: Created template

5. Get Affected Bookings
	•	GET /admin/rooms/{id}/affected-bookings
	•	Query: ?status=pending
	•	Response: List of bookings yang akan terpengaruh

User Endpoints:

1. Get Available Rooms
	•	GET /rooms?status=available
	•	Response: Only available rooms

2. Get All Rooms with Status
	•	GET /rooms?include_unavailable=true
	•	Response: All rooms dengan status indicator

3. Check Room Availability
	•	GET /rooms/{id}/availability
	•	Response: { available, status, status_note, alternative_rooms }

14.11 UI Components Specification

Status Badge Component

Props:
	•	status: string (available|maintenance|unavailable|reserved)
	•	size: string (sm|md|lg)
	•	showIcon: boolean
	•	showTooltip: boolean
	•	statusNote: string (optional)

Variants:
	•	available: bg-green-100 text-green-800 border-green-300
	•	maintenance: bg-yellow-100 text-yellow-800 border-yellow-300
	•	unavailable: bg-red-100 text-red-800 border-red-300
	•	reserved: bg-blue-100 text-blue-800 border-blue-300

Status Change Modal Component

Sections:
	1.	Current status display
	2.	New status selection (dropdown)
	3.	Status note input (textarea)
	4.	Affected bookings warning (if any)
	5.	Notification options (checkboxes)
	6.	Action buttons (Cancel, Update)

Validation:
	•	Status note required jika status !== 'available'
	•	Min length: 10 characters
	•	Max length: 500 characters
	•	Confirmation required jika ada affected bookings

Status History Timeline Component

Display:
	•	Vertical timeline layout
	•	Each entry shows:
	•	Date & time
	•	Status change (old → new)
	•	Status note
	•	Changed by (admin name + avatar)
	•	Affected bookings count
	•	Color-coded dots based on status

Alternative Rooms Suggestion Component

Trigger: When room status !== 'available'

Display:
	•	Section title: "Ruangan Alternatif yang Tersedia"
	•	Filter criteria:
	•	Same or higher capacity
	•	Same type (if applicable)
	•	Available status
	•	Similar facilities
	•	Show 3-5 suggestions
	•	Each card shows:
	•	Room photo
	•	Name + capacity
	•	Rating
	•	Key facilities
	•	"Lihat Detail" button

14.12 Business Rules & Constraints

Status Change Rules:
	1.	Only admin PIC dapat change status untuk assigned rooms
	2.	Super admin dapat change status untuk semua rooms
	3.	Status change requires reason (status_note) jika bukan "available"
	4.	Status change logged untuk audit trail
	5.	Affected users harus di-notify (configurable)

Booking Rules:
	1.	User hanya dapat booking room dengan status "available"
	2.	Admin dapat override dan booking room dengan status "reserved"
	3.	Pending bookings auto-flagged jika room status changed
	4.	Approved bookings tidak auto-cancel, requires manual review

Display Rules:
	1.	Default view: only show available rooms
	2.	User dapat toggle untuk see all rooms
	3.	Non-available rooms displayed dengan clear indicator
	4.	Status note always visible untuk transparency

14.13 Testing Requirements

Unit Tests:
	☐	Test status validation rules
	☐	Test status change logic
	☐	Test affected bookings identification
	☐	Test status history logging
	☐	Test template CRUD operations

Integration Tests:
	☐	Test status change dengan notification
	☐	Test booking creation dengan status check
	☐	Test status filter di room list
	☐	Test alternative room suggestions
	☐	Test admin authorization untuk status change

E2E Tests:
	☐	Admin changes room status
	☐	User attempts booking non-available room
	☐	User receives notification untuk affected booking
	☐	User views status history
	☐	Admin uses status template

Manual Testing:
	☐	Test all status badge variants
	☐	Test status change modal UX
	☐	Test notification delivery
	☐	Test mobile responsive layout
	☐	Test accessibility (screen reader, keyboard nav)

14.14 Implementation Checklist

Database:
	☐	Create migration untuk rooms table updates (status, status_note)
	☐	Create migration untuk room_status_history table
	☐	Create migration untuk status_note_templates table
	☐	Create seeders untuk default status templates
	☐	Add indexes untuk performance

Backend:
	☐	Create RoomStatus enum
	☐	Update Room model (add status fields, relationships)
	☐	Create RoomStatusHistory model
	☐	Create StatusNoteTemplate model
	☐	Create RoomStatusService (status change logic)
	☐	Update BookingService (add status validation)
	☐	Create status change notification (email, in-app)
	☐	Create API endpoints untuk status management
	☐	Update booking validation rules
	☐	Create policy untuk status change authorization

Frontend:
	☐	Create StatusBadge component
	☐	Create StatusChangeModal component
	☐	Create StatusHistoryTimeline component
	☐	Create AlternativeRoomsSuggestion component
	☐	Update RoomList component (add status filter)
	☐	Update RoomDetail component (add status display)
	☐	Update BookingForm component (add status check)
	☐	Create admin status management interface
	☐	Add status templates management UI
	☐	Implement real-time status validation

Admin Panel:
	☐	Create room status management page
	☐	Create status history view
	☐	Create status templates CRUD interface
	☐	Create affected bookings review interface
	☐	Add status analytics widgets to dashboard
	☐	Create status change notification settings

Testing:
	☐	Write unit tests
	☐	Write integration tests
	☐	Write E2E tests
	☐	Perform manual testing
	☐	Test notification delivery
	☐	Test mobile responsiveness

Documentation:
	☐	Update API documentation
	☐	Create admin user guide untuk status management
	☐	Create user guide untuk understanding room status
	☐	Document status change workflow
	☐	Document notification system

14.15 Success Metrics

KPIs:
	•	User awareness: % users yang understand status indicators
	•	Booking errors: Reduce attempts to book unavailable rooms by 90%
	•	Admin efficiency: Average time to update room status < 30 seconds
	•	Transparency: 100% status changes logged dan visible
	•	User satisfaction: Positive feedback on status visibility

Monitoring:
	•	Track status change frequency per room
	•	Monitor average downtime per room
	•	Track user inquiries about unavailable rooms
	•	Monitor notification delivery success rate
	•	Track alternative room suggestion click-through rate

⸻

END OF DOCUMENT