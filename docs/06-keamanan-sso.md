# Security & Single Sign-On (SSO)

## SSO Architecture

SMART Absen SMA UII uses a **self-managed Single Sign-On (SSO)** approach built on top of Laravel. This system is designed as the first **Identity Provider (IdP)** in the foundation environment, which will later become the authentication standard for all SMA UII applications.

### Identity Provider Concept

```
┌─────────────────────────────────────────────────────────┐
│                   Identity Provider                      │
│                      (SMART Absen)                       │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  User Store   │  │  Token       │  │  Role        │  │
│  │  (users tbl)  │  │  Management  │  │  Resolver    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            SSO Session (Sanctum)                   │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────┘
                       │
         ┌──────────────┼──────────────┐
         │              │              │
         ▼              ▼              ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ e-Learning │ │ Financial  │ │ Library    │
│ App        │ │ App        │ │ App        │
│ (future)   │ │ (future)   │ │ (future)   │
└────────────┘ └────────────┘ └────────────┘
```

### Technology Used

**Laravel Sanctum + Spatie Laravel Permission**

| Reason | Detail |
|---|---|
| **Simple** | Database-based API token, suitable for SPA and mobile |
| **Stateful** | Session-based auth for Inertia pages |
| **Stateless** | API token for third-party applications in the future |
| **Built-in** | Sanctum and Spatie are natively integrated with Laravel |
| **RBAC** | Spatie provides role, permission, team, and blade directives |

---

## Role-Based Access Control (RBAC)

### Role Hierarchy

```
┌─────────────────────────────────────────┐
│              SUPER ADMIN                 │
│  (Full system access + bypass)          │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│                 ADMIN                    │
│  (CRUD master data, settings, recap)    │
└─────────────────────────────────────────┘
                    │
         ┌───────────┴───────────┐
         │                       │
         ▼                       ▼
┌──────────────┐    ┌──────────────────────┐
│  Homeroom    │    │    Duty Teacher      │
│  Teacher     │    │ (real-time monitor)  │
│ (verify      │    └──────────────────────┘
│  leave,recap)│
└──────┬───────┘
       │
       ▼ (no hierarchy, separate roles)
┌──────────────┐    ┌──────────────────────┐
│   Student    │    │     Guardian         │
│ (attendance) │    │ (monitor child +    │
│              │    │  submit leave)       │
└──────────────┘    └──────────────────────┘
```

### Implementation with Spatie Laravel Permission

```bash
# Installation
bun add composer spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

```php
// Role & permission definition in seeder / Service Provider
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

// Create permissions
Permission::create(['name' => 'create-attendance']);
Permission::create(['name' => 'view-reports']);
Permission::create(['name' => 'manage-user']);
Permission::create(['name' => 'manage-class']);
Permission::create(['name' => 'approve-leave']);

// Create roles + assign permissions
$superAdmin = Role::create(['name' => 'super-admin']); // all permissions via gate
$admin = Role::create(['name' => 'admin']);
$admin->givePermissionTo(['create-attendance', 'view-reports', 'manage-user', 'manage-class', 'approve-leave']);

$teacher = Role::create(['name' => 'teacher']);
$teacher->givePermissionTo(['create-attendance', 'view-reports', 'approve-leave']);

$student = Role::create(['name' => 'student']);
$student->givePermissionTo(['create-attendance']);

$guardian = Role::create(['name' => 'guardian']);
$guardian->givePermissionTo(['view-reports', 'approve-leave']);
```

```php
// Usage in Routes
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum', 'role:admin|super-admin'])->group(function () {
    Route::resource('users', UserController::class);
});

Route::middleware(['auth:sanctum', 'permission:create-attendance'])->group(function () {
    Route::post('/attendance', [AttendanceController::class, 'store']);
});
```

```php
// Usage in Blade/Inertia
@role('admin')
    <!-- display admin panel -->
@endrole

@can('view-reports')
    <!-- display report button -->
@endcan
```

---

## Triple-Layer Attendance Validation

Before the system saves attendance data, the server MUST perform three layers of validation:

### Layer 1: Academic Calendar Validation

```php
// Check if today's date exists in the academic_calendars table
$isHoliday = AcademicCalendar::where('date', today())
    ->where('is_holiday', true)
    ->exists();

if ($isHoliday) {
    return response()->json([
        'status' => 'rejected',
        'message' => 'Today is an academic holiday'
    ], 403);
}
```

### Layer 2: Active Day Validation

```php
// Check if today is an active day
$hariIni = today()->format('l'); // Monday, Tuesday, etc.

$setting = AttendanceTimeSetting::where('day', $hariIni)->first();

if (! $setting) {
    return response()->json([
        'status' => 'rejected',
        'message' => 'No attendance schedule for today'
    ], 403);
}
```

### Layer 3: Time Range Validation

```php
$currentTime = now()->format('H:i:s');

if ($currentTime < $setting->open_time) {
    // Not yet time for attendance
    return response()->json(['status' => 'too_early'], 403);
}

if ($currentTime <= $setting->late_threshold) {
    $status = 'Present';
} elseif ($currentTime <= $setting->close_time) {
    $status = 'Late';
} else {
    // Attendance closed
    return response()->json(['status' => 'closed'], 403);
}
```

### Validation Diagram

```
[Attendance Request]
        │
        ▼
┌───────────────────┐   NO    ┌──────────────────┐
│  Check Academic   │─────────│  Reject: "Acade- │
│  Calendar         │         │  mic Holiday"    │
└────────┬──────────┘         └──────────────────┘
         │ YES
         ▼
┌───────────────────┐   NO    ┌──────────────────┐
│  Check Active Day │─────────│  Reject: "No     │
│  (exists in      │         │  Schedule"        │
│  settings?)      │         └──────────────────┘
└────────┬──────────┘
         │ YES
         ▼
┌──────────────────────────────────────────────────┐
│              Check Time Range                     │
├──────────┬──────────┬──────────┬──────────────────┤
│ < Open   │ Open to  │ Late to  │ > Close          │
│          │ Late     │ Close    │                   │
├──────────┼──────────┼──────────┼──────────────────┤
│ "Too     │ "Present"│ "Late"   │ "Attendance      │
│  Early"  │          │          │ Closed"          │
└──────────┴──────────┴──────────┴──────────────────┘
```

---

## Other Security Principles

### 1. Dynamic Configuration (Anti-Hardcode)

```
❌ PROHIBITED:
if ($currentHour < 6 && $currentHour > 8) { ... }

✅ REQUIRED:
$setting = AttendanceTimeSetting::where('day', today()->dayName)->first();
if ($currentTime < $setting->open_time) { ... }
```

### 2. Tokenization & Session

| Aspect | Method |
|---|---|
| Web (Browser) | Session-based via Laravel cookie |
| API (Mobile future) | Sanctum token |
| CSRF Protection | Laravel built-in CSRF token |
| Password hashing | Bcrypt (12 rounds) |

### 3. URL & Data Protection

- All routes wrapped in `auth:sanctum` + `role:xxx` middleware
- Policy/Guard for specific operations (e.g., students can only attend for themselves)
- Server-side validation for all inputs
- Rate limiting on attendance endpoints to prevent spam

### 4. Object Storage Security

- Files uploaded directly to Object Storage (not via Laravel server)
- File URLs are signed/temporary (expire after a certain time)
- File type validation on server before saving URL

### Web-Based Advantages & Disadvantages

| Aspect | Advantage | Disadvantage |
|---|---|---|
| Installation | No install, just browser | — |
| Update | Instant, no download | — |
| Offline | — | Cannot attend without internet |
| Device permissions | — | Inexperienced users may block camera/location |
