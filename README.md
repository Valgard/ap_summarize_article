# summarize-article

A Claude Code plugin that turns a web article into a German Markdown summary and keeps
the cleaned original text in the same file.

## Usage

The skill only runs when invoked explicitly. It never triggers on its own.

~~~
/summarize-article:summarize-article <argument>
~~~

The argument decides the mode:

| Argument | Mode | What happens |
|----------|------|--------------|
| URL (`http://…`, `https://…`) | Create | Fetch the article, summarize it, save it with the original text |
| Path to a `.md` file | Update one file | Re-fetch the source URL from the file's metadata and regenerate the file; the file name stays |
| Path to a directory | Update in bulk | Show how many `.md` files will be updated, ask for confirmation, then update each one and report updated vs. skipped |

Summaries are saved to `~/Documents/!AI/article_summaries/`. If a file for the article
already exists, the skill asks whether to overwrite it, abort, or pick a new name.

If an article can't be fetched (paywall, or a 403 that the cURL fallback can't get
past either), the skill stops and says so. It never writes a file from partial content,
and an update leaves the existing file untouched.

## Output

File names follow the pattern *title keywords + source or author + year*, in lower case
with hyphens and without stop words, for example
`spec-driven-development-thoughtworks-2025.md`.

Each file has five blocks, separated by `---`:

1. **Metadata:** author, source URL and publication date
2. **Kernthese:** the article's central claim in one to three sentences
3. **Main part:** one `##` section per topic, roughly 20–30 % of the original length
4. **Fazit:** the conclusion
5. **Original text:** the cleaned article in its original language, inside a collapsed
   `<details>` block

The summary is in German; technical terms may stay in English. The exact format rules
(heading names, metadata layout, fallbacks for a missing author or date) are in
[`skills/summarize-article/SKILL.md`](skills/summarize-article/SKILL.md).

## Repository layout

~~~
.claude-plugin/plugin.json          plugin manifest
skills/summarize-article/SKILL.md   the skill
~~~
