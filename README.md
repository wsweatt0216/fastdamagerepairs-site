# fastdamagerepairs-site

## Project Mission

Lead generation platform for water damage mitigation companies. Incoming calls → IVR menu → routed to buyers → recorded & logged → qualified calls billed via Stripe.

## System Architecture

```
fastdamagerepairs.com → Vercel landing page
             ↓
+1 (813) 498-3847 → Twilio IVR (WaterDamage_IVR)
             ↓
Make webhook router (Started/Completed/Recording branches)
             ↓
Airtable (PPL Water Damage CRM) + Stripe (invoice on qualified)
```

## Current Status

### What's Working ✅

- ✅ **Domain**: fastdamagerepairs.com → Cloudflare registrar, DNS to Vercel
- ✅ **Landing page**: Deployed on Vercel
- ✅ **Twilio account**: Phone number +1 (813) 498-3847 (Tampa)
- ✅ **TwiML bins created**:
  - WaterDamage_IVR
  - WaterDamage_Dial
  - WaterDamage_HOLD
  - WaterDamage_WHISPER
- ✅ **Make scenario**: 3-branch router framework
- ✅ **Airtable base**: PPL Water Damage CRM with Leads table schema
- ✅ **Stripe**: Test customer ready

### What's Pending ⏳

- ⏳ **Validate Make filters/mappings** + run 2 test calls

## Key Components & Configurations

### Twilio

**Phone Number**: +1 (813) 498-3847 (Tampa area code)

**TwiML Bins**:
- **WaterDamage_IVR**: Main IVR menu that greets callers and provides options
- **WaterDamage_Dial**: Connects caller to available buyer
- **WaterDamage_HOLD**: Hold music/message while connecting
- **WaterDamage_WHISPER**: Whisper message to buyer before connecting

**IVR Flow**:
1. Caller dials +1 (813) 498-3847
2. IVR greeting plays with menu options
3. Caller presses 1 to connect with a mitigation specialist
4. System routes to available buyer
5. Call is recorded and logged

### Make (Integromat)

**Webhook URL**: Receives events from Twilio

**3-Branch Router**:
- **Started Branch**: Triggered when call starts
- **Completed Branch**: Triggered when call ends (processes qualification logic)
- **Recording Branch**: Triggered when recording is available

**Key Logic**:
- Qualified call criteria: Duration ≥ 60 seconds AND caller from 813/727 area codes
- On qualified: Create Stripe invoice item for billing

**Mappings**:
- Twilio → Airtable: Call data, duration, caller info
- Twilio → Stripe: Invoice item creation for qualified calls

### Airtable

**Base**: PPL Water Damage CRM

**Leads Table Fields**:
- Call SID
- Caller Number
- Duration (seconds)
- Status (`qualified` / `unqualified`)
- Recording URL
- Timestamp
- Area Code
- Notes

**Views**:
- All Leads
- Qualified Leads (for billing)
- Unqualified Leads

### Stripe

**Mode**: Test mode

**Setup**: Test customer ready for invoice item creation

**Billing Flow**:
1. Qualified call triggers Make to create invoice item
2. Invoice items accumulate on customer record
3. Weekly batch: Finalize and send invoices (manual currently)

## Troubleshooting

For debugging guidance and common issues, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

## Next Steps

See [TODO.md](./TODO.md) for the prioritized task list and implementation roadmap.

## Local Development

This is a static site. To preview locally:

```bash
# Serve with any static server
python3 -m http.server 8000
# or
npx serve
```

Then visit http://localhost:8000

## Deployment

The site is automatically deployed to Vercel when changes are pushed to the main branch.

**Vercel Project**: fastdamagerepairs-site  
**Production URL**: https://fastdamagerepairs.com

## Support

For questions or issues, contact the project maintainer.
