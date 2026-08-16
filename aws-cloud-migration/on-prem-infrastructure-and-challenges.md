WileyPLUS is an online teaching & learning platform that provides:
* **Institutions:** Integration with major Learning Management Systems (LMS) like Blackboard, Canvas, Moodle, and Brightspace, enabling single sign-on (SSO).
* **Instructors:** The ability to assign Wiley-provided resources directly within their LMS courses.
* **Students:** Access to Wiley content such as e-textbooks, quizzes, and assignments, with instant grading and feedback (John Wiley & Sons, 2025c, 2025a).

This report was developed using insights gained through meetings with Wiley to outline the WileyPLUS on-premises architecture, including its components, functions, operational processes, challenges, and service level agreement targets.

Section 2 analyzes the WileyPLUS on-premises IT infrastructure.

---

## 1. WileyPLUS On-Premises IT Infrastructure

WileyPLUS operates on a multi-zone architecture, with its primary data center located in Hoboken, New Jersey, USA. The platform combines core components such as Akamai Content Delivery Network (CDN), Application Servers, Enterprise Server, and Content Servers, with LMS integrations. The WileyPLUS architecture is illustrated in Figure 1, while the main components and their functions are detailed in Table 1.

![Figure 1. WileyPLUS On-Premises Multi-Zone IT Architecture](images/figure-1.png)

**Table 1. WileyPLUS On-Premises Infrastructure Components and Their Functions**

| Zone | Component | Function |
| :--- | :--- | :--- |
| **Internet Layer** | WileyPLUS Direct Login, External LMS (Blackboard, Canvas, Moodle, Brightspace) Platforms | Serves as entry points for users (Students and Instructors). LMSs initiate Learning Tools Interoperability (LTI) v1.1/v1.3 launch requests to connect with WileyPLUS (John Wiley & Sons, 2025a). |
| | Akamai CDN | Provides global caching and acceleration for content via an edge network. Handles SSL termination and offloads traffic from origin servers (Akamai Technologies, 2025). |
| **Perimeter Security Zone** | Internet Line Firewall Cluster (FW1) | Filters inbound internet traffic and enforces network policies (e.g., port/IP rules) before it enters internal zones. |
| | Internet Line Routing Cluster (RT1) | Directs incoming traffic to the appropriate downstream services in the Demilitarized Zone (DMZ). |
| **Isolated Demilitarized Zone (DMZ)** | Brocade Load Balancing Cluster | Distributes traffic across multiple application server clusters. |
| | WileyPLUS Application Server Clusters (JBoss 4.2.3) | Hosts the WileyPLUS web application and orchestrates user interactions by routing requests to the appropriate backend components. The application servers deliver dynamic pages to end users, while relying on the Enterprise Server for authentication, the Content Servers for static assets, and Tibco middleware for LMS integrations. |
| | Enterprise Server - Tomcat (ENT) | Handles user authentication and SSO. |
| | WX Middleware - Tibco Services | Orchestrates LMS integrations (e.g., roster and grade synchronization) and translates data formats for non-Blackboard LMSs. |
| | WileyPLUS Large Content Server Cluster (Apache 2.0) | Serves static course media files (e.g., textbooks, chapter PDFs, images, CSS) fetched by Akamai when not cached at the edge. |
| **Backbone Security Zone** | Backbone Firewall Cluster (FW2) | Secures traffic between the DMZ and database layer, enforcing internal network segmentation. |
| | Backbone Routing Cluster (RT2) | Manages routing between application and database zones. |
| **Isolated Database Zone** | WileyPLUS Databases (Oracle and MySQL) | Store user, course, enrolment, grade, quiz, and content metadata. Multiple schemas (UNI, SPAWP, HEBCS, etc.) reside on Oracle 11g, while MySQL supports enterprise operational data. |

---

## Challenges of the On-Premises IT Infrastructure

Even though the on-premises model historically supported thousands of concurrent sessions, it suffers from several major challenges that hinder WileyPLUS’s availability goals and digital transformation strategy. These challenges can be grouped into six interrelated themes as shown below:

1. **Global Content Delivery Latency:** Although Akamai CDN accelerates the delivery of static assets, its effectiveness is limited to cached content. Cache misses and dynamic content requests are processed by the legacy monolithic application servers, which cannot scale elastically during peak demand (e.g., examinations). The manually scaled, tightly coupled database schemas further constrain dynamic content delivery, while multiple firewall and routing layers compound network latency.
2. **Network and Security Architecture Constraints:** The perimeter and backbone security layers rely on hardware-based firewall (FW1/FW2) and routing clusters (RT1/RT2). Although this architecture provides defense-in-depth, it also introduces network complexity and latency. Scaling these clusters requires physical upgrades and manual reconfiguration, limiting responsiveness during traffic surges. Each additional firewall or routing layer adds potential failure points, making the network harder to troubleshoot and less resilient to outages or misconfigurations. Furthermore, the on-premises model lacks adaptive mechanisms to counter emerging attack patterns and application-layer threats like SQL injection.
3. **Application Layer Limitations:** The WileyPLUS application runs on legacy JBoss 4.2.3, deployed as a tightly coupled monolith. Scaling is achieved by adding physical front-end servers. This approach is costly and operationally cumbersome, as the entire application must be scaled regardless of which features experience demand. Deployments and updates are slow and risky because changes affect the entire codebase, increasing downtime risks and limiting the platform’s ability to evolve rapidly. The Brocade Load Balancing Cluster distributes traffic across the monolithic application servers but provides limited elasticity and requires manual scaling and maintenance.
4. **Authentication and SSO Bottlenecks:** All authentication and single sign-on flows are centralized through the ENT, creating a single point of failure. If ENT experiences an outage, all user logins and LMS launches are disrupted. ENT’s scalability is largely manual, requiring additional hardware to support peak loads. Consequently, maintenance windows or unplanned downtime at the authentication layer cause system-wide impact.
5. **LMS Integration Complexity:** WileyPLUS supports multiple LMS platforms (Blackboard, Canvas, Moodle, Brightspace) through fragmented integration mechanisms. Blackboard uses custom APIs (building block and JSON) that interface directly with the application servers, while other LMSs integrate through WX Tibco middleware. Maintaining two separate integration paths increases operational complexity and risk, as multiple components must be monitored, updated, and synchronized with external LMS changes. Tibco also introduces a dependency; if Tibco fails, all non-Blackboard LMS integrations are disrupted. Its legacy, proprietary nature adds latency (TIBCO Technologies, 2025), requires specialist expertise, and complicates troubleshooting during incidents, making the integration layer a fragile operational bottleneck.
6. **Data Layer Limitations:** The data layer is composed of vertically scaled Oracle 11g and MySQL databases (Oracle Corporation, 2025b, 2025a). This heterogeneous setup requires separate replication, backup, and maintenance processes for each system, all of which are manual, increasing operational complexity. The legacy database schemas, combined with tightly coupled data structures degrade query performance. During high-traffic periods such as examinations, database workloads intensify, resulting in query slowdowns and occasional outages. The static scaling model demands provisioning for peak capacity in advance, leading to inefficient resource utilization and limited elasticity. In addition, licensing, patching, and maintenance of these on-premises databases impose significant operational costs and administrative overhead.

---

## 2. Impact of On-Premises Challenges and Strategic Need for Cloud Migration

The above architectural challenges directly affect students, instructors, and Wiley’s business operations. For students and instructors, slow content delivery, authentication bottlenecks, and database latency degrade the learning and teaching experience due to delayed page loads, login failures, and inconsistent roster or grade synchronization. Instructors face additional administrative burdens, often resorting to manual interventions. These disruptions are particularly damaging during high-stakes examination periods, undermining trust in the platform’s reliability. 

For Wiley, the reliance on manual scaling, fragmented integrations, and aging infrastructure leads to high operational and infrastructure costs, staff fatigue, extended maintenance windows, and limited agility in responding to demand fluctuations or security threats. Prolonged outages can impact both revenue and Wiley’s reputation with institutional partners. Collectively, these issues reduce WileyPLUS’s operational agility, compromise user experience, and weaken its competitive position in the global digital learning market.

The WileyPLUS on-premises architecture has evolved organically over the years to support thousands of users worldwide. However, to address the on-premises architectural challenges, achieve WileyPLUS’s target of “100% uptime”, and align infrastructure with digital modernization strategy, a phased cloud migration is proposed as a strategic solution. Part 02 - Artefact examines two phases of migration to Amazon Web Services (AWS): 
* (i) Lift-and-Shift
* (ii) Re-engineering

Both phases are analyzed in detail to demonstrate how they address the six key challenge themes outlined in Section 3 of this report. 

---

## 3. Summary

In summary, the WileyPLUS on-premises architecture has enabled global digital learning delivery at scale but faces critical challenges in scalability, integration flexibility, and operational efficiency. Authentication and content delivery rely on monolithic applications and legacy network infrastructure, while LMS integrations are fragmented through custom Blackboard APIs and Tibco middleware. Static scaling of databases and network appliances limits responsiveness to peak demand, and operations remain highly manual. These issues collectively degrade user experience during critical academic periods, increase operational risk and cost, and obscures business goals. This necessitates a strategic migration to AWS, where cloud-native architectures can address these systemic constraints.