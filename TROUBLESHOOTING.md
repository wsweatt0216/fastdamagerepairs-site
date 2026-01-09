# Troubleshooting Guide

This guide helps debug common issues with the lead generation system.

---

## Table of Contents

- [Call Issues](#call-issues)
  - [Short call didn't create row](#short-call-didnt-create-row)
  - [Long call not marked qualified](#long-call-not-marked-qualified)
  - [No recording URL](#no-recording-url)
- [Make/Webhook Issues](#makewebhook-issues)
  - [Make scenario not triggering](#make-scenario-not-triggering)
  - [Duplicate entries in Airtable](#duplicate-entries-in-airtable)
- [Airtable Issues](#airtable-issues)
  - [Data not appearing](#data-not-appearing)
  - [Wrong field values](#wrong-field-values)
- [Stripe Issues](#stripe-issues)
  - [No invoice item created](#no-invoice-item-created)
  - [Double billing](#double-billing)
- [Twilio Issues](#twilio-issues)
  - [Call not connecting](#call-not-connecting)
  - [IVR not working](#ivr-not-working)

---

## Call Issues

### Short call didn't create row

**Symptom**: You made a test call under 60 seconds, but no row appeared in Airtable.

**Debugging Steps**:

1. **Check Twilio Call Logs**:
   - Go to Twilio Console → Monitor → Logs → Calls
   - Find your call by time or caller number
   - Verify call status is "completed"
   - Check if webhooks were sent (look for webhook delivery logs)

2. **Check Make Execution History**:
   - Go to Make dashboard → Scenarios
   - Click on your scenario → History
   - Look for execution around the time of your call
   - Check if "Completed" branch was triggered
   - Review data that was received from Twilio

3. **Verify Make Webhook URL**:
   - In Twilio Console → Phone Numbers → +1 (813) 498-3847
   - Check "Voice & Fax" section
   - Verify webhook URLs are pointing to your Make scenario
   - Ensure StatusCallback webhook is configured

4. **Check Make → Airtable Connection**:
   - In Make scenario, find the Airtable module
   - Verify connection is active (green checkmark)
   - Test connection manually
   - Check field mappings are correct

**Common Fixes**:
- Reconnect Airtable module if connection expired
- Verify Airtable base ID and table name are correct
- Check that all required fields are mapped
- Ensure Make scenario is active (not paused)

---

### Long call not marked qualified

**Symptom**: Call was >60 seconds from correct area code, but Status is `unqualified` or no Stripe invoice created.

**Debugging Steps**:

1. **Check Airtable Record**:
   - Find the call record in Airtable
   - Verify Duration field shows ≥60 seconds
   - Check Caller Number field has correct area code (813 or 727)
   - Look at Status field value

2. **Review Make Qualification Logic**:
   - Go to Make scenario → Completed branch
   - Find the filter/router that checks qualification criteria
   - Verify filter conditions:
     - Duration ≥ 60
     - Area code is 813 OR 727
   - Check if filter is using correct operators (AND/OR)

3. **Check Make Execution for This Call**:
   - Find the execution in Make history
   - Click to expand and see each module's output
   - Look at the qualification filter/router
   - See which path was taken and why

4. **Verify Stripe Module**:
   - In Make scenario, find Stripe "Create Invoice Item" module
   - Check if it was executed (in history)
   - Verify Stripe connection is active
   - Check customer ID is correct

**Common Fixes**:
- Update filter logic if conditions are wrong
- Fix field mapping (e.g., duration might be string instead of number)
- Add type conversion if needed (e.g., parse duration to integer)
- Reconnect Stripe module if connection expired
- Verify Stripe is in test mode if doing test calls

---

### No recording URL

**Symptom**: Call completed but Recording URL field is empty in Airtable.

**Debugging Steps**:

1. **Check Twilio Recording Settings**:
   - Go to Twilio Console → Phone Numbers → +1 (813) 498-3847
   - Verify "Record calls" is enabled
   - Check recording format and channels

2. **Check Make Recording Branch**:
   - Go to Make scenario → Recording branch
   - Verify this branch has a webhook trigger for recording events
   - Check if it updates the Airtable record with Recording URL

3. **Verify Recording Webhook**:
   - In Twilio Console, check if recording webhook is configured
   - Webhook URL should point to Make scenario (Recording branch)
   - Verify webhook is being called (check Twilio logs)

4. **Check Airtable Update**:
   - In Make Recording branch, verify Airtable "Update Record" module
   - Ensure it's searching for the correct record (by Call SID)
   - Verify Recording URL field mapping is correct

**Common Fixes**:
- Enable call recording in Twilio phone number settings
- Configure recording webhook to trigger Make scenario
- Update Airtable module to map recordingUrl correctly
- Check for timing issues (recording may process after call ends)

---

## Make/Webhook Issues

### Make scenario not triggering

**Symptom**: Webhooks are being sent from Twilio, but Make scenario isn't running.

**Debugging Steps**:

1. **Verify Scenario is Active**:
   - Go to Make dashboard → Scenarios
   - Check if scenario has green "ON" indicator
   - If off, click to activate

2. **Check Webhook URL**:
   - Copy webhook URL from Make scenario (webhook module)
   - Compare with URL in Twilio configuration
   - Ensure they match exactly (no extra spaces or characters)

3. **Test Webhook Manually**:
   - In Make, right-click webhook module → "Run this module only"
   - Use a tool like Postman or curl to send test data
   - Verify Make receives and processes it

4. **Check Twilio Webhook Logs**:
   - Go to Twilio Console → Monitor → Logs → Webhooks
   - Find recent webhook attempts
   - Check response codes (should be 200)
   - Look for error messages

**Common Fixes**:
- Activate scenario if it's paused
- Update webhook URL in Twilio if it changed
- Check Make account for quota/limit issues
- Verify webhook module is set to accept POST requests

---

### Duplicate entries in Airtable

**Symptom**: Same call creates multiple rows in Airtable.

**Debugging Steps**:

1. **Check for Multiple Webhook Calls**:
   - Review Twilio webhook logs
   - See if webhook is being called multiple times for one call
   - Check for retry attempts (status != 200)

2. **Review Make Execution History**:
   - Check if scenario ran multiple times for same Call SID
   - Look at timestamps to see if they're retries or separate triggers

3. **Check Idempotency Guard**:
   - See if Task 1 in Phase 2 of TODO.md was implemented
   - Verify filter checks if record already exists

**Common Fixes**:
- Implement idempotency guard (see TODO.md Phase 2, Task 1)
- Add Make filter to check if Call SID already exists in Airtable
- Ensure webhook responds with 200 status quickly
- Add unique constraint on Call SID in Airtable (if supported)

---

## Airtable Issues

### Data not appearing

**Symptom**: Make says it created/updated record, but nothing shows in Airtable.

**Debugging Steps**:

1. **Check Airtable Base & Table**:
   - Verify you're looking at correct base
   - Verify you're in the correct table (Leads)
   - Check if any filters are applied to the view

2. **Check Make Airtable Module Output**:
   - In Make execution history, expand Airtable module
   - Look at output to see record ID that was created
   - Copy record ID and search in Airtable

3. **Verify Field Names Match**:
   - In Make, check field mappings
   - Compare with actual field names in Airtable (case-sensitive)
   - Ensure field types match (e.g., number field getting number, not string)

**Common Fixes**:
- Remove filters from Airtable view
- Update Make field mappings to match exact field names
- Create missing fields in Airtable if needed
- Check Airtable permissions (Make needs write access)

---

### Wrong field values

**Symptom**: Data appears in Airtable but values are incorrect or formatted wrong.

**Debugging Steps**:

1. **Check Make Field Mappings**:
   - Review which Twilio fields are mapped to which Airtable fields
   - Verify the correct data is being pulled from Twilio

2. **Check Data Types**:
   - Ensure numeric fields receive numbers, not strings
   - Check date/time formatting matches Airtable expectations
   - Verify phone number formatting

3. **Review Make Transformation**:
   - Check if any formulas or functions are applied
   - Verify they're producing expected results

**Common Fixes**:
- Use Make's built-in functions to convert data types
- Add formatting functions for phone numbers or dates
- Update formulas if calculation is incorrect
- Map to correct source fields from Twilio webhook data

---

## Stripe Issues

### No invoice item created

**Symptom**: Call was qualified, but no invoice item shows in Stripe.

**Debugging Steps**:

1. **Check Make Execution**:
   - Find execution in Make history
   - Verify "Qualified" path was taken
   - Check if Stripe module was executed
   - Look for error messages in Stripe module output

2. **Verify Stripe Connection**:
   - In Make Stripe module, check connection status
   - Test connection
   - Verify API key has correct permissions

3. **Check Customer ID**:
   - Verify Stripe customer ID is correct
   - Ensure customer exists in Stripe (same mode: test/live)

4. **Review Stripe Logs**:
   - Go to Stripe Dashboard → Developers → Logs
   - Look for API request around time of call
   - Check response code and error messages

**Common Fixes**:
- Reconnect Stripe module if connection expired
- Update Stripe API key if it changed
- Verify using test mode for test calls
- Check customer ID is valid and in correct mode
- Ensure invoice item parameters are correct (amount, description, etc.)

---

### Double billing

**Symptom**: Same call creates multiple invoice items in Stripe.

**Debugging Steps**:

1. **Check Airtable for Duplicate Records**:
   - Search by Call SID
   - See if multiple records exist

2. **Review Make Execution History**:
   - Check if scenario ran multiple times for same call
   - Look for retry attempts

3. **Check Idempotency Implementation**:
   - Verify if Task 1 (Phase 2) was implemented
   - Check filter before Stripe module

**Common Fixes**:
- Implement idempotency guard (TODO.md Phase 2, Task 1)
- Add filter to check if Status is already "qualified" before billing
- Use Stripe's idempotency key feature
- Manually void/delete duplicate invoice items in Stripe

---

## Twilio Issues

### Call not connecting

**Symptom**: Caller hears ringing but call never connects, or goes to error message.

**Debugging Steps**:

1. **Check Twilio Call Logs**:
   - Go to Twilio Console → Monitor → Logs → Calls
   - Find the call
   - Look at call flow and status
   - Check for error codes

2. **Verify TwiML Bin Configuration**:
   - Go to Twilio Console → TwiML Bins
   - Open WaterDamage_Dial bin
   - Verify phone number to dial is correct and formatted properly
   - Check for TwiML syntax errors

3. **Test Buyer Phone Number**:
   - Call the buyer's number directly
   - Verify it's working and can receive calls
   - Check if number is in correct format (+1XXXXXXXXXX)

4. **Check Phone Number Configuration**:
   - Go to Twilio Phone Numbers → +1 (813) 498-3847
   - Verify "Voice & Fax" webhook points to correct TwiML bin
   - Check if number is active and not suspended

**Common Fixes**:
- Update buyer phone number in TwiML
- Fix phone number format (must include country code)
- Update webhook URL if TwiML bin changed
- Verify TwiML bin is public/accessible
- Check Twilio account balance (low balance can cause issues)

---

### IVR not working

**Symptom**: Caller doesn't hear menu options, or pressing numbers doesn't work.

**Debugging Steps**:

1. **Check TwiML Bin**:
   - Open WaterDamage_IVR bin
   - Verify `<Say>` or `<Play>` elements have correct text/audio
   - Verify `<Gather>` element is configured to capture digits

2. **Test IVR Flow**:
   - Call the number yourself
   - Listen to prompts
   - Try pressing different numbers
   - Note where it fails

3. **Check Twilio Call Logs**:
   - Look at call flow diagram
   - See which TwiML bins were executed
   - Check for errors or unexpected redirects

4. **Verify Gather Action**:
   - In `<Gather>` element, check `action` attribute
   - Ensure it points to correct next step (e.g., WaterDamage_Dial)

**Common Fixes**:
- Update TwiML with correct prompts and options
- Fix `<Gather>` action URL
- Ensure numDigits or timeout settings are appropriate
- Test TwiML in Twilio's TwiML bin editor
- Check audio file URLs if using `<Play>` (must be public and accessible)

---

## General Debugging Tips

1. **Always check logs in this order**:
   - Twilio Console → Call logs & webhook logs
   - Make → Execution history
   - Airtable → Recent activity
   - Stripe → API logs

2. **Use test mode for everything** until production ready:
   - Stripe test mode
   - Test phone numbers if available
   - Make scenarios in draft/test

3. **Document your changes**:
   - Keep notes of what you changed
   - Take screenshots of configurations
   - Update this troubleshooting guide with new issues

4. **Start simple**:
   - Test one component at a time
   - Use manual triggers in Make to isolate issues
   - Create test data to verify each integration

---

## Getting Help

If you've tried the steps above and still have issues:

1. **Gather Information**:
   - Call SID from Twilio
   - Make execution ID
   - Airtable record ID (if exists)
   - Screenshots of errors
   - Timestamp of issue

2. **Check Documentation**:
   - [Twilio Docs](https://www.twilio.com/docs)
   - [Make Help Center](https://www.make.com/en/help)
   - [Airtable Support](https://support.airtable.com/)
   - [Stripe Docs](https://stripe.com/docs)

3. **Contact Support**:
   - Twilio Support (for call/webhook issues)
   - Make Support (for scenario issues)
   - Project maintainer (for system-specific questions)
