# TODO: fastdamagerepairs-site Task List

This document tracks all pending tasks for the lead generation system, organized by priority and phase.

---

## PHASE 1: VALIDATION (REQUIRED - Do This First)

**⚠️ These tests must pass before deploying to production or proceeding to Phase 2.**

### Test A: Short Call (Unqualified)

**Time**: 10 minutes  
**Objective**: Verify unqualified calls are logged correctly

- [ ] **Execute Test**:
  - Call +1 (813) 498-3847
  - Press 1 when prompted
  - Stay on line for ~20 seconds
  - Hang up

- [ ] **Expected Outcome**:
  - New row appears in Airtable Leads table
  - Status field = `unqualified`
  - Duration < 60 seconds
  - Recording URL present
  - No Stripe invoice item created

- [ ] **If Test Fails**: See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) → "Short call didn't create row"

---

### Test B: Long Call (Qualified)

**Time**: 10 minutes  
**Objective**: Verify qualified calls are logged, marked qualified, and billed

- [ ] **Execute Test**:
  - Call +1 (813) 498-3847 from an 813 or 727 area code phone
  - Press 1 when prompted
  - Stay on line for at least 75 seconds
  - Hang up

- [ ] **Expected Outcome**:
  - New row appears in Airtable Leads table
  - Status field = `qualified`
  - Duration Sec ≥ 60
  - Recording URL filled in
  - Caller Number shows 813 or 727 area code
  - Stripe invoice item created on test customer

- [ ] **If Test Fails**: See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) → "Long call not marked qualified"

---

## PHASE 2: HARDENING (STRONGLY RECOMMENDED)

**⚠️ These safeguards prevent double-billing and ensure system reliability.**

### Task 1: Idempotency Guard (Prevent Double-Billing)

**Time**: 15 minutes  
**Priority**: HIGH  
**Issue**: [#2](https://github.com/wsweatt0216/fastdamagerepairs-site/issues/2) *(if tracked)*

- [ ] **Problem**: Make webhook retries could create duplicate Stripe invoice items for the same call

- [ ] **Solution**:
  - In Make scenario, before the Stripe module in the Qualified sub-branch
  - Add filter: Check if Airtable Status field != `qualified`
  - Only proceed to Stripe if Status is NOT already qualified
  - This ensures we only bill once per call

- [ ] **Implementation**:
  - 1 Make filter module
  - Filter condition: `Status` field does not equal `qualified`

- [ ] **Test**: Call twice with same Call SID simulation (or trigger webhook twice) and verify only one Stripe item is created

---

### Task 2: 7-Day Duplicate Caller Guard

**Time**: 15 minutes  
**Priority**: MEDIUM  
**Issue**: [#3](https://github.com/wsweatt0216/fastdamagerepairs-site/issues/3) *(if tracked)*

- [ ] **Problem**: Same caller calling multiple times within 7 days should only count first call as qualified

- [ ] **Solution**:
  - Before marking a call as qualified, search Airtable for existing records
  - Search criteria: Same Caller Number within last 7 days
  - If found: Mark current call as `unqualified` and skip Stripe billing
  - If not found: Proceed with normal qualification logic

- [ ] **Implementation**:
  - 1 Airtable Search module (search by Caller Number, date range)
  - 1 Router/Filter based on search results
  - Update logic to set Status = `unqualified` if duplicate found

- [ ] **Test**: Call twice from same number within minutes, verify second call is marked unqualified

---

### Task 3: Weekly Invoicer Scenario (Auto-Finalize & Send)

**Time**: 30 minutes  
**Priority**: MEDIUM  
**Issue**: [#4](https://github.com/wsweatt0216/fastdamagerepairs-site/issues/4) *(if tracked)*

- [ ] **Problem**: Qualified calls create invoice items, but invoices must be manually finalized and sent

- [ ] **Solution**:
  - Create new Make scenario: "Weekly Invoice Finalization"
  - Schedule: Every Monday at 9:00 AM ET
  - Actions:
    1. Stripe: List all draft invoices
    2. For each invoice: Finalize invoice
    3. For each finalized invoice: Send via email
    4. On failure: Send error notification email

- [ ] **Implementation**:
  - New Make scenario (copy/paste from template if available)
  - Stripe modules: List Invoices (filter: status=draft), Finalize Invoice, Send Invoice
  - Error handler: Email notification module

- [ ] **Test**: Create test draft invoice in Stripe, trigger scenario manually, verify invoice is finalized and sent

---

## PHASE 3: BUYERS (NEXT SPRINT)

**⚠️ Required for scaling beyond single buyer. Currently phone number is hardcoded.**

### Task 1: /partners Page + Intake Form

**Time**: 1 hour  
**Priority**: MEDIUM  
**Issue**: [#5](https://github.com/wsweatt0216/fastdamagerepairs-site/issues/5) *(if tracked)*

- [ ] **Problem**: No way for buyers to sign up and specify service areas

- [ ] **Solution**:
  - Create `/partners` page (or `/partners.html`)
  - Add intake form with fields:
    - Company Name
    - Contact Name
    - Phone Number
    - Email
    - Service Areas (checkboxes: Tampa, St. Pete, Clearwater, Raleigh, Durham, Chapel Hill)
    - License Number
    - Insurance Info
  - Form submits to Make webhook
  - Make creates row in Airtable "Buyers" table

- [ ] **Implementation**:
  - HTML page with form (can use Tally embed or custom HTML)
  - Make webhook to receive form data
  - Airtable "Buyers" table creation with fields
  - Link from main page (optional)

- [ ] **Test**: Submit form, verify row appears in Airtable Buyers table

---

### Task 2: Auto-Routing (Twilio Function or Make)

**Time**: 1.5 hours  
**Priority**: HIGH (for multi-buyer scaling)

- [ ] **Problem**: Phone number to dial is currently hardcoded in TwiML. Need dynamic routing based on:
  - Caller's area code
  - Buyer availability
  - Service area coverage
  - Round-robin or priority logic

- [ ] **Solution Option A - Twilio Function**:
  - Create Twilio Function that queries Airtable Buyers table
  - Filter buyers by service area matching caller's area code
  - Return phone number of available buyer (or first in rotation)
  - Update WaterDamage_Dial TwiML to call this function

- [ ] **Solution Option B - Make Backend API**:
  - Create Make scenario with webhook endpoint
  - Twilio TwiML calls this webhook with caller info
  - Make queries Airtable and returns TwiML with <Dial> element
  - More flexible but adds latency

- [ ] **Implementation**:
  - Recommended: Twilio Function (lower latency)
  - Airtable Buyers table with fields: Phone, Service Areas, Status (active/inactive)
  - Function logic: Query, filter, select
  - Update TwiML bin to reference function

- [ ] **Test**: Call from different area codes, verify routed to correct buyer

---

## Backlog / Future Enhancements

Lower priority items for future consideration:

- [ ] **Call quality scoring**: Track buyer pickup rate, customer satisfaction
- [ ] **Buyer dashboard**: Self-service portal to view call logs and invoices
- [ ] **Multi-language IVR**: Spanish language support
- [ ] **SMS notifications**: Text buyer when call is incoming
- [ ] **Voicemail handling**: Route to voicemail if no buyer available
- [ ] **Analytics dashboard**: Call volume, conversion rates, revenue tracking
- [ ] **Geographic expansion**: Add more service areas beyond FL and NC

---

## Notes

- All tasks should be tested in Stripe test mode before production
- Document any changes to Make scenarios with screenshots/notes
- Keep Airtable schema documented if new fields are added
- Review and update this TODO.md as tasks are completed
