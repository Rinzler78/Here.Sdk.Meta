# Coding Assertions

## Gates

| Command | When | Proves |
| --- | --- | --- |
| `git submodule update --init --recursive` | after clone | Submodules resolve at their pinned commits. |
| `git log --show-signature -1` | before opening a pull request | The commit is signed, as the ruleset requires. |

## Prerequisites

- Git with SSH commit signing configured per clone.
- `gh` authenticated, to open and merge pull requests.
- No pre-commit hook runs in this repository today. The specification requires four blocking checks per language, with no bypass.
- The specification requires English everywhere, checked by cspell.
