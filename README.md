# Sunrise Dental Clinic Backend

Spring Boot REST backend for patient registration, appointment management, treatment records and billing. It uses a file-backed H2 database and starts on `http://localhost:8080`.

## Run

```powershell
.\gradlew.bat bootRun
```

Run the test suite with `./gradlew.bat test`. The H2 browser console is available at `http://localhost:8080/h2-console`; its JDBC URL is configured in `src/main/resources/application.properties`.

All protected endpoints expect:

```http
Authorization: Bearer <accessToken>
```

Tokens are issued by the login endpoint, stored in H2, expire after eight hours, and are revoked at logout.

## Seeded development accounts

| Portal / role | Username | Password |
| --- | --- | --- |
| Main admin | `mainadmin` | `Main@123` |
| Super admin | `superadmin` | `Super@123` |
| Doctor | `doctor1` | `Doctor@123` |
| Doctor staff | `staff1` | `Staff@123` |
| Patient | `patient1` | `Patient@123` |

Change these credentials before any non-demo use.

## API endpoints

### Authentication and accounts

| Method | Path | Access / purpose |
| --- | --- | --- |
| POST | `/api/auth/login` | Login with any valid account |
| POST | `/api/auth/login/{portalRole}` | Role-specific login; use `main-admin`, `super-admin`, `doctor`, `doctor-staff`, or `patient` |
| GET | `/api/auth/me` | Current logged-in user |
| POST | `/api/auth/logout` | Revoke current bearer token |
| POST | `/api/patients/register` | Public patient registration and patient-account creation |
| GET, POST | `/api/users` | Admin user listing/creation (main or super admin) |
| GET | `/api/users/doctors` | Active doctors |

### Patients, treatments and appointments

| Method | Path | Access / purpose |
| --- | --- | --- |
| GET | `/api/patients`, `/api/patients/{id}` | Clinic staff; patients can only read their own profile |
| GET | `/api/patients/me` | Patient's own profile |
| PUT | `/api/patients/{id}` | Clinic staff or that patient |
| GET | `/api/treatments` | Active treatment catalogue |
| GET, POST, PUT | `/api/treatments/all`, `/api/treatments`, `/api/treatments/{id}` | Treatment administration (main/super admin) |
| GET | `/api/appointments/doctors` | Active doctors available for booking |
| POST | `/api/appointments/online` | Patient-only online booking |
| POST | `/api/appointments/physical` | Clinic-staff physical/walk-in booking |
| GET | `/api/appointments/mine` | Appointments visible to current role |
| GET | `/api/appointments/{appointmentNumber}` | Appointment-number search with ownership checks |
| PATCH | `/api/appointments/{appointmentNumber}/status` | Staff/assigned doctor; patient may cancel their own |
| POST, GET | `/api/appointments/{appointmentNumber}/treatment-records` | Assigned doctor adds clinical record; authorised users view it |

The booking request includes `doctorId`, `treatmentCode`, `appointmentDateTime`, optional `notes`, and `patientId` for physical bookings. Slots are limited to every 30 minutes from 08:00 through 17:30, and each dentist/time pair is unique.

### Bills and payments

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/bills/appointment/{appointmentNumber}` | Calculate/create a bill from treatment cost + LKR 1,500 consultation fee |
| GET | `/api/bills/{billNumber}` | Read bill |
| GET | `/api/bills/{billNumber}/receipt` | Printable receipt data |
| POST | `/api/bills/{billNumber}/dummy-card-payment` | Luhn-validated dummy card payment for online booking; stores only final four digits |
| POST | `/api/bills/{billNumber}/cash-payment` | Clinic staff records cash for physical booking |

### Other

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/dashboard` | Role-aware dashboard summary |
| GET | `/api/help` | Step-by-step in-app help text |

## Error behaviour

Every validation, not-found, access-control, conflict, or unexpected failure returns a consistent JSON response with `timestamp`, `status`, `message`, and (for validation) field-level `details`. Typical protected-path outcomes are `401` for missing/expired login, `403` for a wrong role/ownership, and `409` for a duplicate doctor slot or duplicate payment.
