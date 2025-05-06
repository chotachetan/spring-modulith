# spring-modulith
Spring Modulith architecture, source codes and notes



### 🔷 What Is Spring Modulith?

Spring Modulith is an extension of **Spring Boot** that promotes **modular monolith architecture**. It helps developers design, structure, and maintain **modular applications** without needing to jump to microservices immediately.

From an **enterprise architect's view**, it’s about **enforcing boundaries, visibility, and communication patterns** between functional modules within the monolith—enabling **clean architecture** principles, maintainability, and scalability.

---

### 🧱 Core Concepts of Spring Modulith

We'll break this into the following pillars:

1. **Modular Architecture**
2. **Explicit Module Boundaries**
3. **Application Events (Asynchronous Communication)**
4. **Module API Visibility**
5. **Module Testing & Documentation**
6. **Architecture Verification**
7. **Spring Modulith Runtime**

---

## 1. Modular Architecture

**Enterprise View:**
You structure your application into **functional units (modules)** based on business capabilities—like “Orders,” “Payments,” “Inventory”—instead of technical layers (Controller/Service/Repository).

**Java Developer View:**
Each module is a Java package (or group of packages) with:

* a `@Module` annotation
* its own domain, logic, services, etc.

### 📊 Diagram: Modular Monolith

```plaintext
+--------------------------+
|      Spring Modulith     |
+--------------------------+
| Orders     | Payments    | Inventory    |
| (module)   | (module)    | (module)     |
+--------------------------+
```

Each module is **encapsulated** and communicates via **events** or exposed APIs.

---

## 2. Explicit Module Boundaries

Spring Modulith scans the package structure to detect modules. You annotate them with:

```java
@Module
package com.example.orders;
```

💡 **Why it matters:** From an architect’s point of view, this:

* Forces well-defined interfaces
* Prevents cyclic dependencies
* Enables team autonomy per module

### ✅ Benefits:

* Easier testing
* Replaceable components
* Encourages domain-driven design

---

## 3. Application Events (Asynchronous Communication)

Instead of direct service calls between modules, Spring Modulith encourages **event-driven communication** using **Spring’s ApplicationEventPublisher**.

```java
// From Inventory module
publisher.publishEvent(new InventoryLowEvent(productId));
```

### 📊 Diagram: Event-Driven Interaction

```plaintext
+---------+    (event)    +----------+
| Orders  | ------------> | Inventory|
+---------+               +----------+
```

Modules don’t directly call each other. They respond to events, reducing tight coupling.

---

## 4. Module API Visibility (Access Control)

Spring Modulith enforces **visibility constraints** using the concept of **API vs. internal** classes.

```java
@ApplicationModule(
    allowedDependencies = {"payments", "inventory"}
)
```

This tells Spring Modulith:

* What dependencies are allowed
* What’s considered public API

You also use **jMolecules** annotations like `@ServiceInterface`, `@ApplicationService`, etc., for semantic clarity.

### ✅ Why It Matters:

* Prevents “leaky abstractions”
* Enforces architecture decisions programmatically

---

## 5. Module Testing & Documentation

Spring Modulith includes **module testing** support:

```java
@SpringModulith
public class ModulithTest {
    // Integration test for module boundaries
}
```

It can also generate **UML diagrams** for architecture documentation using `ApplicationModules#toUmlDiagram()`.

### 📊 Example: Auto-generated Diagram

```plaintext
[Orders] --> [Inventory]
[Orders] --> [Payments]
```

This auto-documentation helps architects **review actual module dependencies** against intended architecture.

---

## 6. Architecture Verification

Spring Modulith validates module boundaries at build time and runtime.

### Features:

* Detects cyclic dependencies
* Identifies unintended module access
* Generates module dependency matrix

```java
ApplicationModules.of(MyApp.class).verify();
```

✅ This ensures **architecture compliance** even as the codebase grows.

---

## 7. Spring Modulith Runtime

Though modular, it’s still a monolith—**one deployable unit**.

But with:

* Independent module lifecycles
* Clean decoupling
* Clear boundaries

Which means:

* You can **scale individual modules into microservices** later without major refactoring.
* You can enforce **bounded contexts** similar to microservices within the monolith.

---

## 🔄 Comparison with Microservices

| Feature           | Microservices        | Spring Modulith                |
| ----------------- | -------------------- | ------------------------------ |
| Deployment        | Multiple deployables | Single monolith                |
| Module Boundaries | Network boundaries   | Package/module boundaries      |
| Communication     | REST/gRPC            | Events / method calls          |
| Testing           | Complex              | Simpler                        |
| Evolution         | Complex              | Gradual (toward microservices) |

---

## 🧩 Sample Project Structure

```plaintext
src/
├── com.example.orders
│   ├── Order.java
│   ├── OrderService.java
│   └── OrderController.java
│   └── package-info.java  // with @Module
├── com.example.inventory
│   ├── Inventory.java
│   ├── InventoryService.java
│   └── package-info.java
```

Each `package-info.java` uses `@Module` to declare it as a module.
The services communicate using domain events.

---

### ✅ Summary: Key Takeaways for Architects

* **Bounded Contexts** = Modules
* **Module Independence** = Decoupled & testable
* **Event-Driven Communication** = Loose coupling
* **Architecture Enforcement** = Programmatically checked
* **Documentation & Diagrams** = Built-in
* **Microservice-ready Design** = Without microservice complexity

---

Would you like:

* A **sample Spring Modulith project** scaffolding?
* Or a **deep dive into event patterns, error handling, or transitioning to microservices**?

Let me know how you’d like to continue—code, diagrams, or design discussions!


Great! Let's go deep on all three areas: **event patterns**, **error handling**, and **transitioning to microservices** — with **code**, **diagrams**, and **enterprise design discussions** for each. Since you're a Java enterprise dev, I’ll lean into pragmatic and architecturally relevant patterns.

---

## 🔁 1. Deep Dive: **Event Patterns in Spring Modulith**

### 💡 Why Events?

Events in Modulith decouple modules by eliminating direct service-to-service calls. This enables clean module isolation, scalability, and domain-driven design.

---

### 🧱 Pattern 1: Domain Event + Listener

📌 Example: `OrderCreatedEvent` triggers inventory allocation.

#### ✅ Domain Event

```java
public record OrderCreatedEvent(UUID orderId, List<UUID> productIds) {}
```

#### ✅ Publisher (in `orders` module)

```java
@Component
@RequiredArgsConstructor
public class OrderService {
    private final ApplicationEventPublisher publisher;

    public void createOrder(Order order) {
        // Save order...
        publisher.publishEvent(new OrderCreatedEvent(order.getId(), order.getProductIds()));
    }
}
```

#### ✅ Listener (in `inventory` module)

```java
@Component
public class InventoryEventListener {

    @EventListener
    public void handle(OrderCreatedEvent event) {
        // Check and reserve inventory...
    }
}
```

---

### 🧱 Pattern 2: Asynchronous Event Processing

By default, Spring Modulith uses **synchronous delivery**. Use `@Async` to handle in background.

```java
@EventListener
@Async
public void handle(OrderCreatedEvent event) {
    // Long-running logic
}
```

🧰 Best with:

* `@EnableAsync`
* Executor configuration

---

### 🧱 Pattern 3: Events as Integration Points

Use events to **simulate eventual microservices** communication—helpful for eventual decomposition.

📌 Each event becomes an “integration contract.”

```plaintext
+----------+           Event            +------------+
| Orders   | -------------------------> | Inventory  |
+----------+                           +------------+
```

Add `@DomainEvent` to annotate significant events:

```java
@DomainEvent
public record PaymentSuccessful(UUID paymentId, UUID orderId) {}
```

---

## 🚨 2. Error Handling in Event-Driven Modular Monoliths

### 🔄 Synchronous Events

Errors propagate back to the caller.

```java
@EventListener
public void handle(OrderCreatedEvent event) {
    // If exception thrown here, caller knows
}
```

📌 Good for critical workflows.

---

### 🔄 Asynchronous Events

Errors are **swallowed by default**. You must handle them explicitly.

#### Option A: Logging

```java
@Async
@EventListener
public void handle(OrderCreatedEvent event) {
    try {
        // logic...
    } catch (Exception ex) {
        log.error("Inventory update failed", ex);
    }
}
```

#### Option B: Spring’s `@TransactionalEventListener`

Gives better control:

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void handleEvent(OrderCreatedEvent event) {
    // Guaranteed to run after commit
}
```

---

### 🧰 Enterprise Patterns for Resilience

| Pattern             | Use When                                     |
| ------------------- | -------------------------------------------- |
| Retry with Backoff  | Transient failures (e.g., external API fail) |
| Dead Letter Queue   | Persist failed event data                    |
| Circuit Breaker     | If listener depends on flaky service         |
| Compensating Action | Eventual consistency needed                  |

---

## 🚀 3. Transitioning to Microservices from Modulith

### 🧱 Modulith → Microservices = Gradual Path

Spring Modulith’s modular design lets you **extract modules** into services one at a time.

---

### 📦 Step 1: Identify Candidate Modules

Use `ApplicationModules.of(MyApp.class).verify()` + UML diagram to assess coupling.

Modules with:

* Minimal inbound dependencies
* High cohesion
* Clear event APIs

📌 Example: `Inventory`, `Notifications`

---

### 🛠 Step 2: Replace Internal Events with External Messaging

#### Before (intra-JVM):

```java
publisher.publishEvent(new OrderShippedEvent(...));
```

#### After (external messaging):

```java
rabbitTemplate.convertAndSend("shipping-exchange", new OrderShippedEvent(...));
```

Transition happens here:

```plaintext
+------------+       RabbitMQ/EventBridge      +-------------+
| Orders     | ------------------------------> | Shipping MS |
+------------+                                 +-------------+
```

Use:

* **Spring Cloud Stream**
* **Kafka, RabbitMQ**
* **Event contracts with Avro or JSON Schema**

---

### 💬 Step 3: Convert Internal Calls to REST/Feign/GraphQL

If `orders` calls `payments` directly:

```java
@Autowired
PaymentService service;
```

Refactor to use:

* **OpenFeign**
* **RestTemplate/WebClient**

Expose APIs via:

```java
@RestController
@RequestMapping("/api/payments")
```

---

### 🗂 Step 4: Extract Gradually

* Start with **database decoupling** (if shared).
* Externalize one module at a time.
* Use **backward-compatible events** and **shared schema** libraries.

---

### 📊 Diagram: Transition Plan

```plaintext
Phase 1: Modulith
+------------------------------+
| Orders | Payments | Shipping|
+------------------------------+

Phase 2: Hybrid
+---------+     Kafka     +------------+
| Orders  | ------------> | ShippingMS |
+---------+               +------------+

Phase 3: Microservices
+---------+     Kafka     +------------+
| OrdersMS| ------------> | ShippingMS |
+---------+               +------------+
```

---

## 🧠 Summary

| Area                   | Patterns & Practices                                               |
| ---------------------- | ------------------------------------------------------------------ |
| **Event Patterns**     | Synchronous, Asynchronous, Domain Events, Integration Points       |
| **Error Handling**     | `@Async`, Retry, Dead Letter Queues, `@TransactionalEventListener` |
| **Microservices Move** | Gradual extraction, Replace internal events with messaging         |

---

## ✅ **Part 2: Modulith-to-Microservice Migration Checklist**

Here’s a **step-by-step enterprise-grade migration checklist**:

---

### 🔶 1. 📦 **Modularize the Monolith**

* [ ] Annotate domain packages with `@Module`
* [ ] Ensure package-private access where possible
* [ ] Move inter-module calls to events

---

### 🔶 2. 🔍 **Validate Module Boundaries**

* [ ] Use `ApplicationModules.of(...).verify()`
* [ ] Generate UML & analyze dependencies
* [ ] Fix any cyclical dependencies

---

### 🔶 3. 🧪 **Isolate Integration Points**

* [ ] Wrap outbound calls (e.g., payment gateways) in adapters
* [ ] Replace method calls with domain events

---

### 🔶 4. 🛡 **Harden Events and Listeners**

* [ ] Use `@Async` for non-critical listeners
* [ ] Add retry policies (`@Retryable`)
* [ ] Add fallback/error logging
* [ ] Persist dead-letter messages if needed

---

### 🔶 5. 🚚 **Extract Candidate Module**

* [ ] Pick a low-coupled module (e.g., Notifications, Shipping)
* [ ] Move to standalone Spring Boot app
* [ ] Set up RabbitMQ/Kafka for event transport
* [ ] Replace internal events with external messaging

---

### 🔶 6. 🌐 **Expose APIs for Integration**

* [ ] REST controllers for shared module functionality
* [ ] Secure with OAuth2/JWT if needed
* [ ] Provide client SDK or Feign client for reuse

---

### 🔶 7. 📊 **Observe and Monitor**

* [ ] Add tracing/log correlation
* [ ] Include Prometheus/Grafana for monitoring
* [ ] Use Zipkin/Jaeger for distributed tracing (if async)

---
## ✅ **Part 1: GitHub Repository Plan**

I'll describe the structure of a **Spring Modulith repo** covering:

* Domain modules with `@Module`
* Event publishing/listening
* Synchronous & async handling
* Error handling patterns
* Gradual microservice extraction simulation

---

### 🔧 Repo Name: `spring-modulith-enterprise-patterns`

#### ✅ Modules:

```plaintext
src/
├── com.example.orders      // Publishes OrderCreatedEvent
├── com.example.inventory   // Listens to events, checks inventory
├── com.example.payments    // Processes payments, listens for OrderPaidEvent
├── com.example.shipping    // Separate app in phase 2 (microservice extract)
```

#### ✅ Key Features:

| Feature                           | Implementation                                                   |
| --------------------------------- | ---------------------------------------------------------------- |
| Modular Boundaries (`@Module`)    | Each domain package has `@Module` annotation                     |
| Event Publishing                  | `ApplicationEventPublisher` for domain events                    |
| Event Listeners                   | `@EventListener` + `@Async` listeners across modules             |
| Transactional Event Handling      | `@TransactionalEventListener` for after-commit event consistency |
| Retry Logic                       | Spring Retry with backoff and max attempts                       |
| Dead Letter Queue (DLQ) Simulated | Failed event handling captured and persisted in DB/log           |
| Gradual Microservice Extraction   | Shipping module moves to Spring Boot app + RabbitMQ integration  |

#### ✅ Extras:

* Test classes per module using `@SpringModulith`
* Diagrams generated via `ApplicationModules.toUmlDiagram()`
* README with architecture diagrams


---

