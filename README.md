
Serverless Zero-Trust SIEM &amp; risk scoring engine built on AWS using Python. Leverages S3, Lambda, and DynamoDB for automated malware detection, dynamic entity risk scoring, and centralized logging. Designed for seamless integration with CloudWatch and Wazuh to provide deep threat visibility and scalable incident response in the cloud.


## Overview
This project presents an advanced Serverless Security Middleware Architecture designed to act as an intelligent, automated gatekeeper for cloud-based data pipelines[cite: 1]. Modern cloud environments often suffer from vulnerabilities related to unverified file uploads, where malicious code can bypass access controls if files are not scanned in real-time at the point of entry[cite: 1]. 

To solve this, the system provides a preemptive security strategy[cite: 1]. It intercepts incoming data, performs cryptographic hash analysis via global threat intelligence, and executes identity-based behavioral risk scoring—all before the data is allowed to interact with trusted internal infrastructure[cite: 1]. Built entirely on an event-driven serverless model, this solution is highly scalable, cost-effective, and operates with near-zero latency[cite: 1].

## Core Architecture & AWS Components
The system leverages a modular, cloud-native architecture utilizing Amazon Web Services (AWS)[cite: 1]:

* **Amazon S3 (Multi-Zone Storage):** Implements a strict three-tier storage design to segregate data based on its threat classification[cite: 1].
  * `staging-dmz-aris-7788`: The Demilitarized Zone (DMZ) where unverified files are temporarily held[cite: 1].
  * `main-brain-aris-7788`: The trusted storage zone for verified, safe files[cite: 1].
  * `quarantine-vault-aris-7788`: An isolated vault for malicious or suspicious files[cite: 1].
* **AWS Lambda (`RiskScannerBrain`):** The central processing engine powered by Python 3.14[cite: 1]. It is triggered automatically by S3 event notifications, ensuring real-time execution without the need for dedicated server provisioning[cite: 1].
* **Amazon DynamoDB (State & Logging):** A highly scalable NoSQL database managing two critical tables[cite: 1]:
  * `ArisScanLogs`: Maintains immutable forensic logs of every scanned file, including its SHA-256 hash, filename, and scan outcome[cite: 1].
  * `UserRiskScores`: Tracks dynamic user behavioral profiles and cumulative risk scores based on upload history[cite: 1].
* **VirusTotal API (Threat Intelligence):** External integration that cross-references generated file hashes against a global database of known malware signatures and antivirus engine reports[cite: 1].

## Advanced Security Features
* **Zero-Trust Enforcement:** The system operates on the principle of "never trust, always verify"[cite: 1]. Even authenticated users are subjected to continuous behavioral monitoring[cite: 1]. If a user's risk score reaches a critical threshold (100 points), the system immediately quarantines their identity and rejects subsequent file uploads[cite: 1].
* **Cryptographic Hashing (SHA-256):** Instead of analyzing raw file contents, the system calculates a unique SHA-256 digital fingerprint[cite: 1]. This ensures data privacy, reduces processing time, and securely identifies threats regardless of filename obfuscation[cite: 1].
* **Heuristic Penalty Engine:** Beyond external threat intelligence, the system applies local heuristic penalties[cite: 1]. Files receive risk points for possessing suspicious extensions (e.g., `.exe`, `.py`, `.sh`) or containing threat keywords in their filenames (e.g., `exploit`, `hack`)[cite: 1].
* **Adaptive Throttling:** User privileges are dynamically adjusted based on their risk score (e.g., shifting a user to a "Throttled" state if their score exceeds 50 points)[cite: 1].

## Pipeline Execution Workflow
1. **Ingestion & Trigger:** A user uploads a file to the S3 Staging Bucket[cite: 1]. This generates an `ObjectCreated` event, invoking the AWS Lambda function instantly[cite: 1].
2. **Identity Pre-Check:** Lambda queries DynamoDB to assess the uploading user's current risk score[cite: 1]. If the score is >= 100, the file is moved to the Quarantine Vault, and execution halts[cite: 1].
3. **Hash Generation & API Query:** The file is securely downloaded to memory, and a SHA-256 hash is generated[cite: 1]. The hash is sent to the VirusTotal API via an encrypted HTTPS request[cite: 1].
4. **Decision Engine & Scoring:**
   * The system calculates a total penalty based on the VirusTotal response and local heuristics[cite: 1].
   * If the penalty is > 0, the file is classified as **RISKY**[cite: 1].
   * If the penalty is 0, the file is classified as **SAFE**[cite: 1].
5. **Database Logging:** The file's hash, name, and classification are logged in `ArisScanLogs`[cite: 1]. If the file was risky, the user's score in `UserRiskScores` is incremented, and their privilege level is updated[cite: 1].
6. **File Routing:** The file is copied to its respective target bucket (Safe or Quarantine) and permanently deleted from the Staging DMZ[cite: 1].

## Infrastructure Setup & Deployment
### 1. Storage Configuration
Create three General Purpose Amazon S3 buckets in the same AWS region (e.g., `eu-north-1`)[cite: 1]. Ensure public access is completely disabled for all buckets to maintain security[cite: 1]. Configure the staging bucket to send event notifications to AWS Lambda upon object creation[cite: 1].

### 2. Database Configuration
Create two on-demand Amazon DynamoDB tables[cite: 1]:
* **Table 1:** `UserRiskScores` (Partition Key: `UserId` of type String)[cite: 1].
* **Table 2:** `ArisScanLogs` (Partition Key: `FileName` of type String)[cite: 1].

### 3. Compute & IAM Configuration
* Create an AWS Lambda function with the Python 3.14 runtime[cite: 1].
* **IAM Role:** Assign an execution role that adheres strictly to the principle of least privilege[cite: 1]. The role must grant specific Read/Write/Delete permissions to the designated S3 buckets and `PutItem`/`UpdateItem`/`GetItem` permissions for the DynamoDB tables[cite: 1].
* **Environment Variables:** Store the external threat intelligence key securely in a Lambda environment variable named `VT_API_KEY` to prevent credential exposure in the source code[cite: 1].
* **Code Deployment:** Paste the `RiskScannerBrain` Python logic into the function and deploy[cite: 1].

## Future Enhancements
Based on the project's architectural roadmap, planned future enhancements include[cite: 1]:
* **Machine Learning Integration:** Implementing AI-driven anomaly detection to identify zero-day threats and sophisticated behavioral patterns that bypass signature-based intelligence[cite: 1].
* **Multi-Cloud Support:** Extending the middleware architecture to maintain consistent security policies across AWS, Microsoft Azure, and Google Cloud environments[cite: 1].
* **Dynamic Sandboxing:** Executing highly suspicious files in an isolated sandbox environment for deep behavioral analysis[cite: 1].

## Step-by-Step Implementation Guide

### Step 1: AWS Environment Preparation
* Log into the AWS Management Console[cite: 1].
* Set your AWS Region to **Europe (Stockholm) `eu-north-1`** to ensure all services communicate seamlessly[cite: 1].

### Step 2: Configure Amazon S3 (Storage Layer)
* Navigate to the S3 Dashboard and create three General Purpose buckets[cite: 1]:
  1. `staging-dmz-aris-7788` (The temporary staging zone)[cite: 1].
  2. `main-brain-aris-7788` (The trusted zone for safe files)[cite: 1].
  3. `quarantine-vault-aris-7788` (The isolated zone for malicious files)[cite: 1].
* Disable public access for all buckets to minimize security risks[cite: 1].

### Step 3: Configure Amazon DynamoDB (Database Layer)
* Navigate to DynamoDB and create two new on-demand tables[cite: 1]:
  1. **Table Name:** `UserRiskScores` | **Partition Key:** `UserId` (Type: String)[cite: 1].
  2. **Table Name:** `ArisScanLogs` | **Partition Key:** `FileName` (Type: String)[cite: 1].
* Ensure both tables show a status of "Active"[cite: 1].

### Step 4: Configure AWS Lambda & IAM (Processing Layer)
* Navigate to the IAM Dashboard and create a new execution role (e.g., `RiskScannerBrain-role`) granting access to the S3 buckets, DynamoDB tables, and CloudWatch logging services[cite: 1].
* Navigate to AWS Lambda and click **Create function**[cite: 1].
* **Function Name:** `RiskScannerBrain`[cite: 1].
* **Runtime:** Python 3.14[cite: 1].
* **Architecture:** x86_64[cite: 1].
* Attach the IAM role you created in the previous step[cite: 1].
* Paste the Python logic into the code source editor and deploy[cite: 1].

### Step 5: Integrate Threat Intelligence API
* Create a free account on VirusTotal and generate a personal API key[cite: 1].
* In your AWS Lambda function, navigate to the **Configuration** tab, then **Environment variables**[cite: 1].
* Add a new variable with the Key: `VT_API_KEY` and paste your VirusTotal API key as the Value[cite: 1].

### Step 6: Configure the Event Trigger
* In the Lambda function overview, click **Add trigger**[cite: 1].
* Select **S3** as the source[cite: 1].
* Choose the `staging-dmz-aris-7788` bucket[cite: 1].
* Set the Event type to **All object create events** and save[cite: 1]. This ensures that any file upload instantly initiates the processing workflow[cite: 1].

### Step 7: System Testing & Validation
* **Test 1 (Safe File):** Upload a benign `.txt` file to `staging-dmz-aris-7788`[cite: 1]. Verify that it is automatically moved to `main-brain-aris-7788` and logged as "SAFE" in the `ArisScanLogs` DynamoDB table[cite: 1].
* **Test 2 (Malicious File):** Create a file containing the standard EICAR anti-malware test string (with a benign file name) and upload it to the staging bucket[cite: 1]. Verify that it is moved to `quarantine-vault-aris-7788`, logged as "RISKY", and the user's risk score in `UserRiskScores` increases by 10 points[cite: 1].
## License
This project is licensed under the Apache License 2.0.
