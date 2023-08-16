## Introduction

When deploying your Azure Red Hat OpenShift (ARO) cluster, you can configure many aspects of your worker nodes, but what happens when you need to change your worker nodes after they've already been created? These activities include scaling the number of nodes, changing the instance type, adding labels or taints, just to name a few.

Many of these changes are done using MachineSets. MachineSets ensure that a specified number of Machine replicas are running at any given time. Think of a MachineSet as a "template" for the kinds of Machines that make up the worker nodes of your cluster. These are similar to other Kubernetes resources, like a ReplicaSet is to Pods. One important caveat, is that MachineSets allow users to manage many Machines as a single entity, but are contained to a specific availability zone. If you'd like to learn more, see the [Red Hat documentation on machine management](https://docs.openshift.com/container-platform/latest/machine_management/index.html){:target="_blank"}.

## Scaling worker nodes
### Via the CLI

1. First, let's see what MachineSets already exist in our cluster. To do so, run the following command:

    ```bash
    oc -n openshift-machine-api get machineset
    ```

    By default, ARO clusters have three MachineSets, one for each availability zone. The output will look something like this:

    ```{.text .no-copy}
    NAME                                 DESIRED   CURRENT   READY   AVAILABLE   AGE
    user1-cluster-8kvh4-worker-{{ azure_region }}1   1         1         1       1           4h36m
    user1-cluster-8kvh4-worker-{{ azure_region }}2   1         1         1       1           4h36m
    user1-cluster-8kvh4-worker-{{ azure_region }}3   1         1         1       1           4h36m
    ```

2. Now, let's take a look at the machines that have been created according to the instructions provided by the above MachineSets. To do so, run the following command:

    ```bash
    oc -n openshift-machine-api get machine
    ```

    For this workshop, we've deployed your ARO cluster with six total machines (three workers machines and three control plane machines), one in each availability zone. The output will look something like this:

    ```{.text .no-copy}
    NAME                                       PHASE     TYPE              REGION   ZONE   AGE
    user1-cluster-8kvh4-master-0               Running   Standard_D8s_v3   {{ azure_region }}   1      4h39m
    user1-cluster-8kvh4-master-1               Running   Standard_D8s_v3   {{ azure_region }}   2      4h39m
    user1-cluster-8kvh4-master-2               Running   Standard_D8s_v3   {{ azure_region }}   3      4h39m
    user1-cluster-8kvh4-worker-{{ azure_region }}1-gls9k   Running   Standard_D4s_v3   {{ azure_region }}   1      4h36m
    user1-cluster-8kvh4-worker-{{ azure_region }}2-xmhrw   Running   Standard_D4s_v3   {{ azure_region }}   2      4h36m
    user1-cluster-8kvh4-worker-{{ azure_region }}3-kggpz   Running   Standard_D4s_v3   {{ azure_region }}   3      4h36m
    ```

3. Now that we know that we have three worker nodes, let's pick a MachineSet to and create a new MachineSet using the OpenShift CLI tools, with your user number. To do so, run the following command:

    Change the # of your user in the name of the machine set

    ```bash
    NAME_MACHINESET=arofundamentals-5c5n4-worker-eastus1-user#
    echo ${NAME_MACHINESET}
    ```

    The output of the command should look something like this:

    ```{.text .no-copy}
    arofundamentals-5c5n4-worker-eastus1-user#
    ```

    Create the following MachineSet file

    ```{.text .no-copy}
    cat <<EOF >>${NAME_MACHINESET}.yaml
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    metadata:
      annotations:
        machine.openshift.io/GPU: '0'
        machine.openshift.io/memoryMb: '16384'
        machine.openshift.io/vCPU: '4'
      name: ${NAME_MACHINESET}
      namespace: openshift-machine-api
      labels:
        machine.openshift.io/cluster-api-cluster: arofundamentals-5c5n4
        machine.openshift.io/cluster-api-machine-role: worker
        machine.openshift.io/cluster-api-machine-type: worker
    spec:
      replicas: 1
      selector:
        matchLabels:
          machine.openshift.io/cluster-api-cluster: arofundamentals-5c5n4
          machine.openshift.io/cluster-api-machineset: arofundamentals-5c5n4-worker-eastus1
      template:
        metadata:
          labels:
            machine.openshift.io/cluster-api-cluster: arofundamentals-5c5n4
            machine.openshift.io/cluster-api-machine-role: worker
            machine.openshift.io/cluster-api-machine-type: worker
            machine.openshift.io/cluster-api-machineset: arofundamentals-5c5n4-worker-eastus1
        spec:
          lifecycleHooks: {}
          metadata: {}
          providerSpec:
            value:
              osDisk:
                diskSettings: {}
                diskSizeGB: 128
                managedDisk:
                  storageAccountType: Premium_LRS
                osType: Linux
              networkResourceGroup: dpenagos
              publicLoadBalancer: arofundamentals-5c5n4
              userDataSecret:
                name: worker-user-data
              vnet: aro-vnet-lld49wxe
              credentialsSecret:
                name: azure-cloud-credentials
                namespace: openshift-machine-api
              zone: '1'
              metadata:
                creationTimestamp: null
              publicIP: false
              resourceGroup: aro-infra-lld4cij1-arofundamentals
              kind: AzureMachineProviderSpec
              location: eastus
              vmSize: Standard_D4s_v3
              image:
                offer: aro4
                publisher: azureopenshift
                resourceID: ''
                sku: aro_411
                version: 411.86.20221004
              acceleratedNetworking: true
              subnet: worker-subnet
              apiVersion: machine.openshift.io/v1beta1
    EOF
    ```
4. Now, let's create the new MachineSet file

    ```bash
    oc apply -f ${NAME_MACHINESET}.yaml
    ```

5. Now that we've create the new MachineSet, we can see that the machine is already being created. First, let's quickly check the output of the same command we ran in step 1:

    ```bash
    oc -n openshift-machine-api get machinesets
    ```

    The output should look something like this:

    ```{.text .no-copy}
    NAME                                 DESIRED   CURRENT   READY   AVAILABLE   AGE
    user1-cluster-8kvh4-worker-{{ azure_region }}1   2         2         1       1           4h50m
    user1-cluster-8kvh4-worker-{{ azure_region }}2   1         1         1       1           4h50m
    user1-cluster-8kvh4-worker-{{ azure_region }}3   1         1         1       1           4h50m
    ```

    !!! note
        Note, that the number of *desired* and *current* nodes matches to one

    We can also run the same command we ran in step 2 to see the machine being provisioned:

    ```bash
    oc -n openshift-machine-api get machine
    ```

    The output should look something like this:

    ```{.text .no-copy}
    NAME                                       PHASE         TYPE              REGION   ZONE   AGE
    user1-cluster-8kvh4-master-0               Running       Standard_D8s_v3   {{ azure_region }}   1      4h58m
    user1-cluster-8kvh4-master-1               Running       Standard_D8s_v3   {{ azure_region }}   2      4h58m
    user1-cluster-8kvh4-master-2               Running       Standard_D8s_v3   {{ azure_region }}   3      4h58m
    user1-cluster-8kvh4-worker-{{ azure_region }}1-gls9k   Running       Standard_D4s_v3   {{ azure_region }}   1      4h55m
    user1-cluster-8kvh4-worker-{{ azure_region }}1-zj7dl   Provisioned   Standard_D4s_v3   {{ azure_region }}   1      9s
    user1-cluster-8kvh4-worker-{{ azure_region }}2-xmhrw   Running       Standard_D4s_v3   {{ azure_region }}   2      4h55m
    user1-cluster-8kvh4-worker-{{ azure_region }}3-kggpz   Running       Standard_D4s_v3   {{ azure_region }}   3      4h55m
    ```

### Via the Console

Now let's the worker nodes that yuu create, but this time, from the web console.

1. Return to your tab with the OpenShift Web Console. If you need to reauthenticate, follow the steps in the [Access Your Cluster](../setup/3-access-cluster/) section.

1. Using the menu on the left Select *Compute* -> *MachineSets*.

    ![Web Console - Cluster Settings](/assets/images/web-console-machineset-sidebar.png){ align=center }

1. In the overview you will see the same information about the MachineSets that you saw on the command line. Now, locate your MachineSet which has "1 of 1" machines.

    !!! note
        It may take up to 5 minutes for the MachineSet to create the node 
        while the underlying machine provisions and becomes ready.  Until this time, 
        the machine count will read "0 of 1".

Congratulations! You've successfully created your own worker node.
