# ArgoCD GitOps Template
This repository describes the GitOps approach for Kubernetes applications Continuous Deployments in the context of the Kyriba company.

This repo can be used to:

* Learn how the GitOps is applied for Kuberentes in Kyriba in general.
* Serve as a template scaffold-repo for the new applications deployments.
* Communicate internally about the changes to the approach as well as best practises learned via contributing to this repo.
* Interactively learn the tooling used to implement GitOps in detail.

# Terminology
Before diving into details about how _things_ are structured and working all together let's define some common vocabulary to better see which _things_ we are working with:

### > GitOps
  
   is a way to do Kubernetes cluster management and application delivery.  GitOps works by using Git as a single source of truth for declarative infrastructure and applications. [Read more]([https://www.weave.works/technologies/gitops/#what-is-gitops)

### > ArgoCD GitOps Repo

Is a combination of `Solutions` and `Argo Applications` defining "What to Deploy?" and "Where to Deploy?" correspondingly. It is convenient to dedicate a repo per domain which can have it's own pull of users (teams) which would share the same promotion process and(or) authorization rules. A couple of examples are having a dedidcated repository for bank-api project and the one for ERP project even though the team is techniqueally the same - CAAS, the team consist of two sub-teams located in Paris and San-Diego making it really hard to work with the same pace, hence the promotion process could vary. Another example is the Data team which is located in Paris and Warsaw, it has many sub-teams and different projects in flight but they are more convenient to have a single repo with all their deployments managed the same way.


### > Solution

Is a central unit of this model, it is an aggregation of the coherent `Releases` built around some domain plus their specific configurations described in the config overlay section. It answers the question of "What To Deploy?". The domain itself is a self sufficient product meaning it should not share some configuration with some other solutions. One solution maps to one `Argo Application`. One solution can have either many `Releases` or just a single `Release`.

### > Release

Is an atomic composition unit used to describe the exact `Helm Chart` and `Helm Values` used by the `Helmfile` to produce Kubernetes raw manifests. Typically maps to either some single Kyriba microservice helm chart or some single community helm chart.
### > Argo CD Application

Is an `Argo CD` configuration unit describing the mapping of  `Solution` to the specific `Cluster` and `Environment`. One Solution can be included into many applications but one application can have only one solution. An application is mapped to a single `Argo CD Project`

### > Argo CD Project
Is an `Argo CD` configuration unit describing the set of permissions against a set of resources in the `Argo CD` or Kubernetes cluster. One project can include multiple `Argo CD Applications`.

### > Cluster

Is a Kubernetes cluster whether in onprem or in Cloud connected to `Argo CD`.

### > Environment

Is a tier or level of quality of `solution`, from highest grade - production to the lowest one - proto. One environment can contain many `Clusters` but one cluster can be only in one environment. Also it is a concept used in `Helmfile` to combine a set of configuration parameters under a certain namespace, which is leveraged in our model to have some dynamic configuration overlay described further.

### > Helmfile

Is an [open source tool](https://github.com/roboll/helmfile) describing itself as "Helmfile is a declarative spec for deploying helm charts". Or as one can call it "Helm of Helms" or "Helm on steroids". It is used in the current GitOps approach to render the final Kuberenetes manifests.


### > Argo CD

Is an [open source tool](https://github.com/argoproj/argo-cd/) describing itself as "Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes." Is used with a conjunction of [Helmfile Argo CD plugin](https://github.com/travisghansen/argo-cd-helmfile) to manage the deployment on Kubernetes clusters.


### > Helm Chart

Is a well known [open source tool](https://github.com/helm/helm) describing itself as "Helm is a tool for managing Charts. Charts are packages of pre-configured Kubernetes resources." 

### > Helm Values

Is a concept used by `Helm` to hydrate the helm chart with actual configuration data passed down to `Helm`.
  
# Repository Structure

```yaml
.                                           # - This ArgoCD GitOps repo
├── README.md                               # - This readme file.
├── _argocd                                 # - ArgoCD Application is defined here.
│   ├── environments
│   │   ├── preprod.yaml                    # - each environment defines own set of solutions and clusters
│   │   ├── production.yaml                 # they are separated per file allowing us to have better
│   │   ├── proto.yaml                      # auth controls and audit.
│   │   └── sandbox.yaml
│   └── helmfile.yaml                       # - This is where the ArgoCD Applications are rendered
├── documentation                           # - Here some project documentation/diagramms can be stored.
└── solution                                # - Here we are entering the context of the solution.
    ├── clusters
    │   ├── alpha
    │   │   └── environments
    │   │       └── values.yaml.gotmpl      # - Alpha cluster specific configuration
    │   └── omega
    │       └── environments
    │           └── values.yaml.gotmpl      # - Omega cluster specific configuration.
    ├── environments                        # - A Solution can have environment specific configuration,
    │   ├── _common.yaml.gotmpl             # _common defines the basic common set of values
    │   ├── production.yaml.gotmpl          # environment specific values has higher precedence level then common,
    │   ├── proto.yaml.gotmpl
    │   └── versions.yaml                   # just a shortcut for most dynamic type of values - versions
    ├── helmfile.yaml                       # - This is an entry point for Helmfile which maps the configuration to the
    └── releases                            # releases described inside releases. This solution consists of two releases.
        ├── commonCumminityService
        │   ├── helmfile.yaml               # - Helmfile defined in the solution root is "extended" with definitions
        │   └── values                      # from these "release" helmfiles.
        │       └── _common.yaml.gotmpl
        └── microservice
            ├── helmfile.yaml
            └── values
                ├── _common.yaml.gotmpl     # - This is a template for "Helm Values" rendered
                ├── onprem.yaml.gotmpl     # by Helmfile first to fulfill it with the actual values.
                └── production.yaml.gotmpl  # Also use some overlay logic described furher.
```

# Helmfile in a Nutshell

Our GitOps concepts and the model itself becomes intuitive once you have learned some basic concepts around Helmfile itself. The following sections should help you to learn what is needed to understand the model described. Please make sure you will go through the Helmfile concepts documentations sections mentioned in this section before going furhter, cause a lot of details tight to Helmfile.

> One important note to be considered when we talk about Helmfile is that we are not using Helmfile to actually perform the deployments in a terraform fashion, instead we use it as a template engine only. (see `helmfile template`)

## Why Helmfile
Niether vanilla `Helm` nor other approaches resolve efficintly the config overlays and work with secrets engine. What we aimed to have is [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and simple resolution to the mentioned problem, as such `Helmfile` allows us to achieve that.

* Helmfile supports the [Environments](https://github.com/roboll/helmfile#environment-values) and [templating](https://github.com/roboll/helmfile#templates) which together allows to fine tune almost every aspect of a Helm Chart. 
* Helmfile is essentialy a gotemplate engine with a preset of functions availble. It might sound tricky but one can use Helmfile to render Helmfiles which would render actual Kubernetes manifests through calling Helm.
*  Another crucial feature is [remote secrets](https://github.com/roboll/helmfile/blob/master/docs/remote-secrets.md) which allows to declaratively describe the locations of the secrets instead of their values and delegate the actual secret values hydration to Helmfile, which bring a good level of secrets abstraction.


Mentioned features with a combination of [ability to nest Helmfiles in the Helmfile itself](https://github.com/roboll/helmfile/blob/master/docs/shared-configuration-across-teams.md), [ability to re-use the environment values in the sub-helmfile](https://github.com/roboll/helmfile/blob/master/docs/writing-helmfile.md#re-using-environment-state-in-sub-helmfiles), [and use of environment variables](https://github.com/roboll/helmfile#using-environment-variables) gives us pretty powerful tolling in order to both keep things close to vanilla Helm and comply with DRY principle.

# Deep dive in the concepts

![Image Load Error](documentation/images/BigBang.png?raw=true "Kyriba GitOps Big Bang")

Unlike in precise science there is no golden rule (yet) to decide where there is a clear boundary between Domain, Solution or Release. On the surface Domain is essentially a git repo that forms some completed functionality dedicated to a single Domain hence the named Domain for brevity. The solution is just a composition of one or many coherent releases, and the release is the smallest possible unit just like an atom in this universe, which maps 1 to 1 to certain helm chart release, hence the name release. But what are these concepts? Let's dive in.
## Release
![Image Load Error](documentation/images/Release.png?raw=true "Release")

```yaml
.
├── helmfile.yaml
└── values
    ├── _common.yaml.gotmpl
    ├── onprem.yaml.gotmpl
    └── production.yaml.gotmpl
```

Release is a smallest composition unit in the described GitOps hierarchy. It should have only one Helm Chart managed. On this level we define:

* Helm Repository configuration in `helmfile.yaml`. For example, to add an internal `helm-kyriba-release` repository: (As per `DOWNLOAD_PWD` and other environment variables referenced later don't worry, we will explain this later).
```yaml
repositories:
- name: helm-kyriba-release
  url: https://artifactory.kod.kyriba.com/artifactory/helm-kyriba-release
  username: deploy
  password: {{ requiredEnv "DOWNLOAD_PWD" }}
```

* A particular chart and version of the chart to be used in `helmfile.yaml`. For example a chart `helm-kyriba-release/microservice-1` and version taken from upstream - `{{ .Values.chartVersion }}`. Technically here we can have multiple releases described, Helmfile does not prevent us from doing it but we should have only one release in order to keep this abstraction simple:
```yaml
releases:
- name: microservice
  chart: helm-kyriba-release/microservice
  version: {{ .Values.chartVersion }}
  missingFileHandler: Warn
```

* A correspondent `values` which will be passed down to `Helm` in `helmfile.yaml`. Which is itself composed of a set of templates. For example we have here an overlay of a `values` config composed of `_common` plus a specific environment and provider templates. Please note this `missingFileHandler: Warn` flag which allows Helmfile to tolerate the absence of one of the referenced templates.
```yaml
releases:
- name: microservice
  chart: helm-kyriba-release/microservice
  version: {{ .Values.chartVersion }}
  missingFileHandler: Warn
  values:
  - values/_common.yaml.gotmpl
  - values/{{ requiredEnv "HELMFILE_ENVIRONMENT" }}.yaml.gotmpl
  - values/{{ requiredEnv "CLUSTER_TYPE" }}.yaml.gotmpl
```
* The `values` templates are composed of the overlay of the templates for the release and(or) environment and Kubernetes provider. For example:
```
$ cat _common.yaml.gotmpl 
service:
  version {{ .Values.version }}
databaseX:
  enabled: true
  replicaCount: 1
$
$ cat onprem.yaml 
databaseX:
  persistence:
    enabled: false
$
$ cat production.yaml.gotmpl 
databaseX:
  replicaCount: 3
```

Let's try to render the result Helmfile (so called state Helmfile) by running this command inside the `microservice` release directory.
`HELMFILE_ENVIRONMENT=production CLUSTER_TYPE=onprem DOWNLOAD_PWD=xxx helmfile --state-values-set chartVersion=x.y.z,version=z.y.x build --embed-values` 
you should receive this output:
```yaml
---
#  Source: helmfile.yaml

filepath: helmfile.yaml
helmBinary: helm
repositories:
- name: helm-kyriba-release
  url: https://artifactory.kod.kyriba.com/artifactory/helm-kyriba-release
  username: deploy
  password: xxx
releases:
- chart: helm-kyriba-release/microservice
  version: x.y.z
  missingFileHandler: Warn
  name: microservice
  values:
  - databaseX:
      enabled: true
      replicaCount: 1
    service:
      version: z.y.x
  - databaseX:
      replicaCount: 3
  - databaseX:
      persistence:
        enabled: false
templates: {}
renderedvalues:
  chartVersion: x.y.z
  version: z.y.x
```

You can play with the different values of `HELMFILE_ENVIRONMENT` and `CLUSTER_TYPE` and pass different values with `--state-values-set` to see different results of a Helmfile state.
*To be completed with `helmfile template` examples to see final rendering of the Helm also.

## Solution
![Image Load Error](documentation/images/Solution.png?raw=true "Release")

```yaml
.
├── clusters
│   ├── alpha
│   │   └── environments
│   │       └── values.yaml.gotmpl
│   └── omega
│       └── environments
│           └── values.yaml.gotmpl
├── environments
│   ├── _common.yaml
│   ├── production.yaml
│   ├── proto.yaml
│   └── versions.yaml
├── helmfile.yaml
└── releases
    ├── commonCumminityService
    │   ├── helmfile.yaml
    │   └── values
    │       └── _common.yaml.gotmpl
    └── microservice
        ├── helmfile.yaml
        └── values
            ├── _common.yaml.gotmpl
            ├── onprem.yaml.gotmpl
            └── production.yaml.gotmpl
```

The solution is a next abstraction level above the release. The goal of the solution is to describe a Kubernetes namespace composed of the releases (one or many) and the values passed down to each release. Similar to what we were doing in the example section above with passing some values manually via `--state-values-set` option. So techniqueally here we define the list of releases used and instruct the `Helmfile` to build the map of yaml values and pass them down to to concrete release in order to hydrate release with actual state. The map of yaml values is built using the straightforward overlay rules, let's look in details:


* Define the list of the releases used in solution in `helmfile.yaml`, for example to use `commonCumminityService` and `microservice` releases we need to define this. (note how the namespaced `.Values` are passed to each release).
```yaml
helmfiles:
- path: "releases/commonCumminityService/helmfile.yaml"
  values:
  - {{ toYaml .Values.commonCumminityService | nindent 4 }}
- path: "releases/microservice/helmfile.yaml"
  values:
  - {{ toYaml .Values.microservice | nindent 4 }}
```

* Define the load of `environment` values using some overlay rules in `helmfile.yaml`, for example if we have a specific set of parameters to be used in `production` `proto` environments while want to have a set of values for all other environments `default` this section would do that:
```yaml
environments:
  default:
    missingFileHandler: Warn
    values:
    - environments/common.yaml
    - clusters/{{ requiredEnv "CLUSTER_NAME" }}/environments/values.yaml.gotmpl
    
  production:
    missingFileHandler: Warn
    values:
    - environments/common.yaml
    - environments/production.yaml
    - clusters/{{ requiredEnv "CLUSTER_NAME" }}/environments/values.yaml.gotmpl

  proto:
    missingFileHandler: Warn
    values:
    - environments/common.yaml
    - environments/proto.yaml
    - clusters/{{ requiredEnv "CLUSTER_NAME" }}/environments/values.yaml.gotmpl
```
Now let's take a moment and see what this allow us to do with `Helmfile`, let's emulate `Helmfile` execution:
`HELMFILE_ENVIRONMENT=production CLUSTER_TYPE=onprem DOWNLOAD_PWD=xxx CLUSTER_NAME=yyy helmfile  build --embed-values` (note we don't pass `--state-values-set` anymore)
Review the output, you should see three different `#  Source: helmfile.yaml` yaml blocks. See how the values were mapped to which releases, try to customize the version values passed down on the cluster level using `CLUSTER_NAME=alpha`. Try to play with other environment variables values.

## Domain
![Image Load Error](documentation/images/Domain.png?raw=true "Domain")

As of now there is no special role for this object other then being just a conttainer for Solutions and Argo CD Applications definitions.
# Configuration Layering
![Image Load Error](documentation/images/Overlay.png?raw=true "Overlay")

By configuration layering we mean a mechanism which produces a map of config settings using a Helmfile environment with some overlay technique. It can be defined and used either on solution level Helmfile or on release level. Let's look at the example from `solution/helmfile.yaml` first

```yaml
environments:
  {{ requiredEnv "HELMFILE_ENVIRONMENT" }}:
    missingFileHandler: Warn
    values:
    - environments/_common.yaml.gotmpl
    - environments/versions.yaml.gotmpl
    - environments/{{ requiredEnv "HELMFILE_ENVIRONMENT" }}.yaml.gotmpl
    - clusters/{{ requiredEnv "CLUSTER_NAME" }}/environments/values.yaml.gotmpl
```

There is a documentation explaining the [environments](https://github.com/roboll/helmfile#environment-values) concept, please read it before moving further. If you already went through the "What is solution in detail" section you would see that we managed the environments differently there, we had each environment declared separately, which is good for visibility but complex from a maintainability point of view. This is why we use a more generalized approach. Imagine how simple it is just to add a new environment with this approach.
Both environment variables values used `HELMFILE_ENVIRONMENT` and `CLUSTER_NAME` are set on ArgoCD Application level. And the `missingFileHandler: Warn` enables us to use the overlay by providing correspondent files only when needed. For example to do only a certain environment overlay, or only a specific cluster.
The only difference between `_common.yaml` and `versions.yaml` is the type of usage of them, we expect only `versions.yaml` which contain chart/image versions to be changed frequently. The priority of overrides goes from the first item in the `values:` list which is `_common.yaml` and has lowest priority to cluster specific values which has the top priority.


Let's look at another place where we use overlay - `solution/releases/microservice/helmfile.yaml`

```yaml
releases:
- name: microservice
  chart: helm-kyriba-release/microservice
  version: {{ .Values.chartVersion }}
  missingFileHandler: Warn
  values:
  - values/_common.yaml.gotmpl
  - values/{{ requiredEnv "HELMFILE_ENVIRONMENT" }}.yaml.gotmpl
  - values/{{ requiredEnv "CLUSTER_TYPE" }}.yaml.gotmpl
```

Note how `values:` are composed. Similar to solution level, it is not mandatory to have all overalay files in place due to `missingFileHandler: Warn`. A new environment variable `CLUSTER_TYPE` is something which is passed down from ArgoCD Application and can be either `onprem` or `cloud` allowing to customize the result `values.yaml` with provider specific behavior if needed (like disable persistence `onprem` since it is not yet available there).

## What is Argo CD Application in detail
We use ArgoCD as a control plane of our deployments in K8s, in turn ArgoCD use helmfile to produce the K8s manifests. We need a way to configure ArgoCD to manage the deployments we need in every cluster or namespace where we expect our solution to be deployed. More simply if solution answers to "What to deploy?" we need to have something which answers to "Where to deploy?". This is what the ArgoCD "Application" concept is about. And we use Kubernetes to create these configurations, which allows use to put these configurations under GitOps process as well. We realy on [Argo CD Appliccation Set Controller](https://argoproj.github.io/argo-cd/user-guide/application-set/) to do the `Argo CD Applications` management. Everything related to that story is located in the `_argocd` directory.

![Image Load Error](documentation/images/ApplicationSetController.png?raw=true "ApplicationSetController")




# Promotion Model
![Image Load Error](documentation/images/PromotionModel.png?raw=true "Promotion Model")

Now when you master both `Solutions` and `ArgoCD Applications` you should be able to grasp the promotion model. The beauty of our GitOps approach is that our promotion model is also part of GitOps repository itself. Meaning that the information about how promotion is occurring across environments is described in `_argocd`. Hence the model is configurable from trunk based to complex git flows and can (and probably should) vary from one GitOps repo to another. This template repo is showing how one can use a [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html) promotion model.

Let's review a most important part of Helmfile environment from `_argocd/helmfile.yaml`
```yaml
applications:
  project: default
  repoURL: git@bitbucket.org:kyridev/argocd-gitops-template.git
  bank-api:
    environments:
      proto:
        clusters:
        - alpha:
            branch: master
            namespace: any-namespace
            type: onprem
            autosync: true
      preprod:
        clusters:
        - beta:
            branch: master
            namespace: any-namespace
            type: cloud
      sandbox:
        clusters:
        - omega:
            branch: sandbox
            namespace: any-namespace
            type: cloud
      production:
        clusters:
        - delta:
            branch: production
            namespace: any-namespace
            type: cloud
```
Look at which environment/cluster correspond to which branch in this repo. Knowing some basics of the GitLab Flow we can see easily that the promotion is going from the `proto` environment cluster (everything which was develop in the `master` branch is deployed there). Then one needs to open a Merge Request to `sandbox` branch to let the latest code to be deployed in `sandbox` cluster. After this one need to create anoth Merge Request from `sanbox` to `production` branch to let the code be used for the production deployment.

This is how it will look with GitLab Flow, it will still look the same if we would have real environments with many clusters. It can have a form of trunk-based if needed, imagine how one can do that. The good point here here is that each repo-domain can define its own way of promotion. Everyone who knows how to read `_argocd` definitions would know how the promotion means to work in that particular repo and will be able to maintain that. Given that each environment definition is in a separate file, we can easily use the "codeowners" feature to limit the authorization to do the changes to the customer-facing environments if needed.

# Naming Conventions

To let the model operate and look alike similar independently of the Domain we need to stick to some simple naming rules.

### > ArgoCD GitOps Repo

The name should be

* all lowercase alpahanumeric
* starts and ends with alphanumeric
* have either 1 or 2 dimensions separated by dash `-` 
* comply with the following schema:

2 dimensional:

`argocd-${team}-${domain}`  - when one team wants to have more then one repos for their projects.

Examples:

`argocd-caas-bankapi`  - Connectivity As A Service (CAAS) team's project "Bank API"

`argocd-caas-integration` - Connectivity As A Service (CASS) team's project "Integration"

OR 1 dimensional:

`argocd-${domain}` - if the name of the domain match with the name of the team or serve very specific purpose not related to any team in particular.

Examples:

`argocd-data` - for all Data team related solutions

`argocd-infra` - for foundational Kyriba Kubernetes clusters services (cert manager, metrics, logging, etc.)


### > Release

Is a name used in the context of the Solution (Kubernetes Namespace).

* The name should comply with [DNS Label Names requirments](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names), because `Helm` sets some labels with that name with the following addiitonal limits:
  * no dots symbols in the name
  * Maximum __recommended__ length of the release name is `21` characters, because that can be used as a prefix in Kubernetes objects names and with long names they becomes quite unreadable.
  * Maximum possible lenght is `53` characters, [a limit imposed by Helm](https://github.com/helm/helm/issues/6006) which one day might align with Kubernetes `63` characters.
  * Should be unique within single solution.

Examples:
```
prometheus-storage-adapter
kube-prometheus-stack
```

The name of the Helm release and the name of the directory with that release should be always equal, example for `fluent-bit`:
```yaml
➜  releases git:(master) ✗ tree
.
└── fluent-bit                # Release directory named fluent-bit
    ├── helmfile.yaml
    └── values
        └── _common.yaml.gotmpl

2 directories, 2 files
➜  releases git:(master) ✗ cat fluent-bit/helmfile.yaml 
repositories:
- name: stable
  url: https://charts.helm.sh/stable

releases:
- name: fluent-bit            # Release in Helm named fluent-bit
  chart: stable/fluent-bit
  version: 2.8.17
  namespace: logging
  values:
  - values/_common.yaml.gotmpl
```

### > Solution

This name is used as a part of `Argo CD` applciation name and `Kubernetes namespace`, and can be overriden for the namespace for special cases.
* The name of the solution is used should comply with [DNS Label Names requirments](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names) because `ArgoCD` [uses that name](https://github.com/argoproj/argo-cd/issues/5595#:~:text=There%20is%20an%20implicit%20length,truncate%20it%20to%2063%20characters) as Kubernetes labels with the following additional limits:
  * no dots symbols in the name
  * Must be unique accross all the other solutions from all the domains.
  * Maximum __recommended__ length of the release name is `21` characters, to have the name readable - it is only a part of the `Argo CD` applciation name.
  * The maximum length imposed by the naming convention is `46` characters if Kubernetes namespace name is matching the solution name, otherwise if namespace name is customized the concatenation of the solution name and namespace should not go over `45` characters.

### > Argo CD Application

The name of the application is automatically generated and should always comply with

* [DNS Label Names requirments](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names) because `ArgoCD` [uses that name](https://github.com/argoproj/argo-cd/issues/5595#:~:text=There%20is%20an%20implicit%20length,truncate%20it%20to%2063%20characters) as Kubernetes labels.
* 1 or 2 dimensional schema

1 dimensional:

`${solution}.${clusterName}` - in case if solution name is also used as a namespace in that cluster.

Examples:
```
logging.kfr-fpr-alpha     # 'logging' solution deployed in 'logging' namespace of 'kfr-fpr-alpha' cluster
monitoring.core-pre-a02   # 'monitoring' solution deployed in 'monitoring' namespace of 'core-pre-a02' cluster
```

2 dimensional:
`${solution}.${namespace}.${clusterName}` - in case if custom namespace is used (non common usecase)

Examples:
```
bank-api.bank-api-kfr-fpr-c.kfr-fpr-alpha     # 'bank-api' solution deployed in 'bank-api-kfr-fpr-c' namespace of 'kfr-fpr-alpha' cluster
```

### > Cluster

should comply with:

* all lowercase alphanumeric
* dashes allowed
* maximum length is 16 charaters


Examples:
```
kfr-fpr-alpha
core-rancher-a01
us3-dmo-a01
```

# Live Examples
These repositories follow the approach described, also some experiments are going on in there.

* https://gitlab.com/kyribadevops/argocd-caas-bank-api
* https://gitlab.com/kyribadevops/argocd-infra
* https://bitbucket.org/kyridev/argocd-data/src/master/

# References
* Images used are done in lucidchart https://lucid.app/documents/view/e28b2c57-2900-4c12-bfae-da2540c6ae8d password: GitOps2021

# Q&A [TBD]

# End to end tutorial for ArgoCD and Helmfile [TBD]


