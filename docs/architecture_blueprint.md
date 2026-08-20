# Cloud, Security & IoT Deployment Blueprint

## Scope

This blueprint deploys the fixed **scheduler and Banker's-Algorithm engine from Part 1** without changing its workload or algorithms. The deployed engine includes the fixed `jobs.py` list, FCFS, SJF, SRTF, Round Robin, Priority scheduling, Peterson’s Algorithm demonstration, Banker’s Algorithm, and paging/segmentation translators.

---

## 1. Distributed Architecture and Communication Plan

### Selected architecture: Client-Server

The platform will use a **Client-Server architecture**.

- Zone-A, Zone-B, and Zone-C controllers act as clients.
- The Smart City Operations dashboard and central API act as the server.
- Each zone controller sends alerts, job results, logs, and system-health data to the central dashboard.
- The scheduler and Banker's-Algorithm engine from Part 1 runs on protected cloud compute resources used by the central platform.

### Justification

| Criterion | Justification |
|---|---|
| Transparency | City operators access one dashboard instead of connecting separately to Zone-A, Zone-B, and Zone-C. |
| Scalability | More application instances, message consumers, and storage services can be added as the number of sensors grows. |
| Fault tolerance | Zone controllers buffer messages locally if the dashboard service is temporarily unavailable and retry delivery later. |
| Single point of failure | A central server can become a single point of failure, so the dashboard and scheduler services should run in multiple availability zones behind a load balancer. |

### Data flow decisions

| Data Flow | Communication Type | Protocol | Reason |
|---|---|---|---|
| Zone controller sends a real-time public-safety alert to the dashboard | Asynchronous | MQTT over TLS | MQTT is lightweight and supports publish/subscribe delivery. A zone controller can publish an alert without waiting for a dashboard user response. |
| Zone controller uploads a full day of sensor logs for archival | Asynchronous | HTTPS | Daily logs are large, retryable, and do not require an immediate response. HTTPS provides encrypted authenticated transport for archival uploads. |

---

## 2. VPC-Based Network Boundary

The platform will use **one VPC with separate private subnets**:

- Zone-A private subnet
- Zone-B private subnet
- Zone-C private subnet
- Dashboard and ingestion private subnet
- Management subnet for controlled administrative access

A single VPC provides logical isolation from external networks while allowing customizable subnet CIDR ranges, route tables, network access controls, and security groups. Separate subnets keep each zone’s compute and sensor-ingestion resources logically separated.

### Cross-zone isolation control

Zone-A must not be reachable directly from Zone-B.

This is enforced through:

- Security-group rules that do not allow inbound traffic from Zone-B security-group resources into Zone-A resources.
- Route tables that do not include direct Zone-A-to-Zone-B application routes.
- Allow rules permitting zone controllers to communicate only with the dashboard-ingestion service on approved ports such as MQTT over TLS and HTTPS.

The dashboard is not the enforcement boundary. The **security-group and routing rules** enforce the cross-zone isolation.

---

## 3. Network Security Objectives

| Security Objective | Control or Technology | How It Protects the Platform |
|---|---|---|
| Protect sensitive data | AES-256 encryption using KMS-managed keys | Sensor archives, scheduler outputs, and backups of the fixed `JOBS` list are encrypted at rest. |
| Authentication | Mutual TLS device certificates | A zone controller or gateway proves its identity before it can publish messages to the cloud platform. |
| Authorization | Least-privilege IAM policies | Zone operators can access only their authorized zone resources, while auditors receive read-only permissions. |
| Prevent cyber attacks | IDS/IPS, WAF, and rate limiting | These controls detect suspicious traffic, reduce denial-of-service attacks, and block common web attacks before requests reach the scheduler and Banker's-Algorithm engine from Part 1. |
| Secure communication | TLS 1.3 | MQTT and HTTPS messages are encrypted in transit to prevent interception and tampering. |
| Ensure availability | Multi-AZ deployment, health checks, autoscaling, and backups | The platform continues operating if one compute instance or availability zone fails. |

---

## 4. IAM Roles and Data Protection

### IAM role table

| Role | Permission Set |
|---|---|
| Zone Operator | Can view and operate resources only for the assigned zone, publish sensor alerts, and read local job status. Cannot access other-zone archives. |
| City Dashboard Admin | Can manage dashboard users, view all zone alerts, acknowledge incidents, and monitor system health. Cannot modify immutable audit logs. |
| Auditor | Read-only access to scheduler results, Banker’s Algorithm safety decisions, system logs, and audit trails. Cannot create, modify, or delete resources. |
| Engine Service Role | Can read the fixed `jobs.py` input, run the scheduler and Banker's-Algorithm engine from Part 1, and write signed execution results to approved storage. |

### Data protection map

| Data State | Protection Technique | Concrete Example |
|---|---|---|
| At rest | AES-256 encryption with KMS | Encrypt the fixed `JOBS` list, daily sensor logs, scheduler results, and Banker’s Algorithm reports stored in cloud storage. |
| In transit | TLS 1.3 / MQTT over TLS | Encrypt a public-safety alert sent from the Zone-B controller to the Smart City Operations dashboard. |
| In use | Confidential-computing enclave and memory access controls | Protect the Banker’s Algorithm safety check while its Available, Allocation, Max Need, and Need matrices are processed in memory. |

---

## 5. IoT Connectivity Plan

### Devices and communication technologies

| Sensor or Device | Technology | Reason |
|---|---|---|
| Traffic-camera event trigger | 5G | Traffic cameras may require high bandwidth and low latency for event-triggered media or metadata transmission. |
| Air-quality and environmental sensor | LoRaWAN | LoRaWAN supports long range and low-power communication for small, infrequent environmental readings. |
| Public-safety wearable panic button | NB-IoT | NB-IoT supports low power consumption and wide-area carrier coverage for compact emergency devices. |
| Building occupancy sensor | Zigbee | Zigbee is suitable for short-range, low-power mesh networks inside public buildings. |

### IoT architecture layers

| IoT Layer | Platform Component |
|---|---|
| Physical Environment | Roads, intersections, public buildings, public spaces, and environmental monitoring areas. |
| Perception/Device Layer | Traffic triggers, environmental sensors, wearable panic buttons, and occupancy sensors. |
| Gateway Layer | Zone-A, Zone-B, and Zone-C gateways that aggregate device data, validate devices, and buffer messages during outages. |
| Network Communication Layer | LoRaWAN, NB-IoT, 5G, Zigbee gateway uplinks, MQTT over TLS, and HTTPS. |
| Cloud Platform Layer | The fixed scheduler and Banker's-Algorithm engine from Part 1, hosted on protected cloud compute resources with encrypted storage. |
| Application Layer | Smart City Operations dashboard, incident-management interface, reporting tools, and audit views. |

---

## 6. Threats and Mitigations

| Threat | Impact on Platform | Mitigation |
|---|---|---|
| Compromised IoT device or spoofed telemetry | An attacker could submit false sensor readings or fabricated public-safety events. | Use secure boot, signed firmware, hardware-backed device certificates, mutual TLS, and certificate revocation. |
| Man-in-the-middle attack | An attacker could intercept or alter alerts, sensor readings, or daily uploads. | Require TLS 1.3, mutual TLS for devices, trusted certificate authorities, and reject unencrypted MQTT connections. |
| Distributed denial-of-service attack | Attackers could overload dashboard ingestion endpoints or message brokers. | Use WAF rules, rate limits, autoscaling, DDoS protection, message queues, and isolated ingestion services. |
| Cloud credential theft | An attacker could gain access to zone logs, dashboard functions, or compute resources. | Use MFA for administrators, short-lived IAM roles, least-privilege policies, secret rotation, and audit alerts. |
| Ransomware against archived sensor logs | Historical data and operational evidence could be encrypted or deleted by an attacker. | Use immutable backups, versioned encrypted storage, least-privilege write permissions, and tested recovery procedures. |

---

## 7. Part 1 Engine Deployment

The scheduler and Banker's-Algorithm engine from Part 1 is deployed as a protected cloud compute service.

- The engine uses the exact fixed `jobs.py` workload from Part 1.
- The production scheduler choice is the SJF/SRTF family, specifically SRTF.
- SRTF had the best measured average waiting time of **11.500** and turnaround time of **17.000** for
