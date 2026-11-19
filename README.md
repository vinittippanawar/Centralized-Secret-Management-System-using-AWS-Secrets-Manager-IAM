

# 🔐 Centralized Secret Management System — AWS Secrets Manager & IAM

<div align="center">

### No hardcoded credentials. No exposed passwords. Centralized secret access.

*Securely store application credentials in AWS Secrets Manager and retrieve them from an EC2 instance using IAM roles and least-privilege access.*

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazonaws&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.9-blue?style=for-the-badge&logo=python&logoColor=white)
![Boto3](https://img.shields.io/badge/Boto3-AWS%20SDK-yellow?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-Least%20Privilege-green?style=for-the-badge)
![Secrets Manager](https://img.shields.io/badge/AWS-Secrets%20Manager-orange?style=for-the-badge)

</div>

---

## 📑 Table of Contents

- [🔴 The Problem](#-the-problem)
- [🟢 The Solution](#-the-solution)
- [Architecture](#-architecture)
- [Project Components](#-project-components)
- [Repository Layout](#-repository-layout)
- [Prerequisites](#-prerequisites)
- [🚀 Implementation](#-implementation)
- [IAM Security](#-iam-security)
- [📸 Proof It Works](#-proof-it-works)
- [Security Improvements](#-security-improvements)
- [Why Secret Rotation Matters](#-why-secret-rotation-matters)
- [Limitations](#-limitations)
- [Future Improvements](#-future-improvements)
- [Conclusion](#-conclusion)

---

## 🔴 The Problem

Applications frequently require sensitive information such as:

- Database usernames and passwords
- API keys
- Authentication tokens
- Application credentials

Storing these credentials directly inside source code, configuration files, or Git repositories creates serious security risks.

If credentials are exposed, unauthorized users may gain access to databases, cloud services, or sensitive application resources.

Hardcoded credentials also make password updates difficult because application code must be modified and redeployed whenever credentials change.

---

## 🟢 The Solution

This project implements a centralized secret management system using **AWS Secrets Manager, IAM, EC2, Python, and Boto3**.

Instead of hardcoding credentials into the application:

- 🔐 Credentials are securely stored in AWS Secrets Manager.
- 🖥️ The application runs on an EC2 instance.
- 🛡️ EC2 uses an IAM role instead of static AWS access keys.
- 🔑 IAM permissions restrict access to a specific secret.
- 🐍 Python retrieves the secret using the Boto3 SDK.
- 🔒 Passwords are not displayed by the application.

This approach improves credential security, access control, and maintainability.

---

## 🗺️ Architecture

```text
                   ┌──────────────────────────┐
                   │   AWS Secrets Manager   │
                   │                        │
                   │  db-credentials        │
                   │  Username              │
                   │  Password             │
                   └────────────┬───────────┘
                                │
                                │ GetSecretValue
                                │
                     ┌──────────▼───────────┐
                     │     AWS IAM Role    │
                     │                     │
                     │ Least Privilege    │
                     │ Secret Access Only  │
                     └──────────┬──────────┘
                                │
                                │ Temporary Credentials
                                │
                   ┌────────────▼────────────┐
                   │      Amazon EC2         │
                   │                        │
                   │  Python Application    │
                   │  Boto3 SDK            │
                   └────────────┬───────────┘
                                │
                                ▼
                   ┌────────────────────────┐
                   │  Secret Retrieved     │
                   │  Password Masked     │
                   │  Application Success │
                   └────────────────────────┘
```

### 🔄 Workflow

1. Credentials are created and stored in AWS Secrets Manager.
2. An IAM policy grants permission to retrieve the specific secret.
3. The policy is attached to an IAM role.
4. The IAM role is attached to the EC2 instance.
5. The Python application runs on EC2.
6. Boto3 uses the EC2 role's temporary credentials.
7. The application retrieves the secret using `GetSecretValue`.
8. The application processes the credentials without displaying the password.

---

## 🧩 Project Components

| Component | Purpose |
|---|---|
| **Amazon EC2** | Hosts and executes the Python application |
| **AWS Secrets Manager** | Centralized storage for sensitive credentials |
| **AWS IAM** | Controls access to secrets |
| **IAM Role** | Provides temporary credentials to EC2 |
| **IAM Policy** | Restricts access to the required secret |
| **Python** | Implements secret retrieval logic |
| **Boto3** | Python SDK for interacting with AWS services |
| **AWS CLI** | Used for validation and testing |

---

## 📂 Repository Layout

```text
Internship_project_set-6/
│
├── application/
│   └── app.py
│
├── iam/
│   └── secrets-manager-policy.json
│
├── screenshots/
│   └── secret-retrieval.png
│
└── README.md
```

---

## ✅ Prerequisites

| Requirement | Purpose |
|---|---|
| AWS Account | Access AWS cloud services |
| Amazon EC2 | Host the application |
| AWS IAM | Configure roles and permissions |
| AWS Secrets Manager | Store application credentials |
| AWS CLI | Verify AWS configuration |
| Python 3 | Run the application |
| Boto3 | Access AWS Secrets Manager from Python |
| Linux | Application hosting environment |

---

## 🚀 Implementation

### 1️⃣ Create a Secret in AWS Secrets Manager

Create a secret in AWS Secrets Manager.

**Secret Name:**

```text
internship-project-6/db-credentials
```

**Secret Type:** Other type of secret

Example secret structure:

```json
{
  "username": "appuser",
  "password": "YOUR_DEMO_PASSWORD"
}
```

The secret is stored centrally instead of being hardcoded into the application.

---

### 2️⃣ Create a Least-Privilege IAM Policy

Create an IAM policy that allows the EC2 instance to retrieve only the required secret.

**Policy Name:**

```text
InternshipProject6-SecretsAccess
```

**Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRetrieveProjectSecret",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:ap-south-1:YOUR_AWS_ACCOUNT_ID:secret:internship-project-6/db-credentials-*"
    }
  ]
}
```

### 🔒 IAM Security

This policy follows the principle of least privilege.

It allows:

- `secretsmanager:GetSecretValue`

It does not grant unrestricted access to all secrets or unnecessary permissions to modify secrets.

---

### 3️⃣ Create and Attach the IAM Role

Create an IAM role for EC2.

**Role Name:**

```text
InternshipProject6-EC2-SecretsRole
```

Configure:

- Trusted entity: AWS service
- Use case: EC2
- Attach policy: `InternshipProject6-SecretsAccess`

Attach this role to the EC2 instance.

The EC2 instance can now access AWS services using temporary credentials supplied through the IAM role.

No AWS access keys need to be stored inside the application.

---

### 4️⃣ Configure the Application Environment

Connect to the EC2 instance and create the project directory.

```bash
mkdir -p ~/Internship_project_set-6/application
cd ~/Internship_project_set-6
```

Create a Python virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Install Boto3:

```bash
pip install boto3
```

---

### 5️⃣ Python Application

The application uses Boto3 to retrieve credentials from AWS Secrets Manager.

**File:** `application/app.py`

```python
import boto3
import json
from botocore.exceptions import ClientError

AWS_REGION = "ap-south-1"
SECRET_NAME = "internship-project-6/db-credentials"


def get_secret():
    client = boto3.client(
        "secretsmanager",
        region_name=AWS_REGION
    )

    try:
        response = client.get_secret_value(
            SecretId=SECRET_NAME
        )

        secret = json.loads(response["SecretString"])

        print("Successfully retrieved secret from AWS Secrets Manager!")
        print(f"Username: {secret['username']}")
        print("Password: ******** (hidden)")

        return secret

    except ClientError as error:
        print("AWS error while retrieving secret:")
        print(error.response["Error"]["Message"])
        return None

    except (KeyError, json.JSONDecodeError) as error:
        print("Error parsing secret:", error)
        return None


def main():
    print("====================================")
    print(" Centralized Secret Management Demo")
    print("====================================")

    credentials = get_secret()

    if credentials:
        print("\nApplication started successfully!")
        print("Credentials retrieved securely.")
    else:
        print("\nApplication failed to retrieve credentials.")


if __name__ == "__main__":
    main()
```

---

### 6️⃣ Run the Application

Navigate to the application directory:

```bash
cd ~/Internship_project_set-6/application
```

Activate the virtual environment:

```bash
source ~/Internship_project_set-6/venv/bin/activate
```

Run the application:

```bash
python app.py
```

Expected output:

```text
====================================
 Centralized Secret Management Demo
====================================

Successfully retrieved secret from AWS Secrets Manager!
Username: appuser
Password: ******** (hidden)

Application started successfully!
Credentials retrieved securely.
```

---

## 📸 Proof It Works

### EC2 IAM Role Verification

The EC2 instance uses the attached IAM role to obtain temporary AWS credentials.

The identity can be verified using:

```bash
aws sts get-caller-identity
```

This confirms that the AWS CLI is operating using the EC2 role identity.

---

### Secrets Manager Retrieval

The secret can be retrieved using the AWS CLI:

```bash
aws secretsmanager get-secret-value \
  --secret-id internship-project-6/db-credentials \
  --region ap-south-1 \
  --query SecretString \
  --output text
```

The application also retrieves the secret using Boto3.

---

### Python Application Execution

Running:

```bash
python app.py
```

Successfully retrieves the secret and confirms that the application has received the credentials.

Password output is masked.

![Secret Retrieval](screenshots/secret-retrieval.png)

---

## 🛡️ Security Improvements

| Security Practice | Implementation |
|---|---|
| **Centralized Secrets** | Credentials stored in AWS Secrets Manager |
| **No Hardcoded Credentials** | Secrets are not embedded in application source code |
| **IAM Least Privilege** | Permission limited to retrieving the project secret |
| **IAM Role for EC2** | Uses temporary AWS credentials instead of static access keys |
| **Encrypted Storage** | Secrets Manager encrypts secrets at rest |
| **Password Masking** | Application does not print the actual password |
| **Access Control** | IAM determines which resources the application can access |
| **Centralized Management** | Credentials can be managed independently of application code |

---

## 🔄 Why Secret Rotation Matters

Secret rotation is the process of periodically changing credentials such as passwords, API keys, and database authentication information.

### 🚨 Risks of Not Rotating Secrets

- Credentials may remain valid for a long time.
- Exposed passwords can be reused by unauthorized users.
- Old credentials increase the risk of unauthorized access.
- A compromised credential may continue to work until manually revoked.
- Applications and services may depend on credentials that are difficult to update.

### ✅ Benefits of Secret Rotation

- Reduces the time window during which compromised credentials remain valid.
- Limits the impact of credential exposure.
- Improves security compliance.
- Reduces dependency on long-lived credentials.
- Supports automated credential lifecycle management.

### 🔄 Rotation Workflow

```text
          ┌──────────────────────┐
          │  Existing Password  │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │  Generate New Secret │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Update Target System │
          │  e.g. Database      │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Update Secrets       │
          │ Manager Version      │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Application Retrieves│
          │ New Credentials     │
          └──────────────────────┘
```

> **Note:** Automatic secret rotation was not enabled in this implementation. The project demonstrates centralized secret storage and secure retrieval. Rotation is a recommended future enhancement.

---

## ⚠️ Limitations

- Automatic secret rotation is not configured.
- The demonstration retrieves credentials but does not connect to a production database.
- The application is designed as a secret retrieval demonstration.
- Production deployments should implement credential rotation, monitoring, logging, and application-specific credential refresh handling.

---

## 🚀 Future Improvements

- Implement automatic secret rotation using AWS Lambda.
- Integrate Secrets Manager with an actual database.
- Configure CloudWatch monitoring and logging.
- Add secret access auditing.
- Implement credential refresh logic.
- Use separate secrets for development, staging, and production.
- Integrate the secret management workflow into a CI/CD pipeline.

---

## 🎯 Conclusion

This project demonstrates how AWS Secrets Manager and IAM can be used to implement centralized and secure credential management.

By replacing hardcoded credentials with centrally managed secrets, assigning permissions through IAM roles, and retrieving credentials programmatically using Boto3, the system improves application security and reduces the risks associated with exposed credentials.

The implementation follows important cloud security practices such as **least privilege, temporary credentials, centralized secret storage, and credential protection**.

---

<div align="center">

### 🔐 Built with AWS · Python · Boto3 · IAM

**Secure credentials. Controlled access. Better cloud security.**

</div>
