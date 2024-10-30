**CloudPanel:** ansible-playbook -i hosts.ini cloudpanel_install.yaml

**Reverse Proxy:** ansible-playbook reverse_proxy.yml -e "domain_name=yourdomain.com local_port=3000" -i localhost
