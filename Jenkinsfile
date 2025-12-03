pipeline {
    agent any
    environment {
        VENV_PATH = "${WORKSPACE}/local"
        VENV_ACTIVATE = "${WORKSPACE}/local/bin/activate"
    }
    stages {
        stage('Setup Local Environment') {
            steps {
                echo '-- Setting up Python virtual environment --'
                sh '''
                #!/bin/bash

                rm -rf "$VENV_PATH"

                # Ensure Docker is running
                sudo systemctl restart docker.service

                python3 -m venv local
                . "$VENV_ACTIVATE"

                echo "Python: $(which python3)"
                echo "Pip: $(which pip)"
                echo "VIRTUAL_ENV: $VIRTUAL_ENV"
                '''
            }
        }

        stage('Upgrade Pip') {
            steps {
                echo '-- Upgrading pip --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"
                pip install -U pip
                '''
            }
        }

        stage('Install System Dependencies') {
            steps {
                sh '''
                #!/bin/bash
                #apt update
                #apt install -y git iputils-ping tzdata docker.io
                #systemctl enable --now docker
                '''
            }
        }

        stage('Install Kolla-Ansible and Dependencies') {
            steps {
                echo '-- Installing Kolla-Ansible and required packages --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                pip install 'git+https://opendev.org/openstack/kolla-ansible@master'
                pip install 'jinja2>=3.0.0'
                pip install 'netaddr'
                pip install 'python-openstackclient'

                kolla-ansible install-deps
                '''
            }
        }

        stage('Prepare Infrastructure Config') {
            steps {
                echo '-- Preparing Kolla configuration files --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"

                mkdir -p $VENV_PATH/etc/kolla

                KOLLA_SHARE="$VIRTUAL_ENV/share/kolla-ansible"
                if [ ! -d "$KOLLA_SHARE" ]; then
                    echo "ERROR: Kolla-Ansible share directory not found!" >&2
                    exit 1
                fi

                # Copy config templates
                cp -r "$KOLLA_SHARE/etc_examples/kolla/"* "$VENV_PATH/etc/kolla/"
                cp -r "$KOLLA_SHARE/ansible/inventory/all-in-one" "$VENV_PATH/etc/kolla/"

                # Customize globals.yml
                sed -i 's/^#enable_zun:.*/enable_zun: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_kuryr:.*/enable_kuryr: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#containerd_configure_for_zun:.*/containerd_configure_for_zun: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_etcd:.*/enable_etcd: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#docker_configure_for_zun:.*/docker_configure_for_zun: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#kolla_base_distro:.*/kolla_base_distro: "ubuntu"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#network_interface:.*/network_interface: "ens3"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "ens11"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "192.168.7.15"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_proxysql:.*/enable_proxysql: "no"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#neutron_ovn_distributed_fip:.*/neutron_ovn_distributed_fip: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#neutron_enable_ovn_agent:.*/neutron_enable_ovn_agent: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_manila:.*/enable_manila: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#neutron_plugin_agent:.*/neutron_plugin_agent: "ovn"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_manila_backend_generic:.*/enable_manila_backend_generic: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_cinder_backend_nfs:.*/enable_cinder_backend_nfs: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i 's/^#enable_cinder_backend_lvm:.*/enable_cinder_backend_lvm: "yes"/'g $VENV_PATH/etc/kolla/globals.yml
                sed -i '$a\\enable_package_install: "no"' $VENV_PATH/etc/kolla/globals.yml

                '''
            }
        }

        stage('Generate Secrets') {
            steps {
                echo '-- Generating passwords.yml --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                # Get template if not exists (kolla-genpwd requires it)
                #if [ ! -f /etc/kolla/passwords.yml ]; then
                #    cp "$VIRTUAL_ENV/share/kolla-ansible/etc_examples/kolla/passwords.yml" ./passwords.yml
                #fi

                cp "$VIRTUAL_ENV/share/kolla-ansible/etc_examples/kolla/passwords.yml" ./passwords.yml

                kolla-genpwd -p ./passwords.yml

                cp ./passwords.yml $VENV_PATH/etc/kolla/
                chmod 644 $VENV_PATH/etc/kolla/passwords.yml
                '''
            }
        }


       stage('Install System Dependencies') {
           steps {
               sh '''
               #!/bin/bash
               # Install required OS packages manually (bypass Kolla's apt task)
               sudo apt update
               sudo apt install -y git iputils-ping tzdata

               # Disable package management in Kolla
               if ! grep -q "enable_package_install" /etc/kolla/globals.yml; then
                  echo 'enable_package_install: "no"' >> $VENV_PATH/etc/kolla/globals.yml
               fi
               '''
            }
        }



        stage('Bootstrap Servers') {
            steps {
                echo '-- Running bootstrap-servers --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                kolla-ansible bootstrap-servers -i $VENV_PATH/etc/kolla/all-in-one -vvv
                '''
            }
        }

        stage('Prechecks') {
            steps {
                echo '-- Running prechecks --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                kolla-ansible prechecks -i $VENV_PATH/etc/kolla/all-in-one -vvv
                '''
            }
        }

        stage('Deploy OpenStack') {
            steps {
                echo '-- Deploying OpenStack with Kolla-Ansible --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                kolla-ansible deploy -i $VENV_PATH/etc/kolla/all-in-one -vvv
                '''
            }
        }

        stage('Post-Deployment') {
            steps {
                echo '-- Running post-deploy tasks --'
                sh '''
                #!/bin/bash
                . "$VENV_ACTIVATE"
                export http_proxy="http://192.168.11.10:800"
                export https_proxy="http://192.168.11.10:800"
                export no_proxy="localhost,127.0.0.0/8,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12"

                kolla-ansible post-deploy -i $VENV_PATH/etc/kolla/all-in-one -vvv

                # Source and verify Octavia OpenRC (optional)
                if [ -f $VENV_PATH/etc/kolla/octavia-openrc.sh ]; then
                    . $VENV_PATH/etc/kolla/octavia-openrc.sh
                    env | grep -E "^OS_"
                fi
                '''
            }
        }
    }
}

