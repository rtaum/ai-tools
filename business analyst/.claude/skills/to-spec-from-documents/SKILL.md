---
name: to-spec-from-documents
description: "Create or update a Markdown specification from documents in a named file or folder. Use only when explicitly invoked as /to-spec-from-documents; do not invoke automatically. Requires both an input file/folder and an output spec path; ask for whichever is missing instead of guessing."
disable-model-invocation: true
---

# Create a Spec from Documents

Turn supplied project documents into a living Markdown specification. This is an explicit, file-driven variant of `to-spec`: the source of truth is the input file or folder, not the current conversation alone.

## Required invocation inputs

Before reading or writing anything, verify that the user supplied both:

- **Input**: a file or folder path containing the source documents.
- **Output**: the Markdown spec filename or path.

If either is absent, stop and ask for the missing value. Never infer a path, filename, or output location.

Accept invocations such as:

```text
/to-spec-from-documents
Input: docs/requirements
Output: docs/spec.md
```

or:

```text
/to-spec-from-documents docs/requirements -> docs/spec.md
```

If the request is ambiguous, ask the user to provide the two values explicitly:

```text
Input: <file-or-folder-path>
Output: <spec-markdown-path>
```

## Workflow

1. Validate that the input exists and is a readable file or directory. If it does not, report the exact path and stop.
2. If the input is a directory, inspect its files recursively. Prefer text and Markdown documents. Read supported text formats available through the environment; skip binaries and generated artifacts, and list skipped files in the final response.
3. Read the existing output spec if it exists before drafting changes.
4. Synthesize the source material into the template below. Resolve duplicated statements conservatively and preserve explicit decisions, terminology, constraints, and open questions.
5. For an existing spec, update only what the supplied documents support. Preserve valid existing content that is not contradicted. Do not silently delete requirements; mark conflicts or obsolete statements in **Further Notes** or **Open Questions**.
6. Write the result to the exact requested output path as Markdown. Create parent directories only when necessary.
7. Quickly verify that the file exists, is non-empty, and contains every required section.

Do not publish to an issue tracker. Do not modify source documents. Do not invent requirements to fill gaps.

## Spec template

Use these headings in this order:

```markdown
# Spec: [Name]

## Problem Statement

[The problem faced by the user or organization, grounded in the source documents.]

## Solution

[The proposed solution, grounded in the source documents.]

## User Stories

[A numbered, comprehensive list of user stories.]

## Implementation Decisions

[Modules, interfaces, technical decisions, architecture, schemas, APIs, and interactions. Do not include brittle file paths or code snippets unless the source material makes a precise snippet necessary.]

## Testing Decisions

[External behaviors to test, test levels/modules, and relevant prior art found in the documents.]

## Out of Scope

[Explicit exclusions and boundaries.]

## Further Notes

[Assumptions, unresolved conflicts, source gaps, traceability notes, and skipped input files.]
```

Use `## Open Questions` inside **Further Notes** when unresolved decisions need explicit follow-up. If the source material does not establish a section, write `Not specified in the supplied documents.` rather than guessing.

## Quality checks

Before finishing, confirm:

- The output path is exactly the one requested.
- All required headings are present and ordered correctly.
- Claims are traceable to the supplied documents or clearly labeled as assumptions/open questions.
- Existing-spec updates do not erase unsupported content.
- Conflicting documents are surfaced rather than arbitrarily resolved.
- Unsupported or skipped files are reported.

Report the output path and a concise summary of whether the spec was created or updated.
