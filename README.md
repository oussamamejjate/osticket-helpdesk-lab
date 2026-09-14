# osticket-helpdesk-lab
Helpdesk ticketing system built from scratch on Windows Server 2019 using osTicket, IIS, PHP and MySQL

# 🎫 osTicket Helpdesk Lab

A fully functional IT helpdesk environment built from scratch on Windows Server 2019,
simulating real-world Tier 1 support scenarios end to end — from ticket submission to
resolution — integrated with an existing Active Directory domain.

> This project is a continuation of my
> [Active Directory Home Lab](https://github.com/oussamamejjate/active-directory-home-lab)

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Windows Server 2019 | Hosted the osTicket web server |
| IIS (Internet Information Services) | Web server for osTicket |
| PHP 8.2 | Runtime for osTicket application |
| MySQL 8.0 | Database for tickets, users, and settings |
| osTicket | Open-source helpdesk ticketing system |
| Active Directory | Domain users used for ticket simulation |
| Oracle VirtualBox | Hypervisor |

---

## 🌐 Environment

| Machine | OS | IP | Role |
|---|---|---|---|
| DC-01 | Windows Server 2019 | 172.16.0.1 | Domain Controller |
| osTicket-Server | Windows Server 2019 | 172.16.0.3 | Web / Helpdesk Server |
| CLIENT-01 | Windows 10 Pro | 172.16.0.100 | Domain-joined client |

---

## ⚙️ Installation & Configuration

### What Was Installed on the osTicket Server

- **IIS** with CGI enabled for PHP support
- **PHP 8.2** configured with required extensions:
  mysqli, gd, mbstring, intl, imap, opcache, apcu
- **MySQL 8.0** with a dedicated `osticket` database and user
- **osTicket** deployed under `C:\inetpub\wwwroot\osticket`

### Troubleshooting During Installation

Several issues were encountered and resolved during setup:

**PHP FastCGI crash (HTTP 500.0)**: Caused by missing Visual C++
Redistributable. Fixed by installing `vc_redist.x64.exe`.

**IIS configuration error (HTTP 500.19 — 0x8007000d)**: Caused by
missing URL Rewrite module. Fixed by installing the IIS URL Rewrite
extension from iis.net.

**MySQL Access Denied for osticket user**: Caused by MySQL 8.0's
new default authentication method conflicting with PHP's mysqli
extension. Fixed by recreating the user with
`mysql_native_password` authentication.

**Installation timeout**: Fixed by increasing `max_execution_time`
and `max_input_time` to 120 seconds in php.ini.

![osTicket Installed](screenshots/installation/osticket-installed.png)

---

## 🗂️ osTicket Configuration

### Help Topics Created

| Help Topic | Priority |
|---|---|
| Account Locked Out | High |
| Password Reset | Normal |
| Cannot Access Network Share | Normal |
| Software Issue | Normal |

![Help Topics](screenshots/configuration/help-topics.png)

### Departments Created

- IT Support
- Systems Administration

![Departments](screenshots/configuration/departments.png)

### Agent Created

A helpdesk agent account was created and assigned to the IT Support
department with Full Access permissions. The admin account handles
ticket assignment and triage while the agent account handles
responses and resolution.

![Agent Created](screenshots/configuration/agent-created.png)

---

## 🎯 Helpdesk Scenarios Simulated

### Scenario 1 — Account Locked Out

**Situation:** A domain user submits a ticket reporting they cannot
log in because their account is locked after too many failed
password attempts.

**Resolution workflow:**
1. User submits ticket via the osTicket portal from the client VM
2. Admin assigns ticket to helpdesk agent with High priority
3. Admin adds internal note documenting the issue
4. Technician opens ADUC on the DC, verifies bad password count
   and last bad password timestamp on the user's Account tab
5. Account unlocked via ADUC
6. Agent replies to user confirming resolution and closes ticket

![Ticket Submitted](screenshots/scenario-1-account-lockout/ticket-submitted.png)
![Ticket Assigned](screenshots/scenario-1-account-lockout/ticket-assigned.png)
![Internal Note](screenshots/scenario-1-account-lockout/internal-note.png)
![ADUC Account Locked Details](screenshots/scenario-1-account-lockout/aduc-account-locked-details.png)
![ADUC Account Locked Button](screenshots/scenario-1-account-lockout/aduc-account-locked-button.png)
![Ticket Reply](screenshots/scenario-1-account-lockout/ticket-reply.png)
![Ticket Resolved](screenshots/scenario-1-account-lockout/ticket-resolved.png)

---

### Scenario 2 — Password Reset

**Situation:** A domain user submits a ticket reporting they have
forgotten their password and cannot log into their account.

**Resolution workflow:**
1. User submits ticket via the osTicket portal
2. Admin assigns ticket to helpdesk agent with Normal priority
3. Admin adds internal note documenting the planned action
4. Technician resets the password via ADUC with
   'User must change password at next logon' checked
5. User logs in with temporary password and is prompted to
   set a new password that meets the domain password policy
6. Agent replies to user confirming resolution and closes ticket

![Ticket Submitted](screenshots/scenario-2-password-reset/ticket-submitted.png)
![Ticket Assigned](screenshots/scenario-2-password-reset/ticket-assigned.png)
![Internal Note](screenshots/scenario-2-password-reset/internal-note.png)
![ADUC Password Reset](screenshots/scenario-2-password-reset/aduc-password-reset.png)
![Windows Change Password](screenshots/scenario-2-password-reset/windows-change-password.png)
![Ticket Resolved](screenshots/scenario-2-password-reset/ticket-resolved.png)

---

### Scenario 3 — Cannot Access Network Share

**Situation:** A domain user can open a shared folder on the DC
but receives an access denied error when trying to create or
save files inside it.

**Resolution workflow:**
1. Shared folder `CompanyShare` created on the DC with
   Domain Users set to Read-only permissions
2. Access denied error reproduced on the client VM
3. User submits ticket via the osTicket portal
4. Admin assigns ticket to helpdesk agent with Normal priority
5. Admin adds internal note documenting the permission issue
6. Technician updates share permissions on the DC; granted
   Full Control to the specific user only, applying the
   principle of least privilege rather than modifying
   permissions for all Domain Users
7. User successfully creates a file in the share
8. Agent replies to user confirming resolution and closes ticket

![Share Permissions Read Only](screenshots/scenario-3-network-share/share-permissions-readonly.png)
![Access Denied Client](screenshots/scenario-3-network-share/access-denied-client.png)
![Ticket Submitted](screenshots/scenario-3-network-share/ticket-submitted.png)
![Ticket Assigned](screenshots/scenario-3-network-share/ticket-assigned.png)
![Internal Note](screenshots/scenario-3-network-share/internal-note.png)
![Permissions Updated](screenshots/scenario-3-network-share/permissions-updated.png)
![File Created Success](screenshots/scenario-3-network-share/file-created-success.png)
![Ticket Resolved](screenshots/scenario-3-network-share/ticket-resolved.png)

---

## 💡 Key Lessons Learned

- MySQL 8.0 uses a new default authentication method that conflicts
  with PHP's mysqli; recreating the user with
  `mysql_native_password` resolves this
- IIS requires the URL Rewrite module to process osTicket's
  web.config rewrite rules; without it the site returns a
  500.19 error
- Locked out users can still submit tickets via the osTicket
  guest portal since it requires no domain authentication;
  solving a real operational gap in helpdesk design
- The principle of least privilege applies to share permissions;
  granting access to a specific user rather than all Domain Users
  is the correct approach in a real environment

---

## 🔗 Related Project

This lab is built on top of my Active Directory environment:
[github.com/oussamamejjate/active-directory-home-lab](https://github.com/oussamamejjate/active-directory-home-lab)
