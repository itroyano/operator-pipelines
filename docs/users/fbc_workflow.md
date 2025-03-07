# FBC workflow

If you already have an existing non-FBC operator please continue with the [onboarding](./fbc_onboarding.md) documentation to convert it to FBC. Once you have converted your operator, or you want to introduce a brand new operator, you can start with the FBC workflow.

### FBC operator config
To indicate the operator is using fbc workflow an operator owner needs to indicate this
fact in the `ci.yaml` file.

Example of the `ci.yaml` with FBC config:
```yaml
---
fbc:
  enabled: true
```

## FBC templates
File-based catalog templates serve as a simplified view of a catalog that can be updated
by the user. The OPM currently supports 2 types of templates and it is up to the user which
template the operator will be using.

* Basic template - `olm.template.basic`
* SemVer template - `olm.semver`

More information about each template can be found at [opm doc](https://olm.operatorframework.io/docs/reference/catalog-templates/).

The recommended template from the maintainability point of view is `SemVer`.

### FBC template mapping
To be able to generate a catalog from templates a user needs to provide a mapping between
template and catalog. The mapping is stored in the `ci.yaml` file. Based on your preference
you can map a template to a catalog version with 1:N mapping or 1:1 mapping.

Here is an example of the `ci.yaml` file with single template rendering multiple catalogs (`1:N`):

```yaml
---
fbc:
  enabled: true
  catalog_mapping:
    - template_name: my-custom-semver-template.yaml # The name of the file inside ./catalog-templates directory
      catalogs_names: # a list of catalogs within the /catalogs directory
        - "v4.15"
        - "v4.16"
        - "v4.17"
      type: olm.semver
    - template_name: my-custom-basic-template.yaml # The name of the file inside catalog-templates directory
      catalogs_names:
        - "v4.12"
        - "v4.13"
      type: olm.template.basic
```

And here is an example of the `ci.yaml` file with a single template rendering a single catalog (`1:1`):

```yaml
---
fbc:
  enabled: true
  catalog_mapping:
  - template_name: v4.14.yaml
    catalog_names: ["v4.14"]
    type: olm.template.basic
  - template_name: v4.15.yaml
    catalog_names: ["v4.15"]
    type: olm.template.basic
  - template_name: v4.16.yaml
    catalog_names: ["v4.16"]
    type: olm.template.basic
  - template_name: v4.17.yaml
    catalog_names: ["v4.17"]
    type: olm.template.basic
```

## Generate catalogs using templates
To generate a final catalog for an operator a user needs to execute different `opm`
commands based on the template type. We as operator pipeline maintainers want
to simplify this process and we prepared a `Makefile` with all pre-configured targets.

To get the `Makefile` follow these steps (In case you converted the existing operator and
followed the [onboarding](./fbc_onboarding.md) guide the `Makefile` should be already in
your operator directory and you can skip the step.)

```bash
cd <operator-repo>/operator/<operator-name>
wget https://raw.githubusercontent.com/redhat-openshift-ecosystem/operator-pipelines/main/fbc/Makefile
```

The right place for the Makefile is in the operator's root directory
```
.
├── 0.0.1
|   ├── release-config.yaml
│   ├── manifests
│   └── metadata
├── catalog-templates
├── ci.yaml
└── Makefile

```

> [!IMPORTANT]
> In case an operator isn't shipped to all OCP catalog versions manually update `OCP_VERSIONS`
> variable in the `Makefile` and include only versions supported by an operator.

The command uses the `opm` and converts templates into catalogs. The generated catalogs
can be submitted as a PR in Github and once the PR is processed changes will be released to the
OCP index.

```bash
$ tree (repository-root)/catalogs
catalogs
├── v4.12
│   └── aqua
│       └── catalog.yaml
├── v4.13
│   └── aqua
│       └── catalog.yaml
├── v4.14
│   └── aqua
│       └── catalog.yaml
├── v4.15
│   └── aqua
│       └── catalog.yaml
└── v4.16
    └── aqua
        └── catalog.yaml

```

### Adding new bundle to Catalog
A new bundle can be added automatically to your templates and catalogs if you use
the automated release feature. The process is described in the
[fbc auto-release](./fbc_autorelease.md) documentation.

It is **highly recommended to use the automated release feature** as it simplifies the process
from the user perspective.

However if you want to manually add a new bundle to the catalog follow the steps below.

To add a bundle to the catalog you need to first submit the new version of the operator
using traditional [PR workflow](./contributing-via-pr.md). The operator pipeline builds,
tests, and releases the bundle into the registry. **At this point, the operator is not available
in the catalog yet.** To add the bundle to the catalog you need to update catalog templates
and add a bundle pullspec given by pull request comment and open a new pull request with catalog
changes.

![Release info](../img/release-info.png)

> [!NOTE]
> Currently a workflow requires a 2-step process to release a new bundle into the catalog. In the first step, the operator bundle is released and in the second step, the catalog is updated with the new bundle.
> We are working on a solution to automate this process and make it a single step. However, this will require a usage of `SemVer` catalog template. In case you would like to use this feature once available please consider using `SemVer` template.

#### SemVer
For example if I want to add `v1.1.0` bundle into `Fast` channel of a specific catalog I'll
add it as mentioned in the example below:

```yaml
---
Schema: olm.semver
GenerateMajorChannels: true
GenerateMinorChannels: true
Candidate:
  Bundles:
  - Image: quay.io/foo/olm:testoperator.v0.1.0
  - Image: quay.io/foo/olm:testoperator.v0.1.1
  - Image: quay.io/foo/olm:testoperator.v0.1.2
  - Image: quay.io/foo/olm:testoperator.v0.1.3
  - Image: quay.io/foo/olm:testoperator.v0.2.0
  - Image: quay.io/foo/olm:testoperator.v0.2.1
  - Image: quay.io/foo/olm:testoperator.v0.2.2
  - Image: quay.io/foo/olm:testoperator.v0.3.0
  - Image: quay.io/foo/olm:testoperator.v1.0.0
  - Image: quay.io/foo/olm:testoperator.v1.0.1
  - Image: quay.io/foo/olm:testoperator.v1.1.0
Fast:
  Bundles:
  - Image: quay.io/foo/olm:testoperator.v0.2.1
  - Image: quay.io/foo/olm:testoperator.v0.2.2
  - Image: quay.io/foo/olm:testoperator.v0.3.0
  - Image: quay.io/foo/olm:testoperator.v1.0.0
  - Image: quay.io/foo/olm:testoperator.v1.1.0 # <-- Add new bundle into fast channel
Stable:
  Bundles:
  - Image: quay.io/foo/olm:testoperator.v1.0.0
```
Also see [opm doc](https://olm.operatorframework.io/docs/advanced-tasks/catalog-update-formulary/#semver)
for automate-able step.

#### Basic
For example, if I want to add `v0.2.0` bundle into `stable` channel of specific catalog I'll
add it as mentioned in the example below.

1. Add a new `olm.bundle` entry with bundle pullspec
2. Add bundle into the `stable` channel

```yaml
---
schema: olm.template.basic
entries:
  - schema: olm.package
    name: example-operator
    defaultChannel: stable

  - schema: olm.channel
    package: example-operator
    name: stable
    entries:
      - name: example-operator.v0.1.0
      - name: example-operator.v0.2.0 # <-- Add bundle into channel
        replaces: example-operator.v0.1.0

  - schema: olm.bundle
    image: docker.io/example/example-operator-bundle:0.1.0

  - schema: olm.bundle # <-- Add new bundle entry
    image: docker.io/example-operator-bundle:0.2.0
```

Also see [opm doc](https://olm.operatorframework.io/docs/advanced-tasks/catalog-update-formulary/#fbc)
for automate-able step.

### Updating existing catalogs
A great benefit of FBC is that users can update operator update graphs independently
of operator releases. This allows any post-release modification of the catalogs.
If you want to change the order of updates, remove an invalid bundle, or do any other modification
you are free to do that.

After updating catalog templates don't forget to run `make catalogs` to generate a catalog
from templates and submit the resulting catalog using PR workflow.
