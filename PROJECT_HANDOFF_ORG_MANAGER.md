# PROJECT HANDOFF — Universal Org Manager (Gym + Multi-Org SaaS)

**Date:** 2026-11-15  
**Owner intent:** Sell a universal system (not only gym) for verification, attendance, salary, fees/collection. One download → 10 min setup → working. Multi-tenant. Scale 1 user → lakhs records. Crash-safe backups. Recurring revenue.

---

## 1. PRODUCT VISION (FINAL)

### What it is
A **multi-tenant Organization Manager** for:
- Gyms
- Offices
- Schools
- Factories / coaching / any org that needs attendance + people + salary/fees

### What buyer does
1. Open app / link  
2. Sign up (org name, type, logo, owner)  
3. Set T&C, shifts/batches, UPI/payment info  
4. Add people (or import)  
5. Start attendance / fees / staff — **within ~10 minutes**

### Business model
- **Not** one-time source-code sale  
- **Hosted SaaS**: setup fee + **monthly/yearly renewal**  
- Plan expire → lock / read-only  
- Source code **never** given to client  
- Liability limited via Terms (best-effort uptime, client must export data)

### Unique vs biometric machines
- Face + QR + Quick Present + **VIP Card** (no queue for celebrities/special members)
- Staff salary + penalty + incentive in same app
- Full admission + medical + custom T&C + receipts
- Store / diet optional modules
- Automated WhatsApp + Email alerts

---

## 2. CURRENT LIVE APP (5 Brother’s / legacy)

### Working pieces (reference, do not throw away)
- **admission.html** — rich admission form (Firebase members), photo/ID, cash/online/bakaya, print card with stamp, under-18 WhatsApp buttons, admin desk concepts
- **pay.html** — fees payment, receipt, late payment mention, manual date ideas
- **attendance.html** (latest in artifacts) — search member, Face auto full-screen, Quick Present, live stats (Total / Present / Absent / Fees Bakaya), 1-day-1-entry
- **app.html** — main menu tiles for 5BFC
- Older **Gym Manager** (`index.html` / `gym-manager.html`) — multi-tenant attempt under `gyms/{uid}/…`, Auth email/password, members/att/fees/staff/store/report, Excel backup; **had loading/hang issues** on some deploys (Firebase project / rules / long-polling)

### Firebase projects seen in files
1. `brother-s-fitness-club` — used by admission/pay/attendance (5BFC)  
2. `gym-manager-app-76650` — used by newer Gym Manager index  

**Do not mix data between projects without migration plan.**

### Critical user complaints (must respect)
- Do **not** remove live dashboard: present / absent / total members / fees bakaya  
- Do **not** strip old attendance flows when adding Face — **ADD** Face, don’t replace everything  
- Face UX: search → select → **full screen** camera → **auto capture ~10 sec**  
- Match threshold needs to be practical (~0.68 worked better than 0.52 when score was 0.61)  
- Admission photo must be clear front face or face match fails  
- Multi-org sell requires isolation + backup from day one  

---

## 3. TARGET ARCHITECTURE

```
Firebase Auth (email/password per org owner)
└── orgs/{orgId}                    # orgId === auth.uid for owner (simple model)
      ├── profile: {
      │     name, type: 'gym'|'office'|'school'|'factory'|'other',
      │     logo, phone, addr, owner,
      │     termsHtml,           # editable T&C
      │     batches/shifts,
      │     upiId, branding colors,
      │     alertWhatsApp: true/false,
      │     alertEmail: true/false,
      │     alertPhone, alertEmailAddr,
      │     planExpires, status: 'active'|'readonly'|'blocked'
      │   }
      ├── people/{personId}       # members / employees / students
      ├── attendance/{id}         # date, personId, time, mode: face|qr|quick|card
      ├── fees/{id}               # optional for gym/school
      ├── staff/{id}
      ├── staffAttendance/{id}
      ├── products, sales, expenses  # optional
      └── settings
```

### Security rules (concept)
```
match /orgs/{orgId}/{document=**} {
  allow read, write: if request.auth != null && request.auth.uid == orgId;
}
```
Master admin needs separate privileged path (Cloud Functions or separate admin collection with custom claims) — do not leave master open.

### Hosting
- Prefer **Firebase Hosting** or Cloudflare Pages for production  
- GitHub = code backup only, not sole production host  
- PWA + later Play Store (TWA) for trust  

---

## 4. MODULE REQUIREMENTS (FULL LIST)

### A. Onboarding / Admission (type-aware)
**Gym template (from original 5BFC):**
- Full name, DOB, age, gender  
- Father name / Husband name (female + marital status)  
- Permanent + current address  
- Emergency / Parents contact (mandatory under 18)  
- Height, weight, **BMI auto**  
- Medical: existing disease, supplements, hospital history  
- Photo: Live camera + Gallery  
- ID front/back: Live + Gallery  
- Signature  
- Plan, fees, payment: Online / Cash now / Bakaya + promise date  
- **Per-org Terms & Conditions** (editable)  
- Submit → receipt PDF/image + WhatsApp notify + admin approve path  
- Stamp on print  

**Office/School templates:** different field sets, same engine.

### B. Live Home dashboard
- Aaj present · Total people · Absent  
- Expired memberships / expiring in 7 days  
- Aaj new admissions  
- Aaj fees/collection  
- Staff present + in/out  
- Store sales today (if enabled)  
- Batch/shift-wise counts  

### C. Attendance
- Modes: **Face** (full screen auto) · **QR/Code** · **Quick** · **VIP Card/code**  
- Rule: one successful mark per person per day  
- Live list of today’s marks  
- Optional: unmark (admin only)

### D. Automated alerts (NEW — required)
**WhatsApp**
- On Present mark → optional msg to member / parent (under 18)  
- Daily summary to owner (present count, absent count)  
- Expiry reminders (membership ending)  
- Fees bakaya reminder  

**Email**
- Same events to org alert email  
- Daily digest  

**Implementation note (India):**
- WhatsApp: wa.me deep links (manual) **or** WhatsApp Business API / Interakt / Wati / Twilio (paid, automated)  
- For v1: generate prefilled wa.me + optional Cloud Function later for true push  
- Email: EmailJS / Firebase Extension / SendGrid / Resend from Cloud Function  
- Store alert preferences on `orgs/{id}/profile`

### E. Fees + Receipts
- Collect fees, extend validity  
- Receipt with stamp, late note if applicable  
- Search previous receipt  

### F. Staff + Salary
- Profile, shift hours, monthly salary, weekly off  
- Daily P/A/W + late hours + incentive  
- Month net = (days × per-day) − penalty + incentive  
- Better than pure biometric device salary modules (transparent + export)

### G. Store (optional module)
- Products, stock, low alert, sell, daily profit  

### H. Diet (gym optional)
- Simple BMI/goal based chart assign  

### I. Member digital card
- After verification/approve → card with QR  
- Scan QR = attendance  
- Instant WhatsApp “Present marked” if alerts on  

### J. Report + Backup (CRITICAL)
- Daily report: admissions, fees, sales, expenses, net  
- **Excel export** all collections  
- **Auto backup** on first open of day (client-side download)  
- **Admin vault:** scheduled export to Cloud Storage (platform owner)  
- Client Terms: export regularly; platform keeps best-effort backup  
- On crash: restore from vault / client Excel  

---

## 5. FACE ATTENDANCE TECH NOTES

- Library: face-api.js (browser)  
- Models CDN: `@vladmandic/face-api` model path or face-api.js weights  
- Descriptor from admission photo vs live camera  
- Threshold: start **0.68** (0.52 was too strict when real score ~0.61)  
- UX: select person → full-screen overlay → timer 10s → auto try match at 2.5/5/7.5/10s  
- Fallback: Quick Present if no clear registered face  
- Requires HTTPS + camera permission  

---

## 6. SCALE + DATA SAFETY

| Risk | Mitigation |
|------|------------|
| Org A sees Org B data | Strict Firestore rules by orgId |
| Photos blow up DB | Firebase Storage paths `orgs/{id}/photos/…`; DB stores URL only |
| Lakhs attendance rows | Query by date + personId indexes; archive old years |
| GitHub/hosting down | Production on Firebase/Cloudflare; code on GitHub private |
| Platform crash | Nightly backup to admin Storage + client Excel |
| Client sues on downtime | Terms: limited liability = last 3 months fees; data export duty on client |
| Copy/AI clone | No source to client; license check on server; logic in Functions where possible |

---

## 7. BUILD PHASES

### Phase 0 — Foundation
- Multi-tenant Auth + orgs/{uid} profile  
- Org type selector  
- Editable T&C  
- Backup Excel button + daily auto client backup  
- Fix any loading hang (timeouts, rules, long-polling)

### Phase 1 — Daily ops
- People list + gym-style admission form (full fields)  
- Attendance Quick + Face + live dashboard numbers  
- Fees bakaya list on attendance home  
- WhatsApp prefilled alerts (v1) + email digest hook  

### Phase 2
- Staff salary  
- Receipts + stamp  
- VIP card mode  
- Admin approve flow for admission  

### Phase 3
- Store, Diet, Play Store PWA, true WhatsApp API, lakhs tuning  

---

## 8. ALERTS SPEC (implement later but design now)

```text
Event: attendance.marked
  → if profile.alertWhatsApp: queue WhatsApp to person/parent
  → if profile.alertEmail: send email to org alertEmailAddr

Event: daily.cron (or first open after 8pm)
  → owner summary: present, absent, new joins, collection

Event: membership.expiring_in_3_days
  → member + owner reminder

Event: fees.overdue
  → reminder WhatsApp/Email
```

v1 without paid API: buttons “Send WA summary” that open `wa.me` with text.  
v2: Cloud Functions + Business API.

---

## 9. FILES OF RECORD (paths)

| File | Role |
|------|------|
| /home/workdir/artifacts/admission.html | Full admission (5BFC) |
| /home/workdir/artifacts/pay.html | Fees / receipt |
| /home/workdir/artifacts/attendance.html | Face + Quick + live stats |
| /home/workdir/attachments/index.html | Gym Manager multi-tenant (loading issues) |
| /home/workdir/attachments/gym-manager.html | Earlier multi-tenant single file |
| /home/workdir/attachments/firestore_3.rules | org isolation rules sample |
| /home/workdir/attachments/app.html | 5BFC menu |

---

## 10. NON-NEGOTIABLES (owner words)

1. Multiple organizations — design multi-tenant from start  
2. Admission richness like original (medical, BMI, gender-aware parents, ID+photo dual upload, T&C per gym)  
3. Live attendance board never removed  
4. Face only **added**, not replacing Quick/live  
5. Automated WhatsApp + Email alerts  
6. Backup always ready for owner + admin  
7. Universal org types, not gym-only branding in core  
8. VIP/card path for people who won’t stand for face  
9. Recurring subscription, not one-time code dump  
10. 10-minute onboarding after download  

---

## 11. IMMEDIATE NEXT STEP FOR NEXT AI

1. Confirm Firebase project to use (new clean project recommended for SaaS).  
2. Implement Phase 0: Auth + `orgs/{uid}` + profile + Excel backup.  
3. Port admission fields into tenant-scoped `people`.  
4. Port attendance live stats + Quick + Face under `orgs/{uid}/attendance`.  
5. Add alert settings fields + wa.me summary buttons.  
6. Do not delete 5BFC single-gym features until tenant version feature-parity on dashboard + admission + attendance.

---

## 12. ONE-LINE SUMMARY

**Multi-tenant Org OS (attendance, people, salary/fees, alerts, backup) sold as hosted subscription; gym is the first template; Face+QR+VIP+Quick; data isolated and backed up; no source code to clients.**
