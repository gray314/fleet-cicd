# Git REPO with DEMO of cicd in Rancher Fleet

## Demo1
dir demo - examples with yaml manifest

## Demo2
dir demo2 - examples with helm manifest

## Descriptions

Sample of fleet.yaml
```yaml
# The default namespace to be applied to resources. This field is not used to
# enforce or lock down the deployment to a specific namespace, but instead
# provide the default value of the namespace field if one is not specified
# in the manifests.
# Default: default
defaultNamespace: default

# All resources will be assigned to this namespace and if any cluster scoped
# resource exists the deployment will fail.
# Default: ""
namespace: default

kustomize:
  # Use a custom folder for kustomize resources. This folder must contain
  # a kustomization.yaml file.
  dir: ./kustomize

helm:
  # Use a custom location for the Helm chart. This can refer to any go-getter URL.
  # This allows one to download charts from most any location.  Also know that
  # go-getter URL supports adding a digest to validate the download. If repo
  # is set below this field is the name of the chart to lookup
  chart: ./chart
  # A https URL to a Helm repo to download the chart from. It's typically easier
  # to just use `chart` field and refer to a tgz file.  If repo is used the
  # value of `chart` will be used as the chart name to lookup in the Helm repository.
  repo: https://charts.rancher.io
  # A custom release name to deploy the chart as. If not specified a release name
  # will be generated.
  releaseName: my-release
  # The version of the chart or semver constraint of the chart to find. If a constraint
  # is specified it is evaluated each time git changes.
  version: 0.1.0
  # Any values that should be placed in the `values.yaml` and passed to helm during
  # install.
  values:
    any-custom: value
  # All labels on Rancher clusters are available using global.fleet.clusterLabels.LABELNAME
  # These can now be accessed directly as variables
    variableName: global.fleet.clusterLabels.LABELNAME
  # Path to any values files that need to be passed to helm during install
  valuesFiles:
    - values1.yaml
    - values2.yaml
  # Allow to use values files from configmaps or secrets
  valuesFrom:
  - configMapKeyRef:
      name: configmap-values
      # default to namespace of bundle
      namespace: default 
      key: values.yaml
    secretKeyRef:
      name: secret-values
      namespace: default
      key: values.yaml
  # Override immutable resources. This could be dangerous.
  force: false

# A paused bundle will not update downstream clusters but instead mark the bundle
# as OutOfSync. One can then manually confirm that a bundle should be deployed to
# the downstream clusters.
# Default: false
paused: false

rolloutStrategy:
    # A number or percentage of clusters that can be unavailable during an update
    # of a bundle. This follows the same basic approach as a deployment rollout
    # strategy. Once the number of clusters meets unavailable state update will be
    # paused. Default value is 100% which doesn't take effect on update.
    # default: 100%
    maxUnavailable: 15%
    # A number or percentage of cluster partitions that can be unavailable during
    # an update of a bundle.
    # default: 0
    maxUnavailablePartitions: 20%
    # A number of percentage of how to automatically partition clusters if not
    # specific partitioning strategy is configured.
    # default: 25%
    autoPartitionSize: 10%
    # A list of definitions of partitions.  If any target clusters do not match
    # the configuration they are added to partitions at the end following the
    # autoPartitionSize.
    partitions:
      # A user friend name given to the partition used for Display (optional).
      # default: ""
    - name: canary
      # A number or percentage of clusters that can be unavailable in this
      # partition before this partition is treated as done.
      # default: 10%
      maxUnavailable: 10%
      # Selector matching cluster labels to include in this partition
      clusterSelector:
        matchLabels:
          env: prod
      # A cluster group name to include in this partition
      clusterGroup: agroup
      # Selector matching cluster group labels to include in this partition
      clusterGroupSelector: agroup

# Target customization are used to determine how resources should be modified per target
# Targets are evaluated in order and the first one to match a cluster is used for that cluster.
targetCustomizations:
# The name of target. If not specified a default name of the format "target000"
# will be used. This value is mostly for display
- name: prod
  # Custom namespace value overriding the value at the root
  namespace: newvalue
  # Custom kustomize options overriding the options at the root
  kustomize: {}
  # Custom Helm options override the options at the root
  helm: {}
  # If using raw YAML these are names that map to overlays/{name} that will be used
  # to replace or patch a resource. If you wish to customize the file ./subdir/resource.yaml
  # then a file ./overlays/myoverlay/subdir/resource.yaml will replace the base file.
  # A file named ./overlays/myoverlay/subdir/resource_patch.yaml will patch the base file.
  # A patch can in JSON Patch or JSON Merge format or a strategic merge patch for builtin
  # Kubernetes types. Refer to "Raw YAML Resource Customization" below for more information.
  yaml:
    overlays:
    - custom2
    - custom3
  # A selector used to match clusters.  The structure is the standard
  # metav1.LabelSelector format. If clusterGroupSelector or clusterGroup is specified,
  # clusterSelector will be used only to further refine the selection after
  # clusterGroupSelector and clusterGroup is evaluated.
  clusterSelector:
    matchLabels:
      env: prod
  # A selector used to match cluster groups.
  clusterGroupSelector:
    matchLabels:
      region: us-east
  # A specific clusterGroup by name that will be selected
  clusterGroup: group1

# dependsOn allows you to configure dependencies to other bundles. The current bundle
# will only be deployed, after all dependencies are deployed and in a Ready state.
dependsOn:
  # Format: <GITREPO-NAME>-<BUNDLE_PATH> with all path separators replaced by "-" 
  # Example: GitRepo name "one", Bundle path "/multi-cluster/hello-world" => "one-multi-cluster-hello-world"
  - name: one-multi-cluster-hello-world
```