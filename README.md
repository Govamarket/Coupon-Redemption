# Coupon-Redemption
Race Condition in Coupon Redemption (Business Logic Bypass)

# Race Condition in Coupon Redemption (Business Logic Bypass)

##  Overview

This write-up demonstrates a **race condition vulnerability** in a coupon redemption system that allows a user to redeem a single-use coupon multiple times by sending concurrent requests.

The application enforces a **“Max Uses: 1 per user”** policy, but fails to properly handle concurrent requests, resulting in a **business logic bypass**.

---

##  Objective

Exploit a race condition in the coupon redemption endpoint to:

* Redeem a coupon multiple times
* Increase account balance beyond intended limits

<div>
  <img width="920" height="486" alt="user account" src="https://github.com/user-attachments/assets/72f2275e-1f95-4981-9a12-0151d8bcdfb8" />

</div>
---

##  Environment

* Target: Local Web Security Challenge Lab
* Tool: Burp Suite
* Endpoint:

  ```
  POST /api/redeem-coupon
  ```

---

##  Vulnerability Description

The application follows a **check-then-act pattern**:

1. Check if coupon has been used
2. Apply discount to balance
3. Mark coupon as used

Due to a **~200ms processing delay**, multiple concurrent requests can pass validation before the update occurs.

This leads to a **Time-of-Check to Time-of-Use (TOCTOU)** race condition.

---

##  Exploitation Steps

---

###  Step 1: Intercept Request (Burp Proxy)

Captured the coupon redemption request using Burp Proxy:

http
POST /api/redeem-coupon HTTP/1.1
Host: localhost
Content-Type: application/json

{"couponCode":"WELCOME50"}

---

###  Step 2: Send to Repeater

The request was sent to Repeater to confirm normal behavior.


 Result:

* Balance increased once ($100 → $150)
* Coupon marked as used

---

###  Step 3: Attempt with Intruder (Failed Concurrency)

Used Intruder with multiple payloads:

* Attack type: Sniper
* Payload count: 5–10

 **Screenshot: Intruder Attack Configuration**

<div>
<img width="1366" height="692" alt="turbo" src="https://github.com/user-attachments/assets/76b614b9-1de7-4666-b481-c57be3fa9f54" />

</div>

 **Screenshot: Intruder Results**


<div>
<img width="1366" height="637" alt="turbo py" src="https://github.com/user-attachments/assets/0a009fb3-1c94-4005-b08f-dbe511e62ec4" />

</div>


 Result:

* Only one successful redemption
* Requests processed sequentially

---

###  Step 4: Exploit via Concurrent Requests (Rapid Fire)

Used the lab’s built-in **Rapid Fire Attack** feature to send simultaneous requests.

* Coupon: `WELCOME50`
* Requests: 10

 **Screenshot: Rapid Fire Attack Panel**


<div>
<img width="940" height="452" alt="race at" src="https://github.com/user-attachments/assets/81d0163a-f201-4e44-8136-ef8ad43bccb6" />

<div>

---

###  Step 5: Verify Exploit

Checked the result via API:

http
GET /api/race-status


 **Screenshot: Race Status Response**

<div>
<img width="1366" height="645" alt="suc repeater" src="https://github.com/user-attachments/assets/094ab95a-b7b2-4a8c-babe-a5a9846f2532" />

</div>

---

##  Results

* Initial Balance: `$100`
* Final Balance: `$600`
* Total Redemptions: `10`

 **Screenshot: Final Balance UI**

<div>
<img width="979" height="487" alt="increase act" src="https://github.com/user-attachments/assets/30c28efa-d279-47a2-8275-94e37ceb9acf" />
</div>


---

##  Impact

The coupon, intended for **single use**, was redeemed **10 times**, leading to:

* Unauthorized financial gain
* Business logic bypass
* Integrity failure in transaction handling

---

##  Root Cause

*  Non-atomic operations
*  No locking mechanism
*  Separate check and update logic
*  Exploitable processing delay

---

##  Mitigation

* Implement **database transactions (BEGIN / COMMIT)**
* Use **row-level locking**
* Enforce **unique constraints** at DB level
* Revalidate before update
* Design operations to be **idempotent**

---

##  Key Takeaways

* Race conditions exploit **timing, not input**
* Sequential testing tools may miss concurrency bugs
* Business logic flaws can lead to **critical financial impact**

---

##  Conclusion

This vulnerability highlights how improper handling of concurrent requests can break core application logic. Ensuring **atomicity and synchronization** is critical in financial and transactional systems.

---

##  Author

Clinton Chidera
Web Security Learner | Aspiring Penetration Tester
