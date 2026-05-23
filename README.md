
# 🔐 OWASP Juice Shop — API Security Testing

> Hands-on API penetration testing project performed on OWASP Juice Shop
> using Postman. Covers authentication testing, SQL injection, IDOR, 
> JWT inspection and manipulation.

---

## 📌 Project Overview

| Detail | Info |
|---|---|
| **Target** | OWASP Juice Shop |
| **Tool Used** | Postman, jwt.io |
| **Type** | API Security Testing |
| **Environment** | Local (localhost:5511) |

---

## 🎯 Vulnerabilities Tested

| # | Vulnerability | Result |
|---|---|---|
| 1 | Authentication Testing | ✅ Tested |
| 2 | SQL Injection | ✅ Successful |
| 3 | IDOR | ✅ Successful |
| 4 | JWT Inspection | ✅ Successful |
| 5 | JWT Manipulation | ✅ Successful |
| 6 | Rate Limit Testing | ✅ Tested |
| 7 | Broken Access Control | ✅ Successful |

---

## 🛠 Tools Used

- **Postman** — API testing and Collection Runner
- **jwt.io** — JWT token decoding and inspection
- **OWASP Juice Shop** — Intentionally vulnerable target application

---

---

## 🔍 Vulnerability Details

---

### 1️⃣ Authentication Testing
**Request Body:**
```json
{
  "email": "test@gmail.com",
  "password": "test123"
}
```

**Result:**
- Server returned 200 OK
- JWT token received in response
- Token contained sensitive user data including email, role and hashed password

**Screenshot:**

<img width="1918" height="1013" alt="Screenshot 2026-05-21 184325" src="https://github.com/user-attachments/assets/a268ed5f-b842-4334-a2a9-44cfbca6dc34" />

---

### 2️⃣ SQL Injection

**Endpoint:**
POST http://localhost:5511/rest/user/login

**Payload Used:**
```json
{
  "email": "' OR 1=1--",
  "password": "test"
}
```

**Result:**
- Server returned 200 OK
- Bypassed authentication completely
- Logged in as admin@juice-sh.op without valid credentials
- Admin JWT token returned in response

**Why It Works:**

The SQL query becomes:
```sql
SELECT * FROM Users WHERE email='' OR 1=1--' AND password='test'
```
`1=1` is always true so the query returns the first user (admin)
`--` comments out the rest of the query including password check

**Screenshot:**

<img width="1552" height="988" alt="Screenshot 2026-05-21 190316" src="https://github.com/user-attachments/assets/c077e69a-6705-49e4-b74e-773ddc66316d" />


---

### 3️⃣ IDOR (Insecure Direct Object Reference)

**Endpoint:**
GET http://localhost:5511/api/BasketItems/5
GET http://localhost:5511/api/BasketItems/6

**Headers:**
Authorization: Bearer <token>
**Result:**
- BasketItems/5 returned ProductId 4, BasketId 3, quantity 1
- BasketItems/6 returned ProductId 4, BasketId 4, quantity 2
- Accessed another user's basket data using own token
- No authorization check performed by server

**What Is IDOR:**

IDOR occurs when an application uses user-controllable input to
access objects directly without authorization checks. Simply
changing the number in the URL exposed another user's data.

**Screenshots:**

<img width="1598" height="1018" alt="Screenshot 2026-05-21 185501" src="https://github.com/user-attachments/assets/107926d9-11c9-434e-b1cd-e7de34bad049" />
<img width="1552" height="1018" alt="Screenshot 2026-05-21 185801" src="https://github.com/user-attachments/assets/2924b1a3-48d9-4cd3-8787-0452ed790e1d" />


---

### 4️⃣ JWT Inspection

**Tool Used:** jwt.io

**Token Decoded:**
```json
{
  "status": "success",
  "data": {
    "id": 24,
    "email": "test@gmail.com",
    "password": "cc03e747a6afbbcbf8be7668acfebee5",
    "role": "customer",
    "lastLoginIp": "0.0.0.0",
    "isActive": true
  }
}
```

**Result:**
- JWT payload is base64 encoded not encrypted
- Anyone who intercepts the token can decode it
- Sensitive data exposed including hashed password and role
- Algorithm used: RS256

**Screenshot:**

<img width="1286" height="863" alt="Screenshot 2026-05-21 201348" src="https://github.com/user-attachments/assets/3e4f2609-ca6a-4fbb-85af-6d05eed8a46b" />


---

### 5️⃣ JWT Manipulation

**Endpoint:**
GET http://localhost:5511/rest/user/whoami

**Headers:**
Authorization: Bearer <manipulated_token>

**Result:**
- Server accepted manipulated JWT token
- Returned 200 OK with user data
- No proper token signature validation

**Screenshot:**

<img width="1906" height="1007" alt="Screenshot 2026-05-21 201657" src="https://github.com/user-attachments/assets/1b82a8cd-13f7-487f-8306-68e5f2a16a67" />

---

### 6️⃣ Rate Limit Testing

**Endpoint:**
POST http://localhost:5511/rest/user/login

**Method:**
- Used Postman Collection Runner
- Sent 30 rapid login requests

**Result:**
- All 30 requests returned 401
- No 429 Too Many Requests response
- No account lockout triggered
- Application vulnerable to brute force attacks

---

### 7️⃣ Broken Access Control

**Endpoint:**
GET http://localhost:5511/api/Users/

**Result:**
- Returned complete user database
- Included admin account details
- Accessible without admin privileges
- Exposed emails, roles, profile images of all users

**Screenshot:**

<img width="1583" height="995" alt="Screenshot 2026-05-21 204102" src="https://github.com/user-attachments/assets/ddb6f0c2-d1b1-4a7d-9712-e92a8fd10e09" />

---

## 🚨 Findings Summary

| Vulnerability | Severity | Impact |
|---|---|---|
| SQL Injection | 🔴 Critical | Full admin access without credentials |
| Broken Access Control | 🔴 Critical | Full user database exposed |
| IDOR | 🟠 High | Access to other users private data |
| JWT Inspection | 🟠 High | Sensitive data exposed in token |
| JWT Manipulation | 🟠 High | Forged token accepted by server |
| Missing Rate Limiting | 🟡 Medium | Vulnerable to brute force attacks |

---

## ⚠️ Disclaimer

> This project was performed on OWASP Juice Shop which is an
> intentionally vulnerable application designed for security training.
> All testing was done in a local environment.
> Do not attempt these techniques on real applications without
> explicit written permission. Unauthorized testing is illegal.

---

## 🔗 References

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [jwt.io](https://jwt.io/)
- [Postman](https://www.postman.com/)
- [OWASP IDOR](https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf)
- [SQL Injection OWASP](https://owasp.org/www-community/attacks/SQL_Injection)



