pipeline {
    agent any
    environment {
        VENV_PATH = "${WORKSPACE}/local"
        VENV_ACTIVATE = "${WORKSPACE}/local/bin/activate"
    }
    stages {
        stage('Setup Local Environment') {
            steps {
                echo '-- RUNNING LOCAL ENVIORNMENT --'
                sh '''
                #!/usr/bin/bash
                
                # sudo apt-get update
                # sudo apt-get install python3 python3-dev libffi-dev gcc libssl-dev docker.io -y
                # sudo apt install python3-pip -y
                # sudo apt install python3-venv -y
                
                sudo systemctl restart docker.service
                sudo python3 -m venv local
                . local/bin/activate
                '''
                
            }
            
        }
        stage('INSTALLING PIP') {
            steps {
                echo '-- INSTALLING PIP --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo pip install -U pip
                '''
                
            }
            
        }
        
        stage('INSTALLING Ansible') {
            steps {
                echo '-- INSTALLING Ansible --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo pip install ansible-core
                '''
                
            }
            
        }
        stage('INSTALLING Kolla Ansible') {
            steps {
                echo '-- INSTALLING Kolla Ansible --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo pip install git+https://opendev.org/openstack/kolla-ansible@master
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo kolla-ansible install-deps
                '''
                
            }
            
        }
        
        stage('Preparing Infrastructure') {
            steps {
                echo '-- Preparing Infrastructure Files Structure --'
                sh '''
                #!/usr/bin/bash
                
                sudo mkdir -p /etc/kolla
                sudo chown $(whoami):$(whoami) /etc/kolla
                sudo cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/
                sudo cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla/
                sudo sed -i 's/^#kolla_base_distro:.ls*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
                sudo sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
                sudo sed -i 's/^#network_interface:.*/network_interface: "ens3"/g' /etc/kolla/globals.yml
                sudo sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "ens11"/g' /etc/kolla/globals.yml
                sudo sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "192.168.7.19"/g' /etc/kolla/globals.yml
                # sudo sed -i -e 's|localhost.*ansible_connection.*|192.168.7.15 ansible_host=192.168.7.15 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa|g' /etc/kolla/all-in-one
                '''
                
            }
            
        }
        stage('Secrets Setup') {
            steps {
                echo '-- Generating OpenStack Services Secrets --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo kolla-genpwd -p /etc/kolla/passwords.yml
                '''
                
            }
            
        }
        stage('Boostrap Servers') {
            steps {
                echo '-- Running Ansible Kolla Boostrap Server Script --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo systemctl restart docker.service
                sudo kolla-ansible bootstrap-servers -i /etc/kolla/all-in-one
                '''
                
            }
            
        }
        
        stage('Infrastructure Pre-Checks') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo systemctl restart docker.service
                sudo kolla-ansible prechecks -i /etc/kolla/all-in-one
                '''
                
            }
            
        }
        stage('Deploy Infrastructure') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''
                #!/usr/bin/bash
                . local/bin/activate
                export http_proxy=http://192.168.11.10:800
                export https_proxy=http://192.168.11.10:800
                sudo systemctl restart docker.service
                sudo kolla-ansible deploy -i /etc/kolla/all-in-one
                '''
                
            }
            
        }
        
    }
    
}
