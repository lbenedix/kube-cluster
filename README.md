# Kubernetes Multi Master Cluster
Kubernetes HA(multi master with etcd clustering, multi minion) Secure(Client Certification on etcd and kube-api-server) Cluster on Ubuntu 16.04.
Tested with Kube version v1.4.5 and v1.4.6
## Limitations
  1. **Master nodes are dedicated master only node. kube-proxy,kubelet and flannel are not installed. So master can not reach out to any containers**
  2. **https://master_ip/ui will not work as master can not reach to containers. Dashboard is hosted as a separate internal service available on minion nodes on port 9090 using dashboard service ip address. If required it can be proxied through any reverse proxy server like NGINX. Or Service can expose NodePort and can be available outside the cluster**
  3. **kube-apiserver exposes port 8080 for 127.0.0.1 interface on master. Once https://github.com/kubernetes/kubernetes/issues/13598 is fixed and avialble, --insecure-port will be set to 0.**
  4. **Flannel does not secure data packets. There is a PR (https://github.com/coreos/flannel/pull/290) to add ipsec backend that would encrypt data packets. Once this feature is avialble, setup will be configured to secure it.**
  5. **kubelet and kube-proxy does not support multiple kube-apiserver address. So we still have a single point of failure, as only one ip address can be configured. To workaround, we can expose all the master nodes under an external load balancer and then point to that address. Issue is logged and worked upon in kubernetes. https://github.com/kubernetes/kubernetes/issues/19152**

## Features
  1. Multi Master Cluster with ETCD clustering. 
  2. TLS communication and client cert authentication between ETCD client and peer communication.
  3. TLS communication and client cert authentication between all kube-components
  4. Flannel is used for networking. Flannel does not support TLS yet.

## Instructions to set up
  1. Clone this repo on a local machine. Local machine should not be part of cluster.
  2. Setup SSH (skip this step, if ssh key authentication is already set up.)
    1. ssh-keygen on your local machine
    2. Copy your ssh public key on test servers by using ssh-copy-id
    ```
    ssh-keygen
    ssh-copy-id <server1>
    ssh-copy-id <server2>
    ssh-copy-id <server3>
    ```
  3. Edit config-default.sh
    1. MASTER_NODES, Array of master nodes
    2. WORKER_NODES, Array of worker nodes.
    3. DNS_SERVER_IP, IP Address of dns pod.
    4. SERVICE_CLUSTER_IP_RANGE, IP Range to be used by Kuberenetes Services.
    5. SERVICE_NODE_PORT_RANGE, Port Range which services can expose.
    6. CERT_DIR, Certificate directory where certificates will be stored on Server.
    7. FLANNEL_NET, IP Subnet to be used for pods.
    8. MASTER_NODE_IP, if blank first master node ip address will be used, while communicating from minion to master.
    9. CLUSTER_DNS_EXTERNAL, If not blank will put this in master node extra sans certs so as to communicate with master node using this domain name.
    10.GENERATE_CERTS, if true will generate a ca cert and cert for each node. If false, we need to manually make sure that certificates with proper names are placed in CERT_DIR on all master and minion nodes.
    11.DEPLOY_TEST_DEPLOYMENT, Not supported yet. In future it will deploy a test-api and a NGINX reverse proxy to test the cluster.
  4. Run ./deploy-ha-cluster.sh
