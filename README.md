# 🔐 AWS Security Monitoring & Alerting System

[![AWS](https://img.shields.io/badge/AWS-CloudTrail-orange)](https://aws.amazon.com)
[![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-blue)](https://aws.amazon.com/cloudwatch)
[![SNS](https://img.shields.io/badge/SNS-Alerting-green)](https://aws.amazon.com/sns)
[![Status](https://img.shields.io/badge/Alerts-Live-brightgreen)]()

---

## Project Overview

Built a real-time security monitoring system on AWS that detects sensitive API
activity and sends automated email alerts within minutes of a security event.

Instead of manually checking logs, everything is event-driven — CloudTrail
captures the activity, CloudWatch filters for the exact event, an alarm fires,
and SNS delivers an email notification automatically.

Reduced mean time to detection (MTTD) of unauthorized secret access from
~10-15 minutes to under 2 minutes using event-driven metric filters.

---

## Tech Stack

| Service | Role |
|---|---|
| AWS CloudTrail | Captures all API activity across the account |
| AWS CloudWatch | Ingests logs, applies metric filters, triggers alarms |
| AWS SNS | Sends email notifications when alarm fires |
| AWS Secrets Manager | Stores the secret being monitored |
| AWS S3 | Stores CloudTrail log files |

---

## How It Works

### 1. Secret Creation
Created a secret in Secrets Manager as the monitored resource. Any access to
this secret — via the AWS console or CLI — generates a `GetSecretValue` event
in CloudTrail.

**Secret created in Secrets Manager**

<img width="525" height="166" alt="image" src="https://github.com/user-attachments/assets/5be10897-14b8-4252-a398-e7f1ccf48961" />

---

### 2. Verifying CloudTrail Captures the Event
Retrieved the secret via both the Secrets Manager console and AWS CLI using
`get-secret-value` in CloudShell. Both methods generated a `GetSecretValue`
event in CloudTrail's event history — confirming CloudTrail captures secret
access regardless of how it's triggered.

**GetSecretValue event in CloudTrail event history**

<img width="535" height="162" alt="image" src="https://github.com/user-attachments/assets/b5bfdff7-6e63-4d0a-8aec-31f795843fee" />

---

### 3. CloudWatch Metric Filter
Sent CloudTrail logs to a CloudWatch Log Group and created a metric filter
to count `GetSecretValue` events specifically. The metric increments by 1
each time the event is detected.

**CloudWatch metric filter configured**

<img width="531" height="296" alt="image" src="https://github.com/user-attachments/assets/5640b915-3088-49af-818e-799bfcc8821e" />

---

### 4. CloudWatch Alarm + SNS
Set an alarm to trigger when the SUM of `GetSecretValue` events exceeds 1
within a 5-minute window. Linked to an SNS topic that emails on alarm state
change.

Important: configured as SUM not AVERAGE — using AVERAGE caused the alarm
to not fire correctly during initial testing since a single event averaged
across the period fell below the threshold. Fixing this was the key
troubleshooting step in the project.

**CloudWatch alarm configured with SNS action**

<img width="530" height="273" alt="image" src="https://github.com/user-attachments/assets/ed2f182f-ba3c-4d81-a451-6d2f4271cacb" />

**SNS topic and confirmed subscription**

<img width="543" height="274" alt="image" src="https://github.com/user-attachments/assets/f39cf12a-5ea1-4001-bdf4-11b4590112cd" />

---

## Results

After fixing the threshold, accessed the secret once more and received an
email notification within 1-2 minutes. CloudWatch alarm moved to IN ALARM
state as expected.

**Email notification received — CloudWatch alarm in IN ALARM state**

<img width="536" height="233" alt="image" src="https://github.com/user-attachments/assets/c6192a12-0a1b-4a06-8f2a-86bc801a21f4" />

---

## CloudTrail vs CloudWatch for Alerting

Tested both approaches to understand the difference:

**CloudWatch metric filter + alarm** — targeted and low noise. Only fires
when the specific event threshold is crossed. Email contains actionable context.

**CloudTrail direct SNS** — high volume, low signal. Inbox flooded immediately
as SNS fired for every log file delivery to S3, not for specific events.

Conclusion: CloudWatch metric filters are the right approach for event-specific
security alerting. CloudTrail SNS is better suited for audit log delivery,
not real-time alerting.

**CloudTrail SNS inbox flood vs CloudWatch targeted alert**

<img width="543" height="206" alt="image" src="https://github.com/user-attachments/assets/9e4deb45-254c-480e-be44-2458e85be1c9" />

---

## Key Learnings

- CloudTrail captures API activity uniformly across console, CLI, and SDK
- CloudWatch metric filters enable precise, low-noise security alerting
- SUM vs AVERAGE threshold configuration has significant real-world impact
- Event-driven alerting reduces MTTD without manual log inspection
- CloudTrail SNS and CloudWatch serve fundamentally different alerting purposes
