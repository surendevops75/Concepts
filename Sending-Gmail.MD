# Send Gmail from Linux Using msmtp

This guide explains how to configure a Linux server to send emails through Gmail using **msmtp**.

`msmtp` is a lightweight SMTP client that can be used by shell scripts, cron jobs, monitoring tools, and automation workflows to send email notifications.

Common use cases include:

- Server Health Alerts
- Backup Notifications
- Cron Job Reports
- Monitoring Alerts
- Deployment Notifications
- Custom Shell Scripts

---

# Architecture Overview

```text
Linux Server
      │
      ▼
    msmtp
      │
      ▼
 Gmail SMTP Server
      │
      ▼
 Recipient Inbox
```

Example:

```text
Shell Script
     │
     ▼
 msmtp
     │
     ▼
 smtp.gmail.com
     │
     ▼
 info@example.com
```

---

# Prerequisites

Before configuring Gmail SMTP:

- Gmail Account
- 2-Factor Authentication (2FA) Enabled
- Google App Password
- Internet Access

---

# Step 1: Install msmtp

Install the SMTP client:

```bash
sudo dnf install msmtp -y
```

Verify installation:

```bash
msmtp --version
```

Expected Output:

```text
msmtp version x.x.x
```

Purpose:

```text
Send Emails
SMTP Authentication
Script Integration
```

---

# Step 2: Enable Two-Factor Authentication

Google requires App Passwords for SMTP access.

Open:

```text
Google Account
    ↓
Security
    ↓
2-Step Verification
```

Enable:

```text
2 Factor Authentication (2FA)
```

Without 2FA:

```text
App Passwords cannot be generated
```

---

# Step 3: Create Google App Password

Open:

```text
https://myaccount.google.com/apppasswords
```

Create a new App Password.

Example:

```text
abcd efgh ijkl mnop
```

Important:

```text
Use App Password
Do NOT use Gmail Account Password
```

---

# Why App Passwords?

Google blocks direct SMTP authentication using your normal password.

Instead:

```text
Gmail Password
        ❌
Not Supported

App Password
        ✅
Supported
```

Benefits:

- More secure
- Revocable anytime
- Limited scope access

---

# Step 4: Create Global msmtp Configuration

Create:

```bash
sudo vi /etc/msmtprc
```

Add:

```conf
defaults

auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-bundle.crt
logfile        /var/log/msmtp.log

account        gmail

host           smtp.gmail.com
port           587

from           your_email@gmail.com
user           your_email@gmail.com
password       your_app_password

account default : gmail
```

---

# Configuration Explanation

## Authentication

```conf
auth on
```

Enables SMTP authentication.

Required by Gmail.

---

## TLS Encryption

```conf
tls on
```

Encrypts communication between:

```text
Linux Server
      │
      ▼
 Gmail SMTP
```

Protects:

- Credentials
- Email Content

---

## Trusted Certificates

```conf
tls_trust_file
```

Used to verify Gmail SSL certificates.

Helps prevent:

```text
Man-In-The-Middle Attacks
```

---

## Log File

```conf
logfile /var/log/msmtp.log
```

Stores email activity.

Useful for:

- Troubleshooting
- Audit Logs
- Delivery Verification

---

## Gmail SMTP Host

```conf
host smtp.gmail.com
```

Official Gmail SMTP server.

---

## SMTP Port

```conf
port 587
```

Port 587 uses:

```text
STARTTLS
```

Recommended by Google.

---

## Sender Address

```conf
from your_email@gmail.com
```

Email sender shown to recipients.

Example:

```text
alerts@example@gmail.com
```

---

## Username

```conf
user your_email@gmail.com
```

Gmail account used for authentication.

---

## Password

```conf
password your_app_password
```

Google App Password.

Example:

```text
abcd efgh ijkl mnop
```

Never use:

```text
Your Gmail Login Password
```

---

## Default Account

```conf
account default : gmail
```

Makes Gmail the default SMTP account.

No need to specify account each time.

---

# Step 5: Configure File Permissions

Set configuration file permissions:

```bash
sudo chmod 644 /etc/msmtprc
```

Verify:

```bash
ls -l /etc/msmtprc
```

Purpose:

```text
Allow all users to read configuration
```

---

# Step 6: Create Log File

Create log file:

```bash
sudo touch /var/log/msmtp.log
```

Set permissions:

```bash
sudo chmod 666 /var/log/msmtp.log
```

Verify:

```bash
ls -l /var/log/msmtp.log
```

Purpose:

```text
Track email delivery
Capture SMTP errors
```

---

# Step 7: Send Test Email

Test email sending:

```bash
{
echo "To: $TO_ADDRESS"
echo "Subject: $SUBJECT"
echo "Content-Type: text/html"
echo ""
echo "$EMAIL_BODY"
} | msmtp "$TO_ADDRESS"
```

---

# Example Test

```bash
TO_ADDRESS="info@example.com"

SUBJECT="Server Alert"

EMAIL_BODY="<h2>Server is Healthy</h2>"

{
echo "To: $TO_ADDRESS"
echo "Subject: $SUBJECT"
echo "Content-Type: text/html"
echo ""
echo "$EMAIL_BODY"
} | msmtp "$TO_ADDRESS"
```

---

# HTML Email Support

Because:

```bash
Content-Type: text/html
```

You can send:

```html
<h1>Alert</h1>
<p>Server CPU Usage Exceeded Threshold</p>
```

Example Result:

```text
Rich HTML Email
```

instead of plain text.

---

# Verify Email Delivery

Check logs:

```bash
tail -f /var/log/msmtp.log
```

Successful example:

```text
host=smtp.gmail.com
status=sent
```

---

# Troubleshooting

## Authentication Failed

Error:

```text
authentication failed
```

Possible Causes:

- Wrong App Password
- 2FA Not Enabled

---

## Connection Refused

Error:

```text
cannot connect to smtp.gmail.com
```

Check:

```bash
telnet smtp.gmail.com 587
```

Possible causes:

- Firewall
- Network Restrictions

---

## Certificate Errors

Error:

```text
TLS certificate verification failed
```

Verify:

```bash
ca-certificates package installed
```

---

# Complete Email Flow

```text
Shell Script
      │
      ▼
Generate Email
      │
      ▼
msmtp
      │
      ▼
smtp.gmail.com
      │
      ▼
Google Authentication
      │
      ▼
Recipient Inbox
```

---

# Real DevOps Use Cases

## Monitoring Alerts

Send:

```text
CPU Alerts
Memory Alerts
Disk Alerts
```

---

## Backup Notifications

Notify:

```text
Backup Success
Backup Failure
```

---

## Deployment Notifications

Send:

```text
Deployment Started
Deployment Completed
Deployment Failed
```

---

## Cron Job Reports

Email:

```text
Daily Reports
Automation Results
Scheduled Tasks
```

---

## Security Alerts

Notify:

```text
Failed Logins
Unauthorized Access
Certificate Expiry
```

---

# Best Practices

✅ Use Google App Passwords

✅ Enable TLS

✅ Store logs separately

✅ Test SMTP connectivity

✅ Use HTML emails for better readability

✅ Rotate App Passwords periodically

---

# Security Recommendations

❌ Do not store Gmail login passwords

❌ Do not commit `/etc/msmtprc` to Git

❌ Do not expose App Passwords publicly

✅ Use App Passwords only

✅ Restrict access to configuration files

---

# Benefits of msmtp

- Lightweight
- Easy Configuration
- Script Friendly
- Gmail Compatible
- HTML Email Support
- Automation Ready

---

# Why This Setup Is Important

Email notifications are a critical part of infrastructure automation.

Using msmtp with Gmail allows Linux servers to send:

- Monitoring Alerts
- Deployment Notifications
- Backup Reports
- Automation Results

This is a common requirement for:

- DevOps Engineers
- Linux Administrators
- Platform Engineers
- Site Reliability Engineers (SRE)
- Cloud Engineers