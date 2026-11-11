---
title: 'Publish an automatic explainer book index on GitHub Pages'
type: 'feature'
created: '2026-11-11'
status: 'done'
route: 'oneshot'
review_loop_iteration: 0
context: []
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The explainer-books directory contains five Markdown books without a website entry point. Readers need a home page linking to every book's overview, with new books appearing after they are pushed to GitHub.

**Approach:** Configure GitHub Pages' supported Jekyll build, discover immediate subfolders through their 00-overview.md files, preserve Markdown content and chapter links, and document publishing and adding books. Make only local, reversible changes; GitHub repository creation and publishing remain outside this change.

</frozen-after-approval>

## Implementation Notes

- Existing content: five direct book subfolders, each containing 00-overview.md and three chapters. All overviews have H1 titles; no Markdown front matter exists.
- Add _config.yml, index.md, a book layout with return navigation, Gemfile, README.md, and .gitignore. Reuse GitHub Pages' supported optional-front-matter, titles-from-headings, and relative-links plugins.
- Use GitHub Pages repository metadata for deployment URLs, avoiding unknown username/repository placeholders. Exclude tooling, documentation, dependency files, and BMAD artifacts from publishing.
- Verify five overview entries and existing chapter targets; attempt a real Jekyll build if dependencies are available. Check automatic discovery with a temporary additional book and exclude folders without an overview.
- No Git repository exists, so the VCS sanity check is skipped. No intent gaps or irreversible operations are needed for local setup.
- Added all six site/support files. The book layout links to both the library and the book's own overview; original book Markdown is unchanged. Included WEBrick for local serving with Ruby 3.3.
- Verification passed: Ruby YAML/front-matter parsing, Gemfile syntax, five H1 overview titles, and resolution of all 25 local links across 20 Markdown files. Verified against official dependency metadata that GitHub Pages uses github-pages 232, Jekyll 3.10, and Ruby 3.3; the metadata plugin fills unset production URL/baseurl values.
- Full Jekyll rendering and automatic-discovery fixtures were not run: RubyGems dependency installation failed because rubygems.org could not resolve in this restricted environment. README documents preview and deployment smoke checks; this remains an explicit verification limitation.
- The BMAD runtime was absent and was materialized using its required setup flow. Generated tooling and spec artifacts are excluded from publishing. No Git repository or remote exists, so no commit, push, or deployment was made.

## Review Triage Log

- **low, patched:** Chapters only linked to the library. Confirmed the existing final chapters lack a way back to their book's chapter list. Added a shared overview link using the matching page's generated URL.
- **low, rejected:** An empty library produces an empty list. The directory currently contains five valid books, and the intended workflow adds books; handling removal of every book is an uncommon condition. No additional empty-state logic was added.
- **low, partially patched/remainder rejected:** README did not explain output verification. Added repeatable manual checks for the index, chapter/navigation links, project-prefix URLs, and new-book discovery. A dedicated generated-site test harness would exceed a simple correction for this reversible site configuration change. Full generated-output validation remains unavailable until dependencies can be installed.
