# pve-twingate-ansible
twingate for proxmox homelab

install ansible
edit vars in cpcvm.yml
when setting up connector use 'script-based deployments' select Linux, genterate tokens, click 'copy command' and paste into your environment var

export TWINGATE='pastehere'

ansible-playbook cpcvm.yml

