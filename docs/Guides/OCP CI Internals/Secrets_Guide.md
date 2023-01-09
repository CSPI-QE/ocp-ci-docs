# OpenShift CI Interop Scenario Secrets Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->
- [Overview](#overview)
- [Collections](#collections)
  - [Create Collection](#create-collection)
  - [Get Access to a Scenario Collection](#get-access-to-a-scenario-collection)
  - [Get Access to the cspi-qe Collection](#get-access-to-the-cspi-qe-collection)
- [Vault](#vault)
  - [Sign in to Vault](#sign-in-to-vault)
  - [Secrets in OpenShift CI](#secrets-in-openshift-ci)
    - [OpenShift CI Secret Name](#openshift-ci-secret-name)
    - [OpenShift CI Secret Namespace](#openshift-ci-secret-namespace)
- [Tutorial - Create and Use Secrets in OpenShift CI](#tutorial---create-and-use-secrets-in-openshift-ci)
  - [Create a New Secret](#create-a-new-secret)
  - [Use Secret During OpenShift CI Execution](#use-secret-during-openshift-ci-execution)

## Overview
OpenShift CI provides its own Hashicorp Vault instance that we can use. This makes the use of secrets standard for anyone who needs them. Most of everything that we do for secrets was discovered from the OpenShift CI documentation [Adding a New Secret to CI](https://docs.ci.openshift.org/docs/how-tos/adding-a-new-secret-to-ci/)

## Collections

### Create Collection
Each scenario being tested will have its own collection. This allows us to prevent secret sharing between members of other collections meant to hold secrets for other scenarios.

First go to [selfservice.vault.ci.openshift.org](https://selfservice.vault.ci.openshift.org/secretcollection?ui=true) and login.

If you do not see a collection in the table for the scenario that your testing it either hasn't been created yet or you don't have access to it.

Please reach out on slack at [#forum-qe-cspi-ocp-ci](https://coreos.slack.com/archives/C047Y0DPEJU) to ask if the collection for the scenario you are testing has already been created. If it has go to the [next section](#get-access-to-collection)

If it hasn't you can create the collection by clicking the `New Collection` button and adding the name. Follow the naming structure of `{product short name}-qe`.
### Get Access to a Scenario Collection
If you do not see a collection in the table and you have verified that it exists then you just need to be added by one of the collection owners. Please reach out on slack at [#forum-qe-cspi-ocp-ci](https://coreos.slack.com/archives/C047Y0DPEJU) and provide details about what and why you would like to be added to a specific collection.

### Get Access to the cspi-qe Collection
This collection holds team specific information holding things like AWS creds, pull secrets, ..etc. If you need access to this collection reach out on slack at [#forum-qe-cspi-ocp-ci](https://coreos.slack.com/archives/C047Y0DPEJU) and provide details about why you would like to be added to the cspi-qe collection.

## Vault

### Sign in to Vault
Now that you are a member of a collection you can login to vault and access that collection.

Go to [vault.ci.openshift.org](https://vault.ci.openshift.org/ui/vault/auth?with=oidc%2F) and click `Sign in with OIDC Provider`.

You'll be redirected to your secrets homepage. 
`cubbyhole/` is just your users personal space for any secrets you want to store. 
We care about `kv/` this is where you'll find secret directories that correspond to the collections that you are a member of.

From here you can use the UI to view, create, edit, and delete secrets.

### Secrets in OpenShift CI
In order to use a secret in Vault in OpenShift CI, the secret must include two key/value pairs:
- `secretsync/target-name`: The [name](#openshift-ci-secret-name) of the secret in OpenShift CI
- `secretsync/target-namespace`: The [namespace](#openshift-ci-secret-namespace) of the secret in OpenShift CI

Outside of those two values, any key/value pair desired can be added to the same path for that secret making them available for use in OpenShift CI.

#### OpenShift CI Secret Name
The name of a secret in OpenShift CI must be unique. When deciding on a name for a secret, make sure to be descriptive. Using a descriptive name will help with readability in OpenShift CI configuration files as well as making it less-likely that a name is used twice.

Best practice in naming these secrets would be something similar to `scenario_name-name_of_secret`. As an example, for the MTR scenario, the FTP credentials used are named `mtr-ftp-credentials`. This name is unique and descriptive.

#### OpenShift CI Secret Namespace
The Namespace of your secret in the build clusters. Multiple namespaces can be targeted by using a comma-separated list. For the purpose of this document, `test-credentials` will be used as the namespace. This namespace allows for the use of a secret in the step of a job within OpenShift CI.

From the OpenShift CI [Adding a New Secret to CI](https://docs.ci.openshift.org/docs/how-tos/adding-a-new-secret-to-ci/) documentation:

>The most common case is to use secrets in a step of a job. In this case, we require the user to mirror secrets to test-credentials namespace. The pod which runs the step can access the secrets defined in the credentials stanza of the step definition.

## Tutorial - Create and Use Secrets in OpenShift CI

### Create a New Secret
1. [Create a new collection](#create-collection), if one doesn't already exist.
2. [Sign into vault](#sign-in-to-vault) and navigate to `kv/NAME_OF_COLLECTION`.
3. Click the "Create secret" button in the upper right-hand corner.
4. Choose name for the secret in Vault and append that to the text already in the "Path for this secret" textbox.
   - **Note**: This name is not the same as the name described in the [OpenShift CI Secret Name](#openshift-ci-secret-name) section of this document. This name can be anything and does not need to be unique.
5. Add a key of `secretsync/target-name` to the secret and choose a value. This value must be unique and descriptive.
   - **Note**: See the [OpenShift CI Secret Name](#openshift-ci-secret-name) section of this document for additional information.
   - **Example**: `some-test-secret`
6. Add a key of `secretsync/target-namespace` and set the value to `test-credentials`.
   - **Note**: See the [OpenShift CI Secret Namespace](#openshift-ci-secret-namespace) section of this document for more information. 
7. Add the desired key/value pair(s) to the secret for use during execution.
   - **Example**: Add a `username` key with a value of `SomeUser` and a `password` key with a value of `CleverPassword`. These values will be available during OpenShift CI execution.
8. Click the "Save" button in the bottom left-hand side of the screen

Now that a secret has been created in Vault, it is available for use in OpenShift CI steps.

### Use Secret During OpenShift CI Execution
For the following steps, this document will refer to the configuration of a `ref` step in OpenShift CI. For simplicity, here is the configuration that will be referred to:

`some-test-ref.yaml`
```yaml
ref:
  as: some-test-ref
  from: cli
  commands: some-test-ref-commands.sh
  credentials:
    - namespace: test-credentials
      name: some-test-secret
      mount_path: /tmp/secrets/test
  documentation: |-
    Show how to use secrets in OpenShift CI
```

1. Create the step you would like to use your new secret in, if it doesn't already exist.
2. Add a `credentials` stanza to the step's configuration (`.yaml`) file.
3. Under the new `credentials` stanza, add a new list item for your secret. Multiple secrets can be used in a step, but for the sake of simplicity, there is only one in the configuration above.
4. In the new list object, add the following key/value pairs:
   1. `namespace`: The namespace specified in step 6 of the [Create a New Secret](#create-a-new-secret) section of this tutorial. 
      - **Note**: This will usually be set to `test-credentials`
   2. `name`: The name chosen in step 5 of the [Create a New Secret](#create-a-new-secret) section of this tutorial.
   3. `mount_path`: This will be the directory where the secrets are stored for use during execution.
      - **Note**: If you add more than one secret to the `credentials` stanza, you must use a different `mount-path` value for each secret. 
5. Use the new secret in your execution. 
   1. Each key/value pair added to your secret in Vault ,other than `secretsync/target-namespace` and `secretsync/target-name`, will have have a file in the `mount_path` directory. The file will be named the same as the key in Vault and will contain a single clear-text line with the value from Vault.
      - **Example**: 
        - In step 7 of the [Create a New Secret](#create-a-new-secret) section of this tutorial, we created the following key/value pairs:
          - `username`: `SomeUser`
          - `password`: `CleverPassword`
        - In this example, there would be two files found in `tmp/secrets/test` - one file named `username` and one file named `password`. Each file would hold a single clear-text line - the value ("`SomeUser`" and "`CleverPassword`")
    2. Using these values can be as easy as setting the contents of each file to a variable: `USERNAME=$(cat /tmp/secrets/test/username)`