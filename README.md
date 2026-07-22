# FCJ Internship Report — Tran Quoc Khanh

Internship report for the First Cloud AI Journey Workforce Bootcamp at Amazon
Web Services Vietnam, 05/05/2026 – 30/07/2026. Built with Hugo and the
[hugo-theme-learn](https://github.com/matcornic/hugo-theme-learn) theme.

The workshop in section 5 documents an **Airport Information Management
System** on AWS: a three-tier architecture combining ECS/Fargate for the
business API with a serverless OCR pipeline (S3 → SQS → Textract → Lambda →
RDS → SES) for extracting data from boarding passes.

## Visual overview

`project-overview.html` is a single self-contained page summarising the
architecture, service coverage and verification evidence — intended for a quick
visual assessment. Open it in any browser; it needs no server and no network.

## Companion package

The workshop's source code ships separately as **`TranQuocKhanh-Project.zip`**
— a runnable prototype of that OCR pipeline. Section 5.6 of the report walks
through running it. This repository contains the report only.

## Structure

```
content/
  _index.md                 Cover page (student information)
  1-Worklog/                Weekly worklog, 13 weeks
  2-Proposal/               Proposal for the solution built
  3-BlogsPosted/            Blogs published
  4-EventParticipated/      Events attended
  5-Workshop/               Airport Information Management System on AWS
  6-Self-evaluation/        Self-assessment
  7-Feedback/               Sharing and feedback
```

Every page exists in two languages: `_index.md` (English) and `_index.vi.md`
(Vietnamese). Front matter fields:

- `title` — page title shown in the sidebar and breadcrumb
- `weight` — ordering within its section
- `pre` — the number prefix shown in the menu (e.g. `<b> 1.1. </b>`)
- `chapter` — `true` renders the page as a chapter cover

Images live in `static/images/<section>/` and are referenced with an absolute
path, e.g. `![Diagram](/images/5-Workshop/ocr-pipeline.png)`.

## Running the site locally

Requires [Hugo extended](https://gohugo.io/installation/) 0.134.3 or later:

```bash
hugo server
```

Then open <http://localhost:1313>. The theme is vendored in
`themes/hugo-theme-learn/`, so no submodule checkout is needed.

## Deployment

`.github/workflows/hugo.yml` builds the site with Hugo extended 0.134.3 on every
push to `main` and publishes `./public` to the `gh-pages` branch. Enable GitHub
Pages on that branch in the repository settings.
