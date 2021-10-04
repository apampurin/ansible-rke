
It is designet to work from root user. If your system doesn't work with root (centos and ssh issue for ex) change user in vars.yaml and became_user in playbooks.
Also you will need to "sudo" to all shell commands in playbooks which need superuser privilegies.


From master node install ansible
```bash
    apt update && \
    apt install -y git software-properties-common && \
    add-apt-repository --yes --update ppa:ansible/ansible && \
    apt install ansible -y
```

Clone repo `git clone https://github.com/apampurin/ansible-rke.git`
open folder `cd ansible-rke`

fill ansible "hosts" file and group_vars/all.yaml file

copy public key to all nodes
do `ssh-copy-id` cmd from master to worker node (run `ssh-copy-id user@worker_ip` or copy public key and write it in authorized_keys file in worker user/.ssh/ folder)

run:
`ansible-playbook ./playbooks/install_prerequisites.yaml -i hosts`

run:
`ansible-playbook ./playbooks/install_rke.yaml -i hosts`


Add DNS record for rancher dashboard and set it in group_vars

`ansible-playbook ./playbooks/install_rancher_dashboard.yaml -i hosts`

Add email for cert-manager issuer ACME in group_vars

`ansible-playbook ./playbooks/install_cert_manager.yaml -i hosts`

If you get `Error from server (InternalError): error when creating "/root/issuer-prod.yaml": Internal error occurred: failed calling webhook "webhook.cert-manager.io": Post "https://cert-manager-webhook.cert-manager.svc:443/mutate?timeout=10s": context deadline exceeded` while issuer installation step go to the master node and execute `kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook && kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io cert-manager-webhook`. Then make `kubectl apply -f ~/issuer-prod.yaml && kubectl apply -f ~/issuer-staging.yaml`

Check firewall (this is script which I took from random internet manual)
Firewalld TCP ports:

```bash
for i in 22 80 443 179 5473 6443 8472 2376 8472 2379-2380 9099 10250 10251 10252 10254 30000-32767; do
    sudo firewall-cmd --add-port=${i}/tcp --permanent
done
sudo firewall-cmd --reload
```

Firewalld UDP ports:

```bash
for i in 8285 8472 4789 30000-32767; do
   sudo firewall-cmd --add-port=${i}/udp --permanent
done
```


There is random bug when cattle-cluster pod is not running (could not resolve rancher host).
In that case you need to edit cattle deployments manually
(to check if cattle-cluster run `kubectl get pods -n cattle-system`)

`kubectl edit -n cattle-system deployment.apps/cattle-cluster-agent`
find dnsPolicy string and replace with:

```
dnsPolicy: "None"
      dnsConfig:
        nameservers:
          - 8.8.8.8
          - 8.8.4.4
```