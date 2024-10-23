Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.22.16.13:6443 --token ib5uyz.dqce2th47la6bcsc \
        --discovery-token-ca-cert-hash sha256:cf8a4c09716dfbb787392764fa451b5d0199d834ed9b9423e8bd54f1ba22ba01 



Activate the web console with: systemctl enable --now cockpit.socket

ssh -L 6443:localhost:6443 root@43.128.69.41