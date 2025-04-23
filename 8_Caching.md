# Caching

Caching reduces repeatable work through storage, thus reducing latency.
<img width="737" alt="Screenshot 2025-04-23 at 3 05 59 PM" src="https://github.com/user-attachments/assets/a6a347b2-10f7-49a1-8085-256b5f87b1c5" />

## Benefits
- Reduce network calls  
- Avoid repeated computation  
- Reduce database load  

## Types of Caching

### Server-side Caching
<img width="750" alt="Screenshot 2025-04-23 at 3 06 25 PM" src="https://github.com/user-attachments/assets/de9409e3-9865-4454-8bd6-397ed9070863" />


### Client-side Caching
Used when the user looks at the same feed or content repeatedly.
<img width="672" alt="Screenshot 2025-04-23 at 3 06 56 PM" src="https://github.com/user-attachments/assets/31925d86-c269-4a73-a31b-13a05dbe195c" />

### Database-side Caching

Typically, caching exists in all three: client-side, server-side, and database-side.
<img width="722" alt="Screenshot 2025-04-23 at 3 18 23 PM" src="https://github.com/user-attachments/assets/2ea972eb-bdf3-4872-8510-a97b8d720ece" />

## Popular Queries vs Unpopular Queries
Engineers optimize cache storage by trying to keep data that will be accessed next.

## Cache Policies

1. **How do I handle writes?**  
   When the database gets updated, I will need to update the cache as well.  
   - Do I update cache later or together with the database?

2. **How do I remove data from cache if it fills up?**  
   - Use cache eviction policies like **LRU**, **LFU**, etc.

## Drawbacks of LRU

<img width="750" alt="Screenshot 2025-04-23 at 3 13 38 PM" src="https://github.com/user-attachments/assets/01e7e428-4191-4f10-a7a3-f1638b9e9e2c" />

- **Sequential Querying:**  
  Can lead to **Thrashing** (a state where the system spends an excessive amount of time swapping pages between memory and disk, rather than executing useful tasks).

- **Eventual Consistency:**  
  The cache will be eventually consistent with the database but not immediately.  
  - This is a drawback in financial transactions or finance products where you would want true data immediately.

## Other Solutions

- **Distributed Cache (e.g., Redis)** for large distributed systems.
