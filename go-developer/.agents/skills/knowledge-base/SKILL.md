---
name: knowledge-base
description: Hierarchical per-project knowledge base stored in ../.knowledge-base/{project}/{section}/{subsection}/. Use this skill whenever the user asks to remember, capture, save, record, write down, or "add this to the knowledge base" — especially when they name a project, section, or functionality ("write down about project A, section B, that..."). Use it equally for retrieval: "check the knowledge base", "read the documentation in the knowledge base for section X", "what do we know about Y", "did we decide anything about Z". Trigger on loose phrasing too ("keep this somewhere", "remind me what we agreed on this integration"). Reads are targeted at one branch of the tree rather than the whole project, so prefer this skill over answering from memory or re-deriving the answer from the current conversation.
---

# Knowledge Base

A durable, human-readable store of what is known about the user's projects. It lives outside any single repository or conversation, so knowledge gathered while wearing one hat (business analyst, developer, QA) is available while wearing another.

The organizing principle is **narrow reads**. The user does not want a few sprawling documents that must be loaded whole to answer a small question. They want many small notes filed in a folder hierarchy, so that "what do we know about billing refunds?" reads three files, not thirty. Every decision below — folder-per-section, one subject per note, a single hierarchical index — exists to serve that.

## When to use it

- The user asks to record something: "add this to the knowledge base", "write this down about project A section B", "remember that", "keep this somewhere".
- The user asks what is known: "check the knowledge base", "read the documentation for section X", "what do we know about Y", "did we decide anything about Z".
- You are about to answer a question about a project the base already covers. Check the relevant branch first — a stored decision beats a plausible reconstruction, and the user should not have to remember which facts they already told you.
- A durable fact surfaces in conversation that the user will clearly need again — a decision, a constraint, an owner, a gotcha. Offer to capture it in one line and let the user say yes; do not file it unasked.

## When not to use it

- **The answer is already in front of you.** If the fact is in this conversation or in a file you have open, use it. A round-trip through the base for something at hand costs context and returns nothing new.
- **Documentation that belongs to a codebase** — READMEs, ADRs, docstrings, comments in the repo — stays in that repo next to the code it describes. The knowledge base is for cross-repository, cross-role knowledge that outlives a single checkout.
- **Transient state** — task lists, plans for the current change, scratch notes, "where I left off". These go stale in days and turn the index into noise.
- **General knowledge** that is not specific to the user's projects. How Postgres window functions work belongs in the model, not in the base; why *this* project needs them belongs in the base.
- **Bulk capture.** Do not summarize a whole conversation, a whole meeting, or a whole document into the base because it might be useful. Capture what the user pointed at. Volume is what makes a knowledge base unreadable.
- **Browsing.** Do not read the base at the start of a session, and do not walk sibling sections in case they are relevant. Open a branch when a question points at it.
- **Secrets** — credentials, tokens, private keys, personal data. Record that the credential exists and where it is managed; leave the value out.

## Where it lives

The root is `.knowledge-base/` in the **parent directory of the agent's working folder**:

```bash
KB_ROOT="$(cd .. && pwd)/.knowledge-base"
```

Layout:

```
.knowledge-base/
├── README.md                          # one line per project
└── {project}/
    ├── INDEX.md                       # hierarchical map of everything in this project
    └── {section}/
        ├── overview.md                # what this section is, as a whole
        ├── {topic}.md                 # one subject per file
        └── {subsection}/
            ├── overview.md
            └── {topic}.md
```

Folders and files are kebab-case. Sections nest as deeply as the subject genuinely does — `billing/refunds/disputes/` is fine — but do not invent layers the user has not implied; an empty intermediate folder is just a longer path to the same note.

Create directories and index files on first write. Never assume they exist.

## Resolving the path

The user's own words map onto the path. "Write this down about project A, section B, functionality C" means `$KB_ROOT/A/B/C/`.

1. **Project** — the product, client, or system being discussed, never the agent role folder you are running from. If the user names it, use that. Otherwise `ls "$KB_ROOT"` and look for a match; near matches count (`acme-portal` covers "the Acme portal").
2. **Section and subsections** — read the project's `INDEX.md` and match against existing branches before creating new ones. Reuse the existing name even when the user's phrasing differs slightly; `billing/` and `billing-and-invoicing/` as siblings is exactly the fragmentation this structure is meant to prevent.
3. **When the user names no section, infer one from the subject and file it there anyway.** Knowledge about invoicing goes to `billing/` even if the word "section" was never said; knowledge about sign-up goes to `onboarding/`. Create the folder, write the note as its own file inside it, and add its entry to the index. Never drop a note at the project root — the root holds `INDEX.md` and nothing else, and a flat pile of files is exactly what the tree exists to prevent.
4. **Create new sections without asking; say which path you used.** Name the path in your reply so the user can correct it in one sentence. Ask first only when two existing branches both genuinely fit and picking wrong would hide the note — the cost of a question is a moment, the cost of misfiled knowledge is that nobody finds it again.
5. **Match the granularity of the knowledge, not of the phrasing.** One fact about Stripe retries belongs in `billing/integrations/stripe-webhooks.md`, not in a general `billing/overview.md` that slowly accumulates everything. When in doubt about depth, one level of section plus a well-named file beats a deep chain of folders holding one file each.

## Capturing knowledge

1. Resolve the path as above.
2. **Decide what the knowledge actually is.** Capture the substance and the *why* — reasoning, constraints, trade-offs — not a transcript. "We use Postgres" ages badly; "we use Postgres because the reporting queries need window functions and the team already runs it in production" stays useful.
3. **Look for an existing home** inside the target folder before creating a file. Adding to the right existing note beats a near-duplicate sibling, because retrieval depends on there being one obvious place to look.
4. **Write one subject per file.** Name the file for its subject, not the occasion: `stripe-webhooks.md`, not `notes-from-tuesday.md`. Content about the section as a whole goes in that folder's `overview.md`.
5. **Split rather than grow.** If a note starts answering more than one question, break it into siblings and update the index. A note the user must skim to find one fact is a note that costs context every time it is read.
6. **Do not restate what another note already says.** Link to it by relative path instead. Duplicated facts drift apart and then contradict each other.
7. **Handle contradictions out loud.** When new information conflicts with an existing note, update the note, keep a short line recording what changed and when, and say so in your reply. Superseded decisions are part of a project's history.
8. **Update `INDEX.md` in the same turn** — always, and especially when you created a section or subsection. An index that has fallen behind the tree sends every future read to the wrong place. Update `README.md` too if the project is new.
9. **Report the path back** so the user can open the file.

### Note format

```markdown
---
title: Stripe webhooks
section: billing/integrations
updated: 2026-09-21
---

# Stripe webhooks

The substance, under whatever headings fit: how it works, decisions, constraints,
open questions, who owns it.

## Sources
Conversation 2026-09-21; `services/billing/webhook.go`; ticket ACME-412.
```

Keep frontmatter to these three fields — the hierarchy and the index already do the work that tags would otherwise do. Add a short `## Summary` only once a note is long enough that skimming it costs something; on a twenty-line note it is noise.

`Sources` matters more than it looks: when a note turns out to be wrong six months later, the source is how the user works out why.

### INDEX.md format

A nested list mirroring the folder tree exactly, one line per entry, each with a descriptor short enough to decide "is my answer in here?" without opening the file:

```markdown
# Acme Portal

One-paragraph description of the project.

## Contents

- **billing/** — invoicing, payments and refunds
  - [overview.md](billing/overview.md) — how money moves through the system
  - **integrations/** — third-party payment providers
    - [stripe-webhooks.md](billing/integrations/stripe-webhooks.md) — retry semantics and idempotency keys
  - **refunds/** — refund rules and approvals
    - [policy.md](billing/refunds/policy.md) — who approves what, and the 30-day window
- **onboarding/** — sign-up and tenant provisioning
  - [overview.md](onboarding/overview.md) — the five steps from invite to first login
```

If one section grows large enough that its branch dominates the index, give that section its own `INDEX.md` and collapse the branch in the project index to a single link. That keeps the top-level map cheap to read, which is the whole point of having one.

## Retrieving knowledge

1. Resolve the project, then **read `INDEX.md` first**. It is the map; it tells you which branch to open.
2. **Read only that branch.** If the user named a section, read the files under that path and stop. Do not walk the whole project folder, and do not read sibling sections on the chance they are relevant — the user chose this structure precisely so that a narrow question costs a narrow read.
3. For cross-cutting questions where the index does not point anywhere obvious, `grep -ril` the project for key terms and open only the files that match.
4. When the section is ambiguous across projects, check the indexes rather than guessing. A confident answer about the wrong system is worse than a question.
5. Read the matching notes **in full** — the reasoning is usually the part the user wants, and it is rarely in the first paragraph.
6. Answer in prose and cite the notes you used by path, so the user can open them.
7. **If the knowledge base has nothing, say so plainly.** Then answer from general knowledge if you can, clearly separated, and offer to capture the answer. The value of this store is that the user can trust what comes out of it; blurring stored facts with inference destroys that trust faster than an empty answer ever would.
8. If notes are stale, contradictory, or thin on the question asked, flag it rather than papering over it.
