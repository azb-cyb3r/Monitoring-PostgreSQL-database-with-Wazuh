## Configuration

This section describes the configuration of PostgreSQL monitoring with Wazuh. A PostgreSQL database is deployed on an Ubuntu endpoint, PostgreSQL activity logging is enabled, and the Wazuh agent is configured to collect and forward PostgreSQL logs to the Wazuh server. Custom Wazuh decoders and rules are then configured to identify database activities such as database/table creation, deletion, insertion, and updates.

---

## Ubuntu Endpoint

### 1. Configure the PostgreSQL Repository

Add the PostgreSQL APT repository:

```bash
sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

Import the PostgreSQL repository signing key:

```bash
sudo wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

### 2. Update Packages

```bash
sudo apt-get update
```

### 3. Install PostgreSQL

```bash
sudo apt-get -y install postgresql
```

### 4. Locate the PostgreSQL Configuration File

Run:

```bash
sudo -u postgres psql -c 'SHOW config_file;'
```

Example output:

```text
              config_file
-----------------------------------------
 /etc/postgresql/18/main/postgresql.conf
(1 row)

```

The PostgreSQL configuration file in this example is:

```text
/etc/postgresql/18/main/postgresql.conf
```

### 5. Enable PostgreSQL Statement Logging

Open the PostgreSQL configuration file:

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

Locate `log_statement` and configure it as:

```text
log_statement = 'mod'
```
<img width="1460" height="737" alt="dbmod" src="https://github.com/user-attachments/assets/fbb99c60-7b5c-486c-8ab8-836a0d39a004" />

This enables logging of database modification statements, including operations such as:

* `CREATE`
* `DROP`
* `INSERT`
* `UPDATE`

### 6. Restart PostgreSQL

Apply the configuration changes:

```bash
sudo systemctl restart postgresql
```

### 7. Locate the PostgreSQL Log File

Run:

```bash
sudo pg_lsclusters
```

Example output:

```text
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```

The PostgreSQL log file is:

```text
/var/log/postgresql/postgresql-18-main.log
```

---

## Configure Wazuh Agent

Edit the Wazuh agent configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following configuration inside the `<ossec_config>` block:

```xml
<!-- Collect PostgreSQL logs -->
<localfile>
    <log_format>syslog</log_format>
    <location>/var/log/postgresql/postgresql-18-main.log</location>
</localfile>
```
<img width="1691" height="682" alt="dbagent" src="https://github.com/user-attachments/assets/b5772895-213e-47de-9687-2b890df0bc7f" />

This configuration instructs the Wazuh agent to monitor the PostgreSQL log file and forward the collected events to the Wazuh manager.

Restart the Wazuh agent:

```bash
sudo systemctl restart wazuh-agent
```

---

# Wazuh Server

The Wazuh server requires custom decoders and rules to properly identify PostgreSQL database activities.

## 1. Create the PostgreSQL Decoder

Create the decoder file:

```bash
sudo touch /var/ossec/etc/decoders/postgresql_decoders.xml
```

Edit the file:

```bash
sudo nano /var/ossec/etc/decoders/postgresql_decoders.xml
```

Add the PostgreSQL decoders:

```xml
<decoder name="postgresql">
    <prematch type="pcre2">(?i)statement:</prematch>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\d+:\d+:\d+.\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+create\s+database\s+([^"\s;]+)</regex>
    <order>db_user,database</order>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\d+:\d+:\d+.\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+drop\s+database\s+([^"\s;]+)</regex>
    <order>db_user,database</order>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\d+:\d+:\d+.\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+create\s+table\s+([^"\s;]+)</regex>
    <order>db_user,db_table</order>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\d+:\d+:\d+.\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+drop\s+table\s+([^"\s;]+)</regex>
    <order>db_user,db_table</order>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+insert\s+into\s+([^"\s;]+)</regex>
    <order>db_user,db_table</order>
</decoder>

<decoder name="postgresql_child">
    <parent>postgresql</parent>
    <regex type="pcre2">(?i)\d+-\d+-\d+\s+\S+\s+\[\d+\]\s+(\S+\s+\S+)@\S+\s+LOG:\s+statement:\s+update\s+([^"\s;]+)</regex>
    <order>db_user,db_table</order>
</decoder>
```
<img width="1917" height="926" alt="db decoder" src="https://github.com/user-attachments/assets/6c01cc3b-1207-4c0d-8fc8-672ac656ba3a" />

> **Note:** The decoder patterns should match the exact PostgreSQL log format generated by your installed PostgreSQL version. Test the decoders with your real log entries before using them in production.

---

# 2. Create Wazuh Detection Rules

Create the rules file:

```bash
sudo touch /var/ossec/etc/rules/postgresql_rules.xml
```

Edit it:

```bash
sudo nano /var/ossec/etc/rules/postgresql_rules.xml
```

Add the detection rules:

```xml
<group name="postgresql,">

    <!-- Detect PostgreSQL logs without generating an alert -->
    <rule id="100080" level="0">
        <decoded_as>postgresql</decoded_as>
        <description>No alerts.</description>
    </rule>

    <!-- Database creation -->
    <rule id="100081" level="4">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)create database</match>
        <description>
            A database $(database) has been created by the user $(db_user).
        </description>
    </rule>

    <!-- Database deletion -->
    <rule id="100082" level="6">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)drop database</match>
        <description>
            A database $(database) has been deleted by the user $(db_user).
        </description>
        <mitre>
            <id>T1485</id>
        </mitre>
    </rule>

    <!-- Table creation -->
    <rule id="100083" level="4">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)create table</match>
        <description>
            A table $(db_table) has been created by the user $(db_user).
        </description>
    </rule>

    <!-- Table deletion -->
    <rule id="100084" level="6">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)drop table</match>
        <description>
            A table $(db_table) has been deleted by the user $(db_user).
        </description>
        <mitre>
            <id>T1485</id>
        </mitre>
    </rule>

    <!-- INSERT operation -->
    <rule id="100085" level="4">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)insert into</match>
        <description>
            New values have been inserted into table $(db_table) by user $(db_user).
        </description>
        <mitre>
            <id>T1565.001</id>
        </mitre>
    </rule>

    <!-- UPDATE operation -->
    <rule id="100086" level="4">
        <if_sid>100080</if_sid>
        <match type="pcre2">(?i)update</match>
        <description>
            Table $(db_table) has been updated by the user $(db_user).
        </description>
        <mitre>
            <id>T1565.001</id>
        </mitre>
    </rule>

</group>
```
<img width="1917" height="897" alt="db rule" src="https://github.com/user-attachments/assets/db1de2b6-bdd1-4560-bd8a-d27f3e586fb8" />

---

# 3. Rule Description

| Rule ID  | Level | Detection                 |
| -------- | ----: | ------------------------- |
| `100080` |     0 | PostgreSQL log — no alert |
| `100081` |     4 | Database created          |
| `100082` |     6 | Database deleted          |
| `100083` |     4 | Table created             |
| `100084` |     6 | Table deleted             |
| `100085` |     4 | Data inserted             |
| `100086` |     4 | Table updated             |

The level `0` rule acts as a parent/base rule and does not generate a dashboard alert. The subsequent rules generate alerts when specific PostgreSQL operations are detected.

---

# 4. Restart Wazuh Manager

After adding the decoder and rules, restart the Wazuh manager:

```bash
sudo systemctl restart wazuh-manager
```

You can also verify the configuration before restarting:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

If the configuration is valid, generate PostgreSQL activity and verify that the events appear in Wazuh.

---

# 5. Testing

After completing the configuration, perform controlled database operations such as:

```sql
CREATE DATABASE testdb;

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

INSERT INTO users (username)
VALUES ('testuser');

UPDATE users
SET username = 'updateduser'
WHERE id = 1;

DROP TABLE users;

DROP DATABASE testdb;
```
<img width="1371" height="502" alt="db3" src="https://github.com/user-attachments/assets/eedeffd8-18c7-40a2-a988-d642ba3f6aec" />

Then verify:

```text
PostgreSQL
    ↓
PostgreSQL log
    ↓
Wazuh Agent
    ↓
Wazuh Manager
    ↓
Decoder
    ↓
Custom Rule
    ↓
Wazuh Alert
    ↓
Wazuh Dashboard
```
See alerts in Wazuh Dashboard
<img width="1917" height="732" alt="db" src="https://github.com/user-attachments/assets/b8dfe886-6b14-47cb-8059-385f4c672d0e" />
Let's inside of log
<img width="955" height="846" alt="db2" src="https://github.com/user-attachments/assets/cf566915-922b-4ddd-80af-afae1e9ba1f2" />



