# CloudPedagogy Moodle Course Search — Command Examples

This document provides generic copy-and-paste examples for `src/search_mbz.py`.

Assumptions:

```text
cloudpedagogy-moodle-course-search/
├── input/
├── output/
└── src/
    └── search_mbz.py
```

Replace example terms and filenames with values appropriate to your Moodle courses.

## 1. Show help

```bash
python3 src/search_mbz.py --help
```

## 2. Show version

```bash
python3 src/search_mbz.py --version
```

## 3. Search a single Moodle backup

```bash
python3 src/search_mbz.py \
  input/course.mbz \
  --text "assessment" \
  -o output/single_assessment_search
```

## 4. Search every `.mbz` in the input folder

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  -o output/assessment_search
```

## 5. Search recursively through subfolders

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --recursive \
  --text "assessment" \
  -o output/recursive_search
```

## 6. Search for a phrase

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "learning outcomes" \
  -o output/learning_outcomes
```

## 7. Multiple terms — OR

By default, repeated `--text` terms use OR logic.

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --text "assignment" \
  -o output/assessment_or_assignment
```

A record can contain either term.

## 8. Multiple terms — AND

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --text "assignment" \
  --match-all-terms \
  -o output/assessment_and_assignment
```

Both terms must occur in the same searchable record.

## 9. Exclude unwanted wording

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --exclude-text "formative" \
  -o output/assessment_excluding_formative
```

## 10. Case-sensitive search

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "Assessment" \
  --case-sensitive \
  -o output/case_sensitive
```

Without `--case-sensitive`, matching is case-insensitive.

## 11. Regular expression search

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "week\s+[0-9]+" \
  --regex \
  -o output/week_number_regex
```

## 12. Inventory all forums

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  -o output/forums
```

## 13. Inventory all quizzes

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type quiz \
  -o output/quizzes
```

## 14. Find hidden quizzes

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type quiz \
  --visibility hidden \
  -o output/hidden_quizzes
```

## 15. Find hidden activities of any type

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --visibility hidden \
  --scope activity \
  -o output/hidden_activities
```

## 16. Find visible activities

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --visibility visible \
  --scope activity \
  -o output/visible_activities
```

## 17. Restrict by activity name

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --name-any announcements discussion \
  -o output/selected_forums
```

The activity name must contain at least one supplied phrase.

## 18. Search activity content

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --scope activity_content \
  -o output/activity_content
```

## 19. Search forum message bodies

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --text "deadline" \
  --field message \
  -o output/forum_messages
```

## 20. Require retained forum post/discussion evidence

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --text "deadline" \
  --forum-posts \
  --field message \
  -o output/forum_post_deadlines
```

This can return `UNABLE TO DETERMINE` when a forum exists but the backup contains no detected forum post/discussion records.

## 21. Search forum subjects or messages

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --text "deadline" \
  --field subject \
  --field message \
  -o output/forum_subjects_messages
```

## 22. Combine forum activity-name and message filters

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --name-any announcements discussion \
  --text "deadline" \
  --forum-posts \
  --field message \
  -o output/targeted_forum_search
```

## 23. Restrict records to a section

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --section "assessment" \
  -o output/assessment_section_records
```

This filters records according to the section they belong to.

## 24. Search inside section records

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --scope section \
  -o output/section_text_search
```

This searches section XML itself.

## 25. Search course-level metadata

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "Research Methods" \
  --scope course \
  -o output/course_metadata
```

## 26. Find PDF files

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filetype pdf \
  -o output/pdfs
```

## 27. Find Word documents

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filetype docx \
  -o output/docx_files
```

## 28. Find PowerPoint files

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filetype pptx \
  -o output/pptx_files
```

## 29. Search uploaded filenames

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filename "handbook" \
  -o output/handbook_files
```

## 30. Find assignment-related uploaded files

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --filename "assignment" \
  -o output/assignment_files
```

## 31. Search file metadata as text

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assignment" \
  --scope file \
  -o output/file_metadata
```

This searches the searchable metadata record derived from `files.xml`. It does not search inside uploaded documents.

## 32. Find references to a domain

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --domain example.com \
  -o output/domain_search
```

Subdomains of the supplied domain also match.

## 33. Show only courses with matches

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --text "assessment" \
  --result-mode found \
  -o output/courses_with_assessment
```

## 34. Show courses needing attention

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --text "deadline" \
  --forum-posts \
  --field message \
  --result-mode missing \
  -o output/forum_deadline_missing
```

`missing` includes both:

- `NOT FOUND`
- `UNABLE TO DETERMINE`

The statuses remain distinct.

## 35. Combine several filters

```bash
python3 src/search_mbz.py \
  input \
  --batch \
  --activity-type forum \
  --visibility hidden \
  --text "assessment" \
  --field message \
  -o output/hidden_forum_assessment
```

This is an AND combination of constraints: matching records must satisfy all supplied filters.

## 36. Open the HTML report on macOS

```bash
open output/assessment_search/search_report.html
```

## 37. Inspect generated files

```bash
ls -lh output/assessment_search
```

Expected outputs:

```text
search_report.html
course_summary.csv
matches.csv
search_data.json
run_log.csv
```

## 38. Useful interpretation rule

When reviewing results:

```text
FOUND
```

means matching evidence was found.

```text
NOT FOUND
```

means no searchable record in the backup matched.

```text
UNABLE TO DETERMINE
```

means the backup did not contain sufficient evidence for a reliable answer to that query.

Do not automatically treat `UNABLE TO DETERMINE` as `NOT FOUND`.

## 39. What file searches do not do

These commands:

```bash
--filetype pdf
--filename "assessment"
--scope file
```

search Moodle file metadata.

They do **not** extract and search text inside PDFs, Word documents, PowerPoint files, SCORM packages or H5P packages.
