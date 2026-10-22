# Filter contact form spam with a local Ollama model and email only real enquiries

A free n8n workflow. It gives you a contact form, screens every message, and emails you only the ones written by real people about something they want from your business. SEO pitches, link drops and scams stop inside n8n.

The spam check runs on your own machine through [Ollama](https://ollama.com), so there is no API bill and no message text goes to an AI company.

**Download:** [contact-form-spam-filter-ollama.zip](https://github.com/TuckerCode/n8n-contact-form-spam-filter-ollama/releases/latest/download/contact-form-spam-filter-ollama.zip) (the workflow, this README, six saved test runs and one email it really sent). The same files are in this repository.

## What it does

1. Shows an n8n form with three required fields: Name, Email and Message. Every visitor sees the same thank-you text, whatever happens next, so a spammer learns nothing from it. The form's Ignore Bots option is on.
2. Runs plain rules first. A link in the Name field, or more links in the message than `max_links` (2 by default), marks the message as spam whatever the model says.
3. Counts phrases from your `spam_phrases` list (SEO, backlinks, guest post and so on) and passes the hits to the model as a hint. A phrase on its own does not decide anything.
4. Asks Ollama, with a one-sentence description of your business, whether the message is genuine or spam, and why.
5. Emails genuine messages to `notify_email` with Reply-To set to the visitor, so you can answer with one click. The email says why the message passed.
6. Stops spam at **Spam Not Forwarded**. It stays visible in n8n's execution list if you keep production executions.
7. If the model's answer cannot be read, the message is let through. A missed customer costs more than one spam email.

## Setup

1. Install Ollama and pull a model: `ollama pull qwen3:4b-instruct`.
2. Import `workflow.json` into n8n.
3. Open **Set Filter Settings**. Describe your business in one sentence in `business_description`, and set `notify_email` (where enquiries go) and `from_email` (the address your SMTP account sends from).
4. Connect an SMTP credential on **Email Me The Enquiry**.
5. If n8n runs in Docker, change `ollama_url` to `http://host.docker.internal:11434`.
6. Activate the workflow. Open **Contact Form**, copy the production URL and link to it from your website, or share it directly.

## What it does not do

- It is not a CAPTCHA. It does not stop anything from submitting the form. It decides what reaches your inbox.
- It does not reply to the visitor.
- It does not save submissions to a sheet or database. Add a node after **Email Me The Enquiry** if you want that.
- It does not sort genuine messages into sales, support and so on. Each one simply reaches you.
- It reads the form fields Name, Email and Message by those exact labels. Rename a field and you need to change the Code node too.

## Tested

Six sample submissions were run through a test copy on n8n 2.35.7 with `qwen3:4b-instruct`. See [proof/README.md](proof/README.md).

Get more free templates by email: https://stats.sender.net/forms/dR630O/view

Paid templates: https://airankings-templates.whop.site/?ref=gh-contact-form-spam-filter-ollama
