# Send to Paperless

A quick and easy solution using [Cloudflare Email Workers](https://developers.cloudflare.com/email-routing/email-workers/) to process
emails and turn them into [Paperless-NGX](https://docs.paperless-ngx.com/) documents.

## Setup

You need to set the following variables and secrets on your Worker:

| Type      | Name                    | Description                                                             |
|-----------|-------------------------|-------------------------------------------------------------------------|
| Secret    | CF_ACCESS_CLIENT_ID     | Service token for Cloudflare Access application                         |
| Secret    | CF_ACCESS_CLIENT_SECRET | Service token for Cloudflare Access application                         |
| Secret    | PAPERLESS_API_BASE      | Full path to Paperless API, e.g. `https://paperless.example.com/api`    |
| Secret    | PAPERLESS_TOKEN         | API token for Paperless                                                 |
| Plaintext | POSTMASTER_EMAIL        | Email address validated in Cloudflare to send unprocessable messages to |
| Plaintext | PAPERLESS_DEFAULT_TAG   | (Optional) Tag ID to apply when no hashtags are specified               |
| Plaintext | PAPERLESS_FORCED_TAG    | (Optional) Tag ID to always apply to all uploaded documents             |
| Plaintext | IGNORED_MIME_TYPES      | (Optional) Comma-separated list of MIME types to skip (e.g. `application/pgp-keys,application/pgp-signature`) |

## Features

### Subject-based Tagging

You can control document tags using hashtags in the email subject line. Tag names are automatically resolved to their IDs via the Paperless API.

**Examples:**

- Subject: `#tax Invoice from vendor` → Tagged with "tax", title: "Invoice from vendor"
- Subject: `#tax #todo #urgent` → Tagged with all three, title uses filename
- Subject: `` (empty) → Uses default tag (if set), title uses filename

### Tag Configuration

The worker supports three tag configuration options with the following priority:

1. **PAPERLESS_FORCED_TAG** - Always applied to every document (if set)
2. **Subject hashtags** - Applied when `#tagname` is found in subject (if found)
3. **PAPERLESS_DEFAULT_TAG** - Applied when no hashtags are found (if set)

**Example scenarios:**

| Config | Subject | Result Tags |
|--------|---------|-------------|
| FORCED=11, DEFAULT=24 | `#tax Invoice` | 11, tax |
| FORCED=11, DEFAULT=24 | `Invoice` | 11, 24 |
| DEFAULT=24 | `#tax #todo` | tax, todo |
| DEFAULT=24 | `` (empty) | 24 |
| (none) | `#tax` | tax |

## Motivation

AKA "But Paperless can read email!"

I didn't have the means to set up an entire separate email inbox and give Paperless credentials. It's super easy to add an email route
via Cloudflare though. So now I can either have `bills@example.com` as the recipient for things that send bills, or forward attachments
myself when the need arises.
