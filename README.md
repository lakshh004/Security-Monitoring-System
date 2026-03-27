<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Security Monitoring System

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-monitoring)

**Author:** Lakshya Purohit  
**Email:** theridiculousboy@gmail.com

---

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_reghtjy)

---

## Introducing Today's Project!

In this project, I will demonstrate how to set up a monitoring system in AWS using CloudTrail, CloudWatch and SNS. I'm doing this project to learn how security and monitoring services in AWS work, plus a working system that actually sends us emails too!

### Tools and concepts

Services I used were CloudWatch, CloudTrail and SNS. I also used Secrets Manager and S3 buckets. 
Key concepts I learnt includes Secure Secret Management, CloudTrail vs CloudWatch, Real-time Security Alerting, Activity Monitoring and Auditing and Metric Filtering.

### Project reflection

This project took me approximately 3 hours. The most challenging part was troubleshooting why the email wasn't delivering in the first test. It was most rewarding to compare CloudTrail and CloudWatch and to know when to use which service and why!

---

## Create a Secret

Secrets Manager is AWS' security service for storing secrets i.e. database credentials, account IDs, API keys.... anything that is sensitive information that would cause damage or trouble if it got leaked and shouldn't be lying around in the code.

To set up for my project, I created a secret called TopSecretInfo in Secrets Manager This secret is a string that contains a top secret of me that I need 3 coffees a day to function :)

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_o5p6q7r8)

---

## Set Up CloudTrail

CloudTrail is a monitoring service - it's used to track events and activities in our AWS account. These logs are very helpful for security (i.e. detecting suspicious activity), compliance (i.e. proving that you're following the rules for something) and troubleshooting (i.e. identifying what happened/changed if something breaks).

CloudTrail events include types like management, insight, data and network activity events. In this project, I have set up my trail to track Management events because accessing a secret falls into that category.

### Read vs Write Activity

Read API activity involves accessing, reading, opening a resource. Write API activity involves creating, updating, deleting a resource. For this project, we ticked both to learn about both type of activities, but we really only need the Write activity.

---

## Verifying CloudTrail

I retrieved the secret in two ways: First through the Secrets Manager console where I could easily just select a "Retrieve secrets value" button. Second using the AWS CLI i.e. running a get-secret-value in CloudShell.


To analyze my CloudTrail events i.e. to see the event where I get my secret's value, I visited the event history in CloudTrail. I found that there was a GetSecretValue event tracked regardless of whether I did it over the CLI or over the console. This tells us that CloudTrail can definitely see and track when someone opens my Secrets Manager key.

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_s8t9u0v1)

---

## CloudWatch Metrics

CloudWatch Logs is a monitoring service that brings together logs from other AWS services (including CloudTrail) to help us analyze and create alarms for It's important for monitoring because you get to create insights and get alerted based on events that happens in your account.

CloudTrail's Event History is useful for quickly reading events that happened in the last 90 days, while CloudWatch Logs are better for combining and analysing logs from different sources, storing, accessing logs for longer than 90 days and advanced filtering. 

A CloudWatch metric is a specific way that we count or track events that are in a log group. When setting up a metric, the metric value represents how we increment or 'count' an event when it passes our filters (in my case, I want to increment the metric value by 1 whenever my secret is accessed. Default value is used when the event we're tracking does not occur!

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_a9b0c1d2)

---

## CloudWatch Alarm

A CloudWatch alarm is a feature and alert system in CloudWatch that's designed to "go off" i.e. indicate when certain conditions have been met in my log group. I set my CloudWatch alarm threshold to be about how many times the GetSecretValue event happens in a 5 minute period so the alarm will trigger when the average number of times is above 1.

I created an SNS topic along the way. An SNS topic is like a newsletter/broadcast channel that emails, phone numbers, functions, apps can subscribe to, so they get notified when SNS has a new update to share. My SNS topic is set up to send me an email when my secret gets accessed.

AWS requires email confirmation because it would not automatically start emailing addresses that we subscribe to an SNS topic This helps prevent any unwanted subscription for users that do not wish to receive these emails.

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_fsdghstt)

---

## Troubleshooting Notification Errors

To test my monitoring system, I opened and accessed my secret again. The results weren't successful - I didn't get any emails/notifications about my secret getting accessed.

When troubleshooting the notification issues, I investigated every single part of my monitoring system - whether CloudTrail is picking up on events that are happening when I access my secret, whether CloudTrail is sending logs to CloudWatch, whether the filter is accidentally rejecting the correct events, whether or not the alarm gets triggered, whether the triggered alarm sends an email or not.

I initially didn't receive an email before because CloudWatch was configured to use the wrong threshold - instead of calculating the AVERAGE number of times a secret was accessed in a time period, it should've been the SUM!

---

## Success!

To validate that my monitoring system can successfully detect and alert when my secret is accessed, I checked my secret's value one more time. I received an email within 1-2 minutes of the event. The alarm in CloudWatch is also in "In Alarm" state.

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_ageraergearge)

---

## Comparing CloudWatch with CloudTrail Notifications

In a project extension, I enabled SNS notification delivery in CloudTrail because this lets me evaluate CloudTrail vs CloudWatch for notifying me about events like my secret getting accessed. 

After enabling CloudTrail SNS notifications, my inbox was very quickly filled with emails from SNS (as it was notified by CloudTrail). In terms of the usefulness of these emails, I thought I'm receiving a LOT (it's a little overwhelming) and the logs themselves don't show what happened in terms of management events that occurred. I only saw that new logs have been stored in my bucket.

![Image](http://learn.nextwork.org/thoughtful_blue_happy_trout/uploads/aws-security-monitoring_d7e8f9g0)

---

---
