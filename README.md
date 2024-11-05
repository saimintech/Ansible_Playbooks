**Pre-Step:** sudo apt update && apt -y upgrade && apt -y install curl wget git ansible

____________________________________________________________________________________________
**CloudPanel:** ansible-playbook -i hosts.ini cloudpanel_install.yaml

**Reverse Proxy:** ansible-playbook apache_reverse_proxy.yml -e "domain_name=yourdomain.com local_port=3000"

**Editor:** ansible-playbook add_site.yml -e "domain_name=yourdomain.com"
