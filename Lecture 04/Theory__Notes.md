-

# 📘 Lecture 04: Unified Modeling Language (UML)

## 1. What is a UML Diagram?

* **Definition:** Unified Modeling Language (UML) is a standardized, visual modeling language used to specify, visualize, construct, and document the artifacts of a software system.
* **The SDE Interview Context:** In a Low-Level Design (LLD) interview (e.g., designing a Parking Lot, BookMyShow, or Splitwise), the interviewer will spend the first $15\text{--}20$ minutes evaluating your UML Class Diagram before asking you to write a single line of C++ code. If your structural relationships are fundamentally broken, the code doesn't matter.

---

## 2. Class Diagrams (Structural Modeling)

A Class Diagram provides a static view of the system. It showcases the classes, their internal attributes, their methods, and how they relate structurally to one another.

### A. The Structure of a Class Box

In a standard UML diagram, a class is represented by a rectangle split horizontally into three distinct sections:

1. **Top Section:** The Class Name (e.g., `Car`, `Account`).
2. **Middle Section:** Attributes / Member Variables.
3. **Bottom Section:** Methods / Member Functions.

### B. Access Modifiers (Visibility Notation)

Instead of writing public or private explicitly, UML uses clean shorthand symbols next to attributes and methods:

* `-` $\rightarrow$ **`private`** (Only accessible within the class).
* `+` $\rightarrow$ **`public`** (Accessible from anywhere outside the class).
* `#` $\rightarrow$ **`protected`** (Accessible within the class and its derived/child classes).
* `~` $\rightarrow$ **`package/default`** (Accessible within the package).

---

## 3. The 4 Crucial Class Relationships (Must-Know)

Interviewers heavily judge how you map dependencies between classes. Here is the exact hierarchy from weakest to strongest relationship:

```
[Association] ──> [Aggregation] ──> [Composition] ──> [Inheritance (Generalization)]
  (Weakest)                                                               (Strongest)

```

### 1️⃣ Association (Weakest Interaction)

* **Concept:** A structural relationship where two completely independent classes interact with each other. One class simply utilizes or references another class, often passed temporarily as a method parameter.
* **Relationship Type:** **"Uses-A"** relationship.
* **Real-World Example:** A `Driver` drives a `Car`. The driver isn't part of the car, and the car isn't part of the driver. They just interact over a specific timeline.

### 2️⃣ Aggregation (Loose Ownership)

* **Concept:** A specialized form of association representing a parent-child relationship where the child **can exist completely independently** outside the lifecycle of the parent.
* **Relationship Type:** **"Has-A"** relationship (Weak).
* **Real-World Example:** A `Department` has `Professors`. If the `Department` class object is deleted or shuts down, the `Professor` object is not destroyed; the professor can move to a different department.
* **UML Notation:** Drawn using a line with a **hollow diamond** ($\diamondsuit$) at the parent/owner class.

### 3️⃣ Composition (Strict, Strong Ownership)

* **Concept:** A highly restricted form of aggregation where the child class **cannot exist** if the parent class is destroyed. Their lifecycles are tightly, immutably bound together.
* **Relationship Type:** **"Part-of"** relationship (Strong "Has-A").
* **Real-World Example:** A `Car` has an `Engine`. An `Engine` cannot conceptually exist out on the road working independently without a car structure housing it. If you run `delete myCar;` in memory, the `Engine` object must be cleanly destroyed right along with it inside the destructor.
* **UML Notation:** Drawn using a line with a **solid/filled black diamond** ($\blacklozenge$) at the parent/owner class.

### 4️⃣ Generalization / Inheritance (Strongest Bound)

* **Concept:** A mechanism where a subclass (child) inherits all properties and behaviors from a superclass (base).
* **Relationship Type:** **"Is-A"** relationship.
* **Real-World Example:** A `SportsCar` **is a** `Car`.
* **UML Notation:** Drawn using a solid line with a **hollow triangular arrowhead** ($\longrightarrow$) pointing directly up to the parent class.

---

## 4. Sequence Diagrams (Behavioral Modeling)

While Class Diagrams show how code is structured *statically*, Sequence Diagrams show how objects interact *dynamically over time*. They map out the execution flow of a specific use case.

### A. Core Architectural Components

* **Lifeline:** Represented as a vertical dashed line descending from an object box at the top. It traces the chronological existence of that object during a system transaction.
* **Activation Bar / Focus of Control:** A thin vertical rectangle overlaid onto the lifeline. It visually marks the exact duration during which that specific object is actively executing a task or running an internal method.
* **Actors:** Represented as stick figures, representing external entities (like a `User` or `Admin`) triggering actions on the system UI.

### B. Message Arrow Types

* **Synchronous Message ($\longrightarrow$):** A solid line with a filled arrowhead. The caller stops execution and blocks, waiting for a response before proceeding. (e.g., `ATM.verifyPIN(1234)`).
* **Return Message ($\dashv$):** A dashed line with an open arrowhead. It represents data or validation errors returning back along the call stack to the original sender. (e.g., Returning a boolean status `true` or an `Invalid PIN Error`).
* **Asynchronous Message ($\rightarrow$):** A solid line with a line-drawn arrowhead. The sender passes a command or signal along and immediately moves to the next task without blocking to wait for a reply.

---

## 🏦 Real-World Interview Case Study: The ATM Cash Withdrawal Flow

When an interviewer asks you to map out how an ATM architecture works, you describe it linearly across time using the components you just learned:

```
Actor [User]        UI [ATM Screen]        Brain [ATM Controller]        Backend [Bank Database]
     │                    │                           │                            │
     │───Insert Card─────>│                           │                            │
     │                    │───Read Card Details──────>│                            │
     │                    │                           │───Validate Account────────>│
     │                    │                           │<───Return Status (Valid)───│
     │                    │<───Prompt for PIN─────────│                            │
     │───Enters PIN──────>│                           │                            │
     │                    │───verifyPIN(pin)─────────>│                            │
     │                    │                           │───Check Credentials───────>│
     │                    │                           │<───Auth Success (Dashed)───│
     │                    │<───Prompt for Amount──────│                            │
     │───Enters $100─────>│                           │                            │
     │                    │───withdrawRequest($100)──>│                            │
     │                    │                           │───checkBalance($100)──────>│
     │                    │                           │<───Balance OK (Dashed)─────│
     │                    │                           │                            │
     │                    │                           │───[Internal Cache Check]   │
     │                    │                           │   (Is cash in machine?)    │
     │                    │                           │                            │
     │                    │                           │───deductAccount($100)─────>│
     │                    │                           │<───Success Confirmed───────│
     │                    │<───Dispense Cash Signal───│                            │
     │<───Collect Cash────│                           │                            │

```

1. **The Core Guard Checks:** The `ATMController` acts as a facade orchestrating the flow. It queries the `Bank Database` to check the ledger balance (`checkBalance`), and then accesses its own local hardware component boundary to confirm physical paper cash availability before authorizing the transaction.
2. **State Commit:** The ledger balance modification (`deductAccount`) happens *before* the hardware receives the instruction to physically output the banknotes, protecting the system from edge-case crashes where cash drops but accounts aren't charged.

---
