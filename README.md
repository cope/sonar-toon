# sonar-toon

**Turn bloated SonarQube JSON into lean TOON your LLM will actually read.**

🔗 **Live tool: [cope.github.io/sonar-toon](https://cope.github.io/sonar-toon/)**

No install, no build, no backend. One HTML file. Paste in, copy out.

---

## Why

SonarQube's `/api/issues/search` response is huge. Every issue drags along hashes, timestamps, authors, paging metadata, clean-code taxonomy, duplicated component keys, and more. Feed that to an LLM and most of your context window evaporates on noise.

**sonar-toon** strips each issue down to what matters for fixing it, groups by file, and emits [TOON](https://github.com/toon-format/toon) (Token-Oriented Object Notation), a compact, indentation-based format that reads like YAML but packs uniform arrays into CSV-style tables.

Typical result: **70–90% fewer characters**, same actionable information.

## Before / After

Input (trimmed, real responses are far worse):

```json
{
  "total": 1, "p": 1, "ps": 100,
  "paging": { "pageIndex": 1, "pageSize": 100, "total": 1 },
  "issues": [{
    "key": "AZk1r2x3QmPl9vT0aBcD",
    "rule": "typescript:S6582",
    "severity": "MAJOR",
    "component": "acme-api:src/services/order.service.ts",
    "project": "acme-api",
    "line": 142,
    "hash": "b1946ac92492d2347c6235b4d2611184",
    "textRange": { "startLine": 142, "endLine": 142, "startOffset": 8, "endOffset": 41 },
    "status": "OPEN",
    "message": "Prefer using an optional chain expression instead.",
    "effort": "5min",
    "author": "someone@example.com",
    "creationDate": "2026-03-11T09:12:44+0000",
    "type": "CODE_SMELL",
    "cleanCodeAttribute": "CLEAR",
    "impacts": [{ "softwareQuality": "MAINTAINABILITY", "severity": "MEDIUM" }]
  }],
  "components": [ "..." ],
  "rules": [{ "key": "typescript:S6582", "name": "Prefer using optional chain expressions" }]
}
```

Output:

```
issueCount: 1
fileCount: 1
files[1]:
  - file: src/services/order.service.ts
    issues[1]{line,rule,ruleName,message}:
      142,typescript:S6582,Prefer using optional chain expressions,Prefer using an optional chain expression instead.
```

## Features

- **Accepts anything Sonar throws at you**: full `/api/issues/search` response, hotspots response, bare array of issues, or single issue object
- **Resolves component keys to file paths** using `components[]` from the response
- **Attaches human rule names** from `rules[]`
- **Groups by file**, sorted by path then line, so related issues sit together
- **Tabular TOON output** when issues in a file share the same shape
- **Keeps secondary locations** (`flows`) as `related` entries, deduplicated
- **Hides complexity nags** (`S3776`, `S1541`, `S1067`, or anything mentioning cognitive/cyclomatic complexity) with one checkbox
- **Live stats**: issue count, chars before → after, percent saved
- **Auto-formats pasted JSON**, line-numbered gutters on both panes
- **Remembers input and options** in `localStorage` across reloads
- **Everything stays in your browser**. Nothing is sent anywhere.

## Options

| Toggle | Default | Effect |
|---|---|---|
| `minimal` | on | Drops `type` and `quickFixAvailable` from each issue |
| `group by file` | on | Nests issues under `files[]`. Off gives a flat `issues[]` table with a `file` column |
| `keep flows` | on | Emits secondary locations as `related[]` |
| `hide complexity` | off | Filters out cognitive/cyclomatic complexity issues |

Fields always kept: `line`, `lines` (multi-line range), `rule`, `ruleName`, `message`, plus `status`/`resolution` only when not `OPEN`.

## Usage

1. Open the Sonar API URL for your project in a browser tab (you are already logged in, so it just works):
   ```
   https://sonar.example.com/api/issues/search?componentKeys=my-project&resolved=false&ps=500
   ```
2. Select all, copy
3. Open [cope.github.io/sonar-toon](https://cope.github.io/sonar-toon/), paste into the left pane
4. Click **Copy output**, drop into your LLM prompt

Prefer the terminal? Same thing with a token:

```bash
curl -u "$SONAR_TOKEN:" "https://sonar.example.com/api/issues/search?componentKeys=my-project&resolved=false&ps=500" | clip
```

Or click **Load sample** to see it work.

## Run locally

```bash
git clone https://github.com/cope/sonar-toon.git
```

Open `index.html` in any browser. That is the whole thing.

## License

MIT
