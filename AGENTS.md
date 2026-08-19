# Repository Guidelines

## Scope

This repository contains public user documentation for PalServerBridgeRE. It must remain documentation-only.

Do not add the PalServerBridgeRE mod directory, DLLs, source code, SDKs, build files, symbol files, test tools, reverse-engineering notes, real configuration files, tokens, logs, audit data, player data, or machine-specific absolute paths.

## File Rules

- All text files must be UTF-8 without BOM.
- UTF-16 and GBK are forbidden.
- Write Chinese directly; do not encode it as `\uXXXX`.
- Document public behavior only. Do not document internal implementation details.
- Keep examples in Markdown code blocks instead of adding executable helper tools.

## Changes

- Verify examples against the released mod before documenting them.
- Update `VERSION` and `CHANGELOG.md` when documentation changes materially.
- Run encoding, secret, forbidden-file, and Markdown link checks before committing.
- Commit the completed work at the end of every work round.
