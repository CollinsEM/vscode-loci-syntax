# Loci Syntax for VS Code

Syntax highlighting for the Loci DSL used to describe rules, data dependencies, and computation steps in Loci-based applications.

## What this extension highlights
- Loci directives and declarations like `$include`, `$type`, type names, storage kinds (`store`, `storeVec`, `param`, `Map`, `MapVec`, `blackbox`, `Constraint`, etc.), and `$rule` forms (`pointwise`, `singleton`, `apply`, `unit`, `default`, `optional`, `constraint`, `blackbox`).
- Rule modifiers and helpers: `constraint(...)`, `conditional(...)`, `option(...)`, `inplace(...)`, `parametric(...)`, `comments(...)`, `prelude`/`compute`/`postlude`, Loci reduction tags (`[Loci::Summation]`), constants like `EMPTY`/`UNIVERSE`.
- `$`-prefixed variables (including `$variable{n=0}`) and Loci namespace calls (`Loci::load_module`, `Loci::makeQuery`, etc.).
- Comments (`//`, `/* */`), string and numeric literals, and the `<-ci->`, `<-`, `->` arrows used in rule heads.
- Mixed Loci/C++: unmatched content falls back to the built-in C++ grammar, especially inside `{ ... }` blocks.

## Quick use
- Install the extension, open a `.loci` file, and confirm the status bar shows **Loci DSL**. Colors come from your theme; this extension provides scopes.
- If VS Code doesn’t auto-detect, switch the language mode to **Loci DSL** manually.

## Customize Loci colors (manual theme override)
- Open user settings JSON: `Preferences: Open User Settings (JSON)` or edit:
  - Linux: `$HOME/.config/Code/User/settings.json`
  - macOS: `$HOME/Library/Application Support/Code/User/settings.json`
  - Windows: `%APPDATA%\\Code\\User\\settings.json`
- Add or merge:
```json
"editor.tokenColorCustomizations": {
  "textMateRules": [
    {
      "scope": "keyword.control.rule-statement.loci",
      "settings": { "fontStyle": "bold" }
    },
    {
      "scope": "support.function.rule-kind.loci",
      "settings": { "fontStyle": "italic" }
    },
    {
      "scope": "entity.name.function.rule-output.loci",
      "settings": { "foreground": "#ffcc66" }
    },
    {
      "scope": "variable.parameter.rule-input.loci",
      "settings": { "foreground": "#82aaff" }
    }
  ]
}
```
- Adjust colors to taste. These scopes come from `syntaxes/loci.tmlLanguage.json` and override your theme for Loci files.

## Rule anatomy (quick reference)
- `$rule` keyword and rule kind (`pointwise`, `singleton`, `apply`, `unit`, `default`, `optional`, `constraint`, `blackbox`).
- Rule head: outputs before the rightmost `<-`, inputs after it; the arrow is highlighted separately.
- Trailing modifiers: `constraint(...)`, `conditional(...)`, `inplace(...)`, `option(...)`, `parametric(...)`, `comments(...)`; markers like `prelude`, `compute`, `postlude`.

## Example
```loci
$include "flowPsi.lh"

$type solution store<real> ;
$type stop_iter param<int> ;

$rule default(stop_iter) { $stop_iter = 1000 ; }

$rule pointwise(dtcfl<-dtmax) {
  $dtcfl = real($dtmax) ;
}

$rule apply(cl->qresidual<-qdot)[Loci::Summation],
  constraint(cl->geom_cells) {
  join($cl->$qresidual, $qdot) ;
}
```

---

## Developer info (build, test, package)
- Prereqs: Node.js 20+ for `vsce` packaging/publishing, npm. Node 18 is enough for `npm install` and `npm run compile`, but `npx @vscode/vsce package`, `login`, and `publish` currently fail there with `ReferenceError: File is not defined`.
- Install deps: `npm install`
- Build once: `npm run compile` (or `npm run watch` while developing)
- Run in VS Code: open the folder, press **Run and Debug** (F5) and pick **Run Extension**; reload the Extension Development Host after edits. For a faster loop, run `npm run watch` in the main window while using F5 to reload.
- Package a VSIX (share/install locally):
```bash
npm install
npm run compile
npx @vscode/vsce package
```
  - Outputs `loci-syntax-<version>.vsix`; install via `code --install-extension loci-syntax-*.vsix`.
  - If `vsce` warns about a missing `repository` field, add your Git repo URL to `package.json` or pass `--allow-missing-repository`.

## Publish to Marketplace
- Publisher for this repo: `StreamlineNumerics` (see `package.json`).
- Before publishing, bump the extension version in `package.json`.
- Use Node.js 20+ for the publish step. Current `@vscode/vsce` in this repo declares `node >= 20`, and Node 18 fails during `vsce login/package/publish`.
- Create or confirm a Visual Studio Marketplace publisher named `StreamlineNumerics`.
- Create an Azure DevOps personal access token with Marketplace `Manage` permission.
- Log in once on this machine:
```bash
npx @vscode/vsce login StreamlineNumerics
```
- Publish a new version directly:
```bash
npm install
npm run compile
npx @vscode/vsce publish
```
- Or publish while bumping the version automatically:
```bash
npx @vscode/vsce publish patch
```
```bash
npx @vscode/vsce publish minor
```
```bash
npx @vscode/vsce publish major
```
- If you prefer uploading through the website instead of direct CLI publish:
```bash
npm install
npm run compile
npx @vscode/vsce package
```
  - Then upload the generated `.vsix` at `https://marketplace.visualstudio.com/manage/publishers/`.
- Official docs: `https://code.visualstudio.com/api/working-with-extensions/publishing-extension`

## Contributing
- Adjust scopes in `syntaxes/loci.tmlLanguage.json`.
- Update `language-configuration.json` if bracket or comment behavior changes.
- Provide example snippets/screenshots in this README to showcase improvements.

## Release Notes
- 0.1.0 — Expand grammar coverage, add language activation, and document install/usage.
- 0.0.1 — Initial scaffold.
