---
name: avoid-single-use-wrapper-classes
description: Do not introduce single-use wrapper classes that only re-expose what a standard API already does; call the standard API directly and keep the code simple.
metadata:
    pinned: true
---

# Avoid single-use wrapper classes around standard APIs

The user does not want helper or wrapper classes that exist for one call site and only
re-expose behaviour a standard API already provides. Their words: such a class "is not reused,
it is hard to read and it does something that regular logger does. I want you to minimize or
avoid classes like that. keep it simpler."

The concrete case was .NET logging. Rather than calling `ILogger` directly, a source-generated
`[LoggerMessage]` partial class (`ResilienceLog`, `SyncLog`, `ZammadLog`, ...) had been created
per component. Each wrapped a handful of log statements, added a layer of indirection between
the call site and the message, and duplicated what `logger.LogInformation(...)` already does.
The preference is the plain call at the call site.

The lesson generalises beyond logging: prefer the direct, standard API call over a bespoke
indirection layer, and only introduce a type when it is genuinely reused or carries real logic.

An important corollary concerns how such wrappers get introduced. In this case the wrapper was
not a considered design choice: an analyzer (CA1873, "evaluation of this argument may be
expensive") flagged a log call, and instead of applying the minimal targeted fix — guarding the
one genuinely expensive argument with `logger.IsEnabled(...)` — the whole thing was generalised
into a new class, which then propagated as a repo-wide convention. When a lint or analyzer rule
pushes toward extra indirection, apply the narrowest fix that addresses the actual cost, or
relax the rule deliberately, rather than adding an abstraction to satisfy the tool.
