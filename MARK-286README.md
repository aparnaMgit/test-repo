
  
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




