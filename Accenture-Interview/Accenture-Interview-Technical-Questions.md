# Accenture Interview — Technical Question Bank

> **Preparation Plan**
>
> First, collect and organize all interview questions exactly as asked.
> Then solve **one question at a time** with an interview-ready answer.
>
> **Status:** Q17 completed → Solutions will be added one by one.

---

## AWS

### Q1. Which AWS services are you currently using in your project?
**Status:** ⏳ Pending

### Q2. What is Amazon S3?
**Status:** ⏳ Pending

### Q3. What are the use cases of S3?
**Status:** ⏳ Pending

### Q4. How do you store application data in AWS?
**Status:** ⏳ Pending

### Q5. Which AWS database services have you worked with?
**Status:** ⏳ Pending

### Q6. Difference between Amazon RDS and DynamoDB.
**Status:** ⏳ Pending

### Q7. How does your Spring Boot application connect to the database hosted on AWS?
**Status:** ⏳ Pending

---

## Microservices

### Q8. Suppose the port of a microservice changes. How will another service identify and send requests to the correct instance?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q08-Microservice-Port-Change-Service-Discovery/README.md)

### Q9. Why do we use Service Discovery?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q09-Why-Service-Discovery/README.md)

### Q10. What is the role of API Gateway in service routing?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q10-API-Gateway-Service-Routing/README.md)

### Q11. Why should services communicate using service names instead of IP addresses?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q11-Service-Names-vs-IP-Addresses/README.md)

### Q12. What is the Saga Pattern?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q12-Saga-Pattern/README.md)

### Q13. Why do we use Saga Pattern instead of a distributed database transaction?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q13-Saga-vs-Distributed-Transaction/README.md)

### Q14. Explain Choreography Saga with an example.
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q14-Choreography-Saga/README.md)

### Q15. What are Compensation Transactions?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q15-Compensation-Transactions/README.md)

### Q16. How does Saga maintain data consistency across multiple microservices?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q16-Saga-Data-Consistency/README.md)

### Q17. How do you secure your microservices?
**Status:** ✅ Completed — [Detailed Hinglish Solution](Q17-Microservices-Security/README.md)

### Q18. Explain JWT authentication flow.
**Status:** ⏳ Pending

### Q19. How do services authenticate with each other?
**Status:** ⏳ Pending

### Q20. How does a frontend application communicate with backend services in a microservices architecture?
**Status:** ⏳ Pending

### Q21. Why do we use API Gateway instead of directly calling microservices?
**Status:** ⏳ Pending

### Q22. How do you aggregate responses from multiple services?
**Status:** ⏳ Pending

---

## High Traffic & Scalability

### Q23. Suppose one service suddenly receives very high traffic. How will you handle it?
**Status:** ⏳ Pending

### Q24. What is Load Balancing?
**Status:** ⏳ Pending

### Q25. How does Auto Scaling work?
**Status:** ⏳ Pending

### Q26. How would caching help in reducing database load?
**Status:** ⏳ Pending

---

## Fault Tolerance

### Q27. What are Cascading Failures?
**Status:** ⏳ Pending

### Q28. How do you prevent cascading failures in a microservices architecture?
**Status:** ⏳ Pending

### Q29. Which fault tolerance library have you used in Spring Boot?
**Status:** ⏳ Pending

---

## Core Java — HashMap

### Q30. Explain HashMap internal working.
**Status:** ⏳ Pending

### Q31. What happens when two keys have the same hashCode?
**Status:** ⏳ Pending

### Q32. What is the time complexity before and after Java 8 collision handling?
**Status:** ⏳ Pending

---

## Coding Question

### Q33. Given a list of city names, return the city that contains the highest number of repeated characters.

**Example Input:**
```text
["Delhi", "Mumbai", "Chennai", "Pune", "Bangalore"]
```

**Expected Output:**
```text
Bangalore
```

**Status:** ⏳ Pending

---

# Interview Preparation Workflow

We will **not solve all questions together**.

For every question, use this cycle:

1. **Understand the question**
2. **Concept**
3. **Real-world example**
4. **Project-level explanation**
5. **Interview-ready answer**
6. **Possible follow-up questions**
7. **Common mistakes**
8. **2-minute revision answer**
9. **Update this file**
10. Move to the **next question**

---

# Priority Order

### 🔥 Priority 1 — Must Know
- Microservices
- Saga / Distributed Transactions
- Security / JWT
- API Gateway
- Service Discovery
- Scalability
- Fault Tolerance

### 🔥 Priority 2 — AWS
- S3
- RDS
- DynamoDB
- AWS application/database connectivity

### 🔥 Priority 3 — Core Java
- HashMap Internal Working
- Hash Collision
- Java 8 Tree-based collision handling
- Time Complexity

### 🔥 Priority 4 — Coding
- Repeated-character problem

---

# Progress Tracker

| Range | Topic | Status |
|---|---|---|
| Q1–Q7 | AWS | ⏳ Pending |
| Q8 | Microservices — Service Discovery | ✅ Completed |
| Q9 | Microservices — Why Service Discovery? | ✅ Completed |
| Q10 | Microservices — API Gateway Service Routing | ✅ Completed |
| Q11 | Microservices — Service Names vs IP Addresses | ✅ Completed |
| Q12 | Distributed Transactions — Saga Pattern | ✅ Completed |
| Q13 | Distributed Transactions — Saga vs Distributed Transaction | ✅ Completed |
| Q14 | Distributed Transactions — Choreography Saga | ✅ Completed |
| Q15 | Distributed Transactions — Compensation Transactions | ✅ Completed |
| Q16 | Distributed Transactions — Saga Data Consistency | ✅ Completed |
| Q17 | Security — Microservices Security | ✅ Completed |
| Q18–Q22 | Security | ⏳ Pending |
| Q23–Q26 | High Traffic & Scalability | ⏳ Pending |
| Q27–Q29 | Fault Tolerance | ⏳ Pending |
| Q30–Q32 | Core Java / HashMap | ⏳ Pending |
| Q33 | Coding | ⏳ Pending |

---

## Goal

**Question Bank → One-by-One Solution → Interview Answer → Follow-ups → Revision**

> We will add the solution directly below each question as we complete it.
