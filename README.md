# Network-wide ad blocking with Pi-hole, Kubernetes, and Raspberry Pi

## Prerequisites
1. [Set up](https://santisbon.github.io/reference/rpi/) your Raspberry Pi with a static IP.
2. [Install and configure MicroK8s](https://santisbon.github.io/reference/k8s/#microk8s) (or any other lightweight Kubernetes distribution) on your Pi.

## Install

1. Prepare your Pi's storage
    ```shell
    # On your Pi
    sudo mkdir -p /etc/pihole
    sudo mkdir -p /etc/dnsmasq.d
    ```
2. If installing from source, grab the code
    ```shell
    # On your Pi
    CHART="./piholechart"

    git clone https://github.com/santisbon/pi-hole-k8s.git && cd pi-hole-k8s
    nano $CHART/values.yaml
    # edit the values
    ```
    Or if installing from the repository:
    ```shell
    # On your Pi
    helm repo add santisbon https://santisbon.github.io/charts/
    CHART="santisbon/pihole"
    ```
3. Install the Helm chart which will [enforce the installation order](https://helm.sh/docs/intro/using_helm). Replace parameters with a secure password and the static IP of your Pi if you didn't do it through the `values.yaml` file. If using MicroK8s type the commands as `microk8s helm` and `microk8s kubectl`.
    ```shell
    # On your Pi
    RELEASE=pihole
    NAMESPACE=pihole-n

    helm install $RELEASE $CHART \
        -n $NAMESPACE \
        --create-namespace \
        --set webPassword=supersecret \
        --set externalIP=192.168.XXX.XXX \
        --set webPort=8000
    ```

    https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/use-gitops-with-helm 

    az k8s-configuration create --name deploy-pihole-with-gitops --cluster-name home-pihole --resource-group home --operator-instance-name flux --operator-namespace pihole-ns --operator-params='--git-readonly --git-path=releases' --enable-helm-operator --helm-operator-chart-version='1.2.0' --helm-operator-params='--set helm.versions=v3' --repository-url https://github.com/armandoblanco/pi-hole-k8s.git --scope namespace --cluster-type connectedClusters


    az k8s-configuration create --name azure-arc-sample --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name flux --operator-namespace arc-k8s-demo --operator-params='--git-readonly --git-path=releases' --enable-helm-operator --helm-operator-chart-version='1.2.0' --helm-operator-params='--set helm.versions=v3' --repository-url https://github.com/Azure/arc-helm-demo.git --scope namespace --cluster-type connectedClusters


4. [Configure your router](https://docs.pi-hole.net/routers/asus/) to set Pi-hole as your DNS server.
5. From your desktop, access the web admin interface at http://192.168.XXX.XXX:8000/admin.

## Upgrade
```shell
helm upgrade $RELEASE $CHART -n $NAMESPACE
```

## Uninstall
```shell
helm uninstall $RELEASE -n $NAMESPACE --wait
kubectl delete namespaces $NAMESPACE
```
