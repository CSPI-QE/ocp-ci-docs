# Step Registry Documentation Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->
- [Introduction](#introduction)
- [How to Document a Step in the Step Registry](#how-to-document-a-step-in-the-step-registry)
  - [Getting Started](#getting-started)
  - [Adding Documentation](#adding-documentation)
    - [Purpose](#purpose)
    - [Process](#process)
    - [Requirements](#requirements)
- [Automatically Generated Documentation](#automatically-generated-documentation)
- [Markdown Resources](#markdown-resources)
  - [Documentation](#documentation)
  - [Editors](#editors)
  - [Plugins](#plugins)

## Introduction
Documenting new steps in the step registry is important because it can help use work more efficiently as well as helping users avoid duplicating work in the Step Registry. Duplicated work and lack of documentation, at the time of writing this document, seems to be an issue in the Step Registry and we would like to start solving that issue. When adding a new workflow, ref, chain, _etc._ to the Step Registry, please follow the documentation guide below to write helpful documentation for the new step.

## How to Document a Step in the Step Registry
All steps in the Step Registry, at least written by the Interop team, should have a `README.md` file associated with it in the same folder as the config and BASH script for that step. Documentation should be written in Markdown. If you are unfamiliar with Markdown, please use [the list of resources](#markdown-resources) at the end of this documentation to familiarize yourself with it.

### Getting Started
1. Create the `README.md` file in the same directory that holds the configuration file for your new step.
2. Add a level-1 header to the top of the file with the name of the step. 
   - **Note:** Please add `<!-- omit from toc -->` to the end of the header to avoid adding it to the Table of Contents.
   - **Example:** `# interop-tooling-new-step <!-- omit from toc -->` 
   
3. Add a level-2 header below the first header with the text "Table of Contents"
   - **Example:** `## Table of Contents <!-- omit from toc -->`
4. Add the Table of Contents
   1. In Visual Studio Code, make sure the [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) plugin is installed
   2. Navigate to the line below the "Table of Contents" header and press `CTRL+SHIFT+P` to bring up the Command Pallette.
   3. Type in `Markdown All in One: Create Table of Contents`
   4. Press enter
   - **Note:** This plugin will create the Table of Contents for you and keep it automatically updated with hyperlinks upon every save of the document.

### Adding Documentation
The outline of these documents should generally follow the following structure:
- Purpose
- Process
- Requirements
  - Variables
  - Infrastructure
  - Credentials

If you'd like to expand on this structure, please feel free to. But, this structure should be the minimum for each new step created.

#### Purpose
This section of your document should outline the reason this step was created and what purpose it serves. This section doesn't need to be more than a paragraph or two, but make sure to mention if this step was created for a specific scenario or to be used by multiple scenarios.

#### Process
This section of your document should describe how your new step works. You should try to go into a reasonable amount a detail on how the step works. Here are some things you could include:

- If this step is a workflow or chain, link the steps it uses in order and why each step is needed
- If this step is a ref, outline how the BASH script is used in the ref and how it works
- If this step utilizes any specific (important) files in the test repository, provide a link to the file in GitHub

The documentation doesn't need to exhaustive, but, there should be a general description of how the step works and any outside resources it may utilize. Aim to give someone enough information so that they don't have to step through the code to figure out what your new step does and why.

#### Requirements
This section can provide very useful information to anybody who may need to utilize or debug this step. Make sure to explain why the following requirements are needed and how to get them.

##### Variables<!-- omit from toc -->
Each environment variable in the `env` list of your new step should be outlined. Use the following as a guide for how to document these variables. We will start with an example configuration file:

<sub><sup>`interop-tooling-new-tool-ref.yaml`</sup></sub>
```yaml
ref:
  as: interop-tooling-new-tool
  from: cli
  commands: interop-tooling-new-tool-commands.sh
  env:
  - name: USERNAME
    documentation: Some username for some reason
  - name: PASSWORD
    documentation: Some password for some reason
    default: "reallyC00lP@ssw0rd"
  documentation: |-
    Do something cool in this new tool...
```
This configuration file shows that we have defined two required environment variables in the `env` stanza. These two environment variables have a brief description that is used in the [automatically generated documentation](#automatically-generated-documentation). While it may seem simple to just check the config for these requirements, it is nice to have them in the `README.md` for ease of access when somebody is reading the documentation.

To document these two variables, we will uses the following format:

```markdown
### Variables

- `USERNAME` 
  - **Definition**: The username used in some part of the script. VERY IMPORTANT
  - **If left empty**: The script will fail.
- `PASSWORD`
  - **Definition**: The password associated with the USERNAME variable.
  - **If left empty**: A default password of "reallyC00lP@ssw0rd" will be used.
```
Feel free to add additional information, the information above is mostly a suggestion.

##### Infrastructure<!-- omit from toc -->
There isn't a specific format associated with this section, but it is important. Please make a list of any infrastructure (inside or out of the test cluster) required to run this step. Some examples of this could be:
- A provisioned test cluster
- A container running in the test cluster
- Specific operator(s) that need to installed
- A resource outside of openshift (specific repository or access to a server)

##### Credentials<!-- omit from toc -->
Each item in the `credentials` stanza of the step's configuration should be outlined. Use the following as a guide for documenting these credentials. We will start with an example configuration file:

<sub><sup>`interop-tooling-new-tool-ref.yaml`</sup></sub>
```yaml
ref:
  as: interop-tooling-new-tool
  from: cli
  commands: interop-tooling-new-tool-commands.sh
  credentials:
    - namespace: test-credentials
      name: new-tool-credentials
      mount_path: /tmp/secrets/credentials
```
This configuration file shows that we have defined one credential. Please use the outline below to document any credentials in your new step:

```markdown
### Credentials

- `new-tool-credentials`
  - **Collection**: LINK TO VAULT COLLECTION
  - **Usage**: Used to encrypt a file or something...
  - **Mount Path**: `/tmp/secrets/credentials`
```

In the `usage` section of this documentation, please try to be specific and give any additional information you may  think is needed to understand what that credential is and why it is needed in this step.

## Automatically Generated Documentation
OpenShift CI has implemented a great way to autogenerate documentation for a step, but it does not give as a good way to be as specific as we'd like to be in our documentation. The documentation can be found at the following link: https://steps.ci.openshift.org/

This autogenerated documentation is build using the `description` items in step configuration files. While helpful, it is hard to be verbose in how you document your step. Simpler steps may be able to be documented in this way, but we find the `README.md` documents to be much more helpful.

## Markdown Resources

### Documentation
- [Official Markdown Documentation](https://www.markdownguide.org/getting-started/)
- [Mermaid Documentation](https://mermaid.js.org/intro/)
  - Used to dynamically generate flow (and other) charts within Markdown

### Editors
- [Visual Studio Code](https://code.visualstudio.com/docs/languages/markdown) **Recommended**
- [Obsidian](https://obsidian.md/)
- [Remarkable](https://remarkableapp.github.io/index.html)

### Plugins
> **NOTE:** 
> As our recommended editor, these plugins are specific to Visual Studio Code

- [Markdown All In One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [Markdown Preview Mermaid Support](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)
- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)