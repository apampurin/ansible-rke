
It is designet to work from root user. If your system doesn't work with root (centos and ssh issue for ex) change user in vars.yaml and became_user in playbooks.
Also you will need to "sudo" to all shell commands in playbooks which need superuser privilegies.


From master node install ansible
apt update &&\
  apt install -y software-properties-common && \
  add-apt-repository --yes --update ppa:ansible/ansible && \
  apt install ansible -y

Check firewall (this is script which I took from random internet manual)
Firewalld TCP ports:

for i in 22 80 443 179 5473 6443 8472 2376 8472 2379-2380 9099 10250 10251 10252 10254 30000-32767; do
    sudo firewall-cmd --add-port=${i}/tcp --permanent
done
sudo firewall-cmd --reload

Firewalld UDP ports:

for i in 8285 8472 4789 30000-32767; do
   sudo firewall-cmd --add-port=${i}/udp --permanent
done


fill ansible "hosts" file and vars.yaml

run:
ansible ansible-playbook ./playbooks/install_prerequisites.yaml

do "ssh-copy-id" cmd from master to worker node (run ssh-copy-id user@worker_ip or copy public key and write it in known_hosts file in worker user/.ssh/ folder)

run:
ansible ansible-playbook ./playbooks/install_rke.yaml
ansible ansible-playbook ./playbooks/install_rancher_dashboard.yaml