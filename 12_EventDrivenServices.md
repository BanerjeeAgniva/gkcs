# 📢 Event-Driven Architecture

---

## 🌟 What is Event-Driven Architecture?

In an **event-driven system**, services use **events** to state that *"yes, something has changed."*

* A service (producer) sends events to an **event bus**.
* The event bus sends these events to **other services** (consumers), which might also produce new events if the incoming event concerns them.
* Services typically **store events** in their **own databases** so that the **event bus** is free to discard the events after delivery.
* The **database** of each service, in reality, is **storing event information**.

---

## ✨ Examples of Event-Driven Systems

* Git
* React
* Node.js
* Game servers

---

## 🚀 Advantages

### 1. Higher Availability at the Cost of Consistency

* If one service goes down, events are still available for other services to process.
* However, there's no guarantee that the events being processed are the *latest*.

### 2. Event Rollback

* Since each service has an **event log**, you can **move back to any point in history**.
* Super useful for debugging: if a bug is introduced after timestamp `t`, replay events one-by-one and find the cause.

### 3. Easy Replacements

* Suppose you want to replace `service2` with `service5`.
* Just **replay the events** from `service2`'s event log into `service5`.
* New events from the event bus should now be directed to `service5`.

### 4. Transactional Guarantee

* **At-most-once delivery:** You send a message and don’t care if it gets lost.
* **At-least-once delivery:** You care (example: invoices) and keep retrying until you get an acknowledgment.

### 5. Storing Intent

* Every event stores **intent**: *why* data was changed.
* If replacing a service (say `service2` to `service5`), you can use these intents to **reprocess events differently** to reach a **preferred state**.

---

## ⚡ Disadvantages

### 1. Consistency Issues

* Events might not be perfectly synced across all services.

### 2. Replacement Not Possible on Gateway Services

* Example: `service4` sends emails to external services.
* External responses may **depend on time** (timestamp-sensitive).
* If you replay old events during replacement, new service (`service6`) would get **different responses**, causing **unexpected behavior**.

**Solution:**

* **Always store timestamps** along with responses from external systems.

### 3. Loss of Control

* In **request-response**, you can **set exact timeouts** and know exactly which services you're communicating with.
* In **event-driven**, these are **dependent on the event bus**.
* You have to **set priorities**, **configure retries**, and **design filtering carefully**.
* If a service wants **only some services** to consume its events, that brings **extra complexity**.

### 4. Hidden Flow

* Hard to **trace** what happens after an event is emitted.
* To know which services are subscribers, you need to **inspect the event bus**.

### 5. Migration Challenges

* It's **not easy to move away** from event-driven once you are deep inside it.
* Switching parts of the system back to **request-response** is complicated.

---

## 🔥 Request-Response vs Event-Driven System

| Feature       | Request-Response                    | Event-Driven                       |
| ------------- | ----------------------------------- | ---------------------------------- |
| Communication | Explicit - "Ask and wait"           | Implicit - "Announce and forget"   |
| Control       | High (time limits, targets defined) | Less control (relies on event bus) |
| Clarity       | Clear interaction flow              | Hidden, needs event bus inspection |
| Timing        | Immediate                           | Asynchronous                       |
| Use-Case Fit  | Real-time queries or commands       | Broadcasting state changes         |

---

## 🎯 How to Transition Between States?

If you want to move from one system state to another:

* **Replay events** from the beginning (full rebuild).
* **Diff-based** processing (apply only the changes).
* **Undo** (but some actions like sending emails **cannot be undone**).

**Solution:**

* Use **Compaction** (squash many events into a final, simplified state).

---

# 🧠 TL;DR

> In an **event-driven architecture**, services **announce changes** by sending **events** to an **event bus**, and other services **react** based on their own interest — leading to **high availability, flexibility**, but **lower control and consistency**.
