pipeline {
    agent any
    stages {
        stage('Setup Local Environment') {
            steps {
                echo '-- RUNNING LOCAL ENVIORNMENT --'
                sh '''
                #!/usr/bin/bash
                apt-get update
                apt-get install python3 python3-dev libffi-dev gcc libssl-dev docker.io -y
                apt install python3-pip -y
                apt install python3-venv -y
                python3 -m venv local
                source local/bin/activate
                '''
                
            }
            
        }
        stage('INSTALLING PIP') {
            steps {
                echo '-- INSTALLING PIP --'
                sh '''
                #!/usr/bin/bash
                pip install -U pip
                '''
                
            }
            
        }
        
        stage('INSTALLING Ansible') {
            steps {
                echo '-- INSTALLING Ansible --'
                sh '''
                #!/usr/bin/bash
                pip install 'ansible-core'
                '''
                
            }
            
        }
        stage('INSTALLING Kolla Ansible') {
            steps {
                echo '-- INSTALLING Kolla Ansible --'
                sh '''
                #!/usr/bin/bash
                pip install git+https://opendev.org/openstack/kolla-ansible@master
                kolla-ansible install-deps
                '''
                
            }
            
        }
        
        stage('Preparing Infrastructure') {
            steps {
                echo '-- Preparing Infrastructure Files Structure --'
                sh '''
                #!/usr/bin/bash
                mkdir -p /etc/kolla
                chown $USER:$USER /etc/kolla
                cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/
                cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla/
                sed -i 's/^#kolla_base_distro:.ls*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/ kolla/globals.yml
                sed -i 's/^#network_interface:.*/network_interface: "eth0"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth1"/g' /etc/kolla/globals.yml
                sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "10.0.2.15"/g' /etc/kolla/globals.yml
                '''
                
            }
            
        }
        stage('Secrets Setup') {
            steps {
                echo '-- Generating OpenStack Services Secrets --'
                sh '''
                #!/usr/bin/bash
                kolla-genpwd -p /etc/kolla/passwords.yml
                '''
                
            }
            
        }
        stage('Boostrap Servers') {
            steps {
                echo '-- Running Ansible Kolla Boostrap Server Script --'
                sh '''
                #!/usr/bin/bash
                kolla-ansible -i /etc/kolla/all-in-one bootstrap-servers
                '''
                
            }
            
        }
        
        stage('Infrastructure Pre-Checks') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''#!/usr/bin/bash
                kolla-ansible -i /etc/kolla/all-in-one prechecks
                '''
                
            }
            
        }
        stage('Deploy Infrastructure') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''
                #!/usr/bin/bash
                kolla-ansible -i /etc/kolla/all-in-one deploy
                '''
                
            }
            
        }
        
    }
    
}
