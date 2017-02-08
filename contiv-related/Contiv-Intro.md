# Contiv

Contiv is an Open Source policy-based container for Networking. Contiv makes it easier for end users to deploy micro-services in their environments by providing a higher level of networking abstraction for microservices. Contiv secures your application using a rich policy framework and built-in service discovery and service routing for scale-out services.

With the advent of containers and microservices architecture, there's a need for automated or programmable network infrastructure specifically catering to dynamic workloads which can be formed using containers. With container and microservices technologies, speed, scale, and automation become critical for success. Even network provisioning should
be made through automation to decrease the time to bring-up.

Also with bare metal hosts, VMs, and containers, there are different layers of virtualization abstraction. Packet encapsulation becomes tricky. With public cloud technologies, tenant-level isolation is necessary for our container workloads. Contiv solves both of these problems.

### Architecture Diagram of Contiv

![](https://github.com/gaurav-dalvi/scripts/blob/master/contiv-related/Contiv-HighLevel-Architecture.png?raw=true)

Contiv provides IP address per container and eliminates the need for host-based port NAT. It works with different kinds of networks like pure Layer 3 networks, overlay networks, and Layer 2 networks, and it provides the same virtual network view to containers regardless of the underlying technology. Contiv works with all major schedulers like Kubernetes, Docker Swarm, Mesos, Nomad. These schedulers provide compute resources to your containers and Contiv provides networking to them. Contiv supports both CNM (Docker's networking Architecture) and CNI (CoreOS, Kubernetes's networking architecture). Contiv has L2, L3 (BGP), Overlay (VXLAN) and ACI modes. It has built in east-west service load balancing. Contiv also provides traffic isolation through control and data traffic.

Contiv consists of two components--Netmaster (Contiv Master) and Netplugin (Contiv Host Agent).

### Netmaster and Netplugin

![](https://github.com/gaurav-dalvi/scripts/blob/master/contiv-related/Contiv-Network-Components.png?raw=true)

#### Netmaster:

This one binary performs so many tasks for Contiv. Its REST API server can handle multiple requests simultaneously. It learns routes and distributes to Netplugin nodes. It acts as resource manager which does resource allocation of IP addresses, VLAN and VXLAN IDs for networks. It uses distributed state store like etcd or consul to save all the desire runtime of for contiv objects. Because of which contiv becomes completely stateless, scalable and restart-able. Netmaster has a built-in heartbeat mechanism, through which it can talk to peer netmasters. This avoids risk of single point failure. Netmaster can work with an external integration manager (Policy engine) like ACI.

#### Netplugin:

Each Host agent (Netplugin) implements a [CNI](https://github.com/containernetworking/cni/blob/master/SPEC.md) or [CNM](https://github.com/docker/libnetwork/blob/master/docs/design.md) networking model adopted by popular Container orchestration engines like Kubernates, Docker Swarm, Mesos, Nomad, etc. The host agent communicates with netmaster over a REST API. Contiv also uses json-rpc to distribute endpoints from netplugin to netmaster. 

Netplugin handles Up/Down events from contiv networks defined in Groups. Netplugin handles co-ordination with other entities such as fetching policies, creating container interface, requesting IP allocation, and program host forwarding. Netplugin uses Contiv's custom open-flow based pipeline on a Linux host. It communicates with OpenVswtich (OVS) through the ovs driver. Contiv currently uses OVS for their data path. Because of the plugin architecture of Contiv, it is very easy to plug in any data path (e.g.: VPP, BPF etc). Netplugin also listens and performs **WHAT DOES IT PERFORM?**

### Contiv Custom Open-Flow based Pipeline:

![](https://github.com/gaurav-dalvi/scripts/blob/master/contiv-related/Packet-Pipeline.png?raw=true)

### Contiv Modes:

Contiv can provide you native connectivity (Traditional L2 and L3 network) as well as overlay connectivity (Public Cloud Case, we support AWS). 
In traditional L2 connectivity each packet coming out of Container is marked with certain VLAN so that container workloads can fit in traditional L2 network without any additional settings. 
For L3 connectivity, Contiv uses BGP to distribute routes over network.
![](https://github.com/gaurav-dalvi/scripts/blob/master/contiv-related/Native-Overlay.png?raw=true)

### Contiv + ACI: 

This is special case of running Contiv. 

The integration with ACI, it addresses basic use cases such as Infrastructure automation, Application Aware Infrastructure, scale out models, and Dynamic Applications which are key pillars of modern-day Microservices architectures. Contiv working with ACI, demonstrates how this integration can be achieved in a docker containerized environment to create objects and associations that enable containers to communicate according to policy intent.

Contiv + ACI integration is done using aci-gw docker container. It uses the Python SDK of APIC and allows for communication between contiv and APIC.

This is a typical workflow for ACI and Contiv integration:

![](https://github.com/gaurav-dalvi/scripts/blob/master/contiv-related/aci-integration.png?raw=true)

Step 1 : Configure tenant and dependent resources in APIC.

Step 2 and 4: Contiv Netmaster fetches this information when Contiv is running in ACI mode. 

Step 3: A DevOps person specifies policies for their application workloads to be used by developers. This is Application intent. 

Step 5: Developer launches apps managed by orchestration engines like Docker Swarm or Kubernates.

Step 6: Contiv Netplugin makes sure that policy is implemented correctly. It delegates all policy related context to APIC,
so, that packet forwarding can be taken care of at ACI level.

