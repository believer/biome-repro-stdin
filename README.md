# Mismatch between stdin and check

This is a problem that occurs in Neovim when trying to format a file using [`conform.nvim`](https://github.com/stevearc/conform.nvim/blob/master/lua/conform/formatters/biome.lua) which uses stdin to format files.

1. Run `./node_modules/.bin/biome check apps/test/index.tsx`. Reports one error.

```
apps/test/index.tsx format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ Formatter would have printed the following content:

    1   │ - const·_t·=·[
    2   │ - ··colors.l3BackgroundTertiary,
    3   │ - ··colors.l3BackgroundPositive,
    4   │ - ··colors.l3BackgroundTertiary,
    5   │ - ]
      1 │ + const·_t·=·[colors.l3BackgroundTertiary,·colors.l3BackgroundPositive,·colors.l3BackgroundTertiary]
    6 2 │


Checked 4 files in 18ms. No fixes applied.
Found 1 error.
```

2. Run `cat apps/test/index.tsx | ./node_modules/.bin/biome check --stdin-file-path=apps/test/index.tsx`. Reports that nothing would change with `--write`.

```
const _t = [
  colors.l3BackgroundTertiary,
  colors.l3BackgroundPositive,
  colors.l3BackgroundTertiary,
]
stdin ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ The contents aren't fixed. Use the `--write` flag to fix them.
```

## Debug messages

For the regular check we have the following. Here it gets the correct configuration for the file.

```
2025-06-19T13:21:35.401375Z  INFO main crates/biome_cli/src/commands/mod.rs: Configuration file loaded: Some("/Users/rickard/code/biome-repro-1750338456375/biome.json"), diagnostics detected 0
2025-06-19T13:21:35.403261Z DEBUG main insert_project: crates/biome_service/src/projects.rs: Insert workspace folder /Users/rickard/code/biome-repro-1750338456375
2025-06-19T13:21:35.404514Z DEBUG main crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/package.json 0
2025-06-19T13:21:35.405682Z DEBUG biome::workspace_worker_0 can_handle: crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0
2025-06-19T13:21:35.405830Z DEBUG biome::workspace_worker_0 can_handle:with_settings_and_language: crates/biome_service/src/workspace.rs: The file has the following feature sets: {Assist: Supported, Search: Supported, Debug: Supported, Lint: Supported, Format: Supported} path="/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx"
2025-06-19T13:21:35.405847Z DEBUG biome::workspace_worker_0 can_handle: crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0
2025-06-19T13:21:35.406054Z DEBUG biome::workspace_worker_0 crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0
2025-06-19T13:21:35.406060Z DEBUG biome::workspace_worker_0 with_settings_and_language: crates/biome_service/src/workspace.rs: The file has the following feature sets: {Lint: Supported, Format: Supported, Assist: Supported, Search: Supported, Debug: Supported} path="/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx"
2025-06-19T13:21:35.406062Z DEBUG biome::workspace_worker_0 crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0
2025-06-19T13:21:35.406312Z DEBUG biome::workspace_worker_0 crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0
2025-06-19T13:21:35.407729Z DEBUG biome::workspace_worker_0 cli_lint_guard:pull_diagnostics: crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0 rule_categories=[Syntax, Lint, Action] path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx project_key=ProjectKey(1) skip=[] only=[]
2025-06-19T13:21:35.417035Z  INFO biome::workspace_worker_0 cli_lint_guard:pull_diagnostics: crates/biome_service/src/workspace/server.rs: Pulled 0 diagnostic(s), skipped 0 diagnostic(s) from /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx rule_categories=[Syntax, Lint, Action] path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx project_key=ProjectKey(1) skip=[] only=[]
2025-06-19T13:21:35.417213Z DEBUG biome::workspace_worker_0 format_with_guard:pull_diagnostics: crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0 rule_categories=[Syntax] path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx project_key=ProjectKey(1) skip=[] only=[]
2025-06-19T13:21:35.417383Z  INFO biome::workspace_worker_0 format_with_guard:pull_diagnostics: crates/biome_service/src/workspace/server.rs: Pulled 0 diagnostic(s), skipped 0 diagnostic(s) from /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx rule_categories=[Syntax] path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx project_key=ProjectKey(1) skip=[] only=[]
2025-06-19T13:21:35.417407Z DEBUG biome::workspace_worker_0 format_with_guard:format_file: crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx 0 path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx
2025-06-19T13:21:35.417440Z DEBUG biome::workspace_worker_0 format_with_guard:format_file: crates/biome_service/src/file_handlers/javascript.rs: JsFormatOptions { indent_style: Space, indent_width: 2, line_ending: Lf, line_width: 100, quote_style: Single, jsx_quote_style: Double, quote_properties: AsNeeded, trailing_commas: Es5, semicolons: AsNeeded, arrow_parentheses: Always, bracket_spacing: BracketSpacing(true), bracket_same_line: BracketSameLine(false), source_type: JsFileSource { language: TypeScript { definition_file: false }, variant: Jsx, module_kind: Module, version: ES2022, embedding_kind: None }, attribute_position: Auto, expand: Auto } path=/Users/rickard/code/biome-repro-1750338456375/apps/test/index.tsx
2025-06-19T13:21:35.418984Z DEBUG biome::workspace_worker_0 format_with_guard: crates/biome_cli/src/execute/process_file/format.rs: Format output is different from input: true
```

Using stdin it doesn't seem to resolve the correct configuration and uses `line_width: 80`, which would indicate why it doesn't want to format the code above.

```
2025-06-19T13:20:25.095485Z  INFO main crates/biome_cli/src/commands/mod.rs: Configuration file loaded: Some("/Users/rickard/code/biome-repro-1750338456375/biome.json"), diagnostics detected 0
2025-06-19T13:20:25.099132Z DEBUG main insert_project: crates/biome_service/src/projects.rs: Insert workspace folder /Users/rickard/code/biome-repro-1750338456375
2025-06-19T13:20:25.101514Z DEBUG main crates/biome_service/src/projects.rs: Get settings for /Users/rickard/code/biome-repro-1750338456375/package.json 0
2025-06-19T13:20:25.102527Z DEBUG main crates/biome_service/src/projects.rs: Get settings for __BIOME_INTERNAL_FILE__.tsx 0
2025-06-19T13:20:25.104583Z DEBUG main crates/biome_service/src/projects.rs: Get settings for apps/test/index.tsx 0
2025-06-19T13:20:25.104808Z DEBUG main with_settings_and_language: crates/biome_service/src/workspace.rs: The file has the following feature sets: {Search: Supported, Format: Supported, Debug: Supported, Lint: Supported, Assist: Supported} path="apps/test/index.tsx"
2025-06-19T13:20:25.104911Z DEBUG main crates/biome_service/src/projects.rs: Get settings for apps/test/index.tsx 0
2025-06-19T13:20:25.105145Z DEBUG main format_file: crates/biome_service/src/projects.rs: Get settings for __BIOME_INTERNAL_FILE__.tsx 0 path=__BIOME_INTERNAL_FILE__.tsx
2025-06-19T13:20:25.105265Z DEBUG main format_file: crates/biome_service/src/file_handlers/javascript.rs: JsFormatOptions { indent_style: Space, indent_width: 2, line_ending: Lf, line_width: 80, quote_style: Single, jsx_quote_style: Double, quote_properties: AsNeeded, trailing_commas: Es5, semicolons: AsNeeded, arrow_parentheses: Always, bracket_spacing: BracketSpacing(true), bracket_same_line: BracketSameLine(false), source_type: JsFileSource { language: TypeScript { definition_file: false }, variant: Jsx, module_kind: Module, version: ES2022, embedding_kind: None }, attribute_position: Auto, expand: Auto } path=__BIOME_INTERNAL_FILE__.tsx
```
