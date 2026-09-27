# CloudPedagogy Moodle Course Search

A local-first command-line tool for searching and inspecting Moodle backup (`.mbz`) files without restoring them to a Moodle server.

CloudPedagogy Moodle Course Search can analyse a single backup or search across a folder of backups. It searches Moodle-native XML content and metadata, including course and section information, activities, forum content retained in the backup, visibility, URLs/domains, and uploaded-file metadata.

> **Current version:** 1.4.1

## Why use it?

Moodle backup files contain useful structured information, but normally need to be restored to Moodle before they are easy to inspect. This tool provides a lightweight way to answer questions directly from `.mbz` files, for example:

- Which courses contain a particular phrase?
- Which courses contain a particular Moodle activity type?
- Which activities are hidden?
- Which forum messages contain a term?
- Which courses contain PDF, DOCX or PPTX files?
- Which uploaded filenames contain a particular phrase?
- Which Moodle records reference a particular web domain?
- Which courses do not match a requested condition?
- Which searches cannot be determined reliably because the backup lacks the required evidence?

The tool is designed for Moodle administrators, learning technologists, instructional designers, digital education teams, course reviewers, QA teams and researchers working with Moodle course backups.

## Key features

- Search a single `.mbz` backup.
- Batch search multiple backups.
- Optional recursive folder discovery.
- Plain-text and phrase searching.
- Multiple search terms using OR or AND logic.
- Regular-expression searches.
- Case-sensitive or case-insensitive matching.
- Exclude unwanted terms.
- Filter by Moodle activity type.
- Filter by activity name.
- Restrict searches to particular XML fields such as `message`, `subject`, `name` or `intro`.
- Filter by section name or number.
- Filter by visible or hidden content.
- Search URLs by domain, including subdomains.
- Search uploaded-file metadata by filename or extension.
- Restrict searches to course, section, activity, activity-content or file scopes.
- Search forum post/discussion evidence retained in the backup.
- Distinguish `FOUND`, `NOT FOUND` and `UNABLE TO DETERMINE`.
- Produce HTML, CSV and JSON outputs.

## What the tool searches

The current version searches Moodle-native XML retained in the backup and uploaded-file metadata.

It can search information associated with:

- course metadata;
- sections;
- Moodle activities and resources;
- activity-specific XML content;
- forum discussions/posts when retained in the backup;
- activity visibility;
- URLs and domains;
- uploaded filenames, extensions, MIME types, Moodle components and file areas.

### Important limitation

The tool **does not search inside uploaded binary files** such as PDF, DOCX, PPTX, SCORM or H5P packages.

For example, it can find a file called `assessment-handbook.pdf`, but it does not currently extract and search the text inside that PDF.

## Repository structure

```text
cloudpedagogy-moodle-course-search/
├── input/
│   └── .gitkeep
├── output/
│   └── .gitkeep
├── src/
│   └── search_mbz.py
├── docs/
│   ├── HANDBOOK.md
│   └── COMMAND_EXAMPLES.md
├── README.md
├── LICENSE
├── requirements.txt
└── .gitignore
```

## Requirements

- Python 3.9+ recommended.
- No third-party Python packages are currently required.

The tool uses Python's standard library.


## Creating a Moodle backup for analysis

Moodle Course Search analyses standard Moodle backup (`.mbz`) files. You do not need to restore the backup to a Moodle server before analysing it.

In Moodle, open the course and use the course backup function to create and download an `.mbz` file. The exact menu location can vary by Moodle version, theme and local configuration.

### Lightweight analysis backups

A full archival Moodle backup is not required for many search and inspection tasks. Creating a purpose-built analysis backup can substantially reduce the size of the `.mbz` file and avoid including data that is not needed.

For most structural and metadata searches, the key backup option is:

- **Include activities and resources**

This supports searches involving course activities, resources and associated Moodle data retained in the backup.

If the analysis needs **user-generated activity content**, such as actual forum discussions or posts, also enable:

- **Include enrolled users**
- **Anonymize user information**, where appropriate and supported by the Moodle configuration

Other backup components — such as logs, grade history, completion data, calendar events, badges and similar user/course-history data — can normally be omitted unless they are specifically required for the intended analysis.

The information Moodle Course Search can inspect always depends on what the backup contains. For example, a forum activity may be present while its discussions/posts are absent. In that situation, a forum-post query may return `UNABLE TO DETERMINE` rather than incorrectly reporting `NOT FOUND`.

Some Moodle configurations may create backups that retain references or metadata for files without embedding every file binary. This can be useful for lightweight analysis because the current version of Moodle Course Search searches Moodle XML and uploaded-file metadata rather than the contents of uploaded PDFs, Word documents, PowerPoint files or videos.

Place downloaded `.mbz` files in the `input/` folder, or provide the path to an individual `.mbz` file directly on the command line.

## Quick start

Place one or more Moodle backup files in `input/`.

### Search one backup

```bash
python3 src/search_mbz.py \
  input/course.mbz \
  --text "assessment" \
  -o output/assessment_search
```

### Search all backups in `input/`

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  -o output/assessment_search
```

Open the generated HTML report:

```bash
open output/assessment_search/search_report.html
```

On Linux, open the file using your normal browser or file manager.

## Useful examples

### Find hidden Moodle activities

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --visibility hidden \
  --scope activity \
  -o output/hidden_activities
```

### Inventory quizzes

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type quiz \
  -o output/quizzes
```

### Search forum messages

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --text "deadline" \
  --forum-posts \
  --field message \
  -o output/forum_deadlines
```

### Find PDFs

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filetype pdf \
  -o output/pdf_files
```

### Find assignment-related uploaded files

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filename "assignment" \
  -o output/assignment_files
```

### Find references to a domain

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --domain example.com \
  -o output/domain_search
```

See [docs/COMMAND_EXAMPLES.md](docs/COMMAND_EXAMPLES.md) for a fuller command cookbook.

## Outputs

Each run creates:

| File | Purpose |
|---|---|
| `search_report.html` | Human-readable cross-course report and evidence |
| `course_summary.csv` | One summary row per displayed Moodle course |
| `matches.csv` | Matching records for displayed results |
| `search_data.json` | Complete machine-readable search results |
| `run_log.csv` | Processing status for each backup |

`search_data.json` and `run_log.csv` retain the complete audit trail even when `--result-mode` is used to restrict the user-facing report/CSV results.

## Result statuses

### FOUND

The requested condition was found in searchable data retained in the backup.

### NOT FOUND

No searchable record in the backup matched the requested condition.

This means **not found in the searchable backup**, not necessarily absent from the live Moodle course.

### UNABLE TO DETERMINE

The backup does not contain enough evidence to answer a particular query reliably.

For example, a course may contain forum activities while the backup contains no retained forum post/discussion records. A forum-post search should not incorrectly report that as `NOT FOUND`.

## Privacy and local processing

The tool processes Moodle backups locally. It does not require Moodle server access and does not upload course content to an external service.

Moodle backups may contain personal, confidential or institutionally sensitive information. Handle backups and generated reports according to your organisation's data-protection and information-governance requirements.

The supplied `.gitignore` excludes `.mbz` files, `input/` contents and generated `output/` contents from Git by default.

## Documentation

- [Handbook](docs/HANDBOOK.md) — concepts, interpretation, scopes and limitations.
- [Command examples](docs/COMMAND_EXAMPLES.md) — practical copy-and-paste examples.

## Relationship to the CloudPedagogy Moodle Course Analysis Platform

Moodle Course Search is designed as an independent companion to the broader CloudPedagogy Moodle Course Analysis Platform. It can be used as a standalone command-line tool.

## Development status

Version 1.4.1 has been tested across single-purpose and combined searches including text, multiple terms, exclusions, activity types and names, XML fields, forum evidence, visibility, sections, file metadata, domains, scopes, result filtering, case sensitivity, regular expressions and batch processing.

## Licence

MIT License.

Copyright (c) 2026 Jonathan Wong.

See [LICENSE](LICENSE).
