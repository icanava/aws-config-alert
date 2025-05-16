# 🔐 AWS Security Misconfiguration Detection

A real-time alert system built with AWS Config, Lambda, SNS, and EventBridge to detect public S3 buckets and notify security contacts automatically. This serverless solution simulates a common misconfiguration scenario and responds with <30s alert latency.

---

## 🧰 Stack Used

- **AWS Config** – Detects public S3 bucket misconfigurations
- **EventBridge** – Triggers Lambda when a rule is violated
- **Lambda (Python)** – Parses the Config event and sends an SNS alert
- **SNS** – Sends real-time email notifications
- **CloudTrail** – Captures API activity and changes
- **S3** – Used to simulate a misconfigured bucket with public ACL

---

## 📐 Architecture

<img src="https://github.com/user-attachments/assets/bbe96980-27a4-442e-8690-0499c2cfd1d7" alt="Security Alert Architecture Diagram" height="600"/>

---

## 🛠 Deployment Walkthrough

### 1. Enable AWS Config  
<img src="https://github.com/user-attachments/assets/95a13fa3-011c-449a-90a7-973fdcfb7b7c" alt="AWS Config Dashboard" height="600"/>

Enabled AWS Config to record resource configurations and compliance history. Logs are delivered to an S3 bucket.

---

### 2. Add Managed Rules  
<img src="https://github.com/user-attachments/assets/2cfb5b84-db6f-45b6-bcc3-c9a02c1a83d7" alt="Security Rules" height="600"/>

Added:
- `s3-bucket-public-read-prohibited`
- `s3-bucket-public-write-prohibited`

These rules monitor S3 bucket access and flag public buckets as **NON_COMPLIANT**.

🛠️ Troubleshooting:
At one point, the S3 rules (s3-bucket-public-read-prohibited, etc.) weren’t flagging violations as expected. I discovered that public bucket policies don’t trigger these rules — only ACLs do. The fix was to enable ACLs and manually assign public permissions via the Access Control List (ACL) editor, not just through a policy.


---

### 3. Create SNS Topic  
<img src="https://github.com/user-attachments/assets/70cca090-b827-4331-bfb3-6a8c8a3b6632" alt="SNS Topic" height="600"/>

Created an SNS topic called `SecurityAlertsTopic` for delivering real-time email notifications.

---

### 4. Confirm Email Subscription  
<img src="https://github.com/user-attachments/assets/50110537-8b16-414d-a628-3d5b88e8c596" alt="SNS Email Confirmed" height="600"/>

Subscribed and confirmed an email address to receive alerts from the SNS topic.

---

### 5. Create Lambda Function  
<img src="https://github.com/user-attachments/assets/ff4d27f3-fab1-4190-b516-f94b23b754f7" alt="Lambda Security Function" height="600"/>

Wrote a Python Lambda function that:
- Parses incoming AWS Config violation events
- Extracts relevant details (bucket name, rule, region)
- Publishes an SNS alert

🛠️ Troubleshooting:
The Lambda function initially wasn’t alerting even though test events worked. I eventually found out the issue was a hardcoded bucket name mismatch in the test JSON — I had "test-bucket-name" instead of my actual resource ID. Once corrected, alerts flowed properly on manual trigger.

---

### 6. Update IAM Role  
<img src="https://github.com/user-attachments/assets/cbd7b8cc-9e31-4730-af73-777b9135c565" alt="Lambda IAM Role" height="600"/>

Added `sns:Publish` permissions to the Lambda execution role using the `AmazonSNSFullAccess` policy.

---

### 7. Create EventBridge Rule  
<img src="https://github.com/user-attachments/assets/64d21e2c-918b-4ba4-af43-4e395f3a401c" alt="Event Rule" height="600"/>

Configured EventBridge to listen for `Config Rules Compliance Change` events where complianceType is `NON_COMPLIANT`. Target = the Lambda function.
🛠️ Troubleshooting:
Despite the rule being created correctly, no alerts were being triggered automatically. After testing, I confirmed that EventBridge only fires when AWS Config sends a new compliance evaluation event. Some AWS Config rules (especially S3 public ACL rules) can be flaky or delayed, so for validation, I simulated a real event using an actual compliance violation JSON.
---

### 8. Simulated Violation & Email Alert  
<img src="https://github.com/user-attachments/assets/ece1a8d3-0fed-4946-b1b7-f0ecb022079d" alt="Email Alert" height="400"/>

Simulated a public bucket violation via Lambda test event using real-world AWS Config JSON. SNS delivered a real-time alert to the subscribed email inbox.

---

## 📂 File Structure




---

## ✅ What I Learned

- How to integrate AWS Config rules with Lambda via EventBridge
- How to parse JSON events and dynamically alert on policy violations
- The importance of IAM roles in serverless architecture
- Real-world behavior of S3 permissions and public access controls

---


