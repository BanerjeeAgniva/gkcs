# System Design Basics

## Introduction
- System design involves creating scalable, resilient, and consistent systems.
- If you've written an algorithm that others find useful, you need a way to expose it without sharing your computer.
- **Solution:** Use an API (Application Programming Interface) to handle requests and responses.

## Hosting Considerations
### Problems with Self-Hosting
- Requires database configuration.
- Needs endpoint setup for user connections.
- Susceptible to power outages, which can disrupt services.

### Using Cloud Solutions
- Cloud providers like **AWS** offer computing power on demand.
- Essentially, the cloud is a set of remote computers you can log into.
- Advantages:
  - Better reliability and availability.
  - Configuration handled by cloud providers.

## Scaling Your Business
As user requests increase, your single machine may not be able to handle all connections. There are two approaches:

### **1. Vertical Scaling** (Scaling Up)
- Buying a **bigger** machine.
- **Example:** Upgrading a laptop with more RAM and a faster CPU.
- **Pros:** Faster inter-process communication (IPC), data consistency.
- **Cons:** Limited by hardware constraints, single point of failure.

### **2. Horizontal Scaling** (Scaling Out)
- Buying **more** machines.
- **Example:** Instead of one powerful laptop, using multiple laptops to distribute tasks.
- **Pros:** More resilient, scalable.
- **Cons:** Requires load balancing, network-based communication (slower than IPC).

## Vertical vs. Horizontal Scaling
| Factor | Vertical Scaling | Horizontal Scaling |
|--------|----------------|------------------|
| Load Balancing | Not needed | Required |
| Failure Tolerance | Single point of failure | Resilient |
| Communication Speed | Fast (IPC) | Slower (network calls) |
| Data Consistency | High | More complex, requires distributed databases |
| Scalability | Limited by hardware | Scales well with additional servers |

**Example:**
- If your app needs more power, you can:
  1. Upgrade your server (**Vertical Scaling**)
  2. Add more servers (**Horizontal Scaling**)

## Practical Considerations
- **Hybrid Approach:** Start with vertical scaling (for fast IPC & data consistency), then transition to horizontal scaling (for resilience & scalability).
- **Final Considerations:**
  - **Is it scalable?** Can the system handle more users?
  - **Is it resilient?** Can it recover from failures?
  - **Is it consistent?** Does the system maintain accurate data?

## Conclusion
System design is about balancing trade-offs to meet business and technical requirements effectively.
