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

1. Bump the version number in `__version__`.  E.g., `0.1.0`
2. Commit.
3. Create a tag.  E.g., `hg tag emacs-preload-0.1.0`
4. Get the SHA-1 from `emacs-preload --version`
5. Go to <https://bitbucket.org/adamsmd/emacs-preload/addon/com.releasebucket/releases> and click "Create release".
   - Branch: `default`
   - Version: `0.1.0`
   - Title: `emacs-preload-0.1.0`
   - Description (note two spaces at end of line to form a line break):
       tag: emacs-preload-0.1.0  
       sha-1: 5C712A2ABA07CFCD01AC7385B3793C17811ADEA

       One line description
