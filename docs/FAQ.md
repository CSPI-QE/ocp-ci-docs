# Frequently Asked Questions (FAQ)

**What do I do if I become aware of leaked credentials or any other security issue?**

If possible you should first deactivate the linked credentials. You can no longer continue to use these creds since they have been exposed.

Once you deactivate the creds you'll need to create new ones and populate them wherever they need to go.

Next report a security incident using ServiceNow.

Lastly follow up with team to discuss prevention of this and have them help you with the security incident you filed with infosec.

**How can I authenticate & pull an image from the OpenShift CI central registry to test it locally?**

Everything you need to know is in [this document](https://docs.ci.openshift.org/docs/how-tos/use-registries-in-build-farm/#how-do-i-log-in-to-pull-images-that-require-authentication) (10 min read)

**What is the best way to run rehearsal jobs?**
As a general rule of thumb we should never run the command `pj-rehearse` by itself. This is too risky as it will just run everything that you've changed which may include other teams jobs without us realizing it. So to prevent this only run `pj-rehearse` using a specific job name that you want to rehearse.

For example:

```shell
/pj-rehearse periodic-ci-windup-windup-ui-tests-main-mtr-ocp4.13-lp-interop-mtr-interop-aws
```
