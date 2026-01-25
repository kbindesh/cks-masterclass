# kube-bench Command Line Options and Flags

The kube-bench tool offers various command line flags to control which benchmarks to run, specify output formats, and target specific components. It also uses subcommands to target specific components of a Kubernetes cluster.

## kube-bench - Commands

kube-bench uses specific commands to run tests on different node types.

- kube-bench run master: Validates control plane (master) configurations.
- kube-bench run node: Inspects worker node settings.
- kube-bench run etcd: Checks etcd data store security.
- kube-bench run policies: Runs checks related to Kubernetes policies.
- kube-bench run run --targets master,node: Runs checks for multiple targets at once.

## kube-bench - Key Flags

The following are common flags used with kube-bench to modify its behavior and output:

- --benchmark string: Manually specify a specific CIS benchmark version (e.g., cis-1.5).
- -c, --check string: Run a comma-delimited list of specific checks by their IDs (e.g., 1.1.1,1.1.2).
- --config string: Specify the configuration file to use (default is ./cfg/config.yaml).
- --group string: Run all checks under a comma-delimited list of groups (e.g., 1.1,2.2).
- --skip string: Specify a comma-delimited list of checks or groups to be skipped.
- --version string: Manually specify the Kubernetes version, which helps kube-bench automatically detect the appropriate benchmark.
- --scored (default true): Run the scored CIS checks.
- --unscored (default true): Run the unscored CIS checks

- --json: Prints the results in JSON format.
- -junit: Prints the results in JUnit format.
- -outputfile string: Writes the JSON or JUnit results to a specified file.
