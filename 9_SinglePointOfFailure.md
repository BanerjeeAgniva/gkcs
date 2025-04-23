# Single Point of Failure
<img width="364" alt="Screenshot 2025-04-23 at 3 55 46 PM" src="https://github.com/user-attachments/assets/4c01b202-a050-471a-9380-6360cbcb9564" />


A **Single Point of Failure** (SPOF) is a part of a system that, if it fails, will stop the entire system from working.

Example:  
Humanity as a service will die if a meteor crashes to Earth.

In systems, multiple components are connected to a single point, and if that point crashes, the entire system crashes.

## Avoiding Single Point of Failure

1. **More nodes**  
2. **Master-Slave architecture of databases**  <img width="461" alt="Screenshot 2025-04-23 at 3 57 43 PM" src="https://github.com/user-attachments/assets/8cbbad7a-88e8-4a7b-b556-dea8662556b1" />

3. **Multiple regions** <img width="608" alt="Screenshot 2025-04-23 at 4 02 33 PM" src="https://github.com/user-attachments/assets/f3d363ff-4365-45c4-b0c3-eb7f15998336" />

   - Multiple collections of:
     - **Gateways** (containing the load-balancers)
     - **Nodes** (servers)
     - **Databases**

## Real-World Example

**Netflix → ChaosMonkey**  
- Goes to production and randomly takes down one node  
- Ensures the system is resilient and can handle failures without crashing

Netflix Chaos Monkey is a tool used in chaos engineering, a practice that involves intentionally introducing disruptions into a system to test its resilience. It randomly terminates virtual machine instances and containers within a production environment, forcing engineers to design applications that can handle such failures gracefully
