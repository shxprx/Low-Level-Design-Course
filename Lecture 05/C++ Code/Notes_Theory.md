
# 📗 Lecture 05: SOLID Principles (Part 1)

## 1. Why do we need Design Principles? [00:51](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=51)

When transitioning from simple learning tasks to enterprise systems, a software project shifts from housing a few classes to managing **thousands of classes**. Without a strict blueprint, structural logic becomes chaotic and heavily interwoven.

* **The "Messy Home Wiring" Analogy [01:23](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=83):** Imagine a house where the electrical wires, internet fibers, and water pipes are bound together in a single, tangled bundle. If one internet line snaps, finding it or replacing it safely becomes a nightmare because everything is tightly coupled.
* **The Software Nightmare [02:53](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=173):** Poor architecture destroys a system's **Maintainability** (adding features introduces secondary bugs) and **Readability** (onboarding new engineers takes weeks instead of days).

---

## 2. **S** - Single Responsibility Principle (SRP) [05:25](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=325)

> **The Rule:** "A class should have only one reason to change" or "It should do only one focused thing." [07:11](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=431)

### 🚨 The Violation Case [10:54](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=654)

Putting multiple structural actions under one unit:

```cpp
// VIOLATION: Managing data, printing layouts, and network storage simultaneously.
class ShoppingCart {
public:
    vector<Product> products;
    void calculateTotalPrice(); 
    void printInvoice();   // ❌ Violation: Presentation logic
    void saveToDB();       // ❌ Violation: Persistence logic
};

```

* **Why it breaks:** If you change your layout engine or switch your database schema, you are forced to re-open and modify this exact same `ShoppingCart` class.

### 🛠️ The Clean SDE Refactor [14:26](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=866)

Separate operational boundaries completely by splitting the single class into three tightly targeted single-responsibility components:

```cpp
class ShoppingCart {
public:
    void calculateTotalPrice(); 
};

class ShoppingCartPrinter {
public:
    void printInvoice(const ShoppingCart& cart); // Handles layout only
};

class ShoppingCartStorage {
public:
    void saveToDB(const ShoppingCart& cart);    // Handles database tracking only
};

```

> **Instructor Insight [20:41](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=1241):** SRP does *not* mean a class can only possess a single method. A class can contain multiple functions, provided every single one of those methods directly serves the exact same core responsibility.

---

## 3. **O** - Open/Closed Principle (OCP) [21:35](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=1295)

> **The Rule:** "Software entities should be open for extension, but closed for modification." [21:56](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=1316)

### 🚨 The Violation Case [25:15](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=1515)

Using control blocks (`if-else` / `switch`) to handle shifting application strategies:

```cpp
// VIOLATION: Direct code alterations required for every new storage type
class ShoppingCartStorage {
public:
    void save(string type) {
        if (type == "SQL") { /* SQL logic */ }
        else if (type == "MongoDB") { /* MongoDB logic */ } // ❌ Hard-coded alteration
    }
};

```

### 🛠️ The Clean SDE Refactor [31:02](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=1862)

Introduce an intermediate abstract layer (Interface contract). The operational core remains fully locked down while allowing new child instances to implement custom rules:

```cpp
// The abstract parent contract remains completely unmodified [00:37:48]
class Persistence {
public:
    virtual void save() = 0; 
    virtual ~Persistence() {}
};

class SqlPersistence : public Persistence {
    void save() override { /* SQL execution code */ }
};

class MongoPersistence : public Persistence {
    void save() override { /* MongoDB execution code */ } // ✅ Safe structural extension
};

```

---

## 4. **L** - Liskov Substitution Principle (LSP) [39:24](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=2364)

> **The Rule:** "Subclasses must be completely substitutable for their base classes without breaking the structural integrity of the execution flow." [39:47](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=2387)

### 🚨 The Violation Case [49:53](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=2993)

Forcing an inappropriate inheritance tree that compromises a parent class's behavior:

```cpp
class Account {
public:
    virtual void deposit() = 0;
    virtual void withdraw() = 0;
};

// ❌ CRITICAL VIOLATION: An FD account cannot permit regular cash withdrawals
class FixedDepositAccount : public Account {
    void deposit() override { /* Adds money */ }
    void withdraw() override { 
        throw runtime_error("Withdrawals locked on FD!"); // ❌ Blows up client runtimes
    }
};

```

### ❌ The "Bad" Fix (Do Not Do This in Interviews!) [52:12](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=3132)

Modifying client code to manually identify and switch handling based on an object's dynamic type:

```cpp
// BAD PRACTICE: Tightly couples the client to every single structural type
if (dynamic_cast<FixedDepositAccount*>(acc)) {
    acc->deposit(); // Skip withdraw manually
}

```

* **Why it fails:** Fixing an LSP violation with an explicit type check instantly creates an **OCP violation** because introducing a new account class forces you to break open the client code again to insert more conditional checks [54:02](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=3242).

### 🛠️ The Clean SDE Refactor [56:17](http://www.youtube.com/watch?v=UsNl8kcU4UA&t=3377)

Refactor the base class by splitting it into specific structural interfaces. This ensures child instances never inherit a method contract they cannot completely fulfill:

```cpp
// Interface 1: For any standard funding vehicle [01:05:15]
class DepositOnlyAccount {
public:
    virtual void deposit() = 0;
};

// Interface 2: Extends capabilities only for clear fluid assets
class WithdrawableAccount : public DepositOnlyAccount {
public:
    virtual void withdraw() = 0;
};

// Clean Substitution Mapping:
class SavingsAccount : public WithdrawableAccount { ... }; // Implements both safely
class FixedDepositAccount : public DepositOnlyAccount { ... }; // Implements deposit safely

```

