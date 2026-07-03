---
layout: default
title: Hackathon Prep Bootcamp — Facilitator Lecture Notes
---

# Hackathon Prep Bootcamp — Facilitator Lecture Notes

> **Format:** 3-hour pre-implementation teaching session  
> **Goal:** Students leave with a clear problem design, tech mapping, and implementation plan — ready to build independently  
> **Audience:** College students, mixed levels (Beginner → Intermediate)  
> **Problem Bank:** [Core Java Industry Problem Statements](https://hackathon-java-python.vercel.app/java.html)

---

## Session Overview

| Time | Segment | Duration |
|------|---------|----------|
| 0:00 | Opening & Context | 10 min |
| 0:10 | Problem Framing Workshop | 15 min |
| 0:25 | Core Java Essentials | 30 min |
| 0:55 | OOP + Exception Handling | 15 min |
| 1:10 | Collections Deep Dive | 20 min |
| 1:30 | Spring Boot Blueprint | 30 min |
| 2:00 | MySQL + Schema Design | 25 min |
| 2:25 | Integration Blueprint | 15 min |
| 2:40 | Team Design Sprint | 15 min |
| 2:55 | Handoff + Evaluation Orientation | 5 min |

---

## Delivery Tracks

Split students into two tracks after the Core Java block:

| Track | Who | Covers |
|-------|-----|--------|
| **Foundation** | Beginners (Module A/B problems) | Core Java + Collections only |
| **Full-Stack** | Stronger students (Module C+ problems) | Core Java + Collections + Spring Boot + MySQL |

---

## 0:00–0:10 | Opening & Context

### What to say
> "Today is not a coding race. You will implement your project separately after this session. Right now, the goal is to think like engineers — understand the problem deeply, design the solution clearly, and know what to build before you write a single line."

### Key message
Good hackathon submissions are decided by **problem understanding**, not typing speed.

We are training this mental model:

```
Problem Clarity → Logic Design → Test Thinking → Execution
```

### Write on board
```
Make it clear → Make it correct → Make it clean
```

---

## 0:10–0:25 | Problem Framing Workshop

### What to teach
Convert any problem statement into an engineering contract.

### The 6-Point Framework

| Point | Question to answer |
|-------|--------------------|
| **Inputs** | What data does the system receive? |
| **Outputs** | What must be produced? |
| **Rules** | What business logic applies? |
| **Edge Cases** | Where can logic break silently? |
| **Constraints** | Any limits on values, formats, volumes? |
| **Assumptions** | What are you assuming when unstated? |

### What to say
> "If a team cannot state inputs and outputs clearly in 2 minutes, they are not ready to code. Design the contract first."

### Team task
Each team fills the **Problem Brief Template** (Handout 1) for their chosen problem.

### Quick audit questions to ask teams
- "What is one invalid input your program must reject?"
- "What happens at a boundary — e.g., exactly ₹100 income, exactly 0 units?"
- "What are you assuming the caller always provides correctly?"

---

## 0:25–0:55 | Core Java Essentials

### Teaching principle
> "Most bugs are not syntax bugs — they are logic and boundary bugs."

---

### Topic 1: Data Types and Precision

**What to stress:**
- `int` division silently discards fractions — a career-long trap
- Always use `double` for money/ratios during computation
- Round **only** the final output, not intermediate values

**Example to show:**
```java
int income = 50000;
int emi = 22000;
// WRONG — gives 0 or 1, not 0.44
System.out.println(emi / income);

// CORRECT
System.out.println((double) emi / income);  // 0.44
```

**Real impact (from problem bank):**
- EMI eligibility → FOIR ratio computed as integer = wrong result
- Tax slab → marginal rate applied to full income = over-tax

---

### Topic 2: Control Flow — Conditions and Decision Trees

**What to stress:**
- Order of conditions matters
- Guard clauses (check invalid early, return fast)
- `switch` for clean category/type dispatch

**Guard clause pattern:**
```java
if (income <= 0) {
    System.out.println("Invalid: income must be positive");
    return;
}
// rest of logic only runs if input is valid
```

**Condition order trap (slab logic):**
```java
// WRONG — first slab catches everything >= 250000
if (income >= 250000) { ... }
if (income >= 500000) { ... }  // never reached

// CORRECT — check high first
if (income >= 1000000) { ... }
else if (income >= 500000) { ... }
else if (income >= 250000) { ... }
```

**Real impact:** Tax slab, tariff billing, toll fare — all need ordered condition chains.

---

### Topic 3: Loops and Accumulator Pattern

**What to stress:**
- One clean pass where possible
- Track sum/min/max/count together in one loop
- Off-by-one errors at slab boundaries

**Accumulator pattern:**
```java
double total = 0;
double min = prices[0], max = prices[0];

for (int i = 0; i < prices.length; i++) {
    total += prices[i];
    if (prices[i] < min) min = prices[i];
    if (prices[i] > max) max = prices[i];
}
double avg = total / prices.length;
```

**Slab loop pattern (for billing):**
```java
int[] slabLimits = {100, 200, 300};
double[] slabRates = {1.5, 2.5, 4.0};
int units = 450;
double charge = 0;

for (int i = 0; i < slabLimits.length && units > 0; i++) {
    int billable = Math.min(units, slabLimits[i]);
    charge += billable * slabRates[i];
    units -= billable;
}
charge += units * 6.0; // last slab
```

---

### Topic 4: Method Decomposition

**What to stress:**
> "If your `main` method is doing everything, debugging will be painful."

**Standard method split template:**
```java
public static void main(String[] args) {
    String input = readInput();
    if (!validateInput(input)) return;
    double result = computeResult(input);
    printOutput(result);
}
```

One method = one responsibility.

---

## 0:55–1:10 | OOP + Exception Handling

### Topic 5: Classes and Encapsulation

**When to use OOP:**
> "Use classes when behavior differs by type or when multiple fields belong together and travel together."

**Anti-pattern to avoid:**
```java
// BAD — parallel arrays, easily de-synced
String[] names = {"Alice", "Bob"};
double[] salaries = {50000, 60000};
```

**Better:**
```java
class Employee {
    private String name;
    private double basicSalary;

    public Employee(String name, double basicSalary) {
        if (basicSalary <= 0) throw new IllegalArgumentException("Salary must be positive");
        this.name = name;
        this.basicSalary = basicSalary;
    }

    public double computeNetPay() {
        double hra = basicSalary * 0.4;
        double pf = basicSalary * 0.12;
        return basicSalary + hra - pf;
    }
}
```

---

### Topic 6: Exception Handling

**What to stress:**
- Validate input → throw with clear message
- Never swallow exceptions silently
- Custom exceptions for business rule violations

**Pattern:**
```java
public static double computeEMI(double principal, double rate, int months) {
    if (principal <= 0 || rate <= 0 || months <= 0) {
        throw new IllegalArgumentException("All inputs must be positive");
    }
    double r = rate / 12 / 100;
    return (principal * r * Math.pow(1 + r, months)) / (Math.pow(1 + r, months) - 1);
}
```

---

## 1:10–1:30 | Collections — The Hidden MVP

### Teaching principle
> "Collections solve real-world data structure problems. Choose by behavior, not by habit."

---

### Which collection for which problem?

| Need | Use | Example |
|------|-----|---------|
| Ordered list of items | `List<T>` | Cart items, price ticks, log lines |
| No duplicates | `Set<T>` | Unique customer IDs, unique keywords |
| Key → value lookup | `Map<K,V>` | Word counts, category rates, plate → owner |
| FIFO queue | `Queue<T>` | Task queue, ATM request queue |

---

### List — Ordered items
```java
List<Double> prices = new ArrayList<>();
prices.add(1050.0);
prices.add(1075.5);
prices.add(1040.0);

double max = Collections.max(prices);
double min = Collections.min(prices);
```

---

### Map — Key-value counting and lookup
```java
// Keyword frequency counter
Map<String, Integer> freq = new HashMap<>();
String[] tokens = log.split("\\s+");

for (String token : tokens) {
    freq.put(token, freq.getOrDefault(token, 0) + 1);
}

// Print sorted by frequency
freq.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .forEach(e -> System.out.println(e.getKey() + " : " + e.getValue()));
```

---

### Set — Deduplication
```java
// Find unique customers from transaction list
Set<String> uniqueCustomers = new HashSet<>(transactionList);
System.out.println("Unique customers: " + uniqueCustomers.size());
```

---

### Collections problem mapping from the bank

| Problem | Collection | Why |
|---------|-----------|-----|
| Log keyword counter | `Map<String, Integer>` | count per keyword |
| Seat allocation map | `boolean[][]` (2D array) | grid state |
| Customer dedup key | `Map<String, List<String>>` | canonical key → names |
| Monthly sales heatmap | `Map<String, Map<String, Double>>` | region → month → sales |
| Reward point categories | `Map<String, Double>` | category → accumulated spend |

---

## 1:30–2:00 | Spring Boot Blueprint

> **Delivery note:** Keep this at architecture level. Show a running skeleton, not live-coding everything from scratch.

### What to say
> "Spring Boot is how you expose your Java logic as an API. Your business rules stay in Java. Spring Boot just gives them a front door."

---

### Core layered architecture

```
Client (browser/app/Postman)
        ↓  HTTP Request
   Controller  ← receives and validates the request
        ↓
    Service    ← holds your business logic
        ↓
  Repository   ← talks to the database
        ↓
   Database (MySQL)
        ↑
   Response flows back up
```

### What lives where

| Layer | Responsibility | Contains |
|-------|---------------|----------|
| `controller` | API entry point | `@RestController`, request mapping, input validation |
| `service` | Business logic | All rules, calculations, decisions |
| `repository` | Data access | `JpaRepository`, queries |
| `model/entity` | DB table mapping | `@Entity`, `@Table`, fields |
| `dto` | API data shape | Request/response objects |

---

### Minimal working example (show this pre-built)

**Entity:**
```java
@Entity
@Table(name = "submissions")
public class Submission {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String studentName;
    private String projectTitle;
    private String status;
    // getters + setters
}
```

**Repository:**
```java
public interface SubmissionRepository extends JpaRepository<Submission, Long> {
    List<Submission> findByStatus(String status);
}
```

**Service:**
```java
@Service
public class SubmissionService {
    @Autowired
    private SubmissionRepository repo;

    public Submission create(Submission s) {
        if (s.getStudentName() == null || s.getProjectTitle() == null)
            throw new IllegalArgumentException("Name and title are required");
        s.setStatus("SUBMITTED");
        return repo.save(s);
    }

    public List<Submission> getAll() {
        return repo.findAll();
    }
}
```

**Controller:**
```java
@RestController
@RequestMapping("/api/submissions")
public class SubmissionController {
    @Autowired
    private SubmissionService service;

    @PostMapping
    public ResponseEntity<Submission> create(@RequestBody Submission s) {
        return ResponseEntity.ok(service.create(s));
    }

    @GetMapping
    public List<Submission> getAll() {
        return service.getAll();
    }
}
```

---

### What to emphasize
- Controller is **thin** — no logic, just routing
- Service owns **all rules** — if someone asks "where is the business logic?", answer is always "service"
- Repository never decides anything — just fetches and saves

---

## 2:00–2:25 | MySQL + Schema Design

### Teaching principle
> "Bad schema creates bad application logic. Model data carefully before writing any Java."

---

### Three schema design questions
1. What are my **entities** (things with data)?
2. What are the **relationships** between them?
3. What will I **query** most often?

---

### Schema design from a real problem

**Problem: Hackathon submission tracker**

Step 1 — List entities:
- Student, Project, Submission

Step 2 — Map relationships:
- Student **submits** one Project → Submission links them

Step 3 — Schema:

```sql
CREATE TABLE students (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(100) UNIQUE NOT NULL,
    team        VARCHAR(50)
);

CREATE TABLE projects (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    title       VARCHAR(200) NOT NULL,
    domain      VARCHAR(100),
    difficulty  ENUM('Beginner', 'Intermediate', 'Advanced')
);

CREATE TABLE submissions (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    student_id  INT NOT NULL,
    project_id  INT NOT NULL,
    submitted_at DATETIME DEFAULT NOW(),
    status      ENUM('SUBMITTED','EVALUATED','PASSED','FAILED') DEFAULT 'SUBMITTED',
    score       DECIMAL(5,2),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (project_id) REFERENCES projects(id)
);
```

---

### Essential SQL to know for the hackathon

```sql
-- Insert
INSERT INTO students (name, email, team) VALUES ('Ravi', 'ravi@kl.edu', 'Team A');

-- Read with joins
SELECT s.name, p.title, sub.score
FROM submissions sub
JOIN students s ON sub.student_id = s.id
JOIN projects p ON sub.project_id = p.id
WHERE sub.status = 'EVALUATED';

-- Update
UPDATE submissions SET score = 87.5, status = 'PASSED' WHERE id = 1;

-- Aggregate
SELECT p.domain, AVG(sub.score) as avg_score
FROM submissions sub
JOIN projects p ON sub.project_id = p.id
GROUP BY p.domain;
```

---

### Schema checklist
- Every table has a primary key
- Foreign keys link related tables
- No repeated data columns (normalize)
- ENUM for fixed-value columns
- Timestamps for traceability

---

## 2:25–2:40 | Integration Blueprint

### End-to-end request lifecycle

```
POST /api/submissions
        ↓
Controller: extract body, validate not null
        ↓
Service: check business rules (no duplicate, valid project ID)
        ↓
Repository: save to MySQL
        ↓
DB: persists row
        ↓
Service: returns saved entity
        ↓
Controller: wraps in 200 OK response
        ↓
Client: receives JSON
```

### Error path design

| Failure | Where caught | Response |
|---------|-------------|----------|
| Missing field | Controller/DTO validation | 400 Bad Request |
| Business rule violated | Service layer | 422 with message |
| Record not found | Service layer | 404 Not Found |
| DB error | Repository/global handler | 500 Internal Error |

### What to say
> "Design happy path and failure path together. A system that only works for perfect inputs is not a system."

---

## 2:40–2:55 | Team Design Sprint

### What teams produce in this block
Each team creates their **Design Doc** (Handout 2):

1. Problem brief (inputs/outputs/rules/edge cases)
2. Method breakdown
3. DB schema (if applicable)
4. Tech stack decision (Foundation or Full-Stack track)
5. Test case sheet (Handout 3)

### Mentor review questions during this block
- "Show me your edge cases"
- "Which layer holds your main business rule?"
- "What breaks if the input is null or zero?"

---

## 2:55–3:00 | Evaluation Orientation + Close

### Evaluation rubric

| Criteria | Weight |
|----------|--------|
| Correctness | 40% |
| Edge-case + validation coverage | 25% |
| Code structure (layering, method design) | 20% |
| DB schema quality | 10% |
| Explanation + design clarity | 5% |

### Submission package expected
- Source code
- README (problem + approach + assumptions)
- Test cases with actual output
- Known limitations

### Closing script
> "If your design is clear and your test cases are written, implementation is just execution.  
> Don't improvise blindly. Follow your own plan, validate every rule, and trust your design.  
> You already have the hardest part done."

---

## Handout 1 — Problem Brief Template

```
Team Name      :
Problem ID     :
Problem Title  :

INPUTS
-
-

OUTPUTS
-
-

BUSINESS RULES
1.
2.
3.

EDGE CASES
1.
2.
3.

ASSUMPTIONS
1.
2.

SUCCESS CRITERIA
-
```

---

## Handout 2 — Design Doc Template

```
TECH TRACK        : Foundation / Full-Stack

METHOD BREAKDOWN
Method name       :
  - Input         :
  - Output        :
  - Responsibility:

(repeat for each method)

DB SCHEMA (if applicable)
Table             :
  Columns         :
  Relationships   :

COLLECTIONS USED
- Map / List / Set:  for what purpose

BUILD ORDER
1. Input + validation
2. Core logic
3. Output format
4. Edge cases
5. Spring Boot layers (Full-Stack only)
6. DB integration (Full-Stack only)
7. Final test run
```

---

## Handout 3 — Test Case Template

```
Test ID    :
Type       : Normal / Edge / Invalid
Input      :
Expected   :
Reason     : Why this case matters
```

Minimum **8 test cases** per team:
- 3 normal
- 3 edge (boundary, zero, max)
- 2 invalid input

---

## Quick Reference Card — Repeating Principles

> Use these lines throughout the session to anchor discipline:

1. **"Don't code what you haven't reasoned about."**
2. **"Validate input early, fail clearly, never silently."**
3. **"Controller is thin. Service owns logic. Repository only fetches."**
4. **"Collections are chosen by behavior, not convenience."**
5. **"Bad schema = bad code. Model data first."**

---

*Facilitator notes for KL University Skill Development Division Hackathon*  
*Problem Bank: https://hackathon-java-python.vercel.app/java.html*
