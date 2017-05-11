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
