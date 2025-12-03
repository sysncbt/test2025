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

                ## Clean up any existing virtual environment
                rm -rf $VENV_PATH
                    
                #  apt-get update
                #  apt-get install python3 python3-dev libffi-dev gcc libssl-dev docker.io -y
                #  apt install python3-pip -y
                ##  apt install python3-venv -y
                
                systemctl restart docker.service
                python3 -m venv local
                . $VENV_ACTIVATE

                # Verify
                echo "Python path: $(which python3)"
                echo "Pip path: $(which pip)"
                echo "VIRTUAL_ENV: $VIRTUAL_ENV"

                '''
                
            }
            
        }
        stage('INSTALLING PIP') {
            steps {
                echo '-- INSTALLING PIP --'
                sh '''
                #!/usr/bin/bash

                . $VENV_ACTIVATE

                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                pip install -U pip

                '''
                
            }
            
        }
        
        stage('INSTALLING Ansible') {
            steps {
                echo '-- INSTALLING Ansible --'
                sh '''
                #!/usr/bin/bash


                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                pip install ansible-core

                '''
                
            }
            
        }
        stage('INSTALLING Kolla Ansible') {
            steps {
                echo '-- INSTALLING Kolla Ansible --'
                sh '''
                #!/usr/bin/bash

                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                pip install git+https://opendev.org/openstack/kolla-ansible@master
                pip install 'jinja2>=3.0.0'
                pip install 'netaddr'
                pip install 'python-openstackclient'
                kolla-ansible install-deps

                '''
                
            }
            
        }
        
        stage('Preparing Infrastructure') {
            steps {
                echo '-- Preparing Infrastructure Files Structure --'
                sh '''
                #!/usr/bin/bash

                . $VENV_ACTIVATE                
                mkdir -p /etc/kolla
                mkdir -p /etc/kolla/inventory/
                chown $(whoami):$(whoami) /etc/kolla
                if [ ! -d "/usr/local/share/kolla-ansible" ]; then
                   echo "Kolla-Ansible files not found! Ensure kolla-ansible is installed." >&2
                   exit 1
                fi
                

                cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/
                cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla/inventory/
                
                sed -i 's/^#enable_zun:.*/enable_zun: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_kuryr:.*/enable_kuryr: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#containerd_configure_for_zun:.*/containerd_configure_for_zun: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_etcd:.*/enable_etcd: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#docker_configure_for_zun:.*/docker_configure_for_zun: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#kolla_base_distro:.*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
                sed -i 's/^#network_interface:.*/network_interface: "ens3"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "ens11"/g' /etc/kolla/globals.yml
                sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "192.168.7.15"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_proxysql:.*/enable_proxysql: "no"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_ovn_distributed_fip:.*/neutron_ovn_distributed_fip: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_enable_ovn_agent.*/neutron_enable_ovn_agent: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_manila:.*/enable_manila: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_plugin_agent:.*/neutron_plugin_agent: "ovn"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_manila_backend_generic:.*/enable_manila_backend_generic: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth11"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_cinder_backend_nfs:.*/enable_cinder_backend_nfs: "yes"/g' /etc/kolla/globals.yml
                sed -i 's/^#enable_cinder_backend_lvm:.*/enable_cinder_backend_lvm: "yes"/g' /etc/kolla/globals.yml

                '''
                
            }
            
        }
        stage('Secrets Setup') {
            steps {
                echo '-- Generating OpenStack Services Secrets --'
                sh '''
                #!/usr/bin/bash
                
                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                kolla-genpwd -p /etc/kolla/passwords.yml
                
                '''
                
            }
            
        }
        stage('Boostrap Servers') {
            steps {
                echo '-- Running Ansible Kolla Boostrap Server Script --'
                sh '''
                #!/usr/bin/bash
                

                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                sudo systemctl restart docker.service
                . $VENV_ACTIVATE
                kolla-ansible bootstrap-servers -i /etc/kolla/all-in-one -vvv
                '''
                
            }
            
        }
        
        stage('Infrastructure Pre-Checks') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''
                #!/usr/bin/bash

                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                sudo systemctl restart docker.service
                . $VENV_ACTIVATE                
                kolla-ansible prechecks -i /etc/kolla/all-in-one -vvv
                '''
                
            }
            
        }
        stage('Deploy Infrastructure') {
            steps {
                echo '-- Running Ansible Kolla Prechecks Script --'
                sh '''
                #!/usr/bin/bash


                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                sudo systemctl restart docker.service
                . $VENV_ACTIVATE
                kolla-ansible deploy -i /etc/kolla/all-in-one -vvv
                '''
                
            }
            
        }
        

        stage('Post-Deployment Tasks') {
            steps {
                echo '-- Running Post-Deployment Tasks --'
                sh '''
                #!/usr/bin/bash


                . $VENV_ACTIVATE
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                # Run the kolla-ansible post-deploy script
                kolla-ansible post-deploy -i /etc/kolla/all-in-one -vvv

                # . the OpenRC file for Octavia
                . /etc/kolla/octavia-openrc.sh

                # Optional: Verify the environment is set correctly
                # You can uncomment the line below to see if the variables are loaded.
                env | grep OS_
                '''
            }
        }


    }
    
}
