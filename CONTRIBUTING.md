# Contributing

## File structure

### Reusable Workflows

All reusable workflows live in `.github/workflows/` (GitHub does not support subdirectories for workflows).

## Guidelines

1. **Pin third-party actions to commit SHAs** -- use `actions/checkout@<sha> # v4` instead of floating tags.
2. **Review third-party actions** before adding them -- check maintenance status and source code.
3. **Document all inputs** -- every `workflow_call` input and action input should have a `description`.
4. **Run actionlint locally** before pushing:
   ```
   brew install actionlint
   actionlint
   ```

## CI

Pull requests are automatically linted with [actionlint](https://github.com/rhysd/actionlint).
