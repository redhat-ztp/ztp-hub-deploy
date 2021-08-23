# ztp-hub-deploy
Playbook to deploy a hub cluster with dependencies for ztp. It will run on a clean system, and configure
all the network requirements to deploy a hub cluster using a physical network, but running on
virtual machines.

The cluster will be deployed using a disconnected network. There will be an installer VM that will act as mirror via another IP connected network.

In order to launch it, simply execute:

    ansible-playbook -i /path/to/inventory.yaml playbook.yml

It has 2 tags:
- prepare-environment: it will install all the dependencies, and will prepare network connectivity in the host - create bridge, setup dnsmasq, etc... 
- create-provisioner-cluster: it will install a hub cluster with the needed settings to act as a hub, including mirroring of key components. It will be an ipv6, disconnected cluster

## Inventory definition

The inventory is better expressed using yaml format. There is a `inventory/hosts.yaml.sample` provided in order to reflect all settings. The default examples are based on IPV6.
Following there is an explanation of all:
- **pull_secret**: a JSON-formatted string, contained all the credentials needed for the deployment.  It needs to include credentials for quay.io, registry.ci.openshift.org... depending on the source of your images
- **temporary_path**: just defaulting to /tmp, update it if you need your path to be different
- **installer_disk_size**: the disk size to use for installer VM. The size will vary depending on the amount of content that needs to be mirrored
- **provisioner_cluster_name**: the name to be given to the cluster
- **provisioner_cluster_domain**: the domain to be given to the cluster. A dnsmasq server will be configured using those settings
- **provisioner_cidr**: the IP CIDR range where to deploy the provisioner cluster. It needs to match with a physical network configured in your network
- **provisioner_api_ip**: the IP to which the API endpoint will be associated (vip endpoint). Point to a free ip on the network CIDR.
- **provisioner_ingress_ip**: the IP to which the Ingress endpoint will be associated (vip endpoint). Point to a free ip on the network CIDR.
- **provisioner_ip**: the IP that will receive the bridge that is going to be created in your system
- **provisioner_dnsmasq_from / provisioner_dnsmasq_to**:  ip range where to assign ips for the virtual machines of the cluster
- **provisioner_mask**: mask of the provisioner network
- **provisioner_cluster_network**: name of the bridge that is going to be created in your system
- **bridge_nic**: physical interface to where the bridge is going to be associated. Needs to point to the disconnected network that you are going to use for deploying your cluster
- **default_libvirt_range**: network range for the virtual network. Used for installer VM, this is just going to create a virtual network on your system
- **provisioner_proxy**: if desired, a proxy can be used to cache package installation on installer vm. If you have some varnish/squid you can use, set it there
- **provisioner_cluster_ntp_server**: as the hub cluster is disconnected, it needs to point to a reachable NTP server. You can point it to your provisioner IP host, because an NTP server will be installed there
- **provisioner_cluster_openshift_image**: point to the image where to deploy OpenShift from. It can be pointing to upstream or downstream registries, if your pull secrets allow it
- **provisioner_cluster_libvirt_pool**: path where to store the libvirt images that are going to be created. Point to a volume or partition with plenty of space
- **provisioner_hosts**: dictionary with the following entries: bootstrap, mirror, master-0, master-1, master-2. For each entry there needs to be the following information: mac, hostname, ip.
- **disconnected_operators**: list of operators to mirror to be ready for deployment. Sample inventory shows the needed ones for ZTP deployments.
- **children/provisioner/hosts**: add the IP for the hosts where you are going to run the playbook
