# System Design Basics Notes

## Overview

When designing scalable systems, consider:
1. **Architectural components**
2. **How components interact**
3. **Tradeoffs in usage and design**

Premature scaling is often inefficient. Instead, plan with scalability in mind to avoid costly reworks later.

---

## Key Characteristics of Distributed Systems

### 1. Scalability
**Definition**: Ability of a system to handle growing workload gracefully.

#### Types:
- **Horizontal Scaling**: Add more servers.
  - *Example*: Cassandra, MongoDB
- **Vertical Scaling**: Add resources (CPU/RAM) to existing server.
  - *Example*: MySQL

> **Tip**: Horizontal scaling supports dynamic growth better.

---

### 2. Reliability
**Definition**: The probability that a system fails within a given time.

- Ensures system continues functioning even when some components fail.
- Achieved through **redundancy**.

**Example**: In an e-commerce platform like Amazon, a user’s cart must persist even if a server crashes.

---

### 3. Availability
**Definition**: System uptime during a specific period.

- High availability ≠ High reliability.
- Achievable via rapid failover and repair mechanisms.

**Example**: An aircraft with low downtime is *available*. One that survives all weather conditions is *reliable*.

> A system can be available but not reliable.

---

### 4. Efficiency
**Measures**:
- **Latency**: Time to receive first item.
- **Throughput**: Items delivered per second.

> Consider:
- Network load
- Hardware/software heterogeneity
- Topology variation

---

### 5. Serviceability / Manageability
**Definition**: Ease of diagnosing, maintaining, and repairing the system.

- Early fault detection reduces downtime.
- Auto-diagnosing systems improve maintainability.

**Example**: A server automatically contacting support on fault detection.

---

## Load Balancing

### What is Load Balancing?
Distributes traffic across multiple servers to improve **availability** and **responsiveness**.

### Placement:
- Between user and web servers
- Between web servers and app/cache layers
- Between internal layers and DBs

---

### Benefits
- Faster service and lower latency
- High throughput and reduced downtime
- Predictive insights for bottlenecks
- Reduced stress per server

---

### Load Balancing Algorithms

#### 1. Least Connection
- Chooses server with fewest active connections.
- *Use case*: Long-lived connections.

#### 2. Least Response Time
- Picks server with fewest connections *and* lowest response time.

#### 3. Least Bandwidth
- Chooses server with lowest current traffic (Mbps).

#### 4. Round Robin
- Cycles through servers evenly.
- *Use case*: Equal server specs, short-lived requests.

#### 5. Weighted Round Robin
- Weights servers by capacity.
- Higher weight = more requests.

#### 6. IP Hash
- Uses hash of client IP to pick server.
- Ensures client always hits the same server.

---

### Redundant Load Balancers

> A single LB is a potential **SPOF** (Single Point of Failure).

- Use **dual LBs** in a cluster to monitor and replace each other on failure.

---
