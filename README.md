# ocp-ci-docs

Scenario onboarding documentation for OpenShift CI created by the CSPI team


## Contributing

### Best Practices

- Ensure any new document is linked properly in the [`docs/SUMMARY.md`](docs/SUMMARY.md) file. This is how GitBook will present documents in the generated webpage.
- All pull requests into the `main` branch of this repository should come from a branch in your [personal fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-forks) of the repository.
- Do **NOT** push any changes to the `main` branch of this repository. Any changes to this branch must go through the [pull request process](#pull-requests).

### Pull Requests

- All pull requests must be reviewed by another member of the team before being merged.
- Don't merge your own pull request. Ask someone that did a final review of your changes to merge your pull request when you are ready.
- All pull requests must pass [linting](#linting) before being merged. This check will occur when a pull request is created in GitHub actions.

### Linting

- This repository uses [`markdownlin-cli2`](https://github.com/DavidAnson/markdownlint-cli2) as it's linter.
- The [rules](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md) used for linting can be found in the [`.markdownlint.json`](.markdownlint.json) file.
- Passing the linting is required for merge requests. The GitHub action that enforces that rule can be found at [`.github/workflows/pull_request.yml`](.github/workflows/pull_request.yml).
- To test linting in your local environment prior to creating a pull requests (**recommended**), use the [pre-commit](https://pre-commit.com/) [hook](.pre-commit-config.yaml):
  1. [Install pre-commit](https://pre-commit.com/#install), if you haven't already
  2. Execute `pre-commit run --all-files` in the root of your local copy of this repository
  3. Fix any errors reported by that command.
   

### MarkDown Resources

#### Documentation

- [Official Markdown Documentation](https://www.markdownguide.org/getting-started/)
- [Mermaid Documentation](https://mermaid.js.org/intro/)
  - Used to dynamically generate flow (and other) charts within Markdown

#### Editors

- [Visual Studio Code](https://code.visualstudio.com/docs/languages/markdown) **Recommended**
- [PyCharm](https://www.jetbrains.com/help/pycharm/markdown.html)
- [Obsidian](https://obsidian.md/)
- [Remarkable](https://remarkableapp.github.io/index.html)

#### Plugins

> **NOTE:** 
> As our recommended editor, these plugins are specific to Visual Studio Code

- [Markdown All In One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [Markdown Preview Mermaid Support](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)
- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)
