# Reusable Social Page Content and Publishing Workflow

Version: 1.0

Last updated: 2026-08-19

Reference implementation: Solar Business Philippines

## 1. Purpose

This guide documents the tested workflow used to plan, create, host, schedule, and verify branded Facebook content for Solar Business Philippines. It generalizes that process so it can be repeated for pages in other niches without rebuilding the system from scratch.

The workflow is designed for a human working with an AI agent. It removes the need to download and manually upload every image by using a public GitHub repository as the media host and Metricool as the publishing system.

Use this guide when launching a new page, preparing a monthly content batch, onboarding another AI agent, or recovering the workflow after a tool or account change.

## 2. Target outcome

A completed publishing batch has:

- A clear page identity and target audience
- Approved content pillars and publishing cadence
- A structured content calendar
- Finished captions that follow the page style rules
- Branded images in source and publishing formats
- Public, direct HTTPS URLs for every publishing asset
- Scheduled Metricool posts with the correct date, time, network, caption, and media
- A verification record containing the resulting schedule IDs and status
- No credentials, customer data, or private source files in the public repository

## 3. System architecture

```text
Page brief
   |
   v
Content strategy and calendar
   |
   +--> Caption production and QA
   |
   +--> Image production and QA
             |
             v
       Optimized JPG files
             |
             v
    Public GitHub asset repository
             |
             v
      Raw HTTPS media URLs
             |
             v
       Metricool scheduling
             |
             v
      Schedule verification log
```

GitHub is the media delivery layer. Metricool receives public raw file URLs, fetches the images, and schedules them to the connected social page. Local image paths are never supplied to Metricool.

## 4. What stays standardized and what changes

### Standardized across every page

- Folder structure
- File naming rules
- Content calendar schema
- Caption quality checks
- Image production stages
- GitHub upload method
- Metricool scheduling sequence
- Duplicate and completeness checks
- Schedule result schema
- Security rules
- Definition of done

### Configured for each niche

- Page name and handle
- Business goal
- Audience
- Geographic market
- Brand voice
- Visual identity
- Logo and reference images
- Content pillars
- Calls to action
- Publishing frequency
- Preferred posting times
- Restricted topics and claims
- Required disclaimers
- Language mix
- Hashtag policy

## 5. Required services and tools

### Primary tested tool path

This document uses one primary automation path so an agent does not have to choose among several incomplete alternatives:

1. GPT image generation and content production in the private campaign workspace
2. Git and authenticated GitHub CLI for public asset upload
3. A connected Metricool agent connector for schedule reads and writes
4. Metricool schedule retrieval for reconciliation

If the Metricool connector is unavailable, the automated procedure stops at the verified public assets and completed calendar. A human may then use Metricool's current UI or a supported CSV import, but that is a separate fallback process and must be verified in Metricool afterward.

### Required

1. A Facebook Page with the necessary publishing permissions
2. A Metricool brand connected to that Facebook Page
3. A GitHub account and public asset repository
4. Git installed and GitHub CLI authenticated for the repository
5. A Metricool connector that can read and create scheduled posts
6. GPT image generation or an equivalent image tool that can export PNG and JPG
7. A private working folder where campaign files are retained
8. A way to validate JSON, HTTPS responses, and image files

### Recommended

- A structured JSON calendar for automated scheduling
- A PNG source archive and optimized JPG publishing files
- A brand reference image and a clean logo file
- A spreadsheet or CSV fallback for manual Metricool import

### Optional

- Canva Pro for Brand Kit, master templates, page duplication, team review, and one-batch export
- Canva Enterprise for data-tagged Brand Templates and automated Autofill
- Google Drive for internal briefs, approvals, and private source documents
- A separate private repository for automation scripts

Canva is optional. Use it only when template consistency, collaborative review, or a single batch export is more important than the stronger custom visuals produced by GPT image generation. It does not replace GitHub asset hosting or Metricool scheduling in the primary tested tool path.

Canva Autofill requires an Enterprise plan and a Brand Template with non-empty data fields. A normal Canva Pro subscription still provides a strong batch workflow through Brand Kit, a multi-page master design, duplication, review, and one-batch export.

## 6. Initial setup for a new page

### 6.0 Required input contract

Do not start production until every required input has a value and an approval owner.

| Input | Required value |
|---|---|
| Page identity | Name, niche, goal, audience, location, and language |
| Approval | Approver name or role and the meaning of `approved` |
| Brand | Logo, color palette, voice, visual reference, and image restrictions |
| Content policy | Pillars, CTAs, hashtag policy, banned style, prohibited claims, evidence sources, and disclaimers |
| Campaign | Inclusive start and end dates, cadence, timezone, networks, and post count |
| GitHub | Owner/repository, default branch, write access, and local checkout path |
| Metricool | Connector access, brand ID, connected provider, timezone, and current plan limits |
| Storage | Private campaign path, public asset path, and retention period |

`approved` means a designated human approver has accepted the final caption, final visual, scheduled date, and call to action. Record the approver and approval timestamp in each calendar record. An AI agent must not change approved content during scheduling.

### 6.1 Create the page profile

Create `page-config.json` before producing content. This becomes the source of truth for the page and should be read by every AI agent working on it.

```json
{
  "page_name": "Example Page",
  "niche": "Example niche",
  "business_goal": "Generate qualified inquiries",
  "audience": {
    "primary": "Primary customer description",
    "location": "Target market",
    "language": "English with light local phrasing"
  },
  "brand": {
    "voice": ["helpful", "credible", "practical"],
    "colors": ["#082B4C", "#FFC21C", "#FFFFFF", "#16C4C8"],
    "logo_file": "brand/logo.png",
    "reference_image": "brand/style-reference.png"
  },
  "content": {
    "pillars": ["education", "proof", "community", "offer"],
    "default_call_to_action": "Send us a message to learn more.",
    "banned_style": [
      "em dash",
      "unsupported guarantees",
      "invented statistics",
      "excessive hashtags",
      "generic AI filler"
    ]
  },
  "publishing": {
    "timezone": "Asia/Manila",
    "networks": ["facebook"],
    "posts_per_week": 4,
    "metricool_brand_id": "REPLACE_WITH_BRAND_ID",
    "github_repository": "OWNER/PUBLIC-ASSET-REPOSITORY",
    "github_branch": "main",
    "github_checkout": "C:/PRIVATE-WORKSPACE/PAGE-SLUG-assets",
    "approval_role": "Page owner"
  }
}
```

Keep `page-config.json` private by default. Never place passwords, access tokens, private customer information, or unpublished business records in it.

### 6.2 Connect Metricool

1. Create or select the correct Metricool brand.
2. Connect the Facebook Page.
3. Confirm the connection is active.
4. Record the Metricool brand ID in the page profile.
5. Confirm the page timezone.
6. Check current Metricool plan and monthly scheduling limits before creating the calendar.

Plan features can change. Treat the current account limits shown in Metricool as authoritative.

### 6.3 Create the public asset repository

Use a separate public repository for publishing media. A simple naming convention is:

```text
PAGE-SLUG-assets
```

Recommended repository layout:

```text
PAGE-SLUG-assets/
  README.md
  docs/
    REUSABLE-SOCIAL-PAGE-PUBLISHING-WORKFLOW.md
  posts/
    2026/
      09/
        post-01-topic.jpg
        post-02-topic.jpg
  manifests/
    2026-09-assets.json
```

The repository is public by design. Only finished publishing assets, public documentation, and non-sensitive manifests belong there.

## 7. Campaign folder structure

Keep the campaign working folder private and outside the public repository. Only the files explicitly identified in the public allowlist may be copied into the public repository.

```text
campaigns/
  2026-09/
    page-config.json
    content-calendar.json
    schedule-results.json
    README.md
    brand/
      logo.png
      style-reference.png
    source/
      post-01-topic.png
      post-02-topic.png
    publish/
      post-01-topic.jpg
      post-02-topic.jpg
```

The `source` folder preserves high-quality artwork. The `publish` folder contains the compressed files uploaded to GitHub.

### File classification

| File type | Location | Public? |
|---|---|---|
| Final optimized JPG | Public asset repository | Yes |
| Public workflow documentation | Public asset repository | Yes |
| Asset manifest without private IDs | Public asset repository | Yes |
| `page-config.json` | Private campaign workspace | No |
| `content-calendar.json` | Private campaign workspace | No |
| `schedule-results.json` and Metricool IDs | Private campaign workspace | No |
| Source PNG and editable artwork | Private campaign workspace | No, unless explicitly approved |
| Credentials and CLI configuration | Secure credential storage | Never |

## 8. End-to-end operating procedure

### Step 1: Define the page brief

Document:

- What the page offers
- Who it serves
- What action a reader should take
- Why the page is credible
- Which claims require evidence
- Which topics are prohibited or sensitive
- What visual style should be consistent

Do not begin image generation before the page name, audience, voice, logo, and visual direction are stable.

### Step 2: Set content pillars

Choose three to five pillars that support the business goal. A useful general mix is:

1. Education: answers common questions and explains the niche
2. Problem solving: addresses pain points, myths, and objections
3. Trust: shows process, standards, proof, or behind-the-scenes work
4. Community: invites comments, opinions, stories, or local discussion
5. Conversion: gives the reader a clear next step

Adapt the mix by niche:

| Niche | Example pillars |
|---|---|
| Restaurant | menu education, kitchen stories, customer favorites, offers |
| Real estate | buyer education, neighborhood guides, listings, financing basics |
| Fitness | technique, routines, recovery, member stories, coaching offers |
| Education | study tips, subject explainers, student progress, enrollment |
| Local services | maintenance tips, process transparency, before-and-after proof, booking |

### Step 3: Choose the publishing cadence

Use Metricool's best-time data when available. If no reliable history exists, start with a consistent test schedule and adjust after four weeks.

The Solar reference batch used four weekly slots in Philippine Time:

- Monday at 10:00 AM
- Wednesday at 12:00 PM
- Friday at 10:00 AM
- Sunday at 10:00 AM

These times are an example, not a universal rule. Select each page's schedule using its audience and actual performance data.

### Step 4: Build the content calendar

Create one record per post. Each record should be complete before scheduling.

```json
{
  "campaign": "2026-09",
  "timezone": "Asia/Manila",
  "posts": [
    {
      "post_key": "example-page-2026-09-001",
      "sequence": 1,
      "status": "approved",
      "approved_by": "Page owner",
      "approved_at": "2026-08-30T15:00:00+08:00",
      "scheduled_at": "2026-09-02T12:00:00+08:00",
      "network": "facebook",
      "pillar": "education",
      "objective": "Explain a common customer question",
      "headline": "EXACT IMAGE HEADLINE",
      "caption_body": "Caption body without CTA or hashtags",
      "call_to_action": "Send us a message to learn more.",
      "hashtags": ["#ExampleNiche", "#ExampleMarket"],
      "final_caption": "Caption body without CTA or hashtags\n\nSend us a message to learn more.\n\n#ExampleNiche #ExampleMarket",
      "source_image": "source/post-01-topic.png",
      "publish_image": "publish/post-01-topic.jpg",
      "repository_path": "posts/2026/09/post-01-topic.jpg",
      "media_url": "https://raw.githubusercontent.com/OWNER/REPOSITORY/COMMIT_SHA/posts/2026/09/post-01-topic.jpg",
      "media_sha256": "SHA256_OF_APPROVED_JPG",
      "caption_sha256": "SHA256_OF_FINAL_CAPTION"
    }
  ]
}
```

Use ISO 8601 timestamps with the numeric timezone offset. For Philippine Time, the offset is `+08:00`.

`post_key` is a stable identifier that never changes during the campaign. `final_caption` is the exact string sent to Metricool. The separate body, CTA, and hashtag fields make review easier, but scheduling must use only `final_caption` so text is not omitted or duplicated.

### Step 5: Write and review captions

Each caption should have:

- A specific opening line
- One clear idea
- Useful or interesting supporting detail
- A natural call to action
- Only relevant hashtags

Default quality rules:

- Do not use em dashes.
- Do not invent facts, testimonials, savings, prices, or performance claims.
- Do not use exaggerated certainty.
- Do not repeat the same opening structure across the batch.
- Do not stuff captions with emojis or hashtags.
- Do not make every post promotional.
- Prefer plain, human language over generic marketing phrases.
- Verify technical, medical, legal, or financial claims with reliable current sources.

Recommended content mix for a four-post week:

- Two educational or problem-solving posts
- One trust or community post
- One conversion-focused post

### Step 6: Create the image brief

Write the brief before generating the image. Keep image text short because long text is more likely to be unreadable or misspelled.

Reusable image prompt:

```text
Create a square 1:1 Facebook post for [PAGE NAME], a page about [NICHE] for [AUDIENCE AND LOCATION].

Use the attached logo and visual reference. Match the established brand palette: [COLORS]. Use a clean, credible, modern composition that remains readable on a mobile screen.

Layout:
- [DESCRIBE TEXT PANEL OR COMPOSITION]
- [DESCRIBE PHOTOREALISTIC OR ILLUSTRATED SCENE]
- Place the logo once near [POSITION]

Exact headline:
"[SHORT HEADLINE]"

Requirements:
- Render the exact headline with correct spelling
- Do not add any other words
- No watermark
- No unsupported claim
- No em dash or en dash
- Strong contrast and safe margins
- Keep important elements away from the outer crop area
```

Generate one image first and approve the visual system before producing the full batch.

### Step 6A: Optional Canva production path

Use Canva only when the page benefits from a fixed template system or Canva-based team review. The default workflow uses GPT-generated images because they produced the stronger results in the Solar Business Philippines reference implementation.

#### Canva Pro path

1. Create a Brand Kit for the page with approved colors, fonts, and logos.
2. Create one square 1080 by 1080 Facebook master design.
3. Keep the logo, text safe zones, color blocks, and recurring elements fixed.
4. Use a multi-page design with one page per post in the campaign.
5. Duplicate the approved master page for each content-calendar row.
6. Replace only the tagged content areas: headline, supporting label, photo, and optional CTA.
7. Use Canva comments or approval status for human review.
8. Export all approved pages together as PNG in one batch.
9. Move the exported pages into the private campaign `source` folder, rename them using `post_key`, and continue with image QA and JPG optimization.

This removes per-post downloading. The human performs one campaign export, after which conversion, GitHub upload, and Metricool scheduling are automated.

When the Canva connector is available, an AI agent can search for the master design, copy it, inspect its content, and edit text and image elements through a controlled editing transaction. Canva requires explicit approval before the agent commits those edits.

#### Canva Enterprise Autofill path

1. Convert the approved master into a Brand Template.
2. Add dataset tags to dynamic elements such as `headline`, `supporting_text`, `hero_image`, and `cta`.
3. Prepare tabular data with one row per post.
4. Search Canva Brand Templates for autofill-capable templates.
5. Select the intended template and inspect its exact dataset schema.
6. Map content columns to template fields and obtain Canva asset IDs for image fields.
7. Confirm the mapping before generation.
8. Run a three-row test sequentially.
9. After approval, autofill the remaining rows sequentially and record every design URL and error.
10. Review the designs, export the approved batch, and continue with the public hosting and Metricool steps.

Example Canva data table:

| post_key | headline | supporting_text | hero_image_asset_id | cta |
|---|---|---|---|---|
| example-page-2026-09-001 | FIVE WORD HEADLINE | Short supporting line | CANVA_ASSET_ID | Learn more |

Canva image Autofill fields require Canva asset IDs. If the calendar contains a public image URL, upload it to Canva first and store the returned asset ID. If a local or chat-attached file is supported by the connected Canva tool, upload the file directly through that tool without first exposing it on a temporary public host.

Large Autofill batches create one design per row and may be rate-limited. Process rows sequentially, continue after isolated row failures, and retain a result table with row, status, design URL, and error.

#### Canva scheduling option

Canva may offer direct social scheduling through its own product interface. Do not schedule the same campaign in both Canva and Metricool. The recommended setup keeps Metricool as the single scheduling and analytics source. Use Canva scheduling only if the page intentionally replaces Metricool for that campaign and documents the change.

### Step 7: Perform image QA

Inspect every image at full size and as a small mobile preview.

Check:

- Headline spelling and exact wording
- Correct logo and logo placement
- No extra words or accidental marks
- No visual artifacts, distorted hands, or unsafe scenes
- Correct brand colors
- Strong contrast
- No clipped text
- No misleading representation of the product or service
- Consistency with the rest of the batch

Regenerate a failed image. Do not rely on a caption to correct misleading or misspelled image content.

### Step 8: Prepare publishing files

Retain the original PNG. Create an optimized JPG for publishing.

Recommended defaults:

- Aspect ratio: 1:1
- Color mode: RGB
- JPG quality: about 85 to 92
- Target size: usually below 1 MB when visual quality remains acceptable
- Filename: lowercase, descriptive, and stable

Example:

```text
post-07-rooftop-solar-maintenance.jpg
```

Do not overwrite the approved source image. Source and publishing formats serve different purposes.

### Step 9: Upload assets through GitHub CLI

Authenticate GitHub CLI using secure credential storage. Never place a token in a script, repository, prompt, or documentation file.

Confirm access:

```powershell
gh auth status
gh repo view OWNER/REPOSITORY
```

If the public asset repository is a local checkout, use standard Git commands to add only the approved files, commit them, and push them.

Batch upload sequence:

```powershell
git status -sb
git pull --ff-only
git add -- posts/2026/09 manifests/2026-09-assets.json
git diff --cached --stat
git commit -m "Add September 2026 publishing assets"
git push origin main
git rev-parse HEAD
```

Do not use a broad add command when the checkout contains unrelated files. If `git pull --ff-only` or the push fails, stop and inspect the conflict instead of overwriting remote work. If the default branch is protected, publish through the repository's required branch and review process.

For automation that uploads a new file one at a time, GitHub's Contents API can be called through GitHub CLI:

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\PATH\TO\post-01-topic.jpg")
$encoded = [Convert]::ToBase64String($bytes)
$payload = @{
  message = "Add post 01 publishing asset"
  content = $encoded
  branch = "main"
} | ConvertTo-Json

$payload | gh api `
  --method PUT `
  -H "Accept: application/vnd.github+json" `
  "repos/OWNER/REPOSITORY/contents/posts/2026/09/post-01-topic.jpg" `
  --input -
```

For batch uploads, a normal local commit and push is more efficient than one API call per image.

Updating an existing Contents API file also requires its current GitHub blob SHA. The safer campaign rule is never replace a published asset. Use a new unique filename and update the private calendar record.

Use an immutable, commit-pinned public URL:

```text
https://raw.githubusercontent.com/OWNER/REPOSITORY/COMMIT_SHA/posts/YYYY/MM/FILENAME.jpg
```

For each URL, require HTTP status 200, an `image/jpeg` content type, nonzero content length, and successful image decoding. Confirm the downloaded SHA-256 matches the approved local JPG. Retry the availability check for a short bounded period after a push, then stop if it still fails.

Create `manifests/YYYY-MM-assets.json` with `post_key`, repository path, commit SHA, public URL, byte length, media SHA-256, and verification timestamp. This manifest is public and must not contain Metricool IDs or private campaign details.

### Step 10: Schedule through Metricool

Use this sequence:

1. Read the Metricool brand settings.
2. Confirm the connected network and timezone.
3. Retrieve existing scheduled posts for the full target range.
4. Reconcile the proposed posts against existing posts using the rules below.
5. Retrieve best-time data if the calendar does not already specify times.
6. Create scheduled posts using the caption and public raw GitHub image URL.
7. Record every returned Metricool schedule ID.
8. Retrieve the schedule again and verify the final batch.

Best-time queries may need to be divided into ranges of seven days or less, depending on the connector.

When retrieving a date range for verification, include the full final day. Use an end timestamp such as `23:59:59` instead of midnight at the beginning of the end date.

Never schedule from a local path such as `C:\...\image.jpg`. Metricool must receive a reachable public HTTPS URL.

### Exact reconciliation and resume rules

Before creating a post, derive its identity from:

- Metricool brand ID
- Network
- Exact scheduled timestamp
- SHA-256 of `final_caption`
- SHA-256 of the approved media file

Classify each calendar record:

- `verified_existing`: one Metricool post matches all available identity fields
- `conflict`: the timestamp matches but caption or media differs
- `missing`: no matching post exists
- `ambiguous`: more than one possible match exists

Create only `missing` records. Stop for human review on `conflict` or `ambiguous`. After every successful create, write the returned ID to a temporary checkpoint file before moving to the next post. On restart, retrieve Metricool first and reconcile again rather than trusting the last local state.

If Metricool does not expose the original media hash or URL, match by brand, network, exact timestamp, and exact final caption, then verify the displayed media manually or through the connector's returned media metadata. Count equality alone is never sufficient.

Use one writer per Metricool brand and campaign. Parallel agents may prepare content, but only one scheduling agent may create or modify posts.

### Step 11: Record schedule results

Create `schedule-results.json` after scheduling:

```json
{
  "campaign": "2026-09",
  "metricool_brand_id": "REPLACE_WITH_BRAND_ID",
  "verified_at": "2026-08-19T12:00:00+08:00",
  "summary": {
    "requested": 16,
    "created": 16,
    "verified": 16,
    "errors": 0
  },
  "posts": [
    {
      "sequence": 1,
      "post_key": "example-page-2026-09-001",
      "scheduled_at": "2026-09-02T12:00:00+08:00",
      "metricool_id": "RETURNED_SCHEDULE_ID",
      "status": "pending",
      "media_count": 1,
      "caption_sha256": "SHA256_OF_FINAL_CAPTION",
      "media_sha256": "SHA256_OF_APPROVED_JPG",
      "media_url": "https://raw.githubusercontent.com/OWNER/REPOSITORY/COMMIT_SHA/posts/2026/09/post-01-topic.jpg"
    }
  ]
}
```

The results file is operational evidence. It prevents uncertainty about whether an AI agent merely prepared a post or actually scheduled it.

## 9. Batch scheduling safeguards

Before the first write action:

- Confirm the exact Metricool brand ID.
- Confirm the target network.
- Confirm timezone and numeric offset.
- Confirm each post is marked approved.
- Confirm every media URL is public and reachable.
- Check the target date range for existing posts.
- Check the account's current scheduling limit.

During scheduling:

- Schedule in groups of five or fewer until the connector has demonstrated reliable behavior for the account.
- Save each returned ID immediately.
- Stop if several consecutive posts fail for the same reason.
- Do not blindly retry a validation error.
- Do not create a second copy when the first result is uncertain. Retrieve the schedule first.
- Preserve every checkpoint and previous results file. Write a new timestamped recovery file instead of overwriting evidence.

After scheduling:

- Verify the requested count equals the created count.
- Verify every post has the correct date and local time.
- Verify every post has one intended media item.
- Verify status is pending or the platform's equivalent scheduled state.
- Scan for duplicates.
- Reconcile every post one-to-one. Use visual spot-checks only as an additional review.

## 10. Failure recovery guide

| Problem | Likely cause | Recovery |
|---|---|---|
| Metricool rejects or cannot fetch an image | A local path or non-public URL was supplied | Upload the JPG to the public repository, verify the raw URL, and use that URL |
| GitHub connector cannot upload binary content | The integration lacks binary write permission | Use authenticated GitHub CLI with a local commit and push or the Contents API |
| GitHub CLI repeatedly asks for authorization | The session or configuration directory is unavailable | Run `gh auth status`, restore access to the intended secure configuration, or authenticate again using normal credential storage |
| A scheduled post seems missing from verification | The end date was interpreted as midnight | Repeat the query using an end-of-day timestamp |
| Metricool update fails with a required-field validation error | The update operation needs the full provider or network payload | Retrieve the existing post, supply all required fields once, and do not repeatedly retry the same invalid request |
| Image text is misspelled | Generated text did not match the prompt | Regenerate and recheck the image before upload |
| Duplicate posts appear | Existing schedule was not checked or an uncertain request was retried | Retrieve the schedule, identify the duplicate by time and caption, then remove only the confirmed duplicate |
| Batch stops at an account limit | The current Metricool plan has reached its allowance | Reduce the batch, reschedule within the allowed period, or change plans after human approval |
| Browser automation is unavailable | Desktop browser integration is broken or restricted | Continue through Metricool and GitHub connectors or CLI. Browser control is not required for the core workflow |
| Public URL returns 404 | Wrong owner, repository, branch, path, or filename case | Compare the URL with the repository path and correct the manifest before scheduling |
| Process stops after a post may have been created | The create succeeded but the result was not saved | Retrieve the schedule and run reconciliation before any retry |
| Only part of a batch completed | Network, permission, or connector interruption | Keep completed IDs, refresh authentication if needed, reconcile, and resume only missing records |
| Git push is rejected | Remote changed or branch protection applies | Pull with fast-forward only, inspect the difference, or use the required review branch |
| Wrong brand or timezone is discovered | Input validation failed | Stop all writes, record affected IDs, and obtain human approval before correcting or deleting posts |
| A scheduled post later fails to publish | Facebook connection, platform validation, or delivery failure | Record the delivery error, reconnect if authorized, and reschedule only after confirming no live post exists |

## 11. Security and privacy rules

- Never publish access tokens, passwords, browser cookies, API keys, or authentication files.
- Never commit GitHub CLI configuration directories.
- Add credential folders, campaign files, source art, and local configuration to `.gitignore` before the first commit.
- Run a secret scan and inspect the staged file list before every public push.
- Do not expose credentials through environment dumps, command arguments, shell history, debug logs, prompts, screenshots, clipboard records, or CI logs.
- Never publish customer names, addresses, phone numbers, quotes, or private project photos without approval.
- Keep drafts and editable source documents outside the public asset repository.
- Treat every public repository file as permanently discoverable.
- Use a private working location for automation scripts that contain internal IDs or business logic.
- Use least-privilege access for every connector.
- Revoke old credentials when an account, computer, or operator changes.

## 12. Quality assurance checklist

### Strategy

- [ ] Page goal and audience are explicit
- [ ] Three to five content pillars are defined
- [ ] Posting cadence fits the current plan limit
- [ ] Calendar balances education, trust, community, and conversion

### Copy

- [ ] Every caption has one clear purpose
- [ ] No em dashes are present
- [ ] Claims are supported and current
- [ ] Calls to action are varied and relevant
- [ ] Hashtags are limited and niche-specific
- [ ] Captions do not repeat the same structure

### Images

- [ ] Headline matches the approved text exactly
- [ ] Logo is correct and appears once
- [ ] No extra words or watermark are present
- [ ] Mobile readability is acceptable
- [ ] Scene accurately represents the niche
- [ ] PNG source and optimized JPG are retained
- [ ] Canva master and campaign design links are recorded when Canva is used
- [ ] All Canva pages were exported in one approved batch

### Hosting

- [ ] Only approved public files were uploaded
- [ ] Filenames and repository paths match the calendar
- [ ] Every raw URL opens successfully
- [ ] No credentials or private content are in the repository

### Scheduling

- [ ] Correct Metricool brand and network were used
- [ ] Existing posts were checked first
- [ ] All timestamps include the correct timezone offset
- [ ] Requested, created, and verified counts match
- [ ] Every scheduled post has the intended media
- [ ] Schedule IDs were recorded
- [ ] No duplicates were found
- [ ] Every calendar record was reconciled one-to-one

### Public repository audit

- [ ] Only allowlisted finished JPGs, public manifests, and public documentation are present
- [ ] No private config, calendar, result, source, credential, or customer files are present
- [ ] Public manifest hashes and URLs match the approved assets

## 13. Definition of done

This workflow has two completion states:

- `scheduled_complete`: all approved posts are correctly present in Metricool in a pending scheduled state
- `published_complete`: each scheduled post was later delivered successfully to Facebook and its live post URL or platform ID was recorded

The initial batch operation ends at `scheduled_complete`. A later monitoring pass is required for `published_complete`.

A campaign is complete only when all of these statements are true:

1. The content calendar is approved.
2. Every caption and image passed QA.
3. Every publishing image is available through a verified public HTTPS URL.
4. Every approved post is present in Metricool at the correct local time.
5. Every scheduled post contains the correct caption and media.
6. The schedule results file records IDs, status, counts, and errors.
7. No private or sensitive information was published.
8. A human can inspect the calendar, public assets, and results without relying on the original AI conversation.
9. Every calendar record has a one-to-one reconciliation result.
10. The approval authority and timestamp are recorded.

For `published_complete`, also verify each post after its scheduled time, record the Facebook delivery status and live identifier, and investigate any failed delivery. Where the network and connector support accessibility text, verify it as part of the live-post check.

## 14. Reusable instructions for another AI agent

Give the agent this guide, `page-config.json`, the approved logo, the visual reference, and the requested campaign dates. Then use this instruction:

```text
Implement the social content workflow documented in REUSABLE-SOCIAL-PAGE-PUBLISHING-WORKFLOW.md for the page defined in page-config.json.

Validate every required input before starting. Prepare the content calendar and branded images for [DATE RANGE]. Follow all style, claim, privacy, and banned-style rules in the page profile. Do not use em dashes in captions or image text.

Retain source PNG files and create optimized JPG publishing files. Upload only approved JPG assets to the configured public GitHub repository using GitHub CLI. Verify every raw media URL.

Before scheduling, inspect the existing Metricool schedule and current account limits. Use the documented identity and reconciliation rules. Schedule only missing approved posts to the configured network and timezone. Checkpoint every returned schedule ID, retrieve the final schedule, and verify each record one-to-one.

Do not expose credentials. Do not use local filesystem paths as Metricool media URLs. Stop and report any unresolved approval, claim, permission, or account-limit issue.
```

## 15. New page onboarding checklist

1. Create the page and confirm administrator access.
2. Connect the page to Metricool.
3. Create the public GitHub asset repository.
4. Authenticate GitHub CLI securely.
5. Create `page-config.json`.
6. Add the approved logo and style reference.
7. Define content pillars, voice, restricted topics, and calls to action.
8. Confirm timezone, cadence, and Metricool plan limits.
9. Produce and approve one sample post.
10. Upload the sample JPG and verify its raw URL.
11. Schedule and verify the sample through Metricool.
12. Produce the remaining batch only after the sample passes.
13. Save the calendar and results files.
14. Review performance after four weeks and update the page profile.

## 16. Niche adaptation procedure

Before approving a new niche profile:

1. Identify the buyer or audience decision the page supports.
2. List the top questions, objections, risks, and desired outcomes.
3. Mark regulated or evidence-sensitive subjects, including health, finance, law, children, safety, property claims, and before-and-after representations.
4. Assign authoritative evidence sources and an evidence review date for sensitive claims.
5. Define mandatory disclaimers and prohibited promises.
6. Define imagery restrictions for people, locations, products, uniforms, equipment, food handling, and results portrayal.
7. Test the page voice and visual system with one sample post.
8. Obtain human approval for the sample and page profile.
9. Choose cadence and content mix based on the business goal and available capacity.
10. Review performance after the first month and revise the profile deliberately.

For multilingual pages, require a fluent reviewer for every language used. Do not rely only on literal translation. Localize terminology, tone, calls to action, and disclaimers.

## 17. Reference implementation notes

The Solar Business Philippines implementation validated the following approach:

- Facebook was connected to Metricool.
- The publishing timezone was `Asia/Manila`.
- A public GitHub repository supplied raw HTTPS image URLs.
- GitHub CLI successfully handled publishing when connector-based binary uploads were restricted.
- Source PNGs were retained and JPGs were used for publishing.
- GPT image generation was retained as the primary visual production method, with Canva documented as optional.
- Metricool scheduling was verified by retrieving the final date range and recording schedule IDs.
- A 22-post batch was created and verified with zero scheduling errors.

This reference proves the architecture, but the page strategy, claims, voice, imagery, cadence, and posting times must be reconsidered for every new niche.
