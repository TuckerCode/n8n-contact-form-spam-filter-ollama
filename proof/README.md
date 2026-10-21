# What these six runs prove, and what they do not

On 2026-10-18 a test copy of `workflow.json` was saved to a self-hosted n8n 2.35.7 as "FREE-TEST Contact form spam filter (Ollama)". Six sample submissions were posted to it one at a time, and the model was a local Ollama `qwen2.5:3b`. The test copy, saved here as `test-copy.workflow.json`, makes three changes to the template. The Form Trigger is replaced by a webhook and a test-only Code node, which pass on the same fields the form produces (Name, Email, Message, submittedAt, formMode). **Email Me The Enquiry** is disabled, so no email was sent. The sticky notes are removed. All six executions ended in `success`, and every one took the route it should have. The cake order, the one-line "Are you open on Sundays?" and the complaint were judged genuine by the model and reached the email node. The SEO pitch was judged spam by the model. The message with three links, and the one with a web address in the Name field, were marked spam by the rules alone. Each record's `decided_by` field says whether the rules or the model decided. The first run took 28.3 seconds while the model loaded. The rest took 4 to 8 seconds. **Not tested:** the real n8n form page in a browser, the Ignore Bots option, sending through a real SMTP account, any model other than `qwen2.5:3b`, non-English messages, and the fallback path where the model's answer cannot be read (it did not happen in these runs). Each `run-N.json` is the full record of one execution.

| run | sample | result |
|---|---|---|
| 1 | cake order for 12 people | genuine (model), forwarded |
| 2 | SEO and backlinks pitch | spam (model), not forwarded |
| 3 | three links in the message | spam (rules: 3 links, limit 2) |
| 4 | "Are you open on Sundays?" | genuine (model), forwarded |
| 5 | complaint about a dry cake | genuine (model), forwarded |
| 6 | web address in the Name field | spam (rules) |

## Real SMTP and form test, 2026-10-20

`workflow.json` was imported into the same local n8n 2.35.7 (running natively on 127.0.0.1, not in Docker) as "FREE-TEST contact form SMTP". The only change was an SMTP credential on **Email Me The Enquiry**. It pointed at a local SMTP catcher (aiosmtpd on 127.0.0.1:2525, no TLS, no login), so no real mail account was involved. The Set Filter Settings values were left as shipped (`you@example.com`, `forms@example.com`, Ollama `qwen2.5:3b` on localhost). The workflow was activated, and the real n8n form at its production URL `/form/8a78a675-b5e9-5a55-9a6b-0fc2c7530e28` was filled in and submitted in headless Chromium through Playwright.

- **Genuine enquiry** (a cake order from `priya.shah@example.com`): the model judged it genuine, and one message reached the catcher. It is saved as `smtp-genuine-enquiry.eml`. Envelope: `forms@example.com` to `you@example.com`. Headers: `From: forms@example.com`, `To: you@example.com`, `Reply-To: priya.shah@example.com` (the visitor), `Subject: New enquiry from Priya Shah`, plain text. The body has the visitor's name, address and message, and then the line `Passed the spam filter (model): order`. The reason line is present, but the model gave a one-word reason instead of a sentence.
- **Spam** (an SEO and backlinks pitch from `mark@seo-agency.example.com`, submitted twice): the model called it spam both times ("unsolicited sales pitch"). The run ended at Spam Not Forwarded, and the catcher received nothing.
- **What the visitor sees**: in all three submissions the page showed "Form Submitted" and "Thanks, we got your message and will reply by email.", so a spammer cannot tell the difference.
- **Ignore Bots**: a `curl`, Googlebot or `HeadlessChrome` user agent got `401` on both GET and POST of the form, and no execution started. The Playwright browser had to send an ordinary Chrome user agent to get the form at all, so anyone testing with a headless browser will see the same 401. A bot that fakes a normal browser user agent gets through, and then it is up to the rules and the model.
- `workflow.json` needed no change.

**Still not tested:** a real hosted SMTP provider (TLS/STARTTLS, a login, and deliverability or SPF for the From address), n8n running in Docker (`host.docker.internal` for Ollama and SMTP), a model other than `qwen2.5:3b`, non-English messages, and the fallback path where the model's answer cannot be read. The test workflow, its three executions and the SMTP credential were deleted afterwards.
