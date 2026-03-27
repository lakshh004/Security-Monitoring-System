# 🔐 AWS Security Monitoring & Alerting System

[![AWS](https://img.shields.io/badge/AWS-CloudTrail-orange)](https://aws.amazon.com)
[![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-blue)](https://aws.amazon.com/cloudwatch)
[![SNS](https://img.shields.io/badge/SNS-Alerting-green)](https://aws.amazon.com/sns)
[![Status](https://img.shields.io/badge/Alerts-Live-brightgreen)]()

---

## Project Overview

Built a real-time security monitoring system on AWS that detects sensitive API
activity and sends automated email alerts within minutes of a security event.

Instead of manually checking CloudTrail logs, everything is event-driven —
CloudTrail captures the activity, CloudWatch filters for the exact event,
an alarm fires, and SNS delivers an email notification automatically.

The system detects:
- Unauthorized or unexpected secret access via Secrets Manager
- GetSecretValue API calls regardless of whether made via console or CLI
- Sustained access patterns above defined thresholds

---

## Architecture

<!-- SCREENSHOT: Architecture diagram -->

---

## Tech Stack

| Service | Role |
|---|---|
| AWS CloudTrail | Captures all API activity across the account |
| AWS CloudWatch | Ingests CloudTrail logs, applies metric filters, triggers alarms |
| AWS SNS | Sends email notifications when alarm fires |
| AWS Secrets Manager | Stores the secret being monitored |
| AWS S3 | Stores CloudTrail log files |

---

## How It Works

### 1. Secret Creation
Created a secret in Secrets Manager as the monitored resource.
Any access to this secret — whether via the AWS console or CLI — generates
a `GetSecretValue` event in CloudTrail.
<img width="530" height="174" alt="image" src="https://github.com/user-attachments/assets/88e7862b-9c6f-4c23-b5d0-4149bf0d792c" />


---

### 2. CloudTrail Setup
Configured a CloudTrail trail to capture Management Events across the account.
Management events cover API calls like `GetSecretValue`, `CreateUser`,
`DeleteBucket` — actions that affect AWS resources rather than data inside them.

Both Read and Write activity is tracked, since secret access is a Read event.
<img width="529" height="171" alt="image" src="https://github.com/user-attachments/assets/7b518d6e-c235-4630-b12f-8450447e424b" />


---

### 3. Verifying CloudTrail Captures the Event
Retrieved the secret twice — once via the Secrets Manager console and once
via AWS CLI using `get-secret-value` in CloudShell.

Both methods generated a `GetSecretValue` event in CloudTrail's event history,
confirming that CloudTrail captures secret access regardless of how it's done.

<img width="518" height="288" alt="image" src="https://github.com/user-attachments/assets/16e6e044-f675-4ea1-9020-155807c0b109" />


---

### 4. CloudWatch Metric Filter
Sent CloudTrail logs to a CloudWatch Log Group, then created a metric filter
to count `GetSecretValue` events specifically.

The metric increments by 1 each time the event is detected.
Default value is 0 when the event does not occur in a given period.

<img width="529" height="274" alt="image" src="https://github.com/user-attachments/assets/bcfb6d8b-3c9c-40a0-a0ce-77e625b85547" />


---

### 5. CloudWatch Alarm
Set an alarm to trigger when the SUM of `GetSecretValue` events exceeds 1
within a 5-minute window.

Key distinction: configured as SUM not AVERAGE — using AVERAGE caused the
alarm to not fire correctly during initial testing since a single event
averaged across the period fell below the threshold.

Alarm is linked to an SNS topic that sends an email notification on state
change to `IN ALARM`.

<!-- SCREENSHOT: CloudWatch alarm configured with SNS action -->

---

### 6. SNS Email Notification
Created an SNS topic and subscribed an email address to it.
AWS requires manual email confirmation before SNS will deliver notifications
to prevent unwanted subscriptions.

<img width="529" height="276" alt="image" src="https://github.com/user-attachments/assets/8fab673d-65cc-4915-aed3-454a4b783c9f" />


---

## Results

After fixing the AVERAGE → SUM threshold issue, accessed the secret once more.
Received an email notification within 1-2 minutes of the `GetSecretValue` event.
CloudWatch alarm moved to `IN ALARM` state as expected.

<img width="529" height="276" alt="image" src="https://github.com/user-attachments/assets/c3534f8e-7280-40dc-ac28-9733982b5052" />

<img width="526" height="267" alt="image" src="https://github.com/user-attachments/assets/1febb2c6-cdf9-4bee-b985-013a746ebbb7" />


---

## CloudTrail vs CloudWatch for Alerting

Tested both approaches side by side:

**CloudWatch (metric filter + alarm):** Targeted, low noise. Only fires when
the specific event threshold is crossed. Email contains actionable context.

**CloudTrail direct SNS:** High volume, low signal. Inbox flooded immediately
as SNS fired for every log file delivery to S3, not for specific events.
Logs only indicated new files were stored, not what activity occurred.

**Conclusion:** CloudWatch metric filters are the correct approach for
event-specific security alerting. CloudTrail SNS is better suited for
audit log delivery pipelines, not real-time alerting.

<img width="534" height="226" alt="image" src="https://github.com/user-attachments/assets/c32dd852-04b3-4162-b98f-c353b19504d3" />
<img width="527" height="199" alt="image" src="https://github.com/user-attachments/assets/d01711c1-1d26-430f-867e-5931d8b2005b" />

---

## Key Learnings

- CloudTrail captures API activity across console, CLI, and SDK uniformly
- CloudWatch metric filters enable precise, low-noise security alerting
- SUM vs AVERAGE matters significantly when configuring alarm thresholds
- SNS requires confirmed subscriptions before delivering notifications
- Event-driven alerting reduces MTTD without any manual log inspection
