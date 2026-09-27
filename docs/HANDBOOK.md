# CloudPedagogy Moodle Course Search — Handbook

## 1. Purpose

CloudPedagogy Moodle Course Search is a local-first inspection tool for Moodle backup (`.mbz`) files.

It allows users to search Moodle-native XML and uploaded-file metadata without restoring a course backup to Moodle.

The tool is useful when reviewing one course or when applying the same query across many course backups.

## 2. How a Moodle backup is treated

A Moodle `.mbz` file is an archive containing structured Moodle data. The tool safely extracts the archive to a temporary working directory, reads relevant XML, creates a searchable record stream, performs the requested query and then writes reports.

Conceptually:

```text
Moodle .mbz
    |
    +-- Course metadata
    |
    +-- Sections
    |
    +-- Activities/resources
    |     |
    |     +-- activity metadata
    |     +-- activity-specific XML
    |     +-- forum content when retained
    |
    +-- files.xml
          |
          +-- filename
          +-- extension
          +-- MIME type
          +-- Moodle component
          +-- file area
          +-- context
```

The extracted temporary data is not the output of the tool. The persistent outputs are the generated HTML, CSV and JSON reports.


## 3. Creating an appropriate Moodle backup

The quality and completeness of a search depend on the information retained in the Moodle backup. Moodle Course Search cannot inspect data that was excluded when the `.mbz` file was created.

A full archival backup is not necessary for many analysis tasks. A lightweight, purpose-built backup can reduce file size, processing overhead and the amount of unnecessary data being handled.

### 3.1 Structural and metadata analysis

For common searches involving activities, resources, sections, visibility, URLs and uploaded-file metadata, the principal backup option is:

- **Include activities and resources**

This is a useful lightweight starting point when user-generated activity content is not required.

### 3.2 Analysis requiring forum posts or other user-generated activity data

If the analysis needs actual forum discussions/posts or other relevant user-generated activity data, also enable:

- **Include enrolled users**
- **Anonymize user information**, where appropriate and supported by the Moodle configuration

Including enrolled users is important because forum discussions and posts are user-generated activity data. A backup can contain the forum activity itself without containing the discussions/posts created within it.

Anonymisation can reduce unnecessary exposure of user identities when identities are not relevant to the analysis. Users should still follow their organisation's data-protection and information-governance requirements.

### 3.3 Options that can often be omitted

For a lightweight search/inspection backup, information such as the following can normally be omitted unless it is specifically needed for the intended analysis:

- blocks;
- comments;
- badges;
- calendar events;
- user completion details;
- course logs;
- grade history;
- user role assignments;
- groups/groupings;
- content-bank data;
- H5P user state;
- other user/course-history information unrelated to the search.

The exact options available vary by Moodle version and local configuration.

### 3.4 File references and backup size

Depending on the Moodle configuration and backup options, Moodle may retain file references or metadata without embedding every underlying file binary.

For the current Moodle Course Search workflow, this can be advantageous. The tool searches Moodle-native XML and uploaded-file metadata; it does not currently extract and search text inside uploaded PDFs, DOCX files, PPTX files, videos or other binary resources.

A lightweight backup can therefore be considerably smaller than a full archival backup while still retaining the structure and metadata needed for many searches.

### 3.5 Why backup settings affect result status

Consider a forum-post search.

If a forum and its retained posts are present and a post matches, the result can be:

```text
FOUND
```

If retained forum-post evidence is present but no post matches, the result can be:

```text
NOT FOUND
```

If the forum exists but the backup contains no detected forum discussion/post evidence, the appropriate result is:

```text
UNABLE TO DETERMINE
```

This distinction is intentional. Absence of evidence from a purpose-built or incomplete backup should not be treated as proof that the content is absent from the live Moodle course.

### 3.6 Practical lightweight profiles

For many uses, the backup requirements can be summarised as:

```text
Structural / metadata search
    Include activities and resources

Forum-post / user-generated content search
    Include activities and resources
    Include enrolled users
    Anonymize user information where appropriate
```

Only add further Moodle backup components when they are relevant to the analysis being performed.

## 5. Searchable scopes

The `--scope` option can restrict a query to one or more record categories.

### `course`

Searches course-level XML, such as the course fullname or shortname.

Example:

```bash
python3 src/search_mbz.py input --batch \
  --text "Research Methods" \
  --scope course \
  -o output/course_search
```

### `section`

Searches Moodle section XML, including section names, summaries and other retained fields.

```bash
python3 src/search_mbz.py input --batch \
  --text "assessment" \
  --scope section \
  -o output/section_search
```

### `activity`

Represents synthetic activity metadata used for activity inventory-style queries. This is particularly useful with activity type and visibility.

```bash
python3 src/search_mbz.py input --batch \
  --activity-type quiz \
  --visibility hidden \
  --scope activity \
  -o output/hidden_quizzes
```

### `activity_content`

Searches activity-specific Moodle XML, such as book content, forum fields and other activity data.

```bash
python3 src/search_mbz.py input --batch \
  --text "assessment" \
  --scope activity_content \
  -o output/activity_content_search
```

### `file`

Searches metadata derived from `files.xml`.

```bash
python3 src/search_mbz.py input --batch \
  --text "assignment" \
  --scope file \
  -o output/file_metadata_search
```

This does not search inside the binary file itself.

## 5. Text matching

### Default behaviour

Text matching is case-insensitive by default.

```bash
--text "welcome"
```

can therefore match `welcome`, `Welcome` or other capitalisation variants.

### Case-sensitive matching

Use:

```bash
--case-sensitive
```

when capitalisation must match.

### Multiple terms

Repeat `--text`:

```bash
--text "assessment" --text "assignment"
```

By default, terms use OR logic: a record can match either term.

Use:

```bash
--match-all-terms
```

to require every supplied text term in the same searchable record.

This is record-level AND logic, not course-level AND logic.

### Excluding terms

Use repeatable `--exclude-text` filters:

```bash
--text "assessment" --exclude-text "formative"
```

A record that contains an excluded term is removed from the matching records.

### Regular expressions

Use:

```bash
--regex
```

to treat `--text` values as Python regular expressions.

Example:

```bash
--text "week\s+[0-9]+" --regex
```

Invalid regular expressions are reported as errors.

## 6. Activity searches

### Activity type

Use `--activity-type` to restrict results to Moodle activity types:

```bash
--activity-type forum
```

or:

```bash
--activity-type quiz
```

The option is repeatable.

When an activity type is supplied without a text/content query, the tool can operate as an activity inventory and present one human-readable row per Moodle activity.

### Activity names

Use:

```bash
--name-any announcements discussion
```

to require an activity name to contain at least one supplied phrase.

The phrases use OR logic.

## 7. XML field filtering

Moodle activity XML contains named fields/tags such as:

- `name`
- `subject`
- `message`
- `intro`
- `content`

The generic `--field` option restricts matching to specific XML field names.

Example:

```bash
python3 src/search_mbz.py input --batch \
  --activity-type forum \
  --text "deadline" \
  --field message \
  -o output/forum_messages
```

Multiple fields can be supplied:

```bash
--field subject --field message
```

This means records in either field can match.

Field names are actual XML element/tag names discovered in the backup; they are not semantic categories invented by the tool.

## 8. Forum searches and evidence completeness

Forums require special care because a Moodle backup can contain the forum activity while omitting user-generated posts or discussions.

Use:

```bash
--forum-posts
```

when the query specifically requires forum post/discussion evidence.

Example:

```bash
python3 src/search_mbz.py input --batch \
  --activity-type forum \
  --text "deadline" \
  --forum-posts \
  --field message \
  -o output/forum_deadlines
```

When `--forum-posts` is used, the tool distinguishes three important outcomes:

### FOUND

Relevant retained forum evidence exists and at least one record matches.

### NOT FOUND

Relevant retained forum evidence exists, but no record matches.

### UNABLE TO DETERMINE

Forum activity exists, but the backup contains no detected forum post/discussion records sufficient for the requested search.

This prevents a lack of backup evidence from being incorrectly presented as proof that a message does not exist in the live course.

## 9. Sections

Use repeatable `--section` filters to restrict records by section name or number.

```bash
--section "assessment"
```

This is different from:

```bash
--text "assessment" --scope section
```

The first filters records according to the section context they belong to.

The second searches for the word `assessment` inside section records themselves.

## 10. Visibility

Use:

```bash
--visibility visible
```

or:

```bash
--visibility hidden
```

The default is:

```bash
--visibility any
```

A useful inventory query is:

```bash
python3 src/search_mbz.py input --batch \
  --visibility hidden \
  --scope activity \
  -o output/hidden_activities
```

## 11. File metadata

The tool reads uploaded-file metadata from Moodle's `files.xml`.

### File types

```bash
--filetype pdf
```

Extensions can be supplied without the leading dot.

Other examples:

```bash
--filetype docx
--filetype pptx
```

### Filenames

```bash
--filename "assignment"
```

searches uploaded filenames.

The report can retain contextual information such as the associated Moodle activity, section and visibility when that relationship can be resolved from the backup.

### Binary content limitation

File searches do not extract text from uploaded binary files.

A PDF may therefore be found because:

```text
assessment-guide.pdf
```

matches the filename query, but a phrase that exists only inside the PDF will not currently be found.

## 12. Domain searching

Use:

```bash
--domain example.com
```

to find URLs associated with a domain.

Subdomains also match. For example, a query for:

```text
example.com
```

can match:

```text
media.example.com
```

The search can identify domains in different Moodle XML fields where URLs are retained.

## 13. Combining filters

Filters can be combined.

For example:

```bash
python3 src/search_mbz.py input --batch \
  --activity-type forum \
  --visibility hidden \
  --text "assessment" \
  --field message \
  -o output/combined_search
```

This means:

> Find records that satisfy all of these constraints.

It does **not** mean “run four separate searches”.

This distinction is important when constructing complex queries.

## 14. Result modes

The default is:

```bash
--result-mode all
```

### Found only

```bash
--result-mode found
```

restricts the user-facing HTML report, course summary and matches CSV to courses with `FOUND` status.

### Missing

```bash
--result-mode missing
```

retains:

- `NOT FOUND`
- `UNABLE TO DETERMINE`

The distinction between these statuses remains visible.

`search_data.json` and `run_log.csv` retain the complete processing/audit information even when a result mode filters user-facing outputs.

## 15. Batch and single-course operation

### Single backup

```bash
python3 src/search_mbz.py \
  input/course.mbz \
  --text "assessment" \
  -o output/single_search
```

### Folder of backups

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  -o output/batch_search
```

### Recursive discovery

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --recursive \
  --text "assessment" \
  -o output/recursive_search
```

`--recursive` instructs the tool to include `.mbz` files in subfolders.

## 16. Output files

### `search_report.html`

Human-readable report containing:

- query description;
- course summary;
- status;
- match counts;
- evidence;
- source XML;
- contextual activity/section information where available.

### `course_summary.csv`

One row per course included in the user-facing result set.

It includes course identity, result status, match/activity counts, reason and forum evidence counts.

### `matches.csv`

Detailed matching records included in the visible result set.

### `search_data.json`

Complete machine-readable results, including the full course result set.

### `run_log.csv`

Processing outcome for each backup.

## 17. Interpreting absence carefully

A Moodle backup is a representation of a course created using particular backup settings.

Therefore:

> Absence from a backup is not automatically proof of absence from the live Moodle course.

This is especially important for:

- forum posts;
- user-generated content;
- files stored as references;
- content omitted by backup settings.

The tool reports what can be established from the retained backup data.

## 18. Privacy and information governance

Moodle backups can contain:

- student or staff names;
- forum contributions;
- files;
- course materials;
- assessment information;
- other sensitive institutional data.

Use backups only where you have appropriate authority and follow applicable data-protection, retention and information-governance requirements.

CloudPedagogy Moodle Course Search is local-first: it does not upload backups to an external service.

The repository `.gitignore` is designed to prevent `.mbz` files and generated input/output data from being committed accidentally, but users remain responsible for checking what they publish.

## 19. Current limitations

Version 1.4.1 does not:

- restore or modify Moodle courses;
- search inside uploaded PDF/DOCX/PPTX files;
- perform fuzzy or semantic AI search;
- infer the meaning or intent of a message;
- prove that content absent from a backup is absent from the live Moodle course;
- replace human review for pedagogical, accessibility, compliance or quality judgements.

It is a deterministic search and inspection tool over data retained in Moodle backups.

## 20. Good practice

For repeatable course checks:

1. Use consistent Moodle backup settings.
2. Keep the query command with the resulting report.
3. Review evidence rather than relying only on match counts.
4. Treat `UNABLE TO DETERMINE` separately from `NOT FOUND`.
5. Use narrow filters when the question is precise.
6. Use broad text search for discovery and narrower field/scope filters for verification.
7. Avoid committing real backups or generated institutional reports to public repositories.
