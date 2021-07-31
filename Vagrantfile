IMAGE_NAME = "tiflor/ubuntu2004_containerd"  #bento/ubuntu-20.04"

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
      
    boxes = [
      { :name => "control1", :ip => "192.168.50.10" },
      { :name => "node1", :ip => "192.168.50.11" },
      { :name => "node2", :ip => "192.168.50.12" }
    ]

    boxes.each do |opts|
      config.vm.define opts[:name] do |node|
        node.vm.box = IMAGE_NAME
        node.vm.hostname = opts[:name]
        node.vm.network :private_network, ip: opts[:ip]
            
        # Provision all the VMs in parallel using Ansible after last VM is up.
        if opts[:name] == "node2"
            node.vm.provision "ansible" do |ansible|
                # require 'json'

                # kubernetes_pod_network = [{
                #   # Flannel CNI.
                #   # cni: 'flannel'
                #   # cidr: '10.244.0.0/16'
                #   # Calico CNI.
                #   "cni" => "calico",
                #   "cidr" => "192.169.0.0/16",
                # }]

                ansible.compatibility_mode = "2.0"
                ansible.playbook = "ansible/playbook.yml"
                ansible.limit = "all"
                #ansible.galaxy_role_file = "ansible/requirements.yml"
                ansible.galaxy_roles_path = "ansible/roles"
                #ansible.verbose = "vv"
                ansible.host_vars = {
                    "control1" => {"node_ip" => "192.168.50.10",
                                  },
                    "node1"    => {"node_ip" => "192.168.50.11",
                                   },
                    "node2"    => {"node_ip" => "192.168.50.12",
                                   },
                }
                ansible.groups = {
                  "control_plane" => ["control1"],
                  "control_plane:vars" => {
                    kubernetes_role: "master",
                  },
                  "node" => ["node[1:2]"],
                  "node:vars" => {
                    kubernetes_role: "node",
                  },
                }
                ansible.extra_vars = {
                  ansible_python_interpreter:"/usr/bin/python3",
                  kubernetes_pod_network: {
                      # Flannel CNI.
                      # cni: 'flannel'
                      # cidr: '10.244.0.0/16'
                      # Calico CNI.
                      cni: "calico",
                      cidr: '192.169.0.0/16',
                    },
                  kubernetes_kubelet_extra_args: '--node-ip={{ node_ip }}',
                  # kubernetes_kubeadm_init_extra_opts: '--apiserver-cert-extra-sans="192.168.50.10" ',
                  kubernetes_apiserver_advertise_address: "192.168.50.10", 
                  kubernetes_version: '1.21',
                }
                end
            end
        end
    end
end