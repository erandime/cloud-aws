## 1. Lift-and-Shift AWS Architecture for WileyPLUS

The lift-and-shift migration strategy recreates WileyPLUS’s on-premises architecture within the Amazon Web Services (AWS) cloud through a like-for-like mapping of on-premises components to their equivalent AWS services (Amazon Web Services, 2025t). The original security layers, network zones, and core components are logically re-implemented inside an Amazon Virtual Private Cloud (VPC) in the `us-east-1` region, spanning two Availability Zones (AZs) as shown in Figure 1. In a production environment, network segmentation is achieved through subnets, and the design can be extended across additional AZs to further enhance system resilience. The corresponding AWS services and their mapped functions are detailed in Table 1.

![Figure 1. Lift-and-Shift AWS Architecture for WileyPLUS](figure-1-lift-and-shift.png)

**Table 1. Lift-and-Shift AWS Services and Their Functions**

| Current Component | AWS Replacement (Lift-and-Shift) | Functions |
| :--- | :--- | :--- |
| Internet Line Firewall Cluster (FW1) | AWS Network Firewall | Provides managed, stateful perimeter protection and deep packet inspection at the VPC boundary, via firewall endpoints deployed in dedicated subnets across multiple AZs (Amazon Web Services, 2025l). |
| Internet & Backbone Routing Clusters (RT1 / RT2) | VPC Routing, Internet Gateway (IGW), Network Address Translation (NAT) Gateway | Provides routing between public and private subnets through centrally managed route tables. The IGW enables inbound and outbound internet connectivity, while NAT Gateways allow controlled outbound access for private subnets (Amazon Web Services, 2025i). |
| Backbone Firewall Cluster (FW2) | Security Groups (SGs), Network Access Control Lists (NACLs) | SGs and NACLs enforce fine-grained inbound and outbound rules at the instance and subnet levels (Amazon Web Services, 2025i). |
| Brocade Load Balancing Cluster | Application Load Balancer (ALB) | The ALB is deployed across two AZs, with Elastic Network Interfaces (ENIs) placed in public subnets. It is accessible from the internet through Akamai and the IGW. It performs Layer 7 load balancing with health checks and distributes traffic to Elastic Compute Cloud (EC2)-hosted instances in private subnets across both AZs (Amazon Web Services, 2025j). |
| WileyPLUS Application Server Clusters (JBoss 4.2.3) | Amazon EC2 instances in Auto Scaling Group (ASG) | Hosts the monolithic application. Auto scaling adjusts capacity by adding or removing instances based on demand across multiple AZs (Amazon Web Services, 2025s). |
| Enterprise Server (ENT) - Tomcat | EC2-hosted ENT (Multi-AZ) | ENT rehosted on EC2 instances across multiple AZs. The legacy ENT continues to handle all login and Single Sign-On (SSO) operations. |
| WX Middleware – Tibco Services | EC2-hosted Tibco instances (Multi-AZ) | Middleware rehosted on EC2 instances across multiple AZs while retaining the existing LMS integration model. |
| WileyPLUS Large Content Server Cluster (Apache 2.0) | EC2-hosted content servers (Multi-AZ) | Rehosts existing content servers on EC2 instances deployed across multiple AZs without changing the underlying architecture. |
| WileyPLUS Databases (Oracle and MySQL) | Amazon Relational Database Service (RDS) for Oracle and MySQL (Multi-AZ) | Managed service with automated backups, patching, and synchronous standby replication across AZs for high availability (Amazon Web Services, 2025f). |

---

### 1.1. Challenges Addressed by Lift-and-Shift Migration and Associated Benefits

The lift-and-shift migration approach addresses several of the challenges identified in the on-premises WileyPLUS architecture, as outlined in Part 01 - Research Portfolio. Table 2 describes how each key challenge theme is addressed through this migration strategy and highlights the associated benefits.

**Table 2. On-Premises Challenge Themes Addressed through Lift-and-Shift Migration and Its Benefits**

| No | Challenge Theme | How Lift-and-Shift Addresses and Associated Benefits |
| :--- | :--- | :--- |
| 1 | Global Content Delivery Latency | The existing Akamai CDN configuration is retained to minimise migration complexity and reduce risk. DNS continues to route user requests through Akamai, which provides global caching and acceleration for WileyPLUS content (Akamai Technologies, 2025). Cache misses are served from EC2-hosted content servers within AWS, replicating the on-premises delivery pattern but benefiting from the improved availability and reliability of AWS’s managed infrastructure (Amazon Web Services, 2025a). |
| 2 | Network and Security Architecture Constraints | Hardware firewalls and routing clusters are replaced by a combination of AWS Network Firewall (NWF), Security Groups, NACLs, and VPC routing constructs. This eliminates the need for physical upgrades and manual reconfiguration. NWF multi-AZ design eliminates physical appliance limits and adapts to new attack patterns through managed rule updates, addressing the on-premises model’s static nature (Amazon Web Services, 2025e). Centrally managed, software-defined routing simplifies the network and enhances fault isolation, visibility, and traffic control (Amazon Web Services, 2025p). |
| 3 | Application Layer Limitations | The WileyPLUS monolithic application is rehosted on Amazon EC2 instances within an Auto Scaling Group (ASG) spanning multiple AZs. This enables automatic horizontal scaling during demand spikes and ensures continued availability if an AZ fails (Amazon Web Services, 2025g). An Application Load Balancer (ALB) replaces the Brocade load balancing cluster, providing availability through redundancy and balanced traffic distribution via health checks (Amazon Web Services, 2025j). Although the application remains monolithic, fault tolerance improves and over-provisioning is eliminated. |
| 4 | Authentication and SSO Bottlenecks | The Enterprise Server (ENT) is rehosted on EC2 instances deployed across multiple AZs, providing redundancy and fault tolerance at the infrastructure level (Amazon Web Services, 2025a). If one instance or AZ fails, the other continues to handle authentication requests, improving uptime. |
| 5 | LMS Integration Complexity | WX Tibco middleware is rehosted on EC2 instances across multi-AZs improving uptime and resilience (Amazon Web Services, 2025a). |
| 6 | Data Layer Limitations | Legacy Oracle and MySQL databases are migrated to Amazon RDS in a Multi-AZ configuration. RDS automates backups, patching, and maintains a synchronous standby replica in another AZ for automated failover. Automation and managed service model improves recovery and uptime and reduces operational overhead compared to manual on-premises procedures (Amazon Web Services, 2025f). |

---

### 1.2. Limitations of Lift-and-Shift Approach

The lift-and-shift migration delivers immediate infrastructure-level and operational improvements without requiring application redesign. Migration is faster and lower risk because application, middleware, and database tiers are rehosted with minimal code changes. However, several challenges of the legacy system remain. The key limitations across major architectural areas are summarised below.

1. **Content Delivery:** No significant performance improvement is achieved, as the monolithic application and vertically scaled tightly coupled legacy database schemas remain. Akamai becomes an external service with separate contractual and cost dependencies.
2. **Security:** Application-layer threats are not inherently addressed in this phase; web application vulnerabilities persist.
3. **Application Architecture:** The monolithic structure is retained, meaning scaling occurs at the cluster level rather than per service. Deployments remain slow and risky due to the absence of modular isolation. Scaling the entire monolith for specific feature demand may cause resource inefficiencies.
4. **Authentication and SSO:** The Enterprise Server (ENT) continues as a logical bottleneck, with scalability and maintenance requiring manual intervention.
5. **Integration Layer:** Integration fragmentation between Blackboard and other LMS remains unresolved. The Tibco middleware continues to be a chokepoint that adds latency and requires specialised technical support.
6. **Data Layer:** Amazon RDS remains provisioned capacity, requiring manual vertical scaling rather than automatic scaling on-demand (Amazon Web Services, 2025f). Therefore, resource inefficiencies during low-demand periods remain. Moreover, since the original tightly coupled database schemas remain, query performance limitations persist. Licensing and patching requirements for Oracle add to ongoing overhead.