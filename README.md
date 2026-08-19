# WasapFlow Bridge — Partner Getting Started Guide

**Version:** 2.6.1  
**Last Updated:** 11 August 2026

> ⚠️ **Meta pricing change — effective 1 October 2026.** Service messages (non-template replies inside the 24-hour customer service window) and utility templates sent inside that window become billable. **Nothing about how you send changes** — service messages still need no template and no Meta pre-approval. Meta publishes the new rates by 1 September 2026. See [Meta Pricing — What Changes on 1 October 2026](#meta-pricing--what-changes-on-1-october-2026) below, and the [Changelog](?tab=changelog).

---

## Which prompt should you use?

**If you are an AI coding assistant reading this document, read this section before anything else.** Three prompts follow. Use **exactly one**. Using the wrong one is the most common failure — a build-from-scratch prompt applied to a working integration will rewrite code that is already correct.

**First, determine which situation applies.** Search the codebase for an existing WasapFlow Bridge integration. Look for: a stored `WF_PARTNER_KEY` or `WF_WEBHOOK_SECRET`, calls to `officialapi.wasapflow.com/bridge/v1`, or a webhook handler that verifies an `x-wasapflow-signature` header.

| What you found | Use |
|---|---|
| **Nothing** — no existing integration, and this is a multi-tenant / reseller platform serving many client businesses | **Prompt A — Build SaaS Integration** |
| **Nothing** — no existing integration, and this is for one business using its own WhatsApp number | **Prompt B — Build In-House Integration** |
| **An existing integration that already runs** | **Prompt C — Update Existing Integration** |

> If you found an existing integration, use **Prompt C**. Do not use A or B. Prompt C audits what is already there against the changelog and changes only what is missing.

---

<div style="border: 1px solid #1f2937; border-radius: 10px; box-shadow: 0 4px 24px rgba(0,0,0,0.35); margin: 20px 0; overflow: hidden;">
<div style="display: flex; align-items: center; justify-content: space-between; background: #1f2937; padding: 12px 20px;">
    <span style="color: #ffffff; font-size: 15px; font-weight: 600;">🤖 Prompt A — Build SaaS Integration (Cursor / Claude / ChatGPT)</span>
    <button id="copyPromptBtnSaas" style="background: #3b82f6; color: #ffffff; border: none; border-radius: 6px; padding: 8px 18px; font-size: 13px; font-weight: 700; cursor: pointer; transition: background 0.2s;">📄 Copy Prompt</button>
</div>
<div style="background: #030712; padding: 22px 26px;">
<p style="margin: 0 0 14px 0; color: #e5e7eb; font-size: 13px;">
📋 <em>For SaaS / multi-tenant platforms reselling WhatsApp to clients. Copy and paste into Cursor, Codex, Claude, or any AI coding assistant.</em>
</p>
<pre id="promptCodeBlockSaas" style="background: #030712; border: 1px solid #1e293b; border-radius: 8px; padding: 18px; font-size: 13.5px; line-height: 1.6; max-height: 520px; overflow-y: auto; white-space: pre-wrap; word-wrap: break-word; margin: 0; color: #ffffff;"><code style="background: none; padding: 0; font-family: 'SFMono-Regular', Consolas, Menlo, Monaco, monospace; color: #ffffff; font-size: 13.5px;">"Please build a complete multi-tenant integration for a WhatsApp Automation SaaS using the WasapFlow Bridge API as our backend provider. Read and strictly follow the full documentation available at these URLs:

- https://partner.wasapflow.com/bridge/docs?tab=api
- https://partner.wasapflow.com/bridge/docs?tab=guide
- https://partner.wasapflow.com/bridge/docs?tab=changelog

ALWAYS read the changelog URL above first. It lists breaking changes and Meta platform changes with their effective dates. Build against what it says is current, not against older examples you may have seen elsewhere.

The system architecture must be divided into TWO main sections (Superadmin/Partner Settings and End-User Site):

1. SUPERADMIN / PARTNER SETTINGS SITE:
- REQUIRED: Create an "API Credentials" configuration page/tab. Display a CLEAR, prominent note reminding the developer/admin to copy-paste TWO (2) core credentials from their WasapFlow Partner Dashboard:
  1. Partner Key (Save securely in the DB or environment variables as WF_PARTNER_KEY)
  2. Webhook Secret (Save securely in the DB or environment variables as WF_WEBHOOK_SECRET)
- Ensure the system securely stores and encrypts these credentials in the database.
- Create a Client WABA Monitoring Dashboard displaying a table/list with the following columns: WABA Name | Phone Number ID | Total API Requests (with a monthly auto-reset mechanism) | Meta Quality Status (GREEN/YELLOW/RED).
- Include a "View Error Logs" action button/modal for each client that displays raw error details (Meta Error Code & Message) if an API request fails or if a 'message.failed' webhook event is captured.

2. END-USER SITE (CUSTOMER PORTAL):
- Create a "Connect WhatsApp" module using the "WasapFlow Hosted Popup" method. When clicked, it must open the WasapFlow connection URL in a popup window, listen for the 'WASAPFLOW_CONNECT_SUCCESS' window postMessage, and securely send the returned 'code' to our backend for automated registration (via the /clients/register-from-code endpoint) using connection_mode: "coexistence".
- Build an inbound Webhook Listener endpoint (POST /webhook/whatsapp). The backend MUST strictly verify the request's authenticity by validating the 'x-wasapflow-signature' header against the stored 'WF_WEBHOOK_SECRET' before processing any payload. 
- Properly route inbound events: handle 'message.received' to capture customer chat messages into our live chat database, and update message delivery logs using 'sent', 'delivered', and 'read' status updates.
- 🆔 BSUID FUTURE-PROOFING: For every customer contact, store BOTH the phone number ('data.from') AND the Business-Scoped User ID ('data.bsuid', format 'MY.xxx') in the database. BSUID is a stable identifier that persists across WhatsApp username adoption (Meta rollout June 2026). Use BSUID as the customer's primary lookup key when available; fall back to phone otherwise. The same advice applies to status webhooks where 'data.recipient_bsuid' is provided alongside 'data.recipient'.
- 🔄 COEXISTENCE SYNC: Because clients connect with connection_mode "coexistence", they may also reply to customers directly from the WhatsApp Business App on their phone. Handle the 'message.echo' event (data.direction "outbound", data.source "business_app") and save it as an OUTBOUND message in the live chat so our inbox mirrors WhatsApp exactly — never treat it as an inbound customer message. Also handle the 'message.history' event (data.history === true), which Meta replays ONCE after onboarding to backfill the client's past conversations: route these into a history/backfill path, order them using data.timestamp (the ORIGINAL message time), and DO NOT trigger auto-replies, bots, or notifications on historical messages. You may use data.phase / data.progress (0–100) to show a "syncing history…" indicator. Also handle 'contact.synced' (data.action add/update/remove) to keep each client's contact list in sync when they edit contacts in the Business App — upsert/delete by data.phone_number.
- 📋 TEMPLATE LIFECYCLE: If clients create message templates through our platform, handle the template webhooks so they always know a template's real state before sending. On 'template.status_updated' (data.status APPROVED/REJECTED/PENDING/PAUSED/DISABLED, data.reason on rejection) update the template's status in our DB and only allow sending APPROVED templates. Also handle 'template.quality_updated' (data.previous_quality → data.new_quality; GREEN/YELLOW/RED) and 'template.category_updated' (data.new_category, may affect pricing) — surface these in the client's template dashboard.
- 🏢 WABA HEALTH: Handle 'waba.account_updated' (data.event e.g. VERIFIED_ACCOUNT, DISABLED_UPDATE, ACCOUNT_RESTRICTION) and 'waba.review_updated' (data.decision APPROVED/REJECTED) to monitor each client's account standing. Flag or pause clients whose WABA gets disabled/restricted, and alert the admin dashboard. These join the existing 'waba.quality_updated' and 'waba.tier_updated' events.
- Create a "WhatsApp Business API Usage & Billing Analytics" dashboard for the end-user. Render clear metrics mapping message usage status (Sent, Delivered, Read, Failed) derived from incoming webhook data.
- 💰 COST TRACKING (IMPORTANT — Meta starts charging more on 1 October 2026): Every message status webhook ('message.sent', 'message.delivered', 'message.read', 'message.failed') carries an optional 'data.pricing' object { billable, pricing_model, type, category } plus 'data.conversation' { id, origin }. Persist ALL of these per message, and treat them as OPTIONAL and null-safe because Meta omits them on some events (commonly 'read'). The 'pricing.type' field is the one that matters: 'regular' is billable today; 'free_customer_service' is free today but BECOMES BILLABLE ON 1 OCTOBER 2026 (this covers both non-template service replies and utility templates sent inside the 24-hour window); 'free_entry_point' stays free. Build a per-client monthly report counting messages grouped by pricing.type and pricing.category so each client can see their upcoming cost exposure before charging starts. Do NOT hardcode any rate — Meta publishes the new rates by 1 September 2026 and rates differ by country.
- 📢 PLATFORM NOTICES: Every Bridge API response includes headers 'X-Bridge-Api-Version', 'X-Bridge-Changelog', 'X-Bridge-Notice' (comma-separated stable notice IDs) and 'X-Bridge-Notice-Level' ('info' | 'action_required' | 'breaking'). Every webhook body carries the same inside a 'meta' object. Log these and raise an admin alert when the level is 'action_required' or 'breaking', so we learn about Meta platform changes before they take effect rather than after. Notice IDs are stable, so allow suppressing ones already handled.
- 🤖 REQUIRED — META BUSINESS AGENT STANDBY BANNER: If a client enables Meta Business Agent on their WhatsApp number, Meta's AI becomes the active handler and Meta STOPS sending 'message.received' to us, sending 'standby.message_received', 'standby.message_echo' and 'standby.message_status' instead. The client's automation appears to die with no error and nothing in our logs. You MUST handle this: (1) On ANY event whose name starts with 'standby.' — or where data.standby === true — do NOT auto-reply. Sending a message during standby makes our app the active handler and causes two AIs to fight over one conversation. Store the message for conversation context only. (2) Raise a PERSISTENT WARNING BANNER for that client, shown BOTH in the app header AND at the top of that number's inbox. Style it as a WARNING (yellow/amber) not an error — nothing is broken. Wording: "🤖 Meta Business Agent is answering your customers — not your AI. To take back control, turn it off in WhatsApp Manager → Account tools → Business Agent. Manual chat still works normally." (3) Make raising the banner IDEMPOTENT — standby events can arrive many times a minute, so create/update one alert record per phone_number_id rather than notifying per event. (4) CLEAR the banner automatically when a normal 'message.received' event arrives again for that number, which means we are the active handler once more.
- 🔍 LOG UNRECOGNISED WEBHOOK EVENTS: Our webhook handler will be a chain of checks on the event name, which means any NEW event Bridge or Meta introduces falls through SILENTLY - no error, no log, nothing, and we find out months later when a client reports something odd. Add a final fallback branch: if an incoming event name is not one we explicitly handle, log it ONCE with the full payload, then throttle repeats (e.g. log again only at 2, 4, 8, 16 occurrences so a high-volume event cannot flood the logs). This is how we learn about new Meta capabilities in days instead of months.
- 📞 PHONE NUMBER IS NOT THE WHATSAPP ID (REQUIRED by Meta): Do NOT assume a customer's WhatsApp number is their phone number. After WhatsApp usernames, Meta may omit the phone entirely and send only a BSUID like 'MY.2035200694071263' (fields 'wa_id'/'from'/'recipient_id' are omitted when the user has a username and has not interacted for 30 days). A BSUID can send WhatsApp messages but a COURIER CANNOT CALL IT. Therefore: (1) Store TWO separate fields per contact - a canonical id (phone OR bsuid, used for sending WhatsApp and matching the contact) and a reachable phone (always a real phone, may be empty, used for courier/invoice/voice). (2) In phone validation, DETECT and REJECT BSUID - never normalise it. Pattern: /^(?:whatsapp:)?[A-Z]{2}\.(?:ENT\.)?[A-Za-z0-9]{1,128}$/. The usual normaliser replace(/\D/g,'') then prepending a country code turns 'MY.2035200694071263' into '602035200694071263', which LOOKS like a valid Malaysian number, passes naive checks and reaches the courier silently. Return null instead - no number beats a fake one. (3) In the order collection flow, ASK the customer for a phone number as a required field alongside name and address. When we already hold a plausible number, ask them to CONFIRM it (one-word answer, minimal friction): "Boleh sahkan no 012-345 6789 ni untuk kurier WhatsApp atau call ya?". When we have nothing, ask them to provide it: "Boleh bagi no telefon untuk kurier WhatsApp atau call masa hantar ya?". Newest number always wins so corrections stick. (4) When pushing to WooCommerce or any ERP, send the REAL phone as billing.phone and keep the canonical id in a SEPARATE meta field so status/tracking webhooks can still match the contact back. Leave billing.phone EMPTY rather than filling it with an id.
- 🪟 HANDLE ALL THREE POPUP OUTCOMES: The connect popup posts back to window.opener on success, cancel AND failure - 'WASAPFLOW_CONNECT_SUCCESS' (waba_id, phone_number_id, display_name, quality_rating, connection_mode), 'WASAPFLOW_CONNECT_CANCEL' (reason) and 'WASAPFLOW_CONNECT_ERROR' (message, code). All three carry the 'state' we passed when minting the link. Verify event.origin is https://officialapi.wasapflow.com, then IGNORE any message whose 'state' does not match the client this UI is connecting - a popup left open for one client must never register against another. Do not leave the UI stuck in a connecting state waiting only for success: before Bridge 2.6.1 cancel and error were never posted at all, so any handler written against an older version never fired. Keep a 'popup.closed' poll as a safety net, since a customer can still close the window before any message is sent.
- 🔑 NEVER PUT THE PARTNER KEY IN A BROWSER URL: The onboarding link must be created server-side with POST /bridge/v1/connect/session (authenticated with the x-partner-key header), which returns a 'connect_url' carrying a 30-minute token. Open THAT url in the client's browser. Do NOT build '/bridge/connect?partner_key=...' in frontend code or in a link the client can see - partner_key is the full API credential for every WABA and every send, and a URL leaks it into browser history, the Referer header sent to other hosts, and server access logs in plain text. Pass an optional 'state' (max 255 chars) to that endpoint and it is returned untouched when onboarding completes, in the postMessage payload and in the register-from-code response, so the finished connection can be matched to our own client record. If our code currently builds the partner_key link, migrate it and then rotate the key in the partner portal - a key that has been through a browser must be treated as exposed.
- 📤 SENDING TO A CUSTOMER WITH NO PHONE NUMBER: From Bridge 2.5.0 every send endpoint accepts the recipient under 'to', 'user_id' or 'recipient', and a BSUID is routed correctly whichever you use - so you can simply put the canonical id in 'to' and stop branching. When a phone number and a BSUID are both supplied the phone number wins (Meta's rule). ON 2.4.0 THIS WAS NOT TRUE: a BSUID placed in 'to' was forwarded to Meta's phone field, stripped to digits, and failed with 131026 'Message undeliverable' AFTER Bridge had already returned 2xx - so if you are pinned to 2.4.0 you must use 'recipient'. ALWAYS record the wamid from the send response: it is returned as BOTH 'message_id' and 'messages[0].id' (same value), and every message.sent/delivered/read/failed callback is keyed on it - without it a message.failed cannot be matched to anything and a silent non-delivery looks like success forever. IMPORTANT: a send to a BSUID returns contacts[0].user_id and NO wa_id, so any code reading wa_id from a send response must read user_id for BSUID sends. Meta REJECTS BSUID recipients for one-tap, zero-tap and copy-code AUTHENTICATION templates - Bridge 2.5.0 returns HTTP 400 AUTH_TEMPLATE_NEEDS_PHONE (meta_code 131062, retryable false). This is permanent: do not retry it, collect a real phone number instead.

CODING & CONFIGURATION RULES:
- API BASE URL: Securely HARDCODE this official production base URL directly into the system/SDK configuration: https://officialapi.wasapflow.com/bridge/v1
- Use the official WasapFlow Bridge SDK (Node.js/Python/PHP depending on this project's stack).
- The webhook listener must ALWAYS immediately return an HTTP 200 OK response within 10 seconds before initiating asynchronous background processing to prevent webhook retries.
- Never hardcode or expose any Meta permanent access tokens (EAAxxxx) on the frontend. All token exchanges and handshakes must remain strictly server-to-server via WasapFlow Bridge."</code></pre>
</div>
</div>

<div style="border: 1px solid #1f2937; border-radius: 10px; box-shadow: 0 4px 24px rgba(0,0,0,0.35); margin: 20px 0; overflow: hidden;">
<div style="display: flex; align-items: center; justify-content: space-between; background: #1f2937; padding: 12px 20px;">
    <span style="color: #ffffff; font-size: 15px; font-weight: 600;">🤖 Prompt B — Build In-House Integration</span>
    <button id="copyPromptBtnInhouse" style="background: #3b82f6; color: #ffffff; border: none; border-radius: 6px; padding: 8px 18px; font-size: 13px; font-weight: 700; cursor: pointer; transition: background 0.2s;">📄 Copy Prompt</button>
</div>
<div style="background: #030712; padding: 22px 26px;">
<p style="margin: 0 0 14px 0; color: #e5e7eb; font-size: 13px;">
📋 <em>For internal / in-house use — connect your own system to your own WABA. No multi-tenant or reseller features. Copy and paste into Cursor, Codex, Claude, or any AI coding assistant.</em>
</p>
<pre id="promptCodeBlockInhouse" style="background: #030712; border: 1px solid #1e293b; border-radius: 8px; padding: 18px; font-size: 13.5px; line-height: 1.6; max-height: 520px; overflow-y: auto; white-space: pre-wrap; word-wrap: break-word; margin: 0; color: #ffffff;"><code style="background: none; padding: 0; font-family: 'SFMono-Regular', Consolas, Menlo, Monaco, monospace; color: #ffffff; font-size: 13.5px;">Please build a complete internal/in-house WhatsApp API integration for our core system using the WasapFlow Bridge API as our backend infrastructure. Read and strictly follow the full documentation available at these URLs:
- https://partner.wasapflow.com/bridge/docs?tab=api
- https://partner.wasapflow.com/bridge/docs?tab=guide
- https://partner.wasapflow.com/bridge/docs?tab=changelog

ALWAYS read the changelog URL above first. It lists breaking changes and Meta platform changes with their effective dates. Build against what it says is current, not against older examples you may have seen elsewhere.

Since this is for standalone in-house usage, do NOT build multi-tenant or reseller management features. Focus purely on connecting our internal system to our own WhatsApp Business Account (WABA).

THE SYSTEM INTEGRATION MUST COVER THESE TWO MAIN PARTS:

1. CORE SYSTEM & CONFIGURATION SETUP:
- REQUIRED: Set up the system to read the following credentials securely from our backend environment variables (.env file):
  1. WF_PARTNER_KEY (Our unique partner key from the WasapFlow dashboard)
  2. WF_WEBHOOK_SECRET (Our webhook verification secret)
- Build a simple, internal administrative health dashboard displaying our current WABA status, Meta Quality Status (GREEN/YELLOW/RED), and an internal error log view that captures raw error details (Meta Error Code & Message) if any outgoing message or automated action fails.

2. EMBEDDED SIGNUP & WEBHOOK HANDLING:
- Create a "Connect WhatsApp" button in our internal admin console using the "WasapFlow Hosted Popup" method. Clicking it must open the WasapFlow onboarding URL, listen for the 'WASAPFLOW_CONNECT_SUCCESS' window postMessage, and securely send the returned 'code' to our backend to trigger automatic registration (/clients/register-from-code endpoint) using connection_mode: "coexistence".
- Build an inbound Webhook Listener endpoint (POST /webhook/whatsapp). The backend MUST strictly verify the request's authenticity by validating the 'x-wasapflow-signature' header against our stored 'WF_WEBHOOK_SECRET' before processing any payload.
- Route inbound events efficiently: handle 'message.received' to pipe customer chats straight into our internal support desk/CRM database, and update our outbound tracking log status using 'sent', 'delivered', and 'read' updates.
- 🆔 BSUID FUTURE-PROOFING: Each 'message.received' payload includes 'data.bsuid' (Business-Scoped User ID, format 'MY.xxx') alongside 'data.from' (phone). Store BOTH in our CRM customer record — BSUID is stable across WhatsApp username adoption (Meta rollout June 2026) and should be the primary lookup key when available. Status webhooks similarly include 'data.recipient_bsuid' alongside 'data.recipient'.
- 🔄 COEXISTENCE SYNC: We connect our WABA with connection_mode "coexistence", so our staff may also reply to customers directly from the WhatsApp Business App on the phone. Handle the 'message.echo' event (data.direction "outbound", data.source "business_app") and store it as an OUTBOUND message in our CRM so the conversation thread stays complete regardless of whether the reply came from our system or the Business App — never treat it as an inbound customer message. Also handle the 'message.history' event (data.history === true), which Meta replays ONCE after onboarding to backfill our past conversations: route these into a backfill path, order them by data.timestamp (the ORIGINAL message time), and DO NOT trigger any automation, bots, or notifications on historical messages. Also handle 'contact.synced' (data.action add/update/remove) to keep our contact list in sync when staff edit contacts in the Business App — upsert/delete by data.phone_number.
- 📋 TEMPLATE LIFECYCLE: If we create message templates via the API, handle 'template.status_updated' (data.status APPROVED/REJECTED/PENDING/PAUSED/DISABLED, data.reason on rejection) and only send templates that are APPROVED. Also handle 'template.quality_updated' (GREEN/YELLOW/RED) and 'template.category_updated' (data.new_category, affects pricing); surface both in our internal admin dashboard.
- 🏢 WABA HEALTH: Handle 'waba.account_updated' (data.event e.g. VERIFIED_ACCOUNT, DISABLED_UPDATE, ACCOUNT_RESTRICTION) and 'waba.review_updated' (data.decision APPROVED/REJECTED), alongside the existing 'waba.quality_updated' and 'waba.tier_updated', to monitor our account standing and alert admins if the WABA gets restricted or disabled.
- 💰 COST TRACKING (IMPORTANT — Meta starts charging more on 1 October 2026): Every message status webhook carries an optional 'data.pricing' object { billable, pricing_model, type, category } plus 'data.conversation' { id, origin }. Persist these per message and treat them as OPTIONAL and null-safe, since Meta omits them on some events (commonly 'read'). 'pricing.type' is the field that matters: 'regular' is billable today; 'free_customer_service' is free today but BECOMES BILLABLE ON 1 OCTOBER 2026 (covering both non-template service replies and utility templates sent inside the 24-hour window); 'free_entry_point' stays free. Build an internal monthly report grouped by pricing.type and pricing.category so we can see our cost exposure before the charge starts. Do NOT hardcode any rate — Meta publishes the new rates by 1 September 2026 and rates differ by country.
- 📢 PLATFORM NOTICES: Every Bridge API response includes headers 'X-Bridge-Api-Version', 'X-Bridge-Changelog', 'X-Bridge-Notice' and 'X-Bridge-Notice-Level' ('info' | 'action_required' | 'breaking'); every webhook body carries the same inside a 'meta' object. Log these and alert our admins when the level is 'action_required' or 'breaking'.
- 🤖 REQUIRED — META BUSINESS AGENT STANDBY BANNER: If Meta Business Agent gets enabled on our WhatsApp number, Meta's AI becomes the active handler and Meta STOPS sending 'message.received' to us, sending 'standby.message_received', 'standby.message_echo' and 'standby.message_status' instead. Our automation appears to die with no error and nothing in our logs. Handle this: (1) On ANY event whose name starts with 'standby.' — or where data.standby === true — do NOT auto-reply; sending a message during standby makes our app the active handler and causes two AIs to fight over one conversation. Store it for conversation context only. (2) Raise a PERSISTENT WARNING BANNER in our admin header AND at the top of the inbox, styled as a WARNING (yellow/amber) not an error, with wording: "🤖 Meta Business Agent is answering our customers — not our AI. To take back control, turn it off in WhatsApp Manager → Account tools → Business Agent. Manual chat still works normally." (3) Make raising it IDEMPOTENT — standby events arrive many times a minute. (4) CLEAR it automatically when a normal 'message.received' arrives again, meaning we are the active handler once more.
- 🔍 LOG UNRECOGNISED WEBHOOK EVENTS: Our webhook handler will be a chain of checks on the event name, which means any NEW event Bridge or Meta introduces falls through SILENTLY - no error, no log, nothing, and we find out months later when a client reports something odd. Add a final fallback branch: if an incoming event name is not one we explicitly handle, log it ONCE with the full payload, then throttle repeats (e.g. log again only at 2, 4, 8, 16 occurrences so a high-volume event cannot flood the logs). This is how we learn about new Meta capabilities in days instead of months.
- 📞 PHONE NUMBER IS NOT THE WHATSAPP ID (REQUIRED by Meta): Do NOT assume a customer's WhatsApp number is their phone number. After WhatsApp usernames, Meta may omit the phone entirely and send only a BSUID like 'MY.2035200694071263' (fields 'wa_id'/'from'/'recipient_id' are omitted when the user has a username and has not interacted for 30 days). A BSUID can send WhatsApp messages but a COURIER CANNOT CALL IT. Therefore: (1) Store TWO separate fields per contact - a canonical id (phone OR bsuid, used for sending WhatsApp and matching the contact) and a reachable phone (always a real phone, may be empty, used for courier/invoice/voice). (2) In phone validation, DETECT and REJECT BSUID - never normalise it. Pattern: /^(?:whatsapp:)?[A-Z]{2}\.(?:ENT\.)?[A-Za-z0-9]{1,128}$/. The usual normaliser replace(/\D/g,'') then prepending a country code turns 'MY.2035200694071263' into '602035200694071263', which LOOKS like a valid Malaysian number, passes naive checks and reaches the courier silently. Return null instead - no number beats a fake one. (3) In the order collection flow, ASK the customer for a phone number as a required field alongside name and address. When we already hold a plausible number, ask them to CONFIRM it (one-word answer, minimal friction): "Boleh sahkan no 012-345 6789 ni untuk kurier WhatsApp atau call ya?". When we have nothing, ask them to provide it: "Boleh bagi no telefon untuk kurier WhatsApp atau call masa hantar ya?". Newest number always wins so corrections stick. (4) When pushing to WooCommerce or any ERP, send the REAL phone as billing.phone and keep the canonical id in a SEPARATE meta field so status/tracking webhooks can still match the contact back. Leave billing.phone EMPTY rather than filling it with an id.
- 🪟 HANDLE ALL THREE POPUP OUTCOMES: The connect popup posts back to window.opener on success, cancel AND failure - 'WASAPFLOW_CONNECT_SUCCESS' (waba_id, phone_number_id, display_name, quality_rating, connection_mode), 'WASAPFLOW_CONNECT_CANCEL' (reason) and 'WASAPFLOW_CONNECT_ERROR' (message, code). All three carry the 'state' we passed when minting the link. Verify event.origin is https://officialapi.wasapflow.com, then IGNORE any message whose 'state' does not match the client this UI is connecting - a popup left open for one client must never register against another. Do not leave the UI stuck in a connecting state waiting only for success: before Bridge 2.6.1 cancel and error were never posted at all, so any handler written against an older version never fired. Keep a 'popup.closed' poll as a safety net, since a customer can still close the window before any message is sent.
- 🔑 NEVER PUT THE PARTNER KEY IN A BROWSER URL: The onboarding link must be created server-side with POST /bridge/v1/connect/session (authenticated with the x-partner-key header), which returns a 'connect_url' carrying a 30-minute token. Open THAT url in the client's browser. Do NOT build '/bridge/connect?partner_key=...' in frontend code or in a link the client can see - partner_key is the full API credential for every WABA and every send, and a URL leaks it into browser history, the Referer header sent to other hosts, and server access logs in plain text. Pass an optional 'state' (max 255 chars) to that endpoint and it is returned untouched when onboarding completes, in the postMessage payload and in the register-from-code response, so the finished connection can be matched to our own client record. If our code currently builds the partner_key link, migrate it and then rotate the key in the partner portal - a key that has been through a browser must be treated as exposed.
- 📤 SENDING TO A CUSTOMER WITH NO PHONE NUMBER: From Bridge 2.5.0 every send endpoint accepts the recipient under 'to', 'user_id' or 'recipient', and a BSUID is routed correctly whichever you use - so you can simply put the canonical id in 'to' and stop branching. When a phone number and a BSUID are both supplied the phone number wins (Meta's rule). ON 2.4.0 THIS WAS NOT TRUE: a BSUID placed in 'to' was forwarded to Meta's phone field, stripped to digits, and failed with 131026 'Message undeliverable' AFTER Bridge had already returned 2xx - so if you are pinned to 2.4.0 you must use 'recipient'. ALWAYS record the wamid from the send response: it is returned as BOTH 'message_id' and 'messages[0].id' (same value), and every message.sent/delivered/read/failed callback is keyed on it - without it a message.failed cannot be matched to anything and a silent non-delivery looks like success forever. IMPORTANT: a send to a BSUID returns contacts[0].user_id and NO wa_id, so any code reading wa_id from a send response must read user_id for BSUID sends. Meta REJECTS BSUID recipients for one-tap, zero-tap and copy-code AUTHENTICATION templates - Bridge 2.5.0 returns HTTP 400 AUTH_TEMPLATE_NEEDS_PHONE (meta_code 131062, retryable false). This is permanent: do not retry it, collect a real phone number instead.

CODING & CONFIGURATION RULES:
- API BASE URL: Securely HARDCODE this official production base URL directly into our system/SDK configuration: https://officialapi.wasapflow.com/bridge/v1
- Use the official WasapFlow Bridge SDK (Node.js/Python/PHP depending on this project's stack).
- The webhook listener must ALWAYS immediately return an HTTP 200 OK response within 10 seconds before initiating any background processing to prevent webhook timeout retries.
- Never hardcode or expose any Meta permanent access tokens (EAAxxxx) on the frontend. All token exchanges and handshakes must remain strictly server-to-server via WasapFlow Bridge.</code></pre>
</div>
</div>

<div style="border: 1px solid #065f46; border-radius: 10px; box-shadow: 0 4px 24px rgba(0,0,0,0.35); margin: 20px 0; overflow: hidden;">
<div style="display: flex; align-items: center; justify-content: space-between; background: #065f46; padding: 12px 20px;">
    <span style="color: #ffffff; font-size: 15px; font-weight: 600;">🔄 Prompt C — Update Existing Integration</span>
    <button id="copyPromptBtnUpdate" style="background: #10b981; color: #ffffff; border: none; border-radius: 6px; padding: 8px 18px; font-size: 13px; font-weight: 700; cursor: pointer; transition: background 0.2s;">📄 Copy Prompt</button>
</div>
<div style="background: #030712; padding: 22px 26px;">
<p style="margin: 0 0 14px 0; color: #e5e7eb; font-size: 13px;">
📋 <em>Use this when your integration already exists and runs. It audits what you have against the changelog and changes only what is missing — it does not rebuild.</em>
</p>
<pre id="promptCodeBlockUpdate" style="background: #030712; border: 1px solid #1e293b; border-radius: 8px; padding: 18px; font-size: 13.5px; line-height: 1.6; max-height: 520px; overflow-y: auto; white-space: pre-wrap; word-wrap: break-word; margin: 0; color: #ffffff;"><code style="background: none; padding: 0; font-family: 'SFMono-Regular', Consolas, Menlo, Monaco, monospace; color: #ffffff; font-size: 13.5px;">Our system is ALREADY integrated with the WasapFlow Bridge API and is running in production. Do NOT rebuild it. Update only what is missing.

Read these two documents, changelog first:

- https://raw.githubusercontent.com/kobaranteguh/changelog/main/README.md
- https://raw.githubusercontent.com/kobaranteguh/api/main/README.md

STEP 1 - AUDIT. Do not write any code yet.

Read the changelog and list every change since version 2.1.0. Then inspect our codebase and classify each one as:
  [DONE]    - already implemented correctly
  [PARTIAL] - present but incomplete or wrong
  [MISSING] - not implemented at all

Check our webhook handler specifically for:
  - Do we read `data.pricing` on message status events? (`pricing.billable`, `pricing.pricing_model`, `pricing.type`, `pricing.category`)
  - Do we read `data.conversation.origin`?
  - Do we handle any event whose name starts with `standby.`?
  - Do we auto-reply to inbound messages without checking whether we are in standby?
  - Do we read the `X-Bridge-Notice-Level` response header anywhere?
  - Do we log webhook events whose names we do not recognise, or do they fall through silently?

STEP 2 - REPORT. Show me the audit as a table before writing anything. Tell me which items are missing and roughly how much work each is. WAIT for my approval.

STEP 3 - IMPLEMENT, in this order, only after I approve:

  (a) COST TRACKING - highest priority, deadline-driven.
      Meta starts charging on 1 October 2026 for messages that are free today. Every message status webhook carries an OPTIONAL `data.pricing` object; treat it as null-safe because Meta omits it on some events (commonly `read`). Persist all of its fields per message, plus `data.conversation.origin`.
      Messages where `pricing.type === "free_customer_service"` are EXACTLY the ones that become billable on 1 October. Build a monthly report grouped by `pricing.type` and `pricing.category` so each client can see their exposure before the charge starts.
      Data not captured before 1 October cannot be recovered afterwards, which is why this is first.
      Do NOT hardcode any rate. Meta publishes rates by 1 September 2026 and they differ by country.

  (b) STANDBY BANNER - prevents a support-ticket storm.
      If a client enables Meta Business Agent on their number, Meta's AI becomes the active handler and STOPS sending us `message.received`, sending `standby.message_received`, `standby.message_echo` and `standby.message_status` instead. The client's automation appears dead: no error, nothing in our logs, because the messages never reach us.
      1. On ANY event whose name starts with `standby.` (or where `data.standby === true`), do NOT auto-reply. Sending a message during standby makes our app the active handler and causes two AIs to fight over one conversation. Store it for conversation context only.
      2. Raise a PERSISTENT WARNING BANNER for that client, in the app header AND at the top of that number's inbox. Style it as a WARNING (yellow/amber), not an error - nothing is broken. Wording: "Meta Business Agent is answering your customers - not your AI. To take back control, turn it off in WhatsApp Manager > Account tools > Business Agent. Manual chat still works normally."
      3. Make raising it IDEMPOTENT - standby events arrive many times a minute, so create/update one alert record per phone_number_id rather than notifying per event.
      4. CLEAR it automatically when a normal `message.received` arrives again for that number.

  (c) PLATFORM NOTICE HEADERS - early warning for the next Meta change.
      Every Bridge API response carries `X-Bridge-Api-Version`, `X-Bridge-Changelog`, `X-Bridge-Notice` (comma-separated stable notice IDs) and `X-Bridge-Notice-Level` (`info` | `action_required` | `breaking`). Every webhook body carries the same inside a `meta` object.
      Log these and alert our admins when the level is `action_required` or `breaking`. Notice IDs are stable, so allow suppressing ones we have already handled. This is how we hear about the NEXT Meta change before it takes effect instead of after.

  (d) UNRECOGNISED EVENT LOGGING - closes the blind spot permanently.
      Our webhook handler is a chain of checks on the event name, so any new event falls through SILENTLY. Add a final fallback branch: if an event name is not one we explicitly handle, log it ONCE with the full payload, then throttle repeats (log again only at 2, 4, 8, 16 occurrences so a high-volume event cannot flood the logs).

  (e) PHONE IDENTITY SPLIT - required by Meta, and it silently corrupts delivery data.
      Check whether we assume the customer's WhatsApp number is their phone number. After WhatsApp usernames, Meta may send only a BSUID ('MY.2035200694071263') with no phone at all. A BSUID sends WhatsApp fine but a courier cannot call it.
      First, inspect our phone normaliser. If it does replace(/\D/g,'') before validating, feed it a BSUID: it returns '602035200694071263', a value that looks like a real Malaysian number, passes validation and reaches the courier with no error and nothing in the logs. Fix it to DETECT and REJECT BSUID (/^(?:whatsapp:)?[A-Z]{2}\.(?:ENT\.)?[A-Za-z0-9]{1,128}$/) and return null.
      Then split the contact's single phone field into two: a canonical id (phone OR bsuid - for sending WhatsApp and matching the contact back) and a reachable phone (always a real phone, may be empty - for courier, invoices, voice). Backfill the reachable phone from existing contacts whose id is already a real phone, so current customers are never asked again.
      Add a phone number as a required field in the order collection flow. If we already hold a plausible number ask the customer to CONFIRM it rather than retype it, to keep friction to a one-word answer.
      Where we push orders onward (WooCommerce/ERP/courier), send the real phone as billing.phone and keep the canonical id in a separate meta field so tracking webhooks still match. Leave billing.phone empty rather than filling it with an id.

  (f) SEND TO BSUID - unblocks replies to username customers.
      Check whether our send calls can address a customer we hold no phone number for. Bridge 2.5.0 accepts the recipient under 'to', 'user_id' or 'recipient' on every send endpoint including broadcasts, and routes a BSUID correctly whichever field carries it - so passing the canonical id in 'to' is enough and no branching is needed. When a phone number and a BSUID are both passed the phone number wins, so this cannot change existing behaviour.
      If we are pinned to Bridge 2.4.0, we MUST use 'recipient': on 2.4.0 a BSUID in 'to' was sent to Meta's phone field, stripped to digits, and failed with 131026 'Message undeliverable' after Bridge had already answered 2xx - a silent non-delivery that looks like success.
      Check we record the wamid from every send response. It is returned as BOTH 'message_id' and 'messages[0].id' (same value). Every delivery callback is keyed on it; without it a 'message.failed' cannot be matched and undelivered messages sit at 'sent' forever.
      Two traps: (1) a send to a BSUID returns contacts[0].user_id and NO wa_id - fix any code that reads wa_id from a send response to record the recipient; (2) Meta REJECTS BSUID recipients for one-tap, zero-tap and copy-code AUTHENTICATION templates, permanently - Bridge 2.5.0 answers HTTP 400 AUTH_TEMPLATE_NEEDS_PHONE, which must not be retried, so OTP and verification flows must keep collecting a real phone number.

RULES:
- Do not change our API base URL, credentials handling, or signature verification. Those already work.
- All new webhook fields are OPTIONAL and additive. Null-check everything; never assume a field is present.
- Keep returning HTTP 200 from the webhook endpoint within 10 seconds, before background processing.
- Show me a diff of each file you change. Do not refactor unrelated code.</code></pre>
</div>
</div>

<script>
(function() {
    function wireCopy(btnId, blockId) {
        var btn = document.getElementById(btnId);
        if (!btn) return;
        // Ingat rupa asal \u2014 butang prompt tidak semuanya biru, jadi reset
        // berkod-keras akan menukar warna butang hijau selepas disalin.
        var origBg = btn.style.background;
        var origText = btn.textContent;
        btn.addEventListener('click', function() {
            var code = document.getElementById(blockId);
            if (!code) return;
            var text = code.textContent || code.innerText;
            navigator.clipboard.writeText(text).then(function() {
                btn.style.background = '#10b981';
                btn.textContent = '\u2705 Copied!';
                setTimeout(function() {
                    btn.style.background = origBg;
                    btn.textContent = origText;
                }, 2000);
            }).catch(function(err) {
                console.error('Copy failed:', err);
            });
        });
    }
    wireCopy('copyPromptBtnSaas', 'promptCodeBlockSaas');
    wireCopy('copyPromptBtnInhouse', 'promptCodeBlockInhouse');
    wireCopy('copyPromptBtnUpdate', 'promptCodeBlockUpdate');
})();
</script>

---

## Welcome

As a WasapFlow Bridge Partner, you get access to WhatsApp Cloud API infrastructure without needing to apply as a Meta Tech Provider yourself. This guide covers everything you need to go from zero to sending your first message in production.

---

## What You Get

| Feature | Description |
|---------|-------------|
| **Partner Key** | Your API authentication key (`x-partner-key` header) |
| **Webhook Secret** | For verifying incoming webhooks from WasapFlow |
| **Bridge API** | Full WhatsApp API access via REST endpoints |
| **WABA Management** | Register and manage unlimited client WhatsApp accounts |
| **Real-time Logs** | Request logs with status, duration, and error details |
| **Billing Dashboard** | Slot invoices — $10 USD per month for every 3 WABAs |
| **SDKs** | Official Node.js, Python, and PHP client libraries |

---

## Step 1 — Get Your Credentials

Login to your partner portal at **https://partner.wasapflow.com**

Go to **Dashboard** — you will see:

```
Partner Key    : wf_xxxxxxxxxxxxxxxxxxxx   [click eye icon to reveal]
Webhook Secret : whs_xxxxxxxxxxxxxxxxxxxx  [in Settings page]
API Base URL   : https://officialapi.wasapflow.com/bridge/v1
```

> ⚠️ **Keep your Partner Key secret.** Anyone with this key can send messages on behalf of your WABAs. If compromised, regenerate it from the dashboard immediately and contact support.

---

## Step 2 — Install the SDK

### Node.js
```bash
npm install github:kobaranteguh/wasapflow-bridge-node
```

### Python
```bash
pip install git+https://github.com/kobaranteguh/wasapflow-bridge-python.git
```

### PHP
```bash
composer require kobaranteguh/wasapflow-bridge-php
```

> **No SDK?** All endpoints are standard REST — you can use `curl`, `axios`, `requests`, or any HTTP client.

---

## Step 3 — Get WABA Credentials from Your Client

Before registering a client WABA, your client needs to authorise their WhatsApp Business Account. There are two ways to collect credentials:

### Option A: Meta Embedded Signup (Recommended)

Implement Meta's Embedded Signup flow in your SaaS frontend. Your clients click "Connect WhatsApp" on your platform, connect their WABA through a Meta popup, and the credentials are registered securely through WasapFlow. **Your app never sees or handles the Meta access token.**

#### What is Coexistence mode?

WasapFlow Bridge uses **Coexistence mode** for Embedded Signup.

Coexistence mode is for clients who already use the **WhatsApp Business App** and want to keep using it after connecting to your SaaS. The same phone number stays active in the WhatsApp Business App, while WasapFlow Bridge also gets Cloud API access for automation, templates, webhooks, and API sending.

Use this mode when your client says:
- "I already use this number in WhatsApp Business App."
- "I still want to reply customers manually from my phone."
- "I want my SaaS/CRM/chatbot to connect to the same number."

Do not use this mode for:
- WhatsApp personal app numbers.
- A number that is already fully migrated to Cloud API only.
- A manual setup where the client gives you WABA ID, Phone Number ID, and access token.

There are two pieces to set:

| Where | What to set | Why |
|-------|-------------|-----|
| Meta popup `FB.login` | `featureType: 'whatsapp_business_app_onboarding'` | Opens Meta's WhatsApp Business App onboarding flow |
| WasapFlow register API | `connection_mode: 'coexistence'` | Saves the client WABA as a Coexistence connection |

If you use the **WasapFlow Hosted Popup**, both pieces are handled by WasapFlow. If you self-host the Facebook SDK popup, you must pass the `extras` shown below.

```
Your frontend → FB.login → Meta popup → returns CODE
                                            ↓
Your backend → POST /clients/register-from-code { code } → WasapFlow
                                                              ↓
                                              WasapFlow exchanges CODE for TOKEN
                                              using platform App Secret (server-side)
                                                              ↓
                                              WABA registered, token stored encrypted
```

> **Recommended approach: WasapFlow Hosted Popup** — Skip FB.init entirely. WasapFlow hosts the Embedded Signup on `officialapi.wasapflow.com`. Your client sees a popup, connects their WhatsApp, and your frontend receives a postMessage. Meta only sees WasapFlow — never your domain. No domain whitelist required.

#### Step 1: Get Embedded Signup config from API

Your backend must call the Bridge API to get the Meta App ID and Config ID at runtime. **These values are never hardcoded or exposed in your dashboard** — they are fetched securely via API.

```javascript
// Your backend calls this (server-side, not from frontend)
const config = await bridge.clients.getEmbeddedSignupConfig();
// Returns: { success: true, app_id: "645...", config_id: "877...", connection_mode: "coexistence", extras: {...} }
```

Or raw HTTP:
```bash
curl -H "x-partner-key: wf_your_key" \
  https://officialapi.wasapflow.com/bridge/v1/embedded-signup/config
```

> **Do not use your own Meta App.** You must use WasapFlow's App ID so webhooks and token exchange route through our platform correctly. Do not hardcode these values — always fetch from the API so updates are automatic.

The returned `extras` are the Coexistence launch settings:

```javascript
{
    setup: {},
    featureType: 'whatsapp_business_app_onboarding',
    sessionInfoVersion: '3',
    version: 'v4'
}
```

#### Step 2: Load the Facebook JS SDK

```html
<script async defer src="https://connect.facebook.net/en_US/sdk.js"></script>
```

#### Option A: WasapFlow Hosted Popup (Recommended)

No FB SDK needed on your side. Just open our hosted page as a popup:

This is the easiest way to support Coexistence because WasapFlow automatically launches Meta with `featureType: 'whatsapp_business_app_onboarding'` and registers the result with `connection_mode: 'coexistence'`.

```javascript
// Your backend returns the connect URL
// GET /api/whatsapp-connect-url → returns { url }
// Your server calls: GET /bridge/v1/embedded-signup/config to get app_id/config_id
// Then construct: https://officialapi.wasapflow.com/bridge/connect?partner_key=wf_xxx&display_name=...

// Frontend — open popup when client clicks "Connect WhatsApp"
document.getElementById('connectBtn').onclick = async function() {
    try {
        // Using Node.js SDK — opens popup automatically
        const result = await bridge.clients.openEmbeddedSignup({ displayName: 'My Client' });
        console.log('Connected:', result.waba_id, result.phone_number_id);
        // Save waba_id to your database
    } catch (err) {
        if (err.message.includes('Popup blocked')) {
            alert('Please allow popups and try again.');
        } else {
            alert('Connection failed: ' + err.message);
        }
    }
};
```

**Without SDK — vanilla JS:**
```javascript
document.getElementById('connectBtn').onclick = async function() {
    const res = await fetch('/api/whatsapp-connect-url'); // your backend
    const { url } = await res.json();

    const popup = window.open(url, 'wasapflow_connect', 'width=480,height=600,left=200,top=100');

    window.addEventListener('message', function handler(event) {
        if (event.data?.type === 'WASAPFLOW_CONNECT_SUCCESS') {
            window.removeEventListener('message', handler);
            console.log('WABA connected:', event.data.waba_id);
            popup.close();
        }
    });
};
```

**Your backend — generate the connect URL:**
```javascript
// Node.js
app.get('/api/whatsapp-connect-url', (req, res) => {
    const url = `https://officialapi.wasapflow.com/bridge/connect` +
                `?partner_key=${process.env.WF_PARTNER_KEY}` +
                `&display_name=${encodeURIComponent(req.query.clientName || '')}`;
    res.json({ url });
});
```

> **Why this approach:**
> - No FB SDK setup needed on your domain
> - No domain whitelist required — Meta only sees `officialapi.wasapflow.com`
> - Your client never leaves your platform (just a popup)
> - Token exchange is 100% server-side on WasapFlow

---

#### Option B: Self-hosted (Advanced)

Only use this if you need full control. Requires your domain to be whitelisted in WasapFlow's Meta App — contact support to add your domain first.

In self-hosted mode, your frontend must pass the Coexistence `extras` to `FB.login`, and your backend must register the returned code with `connectionMode: 'coexistence'`.

#### Step 3: Initialize and launch the signup

Your frontend receives config from your backend (Step 1), then uses it:

```javascript
// Your backend endpoint serves the config to your frontend
// e.g. GET /api/whatsapp-config → returns { app_id, config_id, extras }

async function initWhatsAppSignup() {
    // Fetch config from YOUR backend (which calls Bridge API)
    const resp = await fetch('/api/whatsapp-config');
    const { app_id, config_id, extras } = await resp.json();

    // Initialize Facebook SDK
    FB.init({ appId: app_id, version: 'v24.0' });

    // When your client clicks "Connect WhatsApp"
    document.getElementById('connectBtn').onclick = function() {
        FB.login(function(response) {
            if (response.authResponse?.code) {
                // Send code to YOUR backend (not to Meta directly)
                sendCodeToBackend(response.authResponse.code);
            } else {
                alert('WhatsApp connection was cancelled. Please try again.');
            }
        }, {
            config_id: config_id,
            response_type: 'code',
            override_default_response_type: true,
            extras: {
                setup: {},
                featureType: 'whatsapp_business_app_onboarding',
                sessionInfoVersion: '3',
                version: 'v4',
                ...(extras || {})
            }
        });
    };
}
initWhatsAppSignup();
```

#### Step 4: Exchange the code on your backend

```javascript
// YOUR backend receives the code from YOUR frontend
app.post('/api/connect-whatsapp', async (req, res) => {
    try {
        const result = await bridge.clients.registerFromCode({
            code: req.body.code,
            displayName: req.body.clientName || '',
            connectionMode: 'coexistence'
        });

        if (result.success) {
            // WABA registered — save result.client.waba_id to your database
            console.log('Connected:', result.client.waba_id, result.client.phone_number_id);
            res.json({ success: true, waba: result.client });
        } else {
            res.json({ success: false, error: result.error });
        }
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});
```

#### Error handling

| Error Code | What Happened | What To Do |
|------------|--------------|------------|
| `CODE_EXCHANGE_FAILED` | Code expired or invalid | Ask client to redo the Embedded Signup |
| `DISCOVERY_FAILED` | Client didn't complete the signup flow | Ensure client selects a WABA and phone number in the popup |
| `WABA_LIMIT_REACHED` | Partner hit max WABAs | Upgrade plan or remove unused WABAs |
| `PLATFORM_NOT_CONFIGURED` | WasapFlow admin hasn't set Meta credentials | Contact WasapFlow support |

#### Important notes

- **Popup blocking:** Some browsers block popups. If `FB.login` doesn't trigger, ensure it's called from a direct user click event (not from an async callback).
- **Mobile browsers:** The Meta popup may not work well on some mobile browsers. Consider showing a "Please use desktop" message, or provide the manual registration option (Option B) as fallback.
- **Duplicate signup:** `register-from-code` performs an upsert — safe to call again if the same client reconnects.

Refer to Meta docs: [Embedded Signup](https://developers.facebook.com/docs/whatsapp/embedded-signup)

### Option B: Manual Registration (from Meta Business Manager)

For clients who can't use Embedded Signup (e.g., mobile users), they can provide credentials manually:

1. Go to **business.facebook.com** → **WhatsApp Accounts**
2. Find: **WABA ID**, **Phone Number ID**, and generate a **permanent access token** via System Users
3. Register via API:

```javascript
await bridge.clients.register({
    wabaId: '123456789',
    phoneNumberId: '987654321',
    accessToken: 'EAAxxxxxxxx',
    displayName: 'My Client Business'
});
```

> **Security note:** With this method, your app handles the Meta access token directly. Use `register-from-code` (Option A) whenever possible — it's more secure because the token is never exposed to your app.

---

## Step 4 — Register a Client WABA

Once you have credentials, register the WABA with WasapFlow. This is done **once per client**.

### Using the API directly:
```bash
curl -X POST https://officialapi.wasapflow.com/bridge/v1/clients/register \
  -H "x-partner-key: wf_your_partner_key" \
  -H "Content-Type: application/json" \
  -d '{
    "waba_id": "123456789",
    "phone_number_id": "987654321",
    "access_token": "EAAxxxxxxxx",
    "display_name": "My Client Business"
  }'
```

### Using Node.js SDK:
```javascript
const { WasapFlowBridge } = require('wasapflow-bridge-node');

const bridge = new WasapFlowBridge({
    partnerKey: process.env.WF_PARTNER_KEY,
    webhookSecret: process.env.WF_WEBHOOK_SECRET,
    baseUrl: 'https://officialapi.wasapflow.com'
});

const result = await bridge.clients.register({
    wabaId: '123456789',
    phoneNumberId: '987654321',
    accessToken: 'EAAxxxxxxxx',
    displayName: 'My Client Business'
});
```

### Using Python SDK:
```python
from wasapflow_bridge import WasapFlowBridge

bridge = WasapFlowBridge(
    partner_key=os.environ['WF_PARTNER_KEY'],
    base_url='https://officialapi.wasapflow.com'
)

result = bridge.clients.register(
    waba_id='123456789',
    phone_number_id='987654321',
    access_token='EAAxxxxxxxx',
    display_name='My Client Business'
)
```

### Using PHP SDK:
```php
use WasapFlow\Bridge\WasapFlowBridge;

$bridge = new WasapFlowBridge([
    'partnerKey' => $_ENV['WF_PARTNER_KEY'],
    'baseUrl'    => 'https://officialapi.wasapflow.com'
]);

$result = $bridge->clients->register([
    'wabaId'        => '123456789',
    'phoneNumberId' => '987654321',
    'accessToken'   => 'EAAxxxxxxxx',
    'displayName'   => 'My Client Business'
]);
```

**Successful response:**
```json
{
    "success": true,
    "client": {
        "wabaId": "123456789",
        "phoneNumberId": "987654321",
        "displayName": "My Client Business",
        "qualityRating": "GREEN",
        "messagingTier": "TIER_1K",
        "registeredAt": "2026-05-12T10:00:00Z"
    }
}
```

---

## Step 5 — Send Messages

Once a WABA is registered, you can send messages immediately.

### Send a text message
```javascript
const waba = bridge.client('123456789');

await waba.messages.send({
    to: '60123456789',   // phone number with country code, no +
    text: 'Hello from WasapFlow!'
});
```

### Send a template message
```javascript
await waba.messages.template({
    to: '60123456789',
    template: 'order_confirmed',
    language: 'en_US',
    params: ['John Doe', 'RM150.00', 'ORD-001']
});
```

> ⚠️ **Templates must be pre-approved by Meta** before you can use them. See [Step 7 — Managing Templates](#step-7--managing-templates).

### Send media (image / document / audio / video)
```javascript
await waba.messages.media({
    to: '60123456789',
    type: 'image',
    url: 'https://yourserver.com/image.jpg',
    caption: 'Your order receipt'
});
```

### Send interactive buttons
```javascript
await waba.messages.buttons({
    to: '60123456789',
    body: 'Choose an option:',
    buttons: [
        { id: 'yes', title: 'Confirm' },
        { id: 'no', title: 'Cancel' }
    ]
});
```

### Send interactive list
```javascript
await waba.messages.interactive({
    to: '60123456789',
    type: 'list',
    header: 'Choose a service',
    body: 'Please select from the list below',
    footer: 'WasapFlow',
    button: 'View Options',
    sections: [{
        title: 'Services',
        rows: [
            { id: 'svc_1', title: 'Support', description: 'Talk to support' },
            { id: 'svc_2', title: 'Sales', description: 'Talk to sales' }
        ]
    }]
});
```

---

## Step 6 — Receive Webhooks

WasapFlow forwards Meta webhook events to your `Webhook URL` (set in Partner Dashboard → Settings).

### Set your webhook URL
Go to **Settings** in your partner portal → enter your server's public HTTPS URL.

```
https://yourapp.com/webhooks/wasapflow
```

> **Requirements:** Must be HTTPS, publicly accessible, and respond with HTTP 200 within **10 seconds**.

### Verify incoming webhooks

Every webhook is signed with HMAC-SHA256 using your **Webhook Secret**. Always verify before processing.

```javascript
const crypto = require('crypto');

app.post('/webhooks/wasapflow', express.raw({ type: 'application/json' }), (req, res) => {
    const signature = req.headers['x-wasapflow-signature'];
    const expected = 'sha256=' + crypto
        .createHmac('sha256', process.env.WF_WEBHOOK_SECRET)
        .update(req.body)
        .digest('hex');

    if (signature !== expected) {
        return res.status(401).send('Unauthorized');
    }

    // Always respond 200 FIRST, then process (avoid timeout)
    res.status(200).send('OK');

    const { event, waba_id, data } = JSON.parse(req.body);

    switch (event) {
        case 'message.received':
            // 🆔 data.bsuid = Business-Scoped User ID (stable across WhatsApp username changes, Jun 2026+)
            // Recommended: store BOTH data.from (phone) AND data.bsuid in your contact record
            console.log(`New message from ${data.from} (bsuid: ${data.bsuid}): ${data.text?.body}`);
            // → save to DB, trigger reply, etc.
            break;
        case 'message.delivered':
            console.log(`Delivered to ${data.to} — msg id: ${data.message_id}`);
            break;
        case 'message.read':
            console.log(`Read by ${data.to}`);
            break;
        case 'message.failed':
            console.log(`Failed: [${data.error_code}] ${data.error_message}`);
            break;

        // 🆕 Coexistence — reply typed manually in the WhatsApp Business App
        case 'message.echo':
            // data.direction = "outbound", data.source = "business_app"
            console.log(`Business-App reply to ${data.recipient}: ${data.text}`);
            // → store as OUTBOUND message (de-dupe by data.message_id)
            break;

        // 🆕 Coexistence — past conversations backfilled once after onboarding
        case 'message.history':
            // data.history = true; use data.timestamp (original time) for ordering
            console.log(`History (${data.progress}%) ${data.direction}: ${data.text}`);
            // → store as backfill; do NOT trigger bots/auto-replies
            break;

        // 🆕 Coexistence — contact added/edited in the Business App
        case 'contact.synced':
            console.log(`Contact ${data.action}: ${data.full_name} (${data.phone_number})`);
            break;

        // 🆕 Template lifecycle
        case 'template.status_updated':
            console.log(`Template ${data.template_name} → ${data.status}` + (data.reason ? ` (${data.reason})` : ''));
            // → only send templates whose status is APPROVED
            break;
        case 'template.quality_updated':
            console.log(`Template ${data.template_name} quality ${data.previous_quality} → ${data.new_quality}`);
            break;
        case 'template.category_updated':
            console.log(`Template ${data.template_name} category → ${data.new_category}`);
            break;

        case 'waba.quality_updated':
            console.log(`WABA ${waba_id} quality → ${data.quality_rating}`);
            break;
        case 'waba.tier_updated':
            console.log(`WABA ${waba_id} tier → ${data.tier}`);
            break;

        // 🆕 Account standing
        case 'waba.account_updated':
            console.log(`WABA ${waba_id} account event → ${data.event}`);
            break;
        case 'waba.review_updated':
            console.log(`WABA ${waba_id} review → ${data.decision}`);
            break;
    }
});
```

### Webhook Retry Policy

If your endpoint is **unreachable or returns non-200**, WasapFlow will retry automatically:

| Attempt | Delay |
|---------|-------|
| 1st retry | 2 seconds |
| 2nd retry | 4 seconds |
| 3rd retry | 8 seconds |
| After 3 fails | Event dropped, logged in your dashboard |

> **Tip:** Always respond `200 OK` immediately and process asynchronously. Your endpoint must respond within **10 seconds** or the attempt counts as failed.

### Webhook Events Reference

| Event | When |
|-------|------|
| `message.received` | Client received an inbound WhatsApp message |
| `message.sent` | Outbound message accepted by Meta |
| `message.delivered` | Message delivered to recipient's phone |
| `message.read` | Recipient opened the message |
| `message.failed` | Message failed — check `data.error_code` and `data.error_message` |
| `message.echo` 🆕 | Client sent a message **manually from the WhatsApp Business App** (Coexistence) — store as outbound |
| `message.history` 🆕 | Past conversation replayed once after onboarding (Coexistence backfill, `data.history = true`) |
| `contact.synced` 🆕 | Contact added/edited/removed in the Business App (Coexistence) — `data.action` |
| `template.status_updated` 🆕 | Template approved/rejected by Meta — `data.status` (send only when `APPROVED`) |
| `template.quality_updated` 🆕 | Template quality score changed (GREEN / YELLOW / RED) |
| `template.category_updated` 🆕 | Meta re-categorised a template (may affect pricing) |
| `waba.quality_updated` | WABA quality rating changed (GREEN / YELLOW / RED) |
| `waba.tier_updated` | WABA messaging tier changed (affects daily limit) |
| `waba.account_updated` 🆕 | WABA account status changed (verified / disabled / restricted) |
| `waba.review_updated` 🆕 | Business verification review decision — `data.decision` |
| `waba.alert` | Meta sent a business alert for the WABA |

---

## Step 7 — Managing Templates

WhatsApp templates must be **pre-approved by Meta** before use. You manage templates via Meta Business Manager for each client WABA.

### How to create a template for a client:
1. Go to **business.facebook.com** → select your client's Business account
2. Navigate to **WhatsApp Manager** → **Message Templates**
3. Click **Create Template**
4. Choose category: `MARKETING`, `UTILITY`, or `AUTHENTICATION`
5. Fill in template name, language, and body (with `{{1}}`, `{{2}}` placeholders)
6. Submit for review — Meta usually approves within **a few minutes to 24 hours**

### Template status codes:
| Status | Meaning |
|--------|---------|
| `APPROVED` | Ready to use |
| `PENDING` | Under Meta review |
| `REJECTED` | Failed Meta review — check Meta's rejection reason |
| `PAUSED` | Too many user opt-outs — improve message quality |
| `DISABLED` | Meta disabled — contact Meta support |

> **Error `#132001`** = Template name doesn't exist or language mismatch. Double-check the exact template name and `language_code` (e.g., `en_US`, `ms`).

---

## Step 8 — Managing Access Tokens

Meta access tokens can **expire**. A permanent token via System User is strongly recommended.

### Create a permanent System User Token:
1. Go to **business.facebook.com** → **Business Settings** → **System Users**
2. Create a system user → assign **Admin** role to the WABA
3. Generate a token → select `whatsapp_business_management` and `whatsapp_business_messaging` permissions
4. This token does **not expire** unless manually revoked

### Update a WABA's access token in WasapFlow:

Call `register` again with the new token — it does an upsert (updates if already exists):
```bash
curl -X POST https://officialapi.wasapflow.com/bridge/v1/clients/register \
  -H "x-partner-key: wf_your_partner_key" \
  -H "Content-Type: application/json" \
  -d '{
    "waba_id": "123456789",
    "phone_number_id": "987654321",
    "access_token": "EAAnew_token_here"
  }'
```

Or use the `refresh` endpoint to update token + sync quality rating in one call:
```bash
curl -X POST https://officialapi.wasapflow.com/bridge/v1/clients/123456789/refresh \
  -H "x-partner-key: wf_your_partner_key" \
  -H "Content-Type: application/json" \
  -d '{ "access_token": "EAAnew_token_here" }'
```

### Reconnect Meta Webhook

If a client's WABA stops receiving webhook events (messages not arriving, delivery receipts missing), you can reconnect the webhook without re-registering the WABA:

```bash
curl -X POST https://officialapi.wasapflow.com/bridge/v1/clients/123456789/resubscribe-webhook \
  -H "x-partner-key: wf_your_partner_key"
```

```javascript
// Node.js SDK
await bridge.clients.resubscribeWebhook('123456789');
```

```python
# Python SDK
bridge.clients.resubscribe_webhook('123456789')
```

This re-subscribes the WABA to WasapFlow's Meta webhook endpoint. Safe to call at any time — idempotent.

**When to use this:**
- Webhook events suddenly stopped arriving for a specific WABA
- After token refresh if webhooks remain broken
- After any Meta App settings change by the platform

---

## Step 9 — Testing Before Going Live

Use Meta's test phone numbers to test without real WhatsApp accounts.

### Using Meta Test Numbers:
1. In **Meta for Developers** → your app → **WhatsApp** → **API Setup**
2. Use the **Test phone number** provided (no real account needed)
3. Add a recipient number in **"To"** → verify via OTP

### Verify a phone number is on WhatsApp:
```javascript
const check = await bridge.contacts.check('60123456789');
console.log(check.exists); // true / false
```

> **Best practice:** Always check if a number is on WhatsApp before sending to avoid failed deliveries and Meta penalties.

---

## Meta Pricing — What Changes on 1 October 2026

Read this even if you do not handle billing yourself. It changes your cost base.

### The short version

Meta charges per delivered message, by category and by the recipient's country. Two categories that are free today start costing money on **1 October 2026**:

| What | Free since | Charged from |
|---|---|---|
| **Service messages** — any non-template reply inside the 24-hour customer service window | 1 Nov 2024 | 1 Oct 2026 |
| **Utility templates** delivered inside an open customer service window | 1 Jul 2025 | 1 Oct 2026 |

### What does NOT change

This is a billing change, not a policy change. Specifically:

- Service messages remain **free-form**. No template. **No Meta pre-approval.**
- The 24-hour customer service window works exactly as before.
- You still need an approved template only to *start* a conversation or to reply after the window has closed.
- The **72-hour free entry point window** (Click to WhatsApp Ads, Page CTA button) stays free.

If you were worried you would have to convert your automated replies into approved templates — you do not.

### What to know about the new charges

- Service messages are charged at the **same rate as utility and authentication** messages for that market.
- **There are no volume tiers for service messages.** Utility and authentication keep their tiers; service does not. Higher volume does not lower the rate.
- **One charge per message.** A non-template message containing promotional content does *not* additionally incur a marketing charge.
- Meta publishes the rates effective 1 October **by 1 September 2026**.

### Work out your exposure now

Every message status webhook carries a `pricing` object. The messages that become billable are exactly those where `pricing.type` is `free_customer_service`:

```javascript
// In your webhook handler
if (event.startsWith('message.') && data.pricing) {
    await db.messagePricing.upsert({
        messageId: data.message_id,
        billable:  data.pricing.billable,
        type:      data.pricing.type,       // 'regular' | 'free_customer_service' | 'free_entry_point'
        category:  data.pricing.category,   // 'service' | 'utility' | 'marketing' | ...
        origin:    data.conversation?.origin ?? null
    });
}
```

```sql
-- Your added monthly cost = this count × the rate Meta publishes
SELECT category, COUNT(*) AS becomes_billable
FROM message_pricing
WHERE type = 'free_customer_service'
  AND created_at >= NOW() - INTERVAL '30 days'
GROUP BY category;
```

`pricing` is absent on some events (commonly `read`). Always null-check it.

> **Do not hardcode rates.** They vary by country and Meta may change them quarterly. Count volume now; apply the rate when it is published.

### Meta Business Agent — not you, but worth knowing

On 1 July 2026 Meta launched its own AI agent as a separate message category, billed per token from 1 August 2026. Meta's documentation is explicit that any non-template message *not* powered by Meta Business Agent — including messages from a human agent **or a third-party AI solution** — is a **service** message. Everything you send through Bridge is service traffic.

It matters to you for one operational reason, covered next.

### Build this: the "Meta Business Agent took over" banner

If a client enables Meta Business Agent on their number, Meta's AI and your app coexist on that number and only one is the **active handler** at a time. While Meta's agent holds the conversation, Meta **stops sending you `message.received`** and sends `standby.*` events instead.

From your client's point of view, their automation simply stops replying. Nothing errors. **Nothing appears in your logs** — the messages never reached you. They will open a ticket saying the bot is dead, and you will have nothing to show them.

**Please build a persistent warning banner** — in your app header and in the inbox for the affected number:

- **Raise it** on any `standby.*` event
- **Clear it** when normal `message.received` events resume
- Use a **warning** style (yellow/amber), not an error style — nothing has broken
- Make the raise **idempotent**; standby events can arrive many times a minute

Suggested wording:

> 🤖 **Meta Business Agent is answering your customers — not your AI.** To take back control, turn it off in WhatsApp Manager → Account tools → Business Agent. Manual chat still works normally.

And critically — **never auto-reply while in standby**:

```javascript
if (data.standby) {
    await alerts.raise({ clientId, type: 'meta_agent_active', severity: 'warning' });
    return saveContextOnly(data);   // observe only
}
```

Sending a service message during standby makes *your* app the active handler, which is the correct way to escalate to a human — but if a bot does it automatically, you end up with two AIs fighting over one conversation.

WasapFlow ships this exact banner in its own product. Mirroring it means your clients get an explanation instead of a mystery. Full payload reference: [`standby.*` events](?tab=api#standby--meta-business-agent-is-handling-the-conversation-).

### Pricing calendar

Meta may change rates only on 1 January, 1 April, 1 July, or 1 October, with at least one month's notice for a rate card update. You will see any pending change in the `X-Bridge-Notice` header on every API response before it takes effect.

---

## Monitoring — What to Watch

### Dashboard (partner.wasapflow.com/dashboard)
- **Active WABAs** — how many client accounts are registered
- **Requests (24h)** — total API calls in last 24 hours
- **Success / Errors** — breakdown of request outcomes

### Logs (partner.wasapflow.com/logs)
Each log row shows:
- **Time** — when the request was made
- **WABA ID** — which client account
- **Direction** — `outbound` (you → Meta) or `inbound` (Meta → you)
- **Endpoint** — API endpoint called
- **Status** — HTTP response code (200 = ok, 4xx/5xx = error)
- **Meta Error Code** — if Meta returned an error (e.g. `#132001` = template not found)
- **Duration** — response time in ms

### WABAs (partner.wasapflow.com/wabas)
- List of all registered client WABAs
- Status of each (active/inactive)
- Last activity timestamp
- Quality rating and throughput tier (from Meta)

### WABA Quality & Tier
| Quality | Meaning | Action |
|---------|---------|--------|
| 🟢 GREEN | Good standing | No action needed |
| 🟡 YELLOW | Warning — user complaints increasing | Review message content |
| 🔴 RED | High complaints — messaging limit may drop | Pause campaigns, review opt-ins |

| Tier | Daily Message Limit |
|------|---------------------|
| TIER_1K | 1,000 conversations/day |
| TIER_10K | 10,000 conversations/day |
| TIER_100K | 100,000 conversations/day |
| TIER_UNLIMITED | Unlimited |

---

## Billing — WABA Slots

### How it works
- **Creating an account is free** — no credit card, no expiry
- Save a card when you are ready; nothing is charged at that point
- Your **first WABA opens a slot**: $10.00 is charged that day and your monthly cycle starts from that date
- One slot holds **3 active WABAs**. The 4th WABA opens a second slot, prorated for the rest of the cycle
- Removing WABAs lowers your slot count from the next cycle; days already paid are not refunded

### Pricing
| Active WABAs | Slots | Per month |
|------|------|------|
| 0 | 0 | **Free** |
| 1 – 3 | 1 | **$10.00 USD** (net) |
| 4 – 6 | 2 | $20.00 USD |
| 7 – 9 | 3 | $30.00 USD |
| + Stripe processing fee | | $0.31 flat, per invoice |

### Managing billing
- **Save card** — click "Save Card" in the Billing page. Nothing is charged until your first WABA
- **Update card / download invoices** — click "Manage Billing" → Stripe Customer Portal
- **Payment failed** — API is blocked immediately. Click "Pay Now" link in dashboard to pay outstanding invoice and restore access

---

## API Reference Summary

**Base URL:** `https://officialapi.wasapflow.com/bridge/v1`  
**Auth header:** `x-partner-key: wf_your_key`  
**Content-Type:** `application/json`

### Standard Response Format
```json
// Success
{ "success": true, "data": { ... } }

// Error
{ "success": false, "error": { "code": "ERROR_CODE", "message": "Human readable message" } }
```

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/clients/register` | Register a client WABA (upsert) |
| `GET` | `/clients` | List all registered WABAs |
| `DELETE` | `/clients/:wabaId` | Remove a WABA |
| `POST` | `/clients/:wabaId/refresh` | Refresh WABA quality/tier + optional token update |
| `POST` | `/clients/:wabaId/resubscribe-webhook` | Reconnect Meta webhook for a WABA |
| `POST` | `/messages/send` | Send text message |
| `POST` | `/messages/template` | Send template message |
| `POST` | `/messages/media` | Send media message |
| `POST` | `/messages/interactive` | Send interactive buttons/list |
| `GET` | `/contacts/:phone` | Check if phone is on WhatsApp |
| `POST` | `/media/upload` | Upload media to Meta, get `media_id` |

---

## Rate Limits

- Default: **200 requests per second** per partner
- If exceeded: `429 Too Many Requests` with `Retry-After` header
- Contact support to increase limit if needed

**Recommended retry pattern:**
```javascript
async function sendWithRetry(fn, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (err) {
            if (err.status === 429) {
                const wait = (err.retryAfter || 1) * 1000;
                await new Promise(r => setTimeout(r, wait));
            } else throw err;
        }
    }
}
```

---

## Security Best Practices

1. **Never hardcode credentials** — always use environment variables
   ```bash
   # .env
   WF_PARTNER_KEY=wf_xxxxxxxxxxxx
   WF_WEBHOOK_SECRET=whs_xxxxxxxxxxxx
   ```

2. **Always verify webhook signatures** — reject requests without valid `x-wasapflow-signature`

3. **Use HTTPS only** for your webhook endpoint

4. **Store access tokens encrypted** — never store Meta tokens in plain text in your DB

5. **Rotate your Partner Key periodically** — regenerate from dashboard every 3–6 months

6. **Restrict API access by IP** — if your server has a fixed IP, whitelist it in your firewall

---

## Common Errors

| HTTP | Error Code | Meta Error | Meaning | Fix |
|------|-----------|-----------|---------|-----|
| `400` | `WABA_NOT_FOUND` | — | WABA not registered | Register the WABA first |
| `400` | — | `#132001` | Template not found or wrong language | Check template name + language code in Meta BM |
| `400` | — | `#131026` | Phone not on WhatsApp | Use `/contacts/:phone` to check first |
| `400` | — | `#130429` | Meta rate limit (per WABA) | Slow down sending |
| `400` | — | `#131047` | Re-engagement window expired | Can only reply 24h after last customer message for free-form text |
| `401` | `INVALID_KEY` | — | Invalid partner key | Check `x-partner-key` header |
| `402` | `PAYMENT_REQUIRED` / `CARD_REQUIRED` | — | No card saved, or payment failed | Save a card, or pay the outstanding invoice |
| `403` | `SUBSCRIPTION_CANCELLED` | — | Account cancelled | Resubscribe via Billing page |
| `429` | `RATE_LIMIT` | — | WasapFlow rate limit hit | Reduce request rate or contact support |
| `500` | `META_ERROR` | — | Meta API error | Retry — if persistent, check Meta status page |

---

## FAQ

**Q: Can I register multiple phone numbers under one WABA?**  
A: One WABA registration in WasapFlow corresponds to one phone number ID. If your client has multiple numbers, register each separately.

**Q: What does it cost to start?**  
A: Nothing. A partner account is free and never expires. You pay $10 only when you connect your first client WABA, and that slot covers up to 3 WABAs.

**Q: Can I remove a WABA and add it back later?**  
A: Yes. Removing a WABA stops new API access for that WABA immediately, but **if it was active at any point during the billing month, it still counts toward your peak WABA count** for that invoice. Re-registering restores full access. Re-registering the same WABA ID in the same month does not increase peak beyond what was already recorded.

**Q: My client's access token expired. What now?**  
A: Generate a new permanent token from Meta Business Manager (System Users), then call `POST /clients/register` again with the new token (it's an upsert — safe to call on existing WABAs). Alternatively, call `POST /clients/:wabaId/refresh` with `{ "access_token": "new_token" }` to update and sync quality rating in one step.

**Q: A client's webhook events stopped arriving (messages not showing, no delivery receipts). What now?**  
A: Call `POST /clients/:wabaId/resubscribe-webhook` to re-subscribe the WABA to WasapFlow's Meta webhook. This is safe to call at any time and does not affect messaging. If the issue persists, also call `/refresh` to ensure the token is valid.

**Q: Can I white-label the dashboard for my clients?**  
A: The partner dashboard is for your own use to manage your WABAs. Your clients interact with WhatsApp directly through your SaaS app — they don't need access to the WasapFlow partner portal.

**Q: Is there a sandbox/test mode?**  
A: Use Meta's test phone numbers (available in your Meta developer app) for testing without real WhatsApp accounts.

**Q: What Meta permissions does my app need?**  
A: `whatsapp_business_management` and `whatsapp_business_messaging`. Your app doesn't need to be verified if you're using WasapFlow as the Tech Provider.

---

## Support

For technical issues, contact your WasapFlow account manager or email **support@wasapflow.com**.

Include in your support request:
- Your Partner Key (first 8 chars only — **never share full key**)
- WABA ID affected
- Timestamp of the issue (UTC)
- Error message or Meta error code
- Request payload (redact any sensitive tokens)

**Response time:** Business hours (GMT+8), typically within 4 hours.

---

*This document is confidential and intended for WasapFlow Bridge Partners only. Do not distribute.*  
*© 2026 Kobaranteguh Sdn Bhd — All rights reserved.*
