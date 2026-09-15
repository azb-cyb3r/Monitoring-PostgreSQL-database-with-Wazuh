# PostgreSQL Database Monitoring with Wazuh

## 📌 Summary

This project demonstrates the monitoring of PostgreSQL database activity using **Wazuh SIEM**. PostgreSQL logs are collected and analyzed by Wazuh to provide visibility into authentication activity, database errors, and potentially suspicious events.

## 🎯 Objectives

* Integrate PostgreSQL with Wazuh
* Collect PostgreSQL security logs
* Monitor database authentication activity
* Detect failed authentication attempts
* Create custom Wazuh detection rules
* Investigate security alerts

## 🛠️ Technologies

* **Wazuh SIEM**
* **PostgreSQL**
* **Ubuntu Linux**
* **Wazuh Agent**
* **Wazuh Dashboard**

## 🏗️ Architecture

```text
┌──────────────────┐
│    PostgreSQL    │
│     Database     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ PostgreSQL Logs  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Wazuh Agent    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Wazuh Manager  │
│                  │
│ Decoders + Rules │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Wazuh Dashboard  │
│     Alerts       │
└──────────────────┘
```

## 🔍 Monitoring

The project monitors PostgreSQL events including:

* Failed authentication attempts
* Successful authentication
* Invalid database users
* Database connection activity
* PostgreSQL errors
* Suspicious authentication behavior

## ⚙️ Log Collection

PostgreSQL generates database activity logs which are collected by the Wazuh agent and forwarded to the Wazuh manager for analysis.

```text
PostgreSQL Event
       ↓
PostgreSQL Log
       ↓
Wazuh Agent
       ↓
Wazuh Manager
       ↓
Decoder
       ↓
Detection Rule
       ↓
Wazuh Alert
```

## 🚨 Detection

Custom Wazuh rules can be used to identify security-relevant PostgreSQL events.

Example use case:

**Multiple failed PostgreSQL authentication attempts**

```text
Failed Login
     ↓
Failed Login
     ↓
Failed Login
     ↓
Wazuh Detection Rule
     ↓
Security Alert
```

## 🔎 Investigation

When an alert is generated, the following information can be analyzed:

* Timestamp
* Username
* Source information
* Database name
* Authentication result
* Event/message
* Number of failed attempts

The collected information can be used to determine whether the activity is normal or potentially malicious.

## 📸 Evidence

### PostgreSQL Logs

![PostgreSQL Logs](screenshots/postgresql-logs.png)

### Wazuh Alert

![Wazuh Alert](screenshots/wazuh-alert.png)

### Wazuh Dashboard

![Wazuh Dashboard](screenshots/wazuh-dashboard.png)

## 📊 Results

* PostgreSQL logs successfully collected by Wazuh
* Database authentication activity monitored
* Security events displayed in the Wazuh dashboard
* Custom detection rules tested
* PostgreSQL events investigated from a SOC perspective

## 🧠 Skills Demonstrated

* SIEM implementation
* PostgreSQL monitoring
* Linux log analysis
* Wazuh configuration
* Detection engineering
* Security event investigation
* Alert analysis
* SOC monitoring

## 🔐 Disclaimer

All testing was performed in a controlled laboratory environment using systems owned or authorized for security testing.
