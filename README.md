# K8s-Control-Plane
Configuração do kubernetes, vamos utilizar o K8s para criar um control plane em nosso ambiente On-premise (Linux Ubunto).

![Untitled](https://github.com/douglas5001/K8s-Control-Plane/blob/main/K8s.png?raw=true)

### Segue a configuração padrão para todas as maquinas

Interessante que você tenha um server como o por exemplo o meu que é uma VPS Ubunto na Contabo e com 1 ip externo.

1. **Configurar o Sistema Operacional**:
    
    Certifique-se de que seu linux esteja atualizado para a versão mais recente:
    
    ```
    sudo apt update
    sudo apt upgrade -y
    
    ```
    
2. **Instalar o Containerd**:
    
    Kubernete necessita de um container engine, nesse caso vamos utilizar o ContainerD do Docker
    
    ```
    sudo apt install -y containerd
    
    ```
    
    Após intalar, vamos criar o arquivo a seguir. Crie o arquivo **`/etc/containerd/config.toml`** com o seguinte conteúdo:
    
    ```python
   [plugins."io.containerd.grpc.v1.cri"]
     disable_tcp_service = true
     disable_cgroups = false
     stream_server_address = "127.0.0.1:10010"  # Correção aqui
     stream_idle_timeout = "4h0m0s"
     enable_selinux = false
     selinux_category_range = 1024
     enable_apparmor = false
     sandbox_image = "k8s.gcr.io/pause:3.4.1"

   [plugins."io.containerd.grpc.v1.cni"]
     network_dir = "/etc/cni/net.d"
     max_conflict = 0

   [plugins."io.containerd.grpc.v1.containerd"]
     shim_debug = false

   [plugins."io.containerd.grpc.v1.diff"]
     default = "native"

   [plugins."io.containerd.grpc.v1.snapshots"]
     default = "overlayfs"
```
    
    Reinicie o ContainerD
    
    ```
    sudo systemctl restart containerd
    
    ```
    
3. **Configurar o DNS**:
    
    Configure os servidores DNS do Ubuntu para que o Kubernetes resolva os nomes de host. Adicione servidores DNS públicos, como os do Google, ao arquivo. **`/etc/netplan/01-netcfg.yaml`**. Por exemplo:
    
    (NO MEU CASO UTILIZO UMA VPS NA CONTABO, A CONFIGURAÇÂO DEIXEI A PADRAO)
    
    ```
    network:
      version: 2
      ethernets:
        ens160:
          dhcp4: yes
          dhcp4-overrides:
            use-dns: false
          nameservers:
            addresses: [8.8.8.8, 8.8.4.4]
    
    ```
    
    Aplique as configurações de rede:
    
    ```
    sudo netplan apply
    ```
    
4. **Desativar o Swap**:
    
    O Kubernetes não é compatível com swap. Certifique-se de que o swap esteja desativado.
    
    ```
    sudo swapoff -a
    ```
    
    Para desativá-lo permanentemente, você deve editar o arquivo **`/etc/fstab`** e remover a entrada de swap.
    
5. **Instalar kubeadm, kubelet e kubectl**:
    
    Você pode instalar o Kubernetes no Ubuntu usando os seguintes comandos:
    
    ```
    sudo apt update && sudo apt install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    sudo apt install -y kubeadm kubelet kubectl
    ```
    
6. **Inicializar o Cluster Kubernetes**:
    
    Utilize o comando **`kubeadm init`** para inicializar o cluster Kubernetes. Certifique-se de especificar o endereço IP externo do server como parte do comando, utilizando a opção **`--apiserver-advertise-address`**:
    
    ```
    sudo kubeadm init --apiserver-advertise-address=SEU-IP-EXTERNO --pod-network-cidr=10.244.0.0/16
    ```
    
    Anote o token de inicialização para adicionar outros nós ao cluster.
    
7. **Configurar o Kubectl para Usuário Normal**:
    
    Após a inicialização do cluster, configure o kubectl para o usuário normal usando os comandos gerados pelo kubeadm init.
    
    ```
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
8. **Instalar um Plugin de Rede (opcional)**:
    
    Para a comunicação entre os pods de um cluster, é necessário instalar um plugin de rede, como o Calico.
    
    ```
    bashCopy code
    kubectl apply -f https://docs.projectcalico.org/v3.21/manifests/calico.yaml
    ```
    
    Verifique as versões em https://docs.tigera.io/
    
9. **Ingressar em Outros Nós**
    
    Se você tiver outras VPS e quiser adicioná-las ao cluster Kubernetes, use o comando kubeadm join e o token de inicialização gerado no passo 6 para adicionar os nós. Existem mais configuração para este procedimento, verifique a documentação do site Official https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
