# Deploying Apica Ascent PaaS on MicroK8s

[MicroK8s](https://microk8s.io) is a lightweight, pure-upstream Kubernetes aiming to reduce entry barriers for K8s and cloud-native application development. It comes in a single package that installs a single-node (standalone) K8s cluster in under 60 seconds. The lightweight nature of Apica Ascent PaaS enables you to deploy Apica Ascent on lightweight, single-node clusters like MicroK8s.

The following guide takes you through deploying Apica Ascent PaaS on MicroK8s.

## Prerequisites

- Ubuntu x64
The machine will need to be large and resource heavy, as you will be hosting a web application on it. Therefore use the following specifications:
- 16 vCPU
- 32 GB

## Installing MicroK8s

The first step in this deployment is to install MicroK8s on your machine. To install MicroK8s on such systems, do the following.

1.  Update package lists by running the following command.

    ```
    sudo apt-get -y update
    ```
2.  Install `core` using Snap by running the following command.

    ```
    sudo snap install core
    ```
3.  Install MicroK8s using Snap by running the following command.

    ```
    sudo snap install microk8s --classic --channel=1.28/stable
    ```
4.  Join the group created by MicroK8s that enables uninterrupted usage of commands that require admin access by running the following command.

    ```
    sudo usermod -a -G microk8s $USER
    ```
5. **Create the .kube directory**
6.  Add your current user to the group to gain access to the `.kube` caching directory by running the following command.

    ```
    sudo chown -f -R $USER ~/.kube
    ```
7.  Generate your MicroK8s configuration and merge it with your Kubernetes configuration by running the following command.

    ```
    sudo microk8s config > ~/.kube/config
    ```
8.  Check whether MicroK8s is up and running with the following command.

    ```
    sudo microk8s status
    ```

MicroK8s is now installed on your machine.

## Enabling add-ons

Now that we have MicroK8s up and running, let’s set up your cluster and enable the add-ons necessary such as Helm, CoreDNS, ingress, storage, and private registry. MicroK8s readily provides these addons and can be enabled and disabled at any time. Most of these add-ons are pre-configured to work without any additional setup.

To enable add-ons on your MicroK8s cluster, run the following commands in succession.

1.  Enable Helm 3.

    ```
    microk8s enable helm3
    ```
2.  Enable a default storage class that allocates storage from a host directory.

    ```
    microk8s enable storage
    ```
3.  Enable CoreDNS.

    ```
    microk8s enable dns
    ```
4.  Enable ingress.

    To enable the Ingress controller in MicroK8s, run the following command:

    ```
    microk8s enable ingress
    ```

    1. Enable HTTPS
       1. How to Create a Self-Signed Certificate using OpenSSL
          1.  Create server private key

              ```
              openssl genrsa -out cert.key 2048
              ```
          2.  Create certificate signing request (CSR)

              ```
              openssl req -new -key cert.key -out cert.csr
              ```
          3.  Sign the certificate using the private key and CSR

              ```
              openssl x509 -req -days 3650 -in cert.csr -signkey cert.key -out cert.crt
              ```
       2.  To create a TLS secret in MicroK8s using `kubectl`, use the following command:

           ```
           microk8s kubectl create secret tls https --cert=cert.crt --key=cert.key
           ```

           This command creates a secret named "https" containing the TLS keys for use in your Kubernetes cluster. Ensure you have the `cert.crt` and `cert.key` files in your current directory or specify full paths.
       3.  To enable Ingress on microk8s with a default SSL certificate, issue the following command:

           ```
           microk8s enable ingress:default-ssl-certificate=secret/https
           ```
5.  Enable private registry.

    ```
    microk8s enable registry
    ```


6.  Copy over your MicroK8s configuration to your Kubernetes configuration with the following command.

    ```
    microk8s.kubectl config view --raw > $HOME/.kube/config
    ```

## Provisioning an IP address

In this step, we'll provision an endpoint or an IP address where we access Apica Ascent PaaS after deploying it on MicroK8s. You can skip this step if its your local implementation. For this, we'll leverage [MetalLB](https://metallb.universe.tf) which is a load-balancer implementation that uses standard routing protocols for bare metal Kubernetes clusters.

{% hint style="info" %}
**Note:** Since MetalLB is available as an add-on for MicroK8s, you can also run these steps while enabling add-ons for your MicroK8s cluster.
{% endhint %}

To provision an IP address, do the following:

1.  Check your local machine's IP address by running the `ifconfig` command, as shown below.

    ```
     ifconfig:
     wlp60s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
     inet 192.168.1.27 netmask 255.255.255.0 broadcast 192.168.1.255
    ```
2.  Enable MetalLB by running the following command.

    ```
     microk8s enable metallb
     Enabling MetalLB
     Enter each IP address range delimited by comma (e.g.     '10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111'): 192.168.1.27-192.168.1.27
    ```

{% hint style="info" %}
**Note:** MetalLB will not work on macOS due to network filtering that macOS applies. MetalLB might not work if you're provisioning an EC2 instance on AWS due to your private/public IP configuration.
{% endhint %}

## Installing Apica Ascent PaaS

Now that your MicroK8s environment is configured and ready, we can proceed with installing Apica Ascent PaaS on it. To install Apica Ascent PaaS using Helm, do the following:

1.  Add the Apica Ascent PaaS Helm chart to your Helm repository by running the following command.

    ```
    microk8s helm3 repo add logiq-repo https://logiqai.github.io/helm-charts
    ```
2.  Update your Helm repository by running the following command.

    ```
    microk8s helm3 repo update
    ```
3.  Create a namespace on MicroK8s on which to install Apica Ascent PaaS.

    ```
    microk8s kubectl create namespace logiq
    ```
4.  Prepare your values.microk8s.yaml file. You can use the [**starter `values.microk8s.yaml`**](https://github.com/logiqai/logiq-installation/blob/main/values/values.microk8s.yaml) file we've created to configure your Apica Ascent PaaS deployment

    > Optionally, if you are provisioning public IP using Metallb, use the [values.yaml](https://github.com/logiqai/logiq-installation/blob/main/values/values.yaml) instead. run the following command.
    >
    > ```
    > microk8s enable metallb
    > Enabling MetalLB
    > Enter each IP address range delimited by comma (e.g.  '10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111'): 192.168.1.27-192.168.1.27
    > ```
    >
    > In the values file, add the below fields global-> environment section with your own values.
    >
    > ```
    > s3_bucket: <your-s3-bucket>
    > AWS_ACCESS_KEY_ID: <your-aws-access-key-id>
    > AWS_SECRET_ACCESS_KEY: <your-aws-secret-access-key-id>
    > ```
    >
    > In the global -> chart section, change S3gateway to false.
    >
    > ```
    > s3gateway: false
    > ```
    >
    > In the global -> persistence section, change storageClass as below.
    >
    > ```
    > storageClass: microk8s-hostpath
    > ```
5.  Install Apica Ascent PaaS using Helm with the storage class set to `microk8s-hostpath` with the following command.

    ```
    microk8s helm3 install logiq -n logiq --set global.persistence.storageClass=microk8s-hostpath logiq-repo/apica-ascent -f  values.microk8s.yaml  --debug --timeout 10m
    ```

Apica Ascent PaaS is now installed in your MicroK8s environment.

## Accessing Apica Ascent PaaS

Now that Apica Ascent PaaS is installed on your MicroK8s cluster, you can visit the Apica Ascent PaaS UI by accessing the MetalLB endpoint we defined in the pre-install steps. To access the Apica Ascent PaaS UI, do the following:

1.  Inspect the pods in your MicroK8s cluster in the `logiq` namespace by running the following command.

    ```
    microk8s kubectl get pod -n logiq
    ```
2.  Find the exact MetalLB endpoint that's serving the Apica Ascent PaaS UI by running the following command.

    ```
    microk8s kubectl get service -n logiq |grep -i loadbalancer
    ```

    The above command should give you an output similar to the following.

    ```
    logiq-kubernetes-ingress  LoadBalancer   10.152.183.45  192.168.1.27

    80:30537/TCP,20514:30222/TCP,24224:30909/TCP,24225:31991/TCP,2514:30800/TCP,3000:32680/TCP,514:32450/    TCP,7514:30267/TCP,8081:30984/TCP,9998:31425/TCP     18m
    ```
3. Using a web browser of your choice, access the IP address shown by the load balancer service above. For example, `http://192.168.1.27:80`.

Your Apica Ascent PaaS UI is now available in your web browser. You can log into Apica Ascent PaaS using the following default credentials.

* **Username**: `flash-admin@foo.com`
* **Password**: `flash-password`

{% hint style="info" %}
**Note:** You can change the default login credentials after you've logged into the UI.
{% endhint %}
