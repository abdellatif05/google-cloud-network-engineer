# IAM permissions
  - Compute admin
  - Compute network admin
    - All network resources expect firewall rules and ssl certificates
  - Compute network user
    - Access to shared vpc resources
  - Compute Security admin
    - Firewall rules and ssl certificates
  - Compute Shared VPC admin (Compute.xpnAdmin)
    - Enable host projects and attach service projects
    - At the organization level, this role can only be granted by an organization admin.

# Network Modes

### Custom mode
  - You define the network and subnets
  - No explicit firewall rules are created

### Auto mode adresses
  - 10.128.0.0/9
  - You can add subnets manually to automode vpc, by using ip range outside of 10.128.0.0/9
  - You can switch from automode to custom mode

# Defaut network
  - Mode: Automode
  - Explicit ingress firewalls:
    - Default-allow-internall (10.128.0.0/9)
    - Default-allow-ssh (0.0.0.0/0)
    - Default-allow-rdp (0.0.0.0/0)
    - Default-allow-icmp (0.0.0.0/0)

# No-default and default network
  - Implied firewalls
  - Allow all egress
  - Deny all ingress

# Reserved ranges:
  - Range mask length must not be greater than 29
  - 4 addresses are reserved for every primary CIDR range (The first two, The last two)
  - Example: 10.0.0.0/29 (8 hosts)
    - Network: 10.0.0.0
    - Gateway: 10.0.0.1
    - Any future purpose: 10.0.0.6
    - Broadcast: 10.0.0.7
    - From 10.0.0.2 to 10.0.0.5: available for VMs
  - Example: 10.1.2.0/24 (256 hosts)
    - Network: 10.1.2.0
    - Gateway: 10.1.2.1
    - Any future purpose: 10.1.2.254
    - Broadcast: 10.1.2.255
    - From 10.1.2.2 to 10.1.2.253: available for VMs

# Maximum number of network interfaces (NICs) in VM
  - You can only set network interfaces before creating the VM
  - Maximum number of NICs depends on the machine type
  - Number of vCPU (Number of vNICs)
  - 2 or less	(2)
  - 4	(4)
  - 6	(6)
  - 8 or more	(8)
  - Compute Engine bare metal instances	(1)

# Network tiers
  - Standard
    - Use standard internet gateway
    - 200 GB/mo free in every region
  - Premium
    - Recommended for latency sensitive applications
    - More expensive
    - Use google backbone network
  - You can change the tier of a network
    - In project level
    - In resource level (LB, NAT, ip address)
# Private Google Access
  - enabled in subnet
  - connect to GCP API and services without external ip
  - use case: GCS, Spanner, Bigtable, Bigquery, Firestore
  - if you do not have default internet gateway, you can use VIPs:
    - private.googleapis.com    199.36.153.8/30        if you do not use vpc service controls
    - restricted.googleapis.com 199.36.153.4/30        if you use services supported by vpc service controls

# Private Service Access
  - Connect to GCP services without external ip
  - It uses VPC peering
  - Using internal ip of Services like Cloud SQL
  - Use case: Cloud SQL, Alloydb, APIGEE, Memcache, Filestore
  - You must export the VPC network's custom routes so that the service provider's network can import them and correctly route traffic to your on-premises network.

# Cloud NAT
  - Regional resource
  - Support TCP, UDP, ICMP
  - NAT IP address draining works for manual NAT IP address assignment only.
  - A Cloud NAT gateway can apply to multiple network interfaces on the same VM
  - Endpoint-Independent Mapping
    - the destination port, the protocol?the same NAT source IP address and source port tuple can be simultaneously used for many different connections.
    - Example:
      - Source 10.0.0.2:10001 Destination 203.0.113.1:80 TCP Source NAT IP 192.0.2.10:30009
      - Source 10.0.0.2:10002 Destination 203.0.113.2:80 TCP Source NAT IP 192.0.2.10:30009
  - Public NAT:
    - convert private ip to public ip
  - Private NAT:
    - convert private ip to private ip
    - for networks that are connected to a Network Connectivity Center hub, which includes private-to-private NAT for traffic between VPC spokes and between VPC spokes and hybrid spokes.
  - Hybrid NAT:
    - convert private ip to private ip
    - between VPC networks and on-premises or other cloud provider networks that are connected to Google Cloud over Cloud Interconnect or Cloud VPN.

# VPC Serverless Access
  - Used from:
    - Cloud Run
    - Cloud Functions
    - App Engine (standard)
  - To connect your serverless applications to your VPC network
  - You must choose a custom unused range /28 in the VPC network
  - Scaling between 2 and 10 instances (instances not showed in GCP Console)
  - this tags are added to the instances (you can use them to apply firewall rules or routes):
    - Universal network tag (vpc-connector): Applies to all existing connectors and any connectors made in the future.
    - Unique network tag (vpc-connector-REGION-CONNECTOR_NAME): Applies to the connector CONNECTOR_NAME in the region REGION.


# VPC peering
  - Peering between 2 GCP VPCs
  - Ranges should not overlap

# Shared VPC
  - Centralized network management
  - Services projects share the same network (shared VPC host)
  - Use case:
    - Central teams manage network configuration
    - Granular access control
  - compute.xpnAdmin to create host projects and attach service projects
  - compute.networkUser to use the network in service projects
  - A project cannot be both a host and a service project simultaneously.
  - Shared VPC is only available for projects within the same organization.

# Routes (defines egress traffic)
  - Consist of:
    - Destination CIDR    (example: 0.0.0.0/0)
    - Next hop type       (example: default internet gateway)
  - Types:
    - System generated routes
      - Static (example: default internet gateway)
      - Subnet (example: next hope subnet)
    - VPC Peering routes
    - Custom routes
      - static
      - dynamic (cloud router)
    - Policy based routes
      - Let you select a next hop based on more than a packet's destination IP address.
  - Order of routes:
    - System generated routes
    - Custom routes
    - VPC Peering routes
  - Instance tags:
    - The route applies to all instances with any of these tags, or to all instances in the network if no tags are specified


# Cloud Router
  - Used to create dynamic routes
  - Using BGP protocol
  - Global or Regional routes configured in VPC Network
  - A Cloud Router supports multiple interfaces. You don't need to create a separate Cloud Router for each Cloud VPN tunnel or VLAN attachment

### BGP
  - Border Gateway Protocol
  - Used to exchange routes between routers
  - Routers exchange routes between each other
  - It uses ASN (Autonomous System Number)
    - 64512-65535: Private ASNs
  - MED (Multi-Exit Discriminator) priority
    - It is used to select the route from multiple routes with the same destination IP address.

### BGP troubleshooting
  - https://cloud.google.com/network-connectivity/docs/router/support/troubleshoot-bgp-sessions
  - https://cloud.google.com/network-connectivity/docs/router/support/troubleshoot-bgp-peering
  - https://cloud.google.com/network-connectivity/docs/router/support/troubleshoot-bgp-routes
  - https://cloud.google.com/network-connectivity/docs/router/support/troubleshoot-log-messages

# VPN
  - Cloud VPN uses IPsec protocol to create a secure tunnel between your network and GCP
  - Ipsec protocol is mandatory to establish a VPN connection with GCP Cloud VPN
  - Cloud VPN supports between 1gbps and 3gbps of bandwidth
  - BGP or Dynamic routing
    - Classic : is optional for Cloud VPN Classic (will be deprecated in august 2025)
    - HA : is mandatory for Cloud VPN HA
  - SLA:
    - Classic : 99.9%
    - HA : 99.99% for most topologies
  - IPv6:
    - Classic : not supported
    - HA : supported
  - Active/Active (recommended for multiple HA VPN)
    - same priority in routes
  - Active/Passive (recommended for single HA VPN gateway)
    - different priority in routes
### HA VPN topology
###### 99.99% SLA
  In GCP network:
  - VPN gateway -> 2 interfaces -> 2 tunnels -> 1 router

###### 99.9% SLA
  - VPN gateway -> 1 interface -> 1 tunnel -> 1 router

###### Configure HA VPN for more bandwidth
  - To increase the bandwidth of your HA VPN gateways, add more HA VPN tunnels.

### VPN troubleshooting
  - VPN logs are enabled by default
  - Connectivity test:
    - check with ping
    - before checking, you need to ensure that a ICMP connection is allowed in firewall rules in both networks (GCP and on-prem)
  - Common issues:
    - Connectivity works for some VMs, but not for others (firewall rules well configured)
      - you might have traffic selectors that exclude certain sources or destinations.
    - Network latency between VMs in different regions
      - monitor the performance of the entire Google Cloud network using Performance dashboard in Network Intelligence Center

  - https://cloud.google.com/network-connectivity/docs/vpn/support/troubleshooting

# Interconnect
  - Connect your on-premises network to GCP using physical connections
  - your onprem network devices must support the following technical requirements:
    - 10-Gbps circuits, single mode fiber, 10GBASE-LR (1310 nm), or 100-Gbps circuits, single mode fiber, 100GBASE-LR4
    - IPv4 link local addressing
    - LACP, even if you're using a single circuit
    - EBGP-4 with multi-hop
    - 802.1Q VLANs

### Limits
  - Maximum number of physical circuits per Interconnect connection:
    - 8 * 10 Gbps
    - 2 * 100 Gbps
  - Maximum bandwidth per VLAN attachment between 50 Mbps and 50 Gbps
  - MTU:
    - 1400 bytes
    - 1460 bytes
    - 1500 bytes
    - 8896 bytes

### Interconnect creation steps
### Interconnect topology
###### 99.99% SLA
  - At least 4 dedicated interconnect connections
    - 2 dedicated interconnects in one metropolitan area and 2 dedicated interconnects in another metropolitan area
    - Connections that are in the same metro must be placed in different edge availability domains (metro availability zones)
  - At least 2 Cloud Routers placed in at least two distinct Google Cloud regions
    - Each Cloud Router must be attached to a pair of Dedicated Interconnect connections in a metro (two VLAN attachments for each Cloud Router)
  - The dynamic routing mode for the Virtual Private Cloud (VPC) network must be global.
  - Example:
    - metro1:
      - Cloud Router1 (region1)
        - VLAN attachment 1 (metro1-zone-a)
        - VLAN attachment 2 (metro1-zone-b)
    - metro2:
      - Cloud Router2 (region2)
        - VLAN attachment 3 (metro2-zone-a)
        - VLAN attachment 4 (metro2-zone-b)
  - Create a 99.99% topology
    - Change VPC network's dynamic routing mode to GLOBAL
    - Order Dedicated Interconnect connections
    - Google generates LOA-CFAs for your connections and emails them to you
    - Test connections
      - Google sends you an IP address configuration that you must apply to your on-premises router.
    - Create two Cloud Routers, one for each region
    - Create 4 VLAN attachments
      - After your connections are ready to use (in the ACTIVE state)
      - connect connections to Cloud Routers
    - Configure on-premises routers



###### 99.9% SLA
  - At least 2 dedicated interconnect connections must be in the same metro
    - but in different edge availability domains (metro availability zones)
  - At least 1 Cloud Router, Each Cloud Interconnect connection must be attached to the Cloud Router
  - Example:
    - metro1:
      - Cloud Router1 (region1)
        - VLAN attachment 1 (metro1-zone-a)
        - VLAN attachment 2 (metro1-zone-b)


### Interconnect troubleshooting
  - https://cloud.google.com/network-connectivity/docs/interconnect/support/troubleshooting

# Direct Peering
  - Direct peering is used when an organization wants to accelerate and optimize connectivity to Google services like Youtube, Google Maps, Google Workspace or any Google publicly exposed APIs.
  - Direct peering enhances connectivity to reach Google services (Workspace, Youtube etc) by reducing latency and hops to those services.
  - An enterprise may choose Direct peering when consumption is heavy for those services.

# Carrier Peering
  - Same as Direct peering, but the connection is made through a carrier provider.
  - if the enterprise does not have the necessary internal resources, people and skills on BGP.


# DNS
  - Zones
    - Private
      - Options:
        - Default (private zone)
        - DNS peering
        - Forward queries to another DNS server (dns forwarding)
        - Service directory namespace
    - Public
  - Records
    - A (IPv4)
    - AAAA (IPv6)
    - CNAME (canonical name)
    - MX (mail exchange)
    - NS (name server)
    - SOA (start of authority)
  - Delegated subzones
    - To enable delegation in cloud DNS, you can add a NS records for subdomains in the parent zone.
  - DNSSEC
    - Verify that the origin of DNS data and validate that it has not been modified in transit.
  - Import dns formats: BIND, Yaml

# Load balancers
  - Application (HTTP, HTTPS)
    - External
      - Global
        - EXTERNAL_MANAGED (Advanced Traffic Management)
          - CDN supported
          - Cloud Armor supported
          - Supported Backends:
            - instance group
            - Zonal NEG (GKE, GCE)
            - Internet NEG
            - PSC NEG
            - Serverless NEG (Cloud Run, Cloud Functions, App Engine)
            - Hybrid NEG
            - Buckend Bucket
        - Classic (Basic Traffic Management)
          - CDN supported
          - Cloud Armor supported
          - Supported Backends:
            - instance group
            - Zonal NEG (GKE, GCE)
            - Internet NEG
            - Serverless NEG (Cloud Run, Cloud Functions, App Engine)
            - Hybrid NEG
            - Buckend Bucket
      - Regional
        - Cloud Armor supported
        - Supported Backends:
          - instance group
          - Zonal NEG (GKE, GCE)
          - Internet NEG
          - PSC NEG
          - Serverless NEG (Cloud Run)
          - Hybrid NEG
    - Internal
      - Cross-region
        - Supported Backends:
          - instance group
          - Zonal NEG (GKE, GCE)
          - PSC NEG
          - Serverless NEG (Cloud Run)
          - Hybrid NEG
      - Regional
        - Supported Backends:
          - instance group
          - Zonal NEG (GKE, GCE)
          - Internet NEG
          - PSC NEG
          - Serverless NEG (Cloud Run)
          - Hybrid NEG
  - Network (TCP, UDP, TLS)
    - when you need TLS offloading at scale, support for UDP, and exposing IP addresses to your applications
    - Proxy (TCP, SSL)
      - External
        - Global (Proxy LB external global)
          - EXTERNAL_MANAGED (Advanced Traffic Management)
            - Protocol: TCP, SSL
            - Supported Backends:
              - instance group
              - Zonal NEG (GKE, GCE)
              - Hybrid NEG
          - Classic (Basic Traffic Management)
            - Protocol: TCP, SSL
            - Supported Backends:
              - instance group
              - Zonal NEG (GKE, GCE)
              - Hybrid NEG
        - Regional (Proxy LB external regional)
          - Protocol: TCP
          - Supported Backends:
            - instance group
            - Zonal NEG (GKE, GCE)
            - PSC NEG
            - Internet NEG
            - Hybrid NEG
      - Internal
        - Cross-region (Proxy LB internal cross-region)
          - Protocol: TCP, SSL
          - Supported Backends:
            - instance group
            - Zonal NEG (GKE, GCE)
            - PSC NEG
            - Hybrid NEG
        - Regional (Proxy LB internal regional)
          - Protocol: TCP
          - Supported Backends:
            - instance group
            - Internet NEG
            - Zonal NEG (GKE, GCE)
            - PSC NEG
            - Hybrid NEG
    - Passthrough (TCP, UDP, other protocols) (regional)
      - External
        - Protocol: TCP, UDP, other protocols
        - Supported Backends:
          - instance group
          - Zonal NEG (GKE, GCE)
      - Internal
        - Protocol: TCP, UDP, other protocols
        - Supported Backends:
          - instance group
          - Port mapping NEG

# CDN
  - Supported in Application Load Balancer (Global)
  - Default cache time: 3600 seconds
  - Signed requests and Unsigned requests are cached separately

# Cloud Armor
  - Supported in Application Load Balancer (Global and Regional)

# GKE Networking
  - Cluster Network components:
    - GKE user Network (Data plane) (node pool, pods, services)
    - Google Managed Network (Control plane)
  - Cluster Types:
    - Routes based (uses Google cloud routes) (public only)
    - VPC-native (uses alias IPs) (recommended, default) (public and private)
      - Automatically generated subnet routes
      - Ranges:
        - Nodes: Primary Subnet range (10.0.0.0/26)
        - Pods: Secondary Subnet range (10.0.64.0/18)
        - Services: Secondary Subnet range 10.0.128.0/20
      - In shared VPC, gke cluster cannot create the secondary ranges, network admin must create them in the shared VPC in the host project.
  - VPC native public cluster:
    - Nodes have external IP addresses
    - Control plane has only public endpoint
    - Can be accessed from the internet
  - VPC native private cluster:
    - Nodes do not have external IP addresses
    - Control plane has public and private endpoint (you can disable public endpoint)
    - Cannot be accessed from the internet
    - You need specify unique /28 range for vpc peering between Control plane network and data plane network
  - Access services:
    - Container native load balancer Service:
      - Layer 4 (TCP, UDP)
      - GCP configure network (internal or external) passthrough load balancer
    - Ingress:
      - Layer 7 (HTTP, HTTPS)
      - Application (internal or external) load balancer
      - You can expose multiple services through a single ingress
    - Gateway API:
      - Advanced version of ingress
      - Layer 7 (HTTP, HTTPS) and Layer 4 (TCP, UDP)
      - Advanced routing capabilities
      - Multi-cluster routing supported
      - Recommended by GCP
  - Network Security:
    - Authorized networks
    - IAM
    - RBAC (inside the cluster)
    - Workload identity
    - Firewall rules
    - Network policy (between pods)

  - Maximum number of pods per node:
    - Standard: 256 (512 adresses /23)
    - Autopilot: 256 (512 adresses /23)

### GKE limits
  - Default number of pods per node:
    - Standard: 110 (/24)
    - Autopilot: 110 (/24)

  - Pod address range in Route based cluster

  - Assigning secondary CIDR ranges for pods and services in a GKE VPC-native public cluster can be done:
    - at the GKE Cluster networking level
    - at the Network subnet level
  - Primary range, secondary range and authorized networks can access to control plane

# Logging and Monitorin
### Firewall logs
  - enabled in firewall rules level (Legacy network not supported)
  - TCP and UDP
  - log elements:
    - src_ip
    - dest_ip
    - src_port
    - dest_port
    - protocol
  - permissions:
    - compute.securityAdmin to manage firewall rules
    - logging.viewer to view logs

### VPC flow logs
  - enabled in subnet level
  - TCP, UDP, ICMP, ESP, GRE
  - log elements:
    - src_ip
    - dest_ip
    - src_port
    - dest_port
    - protocol


# Network intelligence center
  - Firewall insights
  - Network topology (topology map, trafic between zones)
  - Connectivity test
  - Performance dashboard (latency)
  - Network analyzer (detect misconfigurations and give recommendations)


# CIDR address ranges
  - /32 1
  - /31 2
  - /29 8
  - /28 16
  - /27 32
  - /26 64
  - /25 128
  - /24 256
  - /23 512
  - /22 1024
  - /21 2048
  - /20 4096
  - /19 8192
  - /18 16384
  - /17 32768
  - /16 65536
  - /15 131072
  - /14 262144
  - /13 524288
  - /12 1048576
  - /11 2097152
  - /10 4194304
  - /9 8388608
  - /8 16777216
