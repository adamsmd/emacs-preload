# Development Guidelines

## Dependencies

This program is designed to be self contained:

- No installation needed
- No third party libraries needed

## Performance

- Startup should be below human perception threshold.  (We currently are
  borderline.)
- Prefer library methods over spawning processes

## Messages

- Don't capitalize errors (they must be compatable with `because()`)
- Errors thrown from a given `<line>` include `[#<line>]` at the start to help
  debugging.
- All other documentation capitalized

## Code style

- Indent four columns (even for wrapped statements)
- Line length limit: 100.
- Linters to check against (we currently many warnings from these that we
  don't care about, but maybe in the future we will have customized style
  files):
  + pylint
  + pycodestyle
  + pydocstyle
  + flake8

## Creating a new release

1. Check for outstanding changes
   - `hg diff`
   - `hg pull`
2. Generate usage:
   - `./emacs-preload export-usage`
   - `hg diff`
   - `hg ci -m 'Updated Usage.md'`
3. Bump the version number:
   - `__version__ = 0.1.0 ...`
   - `hg diff`
4. Check for upstream changes again:
   - `hg pull`
5. Commit and tag and push:
   - `hg ci -m 'emacs-preload-0.1.0'`
   - `hg tag emacs-preload-0.1.0`
   - `hg pushh`
6. Get the SHA-1:
   - `emacs-preload --version`
7. Go to <https://bitbucket.org/adamsmd/emacs-preload/addon/com.releasebucket/releases> and click "Create release".
   - Branch: `default`
   - Version: `0.1.0`
   - Title: `emacs-preload-0.1.0`
   - Description (note two spaces at end of line to form a line break):
       tag: emacs-preload-0.1.0  
       sha-1: 5C712A2ABA07CFCD01AC7385B3793C17811ADEA

       Description
