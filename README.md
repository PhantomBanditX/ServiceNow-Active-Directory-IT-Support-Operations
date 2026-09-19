<p align="center">
  <img width="1000" src="https://github.com/user-attachments/assets/d5dd96d9-fe9b-465f-bde0-13b795ed37d7" />
</p>

# ServiceNow + Active Directory IT Support Operations

> Hands-on IT support operations environment integrating ServiceNow ticketing workflows with Active Directory user administration, PowerShell automation, Group Policy, shared-folder access, and end-user validation in Microsoft Azure.

This environment was built to practice realistic **Service Desk / IT Support workflows** in a controlled Azure environment. The focus is not advanced systems administration; it is the support process an entry-level technician should be able to explain clearly:

`Receive request → verify the issue → make the authorized change → validate → document → resolve`

The environment includes four complete ServiceNow workflows:

1. **Account Lockout Incident**
2. **New Employee Provisioning Request**
3. **Shared Folder Access Incident**
4. **Password Reset Incident**

---

## Contents

- [Objectives](#objectives)
- [Technologies Used](#technologies-used)
- [Infrastructure Deployment](#infrastructure-deployment)
- [Active Directory Structure](#active-directory-structure)
- [PowerShell Bulk User Provisioning](#powershell-bulk-user-provisioning)
- [Group Policy - Account Lockout](#group-policy---account-lockout)
- [ServiceNow Workflow Model](#servicenow-workflow-model)
- [Workflow 1 - Account Lockout Incident](#workflow-1---account-lockout-incident)
- [Workflow 2 - New Employee Provisioning Request](#workflow-2---new-employee-provisioning-request)
- [Workflow 3 - Shared Folder Access Incident](#workflow-3---shared-folder-access-incident)
- [Workflow 4 - Password Reset Incident](#workflow-4---password-reset-incident)
- [Skills Demonstrated](#skills-demonstrated)
- [Key Takeaways](#key-takeaways)
- [Environment Scope](#environment-scope)

---

# Objectives

- Build a Windows domain environment in Microsoft Azure.
- Configure Active Directory Domain Services and DNS.
- Join a Windows 11 workstation to the domain.
- Organize users with departmental Organizational Units and security groups.
- Automate bulk test-user creation with PowerShell.
- Configure and validate an account-lockout Group Policy.
- Practice common ServiceNow incident and request workflows.
- Perform basic account, access, and authentication support tasks.
- Validate changes from the end-user side before closing tickets.
- Document work using ServiceNow work notes and resolution notes.

---

# Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Azure | Hosted the virtual machines and virtual network |
| Windows Server 2025 Datacenter | Domain Controller (`DC-1`) |
| Windows 11 Pro | Domain-joined workstation (`Client-1`) |
| Active Directory Domain Services | Domain identity and account management |
| Active Directory Users and Computers | User, OU, group, password, and lockout administration |
| DNS | Domain name resolution for Active Directory |
| Group Policy | Account-lockout control and client policy application |
| PowerShell | Bulk provisioning and account verification |
| ServiceNow | Incident and service-request tracking |
| SMB / NTFS | Departmental shared-folder access |
| Windows Command Line | Authentication and access validation |

---

# Infrastructure Deployment

The environment uses two Azure virtual machines on the same virtual network:

| System | Role |
|---|---|
| `DC-1` | Windows Server 2025 Domain Controller |
| `Client-1` | Windows 11 domain-joined client |

**Active Directory domain:** `stratovainc.com`  
**NetBIOS domain:** `STRATOVAINC`

The Domain Controller was given a stable Azure private IP allocation. `Client-1` was then configured to use the Domain Controller for DNS so it could locate the Active Directory domain and related services.

<details>
<summary><strong>Evidence: Azure networking, AD DS deployment, DNS, and domain join</strong></summary>

### DC-1 private IP configuration
A stable private IP was configured for the Domain Controller's Azure network interface so domain clients could consistently reach DNS and Active Directory services.

<p align="center">
  <img width="900" alt="DC-1 static private IP configuration" src="https://github.com/user-attachments/assets/6ad302d8-1970-4f27-bedd-fad4e3588e90" />
</p>

### Domain Controller verification
Server Manager confirms the server name `DC-1`, the `stratovainc.com` domain, Windows Server 2025, and active Windows security services.

<p align="center">
  <img width="900" alt="DC-1 domain controller verification" src="https://github.com/user-attachments/assets/9b538080-21e3-400c-a7c4-3cbe855cc827" />)
</p>

### Client-1 custom DNS configuration
The Azure NIC for `Client-1` was configured to use the Domain Controller as its custom DNS server.

<p align="center">
  <img width="900" alt="Client-1 Azure DNS configuration" src="https://github.com/user-attachments/assets/9a25fcfa-e56b-4465-9371-87fc5be8a832" />
</p>

### AD DS prerequisite check
The Active Directory Domain Services Configuration Wizard completed its prerequisite checks before promotion.

<p align="center">
  <img width="900" alt="Active Directory Domain Services prerequisite check" src="https://github.com/user-attachments/assets/8a71127c-e7c3-4c8b-ad9d-07a4970d6eb2" />
</p>

### Active Directory forest verification
PowerShell verification confirms the `stratovainc.com` forest and Windows Server 2025 forest mode.

<p align="center">
  <img width="900" alt="Active Directory forest verification" src="https://github.com/user-attachments/assets/56d62464-c0c9-431e-8b51-711a18301340" />
</p>

### Client-1 to DC-1 connectivity
`Client-1` successfully reached the Domain Controller over the Azure virtual network.

<p align="center">
  <img width="900" alt="Client-1 to DC-1 network connectivity verification" src="https://github.com/user-attachments/assets/3e66abe4-beda-4dc0-a5b2-7a9b3a5fb99d" />
</p>

### Client-1 network and DNS verification
`ipconfig /all` confirms `Client-1` received its Azure network configuration and is using `DC-1` for DNS.

<p align="center">
  <img width="900" alt="Client-1 network and DNS configuration verification" src="https://github.com/user-attachments/assets/161629d5-f6e9-40a2-80eb-cd525ae16e64" />
</p>

### DC-1 to Client-1 connectivity
Connectivity was also validated in the opposite direction from `DC-1` to `Client-1`.

<p align="center">
  <img width="900" alt="DC-1 to Client-1 network connectivity verification" src="https://github.com/user-attachments/assets/dd580d1a-c1ad-4e4b-845b-91db11741646" />
</p>

### AD DS installation
The AD DS server role and management tools were successfully installed before promoting the server to a Domain Controller.

<p align="center">
  <img width="900" alt="Active Directory Domain Services installation completed" src="https://github.com/user-attachments/assets/40844fae-475e-4356-9b2f-84e5070658c4" />
</p>

### Client-1 domain join
Windows confirmed that `Client-1` successfully joined the `stratovainc.com` domain.

<p align="center">
  <img width="900" alt="Client-1 successfully joined to the stratovainc.com domain" src="https://github.com/user-attachments/assets/ed9367ed-4e7b-4a67-89ea-7fd5d28a00a7" />
</p>

### Client-1 domain membership verification
A command-line check confirmed the workstation name and domain membership after the restart.

<p align="center">
  <img width="900" alt="Client-1 domain membership verification" src="https://github.com/user-attachments/assets/9ab06e46-d233-42af-867e-2336817b5e7e" />
</p>

### Domain Controller options
The new forest was configured with DNS and Global Catalog capabilities using Windows Server 2025 functional levels.

<p align="center">
  <img width="900" alt="Windows Server 2025 domain controller configuration options" src="https://github.com/user-attachments/assets/c26309ed-9bfd-43db-8f71-06aa067aa7d6" />
</p>

</details>

---

# Active Directory Structure

The custom company OU structure separates employees, groups, and workstations while keeping the environment easy to understand.

```text
_STRATOVA
├── Employees
│   ├── IT
│   ├── Accounting
│   ├── HR
│   ├── Sales
│   └── Operations
├── Groups
└── Workstations
```

Department security groups were created for role-based access:

```text
SG-IT
SG-Accounting
SG-HR
SG-Sales
SG-Operations
```

Rather than assigning departmental access directly to individual users, the environment uses security-group membership so access can be managed consistently.

<details>
<summary><strong>Evidence: OU and security-group design</strong></summary>

### Organizational Unit structure

<p align="center">
  <img width="900" alt="Active Directory organizational unit structure" src="https://github.com/user-attachments/assets/6623fa0f-21f9-40db-8bbd-ccfa250eb90f" />
</p>

### Department security groups

<p align="center">
  <img width="900" alt="Active Directory departmental security groups" src="https://github.com/user-attachments/assets/6c47fc66-840d-4848-ad0e-bc15d0751598" />
</p>

</details>

---

# PowerShell Bulk User Provisioning

PowerShell was used to provision **1,000 synthetic domain users** for the environment. Users were distributed across the five departments and assigned to the corresponding departmental security groups.

This created a larger test environment for authentication, lockout, password-reset, and access-control exercises without manually creating each account.

<details>
<summary><strong>Evidence: PowerShell automation and validation</strong></summary>

### Bulk user creation
The provisioning script completed creation of 1,000 synthetic users and distributed 200 users to each department.

<p align="center">
  <img width="900" alt="PowerShell bulk provisioning of 1000 Active Directory users" src="https://github.com/user-attachments/assets/6db7f848-00be-4f91-add9-f70cff6d4a31" />
</p>

### Department user distribution
A follow-up check confirmed 200 users in each departmental OU.

<p align="center">
  <img width="750" alt="Active Directory department user distribution verification" src="https://github.com/user-attachments/assets/6d1c4474-61f3-4974-a05e-3261597f78bb" />
</p>

### Department security-group membership
A second validation confirmed 200 members in each department security group.

<p align="center">
  <img width="750" alt="Active Directory security group membership verification" src="https://github.com/user-attachments/assets/e9e3ff2d-a590-4d8e-be56-10755f04ca6f" />
</p>

</details>

---

# Group Policy - Account Lockout

A domain account-lockout policy was configured to create a realistic support scenario.

The policy used:

- **Lockout threshold:** 5 invalid logon attempts
- **Lockout duration:** 10 minutes
- **Reset account lockout counter:** 10 minutes

The policy was then verified from `Client-1` before using it in the ServiceNow account-lockout incident.

<details>
<summary><strong>Evidence: Account-lockout policy configuration and application</strong></summary>

### Group Policy configuration

<p align="center">
  <img width="900" alt="Group Policy account lockout configuration" src="https://github.com/user-attachments/assets/5623d5ac-41ae-46be-bc89-5588c02d3ab5" />
</p>

### Client Group Policy application
`gpresult` confirmed that the account-lockout policy was applied to `Client-1`.

<p align="center">
  <img width="900" alt="Client-1 Group Policy application verification" src="https://github.com/user-attachments/assets/1ffd984e-45ee-4027-a517-2cc7e1952e11" />
</p>

### Domain lockout-policy verification
PowerShell confirmed the domain lockout threshold, duration, and observation window.

<p align="center">
  <img width="900" alt="Active Directory domain account lockout policy verification" src="https://github.com/user-attachments/assets/43cca236-30df-4073-abb4-c3560efc13a8" />
</p>

</details>

---

# ServiceNow Workflow Model

The ServiceNow implementation focuses on the basic ITSM distinction between an **incident** and a **service request**.

### Incident
An incident represents something that should already work but is currently broken.

Examples in this environment:

- Account is locked
- User cannot save to a departmental share
- User forgot a password and cannot sign in

Typical workflow:

```text
New → In Progress → Verify → Remediate → Validate → Resolved
```

### Service Request
A service request represents something new that IT is being asked to provide.

Example in this environment:

- Provision a new employee account and departmental access

Typical workflow:

```text
Service Catalog → REQ → RITM → Fulfillment → Validation → Closed Complete
```

---

# Workflow 1 - Account Lockout Incident

## Scenario

**Madelene Wilkens** reported that her account was locked and she could not sign in to her workstation.

The objective was to verify the lockout, restore access, validate authentication from the client, document the work, and resolve the incident.

## Support Workflow

```text
User reports lockout
        ↓
ServiceNow incident created
        ↓
Account lockout reproduced and verified
        ↓
Account unlocked in Active Directory
        ↓
Lockout state rechecked
        ↓
Authentication tested from Client-1
        ↓
Work notes documented
        ↓
Incident resolved
```

## What I Did

I created the ServiceNow incident using the **Access / Identity → Account Lockout** classification. I reproduced the account lockout, confirmed the account was locked in Active Directory, and verified that the failed-password count matched the configured threshold.

I then unlocked the account in Active Directory Users and Computers, rechecked the account status, and tested authentication again from `Client-1`. After confirming that the user could authenticate successfully, I documented the work in ServiceNow and resolved the incident.

<details>
<summary><strong>Evidence: Account Lockout Incident</strong></summary>

### Incident intake
The incident identifies the caller, category, subcategory, priority, and reported sign-in problem.

<p align="center">
  <img width="900" alt="ServiceNow account lockout incident intake" src="https://github.com/user-attachments/assets/0013d477-2553-41b3-af49-9d4bccf6951a" />
</p>

### Lockout reproduced from Client-1
Windows returned error `1909`, confirming the domain account was locked and could not log on.

<p align="center">
  <img width="900" alt="Account lockout reproduced from Client-1" src="https://github.com/user-attachments/assets/a156f642-5525-4a2e-b8b0-29f8b9374a46" />
</p>

### Lockout verified in Active Directory
PowerShell confirmed `LockedOut = True` and `badPwdCount = 5`.

<p align="center">
  <img width="900" alt="Active Directory account lockout verification" src="https://github.com/user-attachments/assets/5a6afcb4-4adc-496c-ad96-bd1da98097c7" />
</p>

### Account unlock remediation
The ADUC account properties showed that the user was locked out and provided the option to unlock the account.

<p align="center">
  <img width="650" alt="Active Directory account unlock remediation" src="https://github.com/user-attachments/assets/f1a4196c-3f40-4bd5-a0f6-baec524cf074" />
</p>

### Unlock verification
A follow-up PowerShell check confirmed `LockedOut = False` and the bad-password count was cleared.

<p align="center">
  <img width="900" alt="Active Directory account unlock verification" src="https://github.com/user-attachments/assets/76e0efde-a78f-4445-8d78-6102fac3f12e" />
</p>

### Access restored from Client-1
A new command session running as `STRATOVAINC\mwilkens` verified successful authentication after the unlock.

<p align="center">
  <img width="900" alt="Client-1 account access restored after unlock" src="https://github.com/user-attachments/assets/5bfdb474-83f7-49e9-88bf-5b90ce8b1871" />
</p>

### Incident resolved
The ServiceNow ticket was moved to **Resolved** with a solution-provided resolution code and clear resolution notes.

<p align="center">
  <img width="900" alt="ServiceNow account lockout incident resolved" src="https://github.com/user-attachments/assets/2b1c47b0-6e56-4d53-92e6-906050c77957" />
</p>

### Work notes and ticket history
The activity history documents the lockout verification, remediation, validation, and final resolution.

<p align="center">
  <img width="900" alt="ServiceNow account lockout work notes and incident history" src="https://github.com/user-attachments/assets/26418ab2-fffa-4a18-89b7-b8e8ebb26fb8" />
</p>

</details>

### Result

The account lockout was verified, the account was unlocked, and successful domain authentication was confirmed from `Client-1` before the incident was resolved.

**Simple support lesson:** `Verify → Correct → Retest → Document → Resolve`

---

# Workflow 2 - New Employee Provisioning Request

## Scenario

A new employee, **Jordan Blake**, required a domain account and standard access for the **Operations** department.

This was treated as a **service request**, not an incident, because IT was being asked to provide a new account and access rather than repair something that had stopped working.

## Request Details

```text
Employee Name: Jordan Blake
Department: Operations
Start Date: 2026-09-21
Required Access: Standard Operations department access
```

ServiceNow created the request hierarchy used to track fulfillment:

```text
REQ0010001
└── RITM0010003
```

## Support Workflow

```text
Catalog request submitted
        ↓
REQ / RITM reviewed
        ↓
Employee account created in AD
        ↓
Operations department assigned
        ↓
SG-Operations membership added
        ↓
First-logon password control validated
        ↓
Domain authentication validated
        ↓
Request marked complete
```

## What I Did

I submitted the New Employee Provisioning catalog request with the employee's name, department, start date, and required access. I reviewed the resulting request and requested item, then created Jordan Blake's Active Directory account in the **Operations** OU.

I assigned the user to `SG-Operations`, configured a temporary password with a first-logon password-change requirement, and tested the account from `Client-1`. After validating the account, the ServiceNow requested item was completed.

<details>
<summary><strong>Evidence: New Employee Provisioning Request</strong></summary>

### New Employee Provisioning catalog request

<p align="center">
  <img width="900" alt="ServiceNow new employee provisioning request" src="https://github.com/user-attachments/assets/186f2308-fbef-4523-9748-c31ad6fb3e68" />
</p>

### Request submitted
ServiceNow generated `REQ0010001` after the catalog request was submitted.

<p align="center">
  <img width="900" alt="ServiceNow new employee request submitted" src="https://github.com/user-attachments/assets/677d3b2f-9435-40b4-824e-046d0037458a" />
</p>

### Requested Item created
`RITM0010003` contains the submitted employee variables and tracks fulfillment of the request.

<p align="center">
  <img width="900" alt="ServiceNow requested item for new employee provisioning" src="https://github.com/user-attachments/assets/81676c4d-fe63-4918-ab5a-14fa4ebbfcf0" />
</p>

### Active Directory user provisioned
PowerShell verification confirms Jordan Blake's account, employee ID, Operations department, and enabled status.

<p align="center">
  <img width="900" alt="Jordan Blake Active Directory account provisioned" src="https://github.com/user-attachments/assets/a084d7ae-ef24-4122-931c-c3f354fa32d7" />
</p>

### Operations group membership assigned
A follow-up check confirms `jblake` was added to `SG-Operations`.

<p align="center">
  <img width="900" alt="Jordan Blake assigned to SG-Operations security group" src="https://github.com/user-attachments/assets/6f252358-2c3b-439c-92ed-0a2455b8fa2b" />
</p>

### First-logon password requirement validated
Windows confirmed that the temporary password required a password change before normal sign-in.

<p align="center">
  <img width="900" alt="New user first-logon password change requirement validation" src="https://github.com/user-attachments/assets/934292f5-e2fd-43a1-b8b6-c71ba4ad7f30" />
</p>

### Domain authentication validated
After validation, a command session running as `STRATOVAINC\jblake` confirmed successful domain authentication.

<p align="center">
  <img width="900" alt="Jordan Blake domain authentication validation from Client-1" src="https://github.com/user-attachments/assets/226b5ba9-0356-4217-82c8-4f21b87aa350" />
</p>

### Requested Item completed
The ServiceNow requested item was moved to **Closed Complete** after the provisioning work was completed.

<p align="center">
  <img width="900" alt="ServiceNow new employee provisioning request completed" src="https://github.com/user-attachments/assets/4913bb1b-c2be-4711-a9cf-51305d578d23" />
</p>

</details>

### Result

Jordan Blake's domain account was provisioned in the correct department, assigned to the Operations security group, validated from the domain-joined client, and documented through the ServiceNow request workflow.

This workflow demonstrates the practical difference between **fulfilling a request** and **resolving an incident**.

---

# Workflow 3 - Shared Folder Access Incident

## Scenario

Accounting employee **Imelda Waterbury** could open the Accounting shared folder but received an **Access is denied** message when trying to save a file.

The Accounting share used group-based access through:

```text
SG-Accounting
```

## Support Workflow

```text
User reports shared-folder problem
        ↓
ServiceNow incident created
        ↓
Access denial reproduced
        ↓
AD group membership checked
        ↓
SG-Accounting found missing
        ↓
Group membership restored
        ↓
User session refreshed
        ↓
Write access retested
        ↓
Incident documented and resolved
```

## What I Did

I reproduced the user's file-write problem and confirmed that the Accounting share returned **Access is denied**. I then checked Imelda Waterbury's Active Directory group memberships and found that `SG-Accounting` was missing.

I restored the correct group membership, refreshed the user's session, and retested the Accounting share. The user was then able to create a file successfully. I documented the troubleshooting and validation steps in ServiceNow and resolved the incident.

<details>
<summary><strong>Evidence: Shared Folder Access Incident</strong></summary>

### Access denied from Client-1
The user context was verified with `whoami`, and an attempted file write returned **Access is denied**.

<p align="center">
  <img width="900" alt="Accounting shared folder access denied from Client-1" src="https://github.com/user-attachments/assets/da19dfb6-2b70-486d-b034-760c913a4ac2" />
</p>

### ServiceNow incident intake
The incident records the caller, access-related category, priority, and reported shared-folder problem.

<p align="center">
  <img width="900" alt="ServiceNow Accounting shared folder access incident intake" src="https://github.com/user-attachments/assets/be84ecee-4a80-4000-bf9a-46affface4aa" />
</p>

### Missing Accounting group membership
ADUC shows that the user's memberships did not include `SG-Accounting`.

<p align="center">
  <img width="900" alt="Active Directory missing SG-Accounting group membership" src="https://github.com/user-attachments/assets/82091091-d940-4175-b277-e5be1b25caeb" />
</p>

### Accounting group membership restored
The user's membership list now includes `SG-Accounting`.

<p align="center">
  <img width="900" alt="Active Directory SG-Accounting membership restored" src="https://github.com/user-attachments/assets/6eea549d-735a-4258-a6cf-4d81ebf3deef" />
</p>

### File access restored
The user successfully created `access-test.txt` in the Accounting shared folder after the membership change was applied.

<p align="center">
  <img width="900" alt="Accounting shared folder access restored from Client-1" src="https://github.com/user-attachments/assets/297c09d0-8316-48b0-8ec2-0fff4a06926c" />
</p>

### Incident resolved with work notes
The ServiceNow activity history records the missing group, remediation, session refresh, successful validation, and resolution.

<p align="center">
  <img width="900" alt="ServiceNow Accounting shared folder incident resolved" src="https://github.com/user-attachments/assets/e040ac61-86d8-4bb9-8fa0-360157b9f200" />
</p>

</details>

### Result

The access problem was traced to missing departmental security-group membership. Restoring `SG-Accounting` returned the intended write access without granting broader permissions directly to the user.

**Simple support lesson:** check the user's assigned access before changing permissions broadly.

---

# Workflow 4 - Password Reset Incident

## Scenario

HR employee **Carmelia Bruen** reported that she forgot her password and could not sign in to her workstation.

## Support Workflow

```text
User reports forgotten password
        ↓
ServiceNow incident created
        ↓
Account status checked
        ↓
Password reset in Active Directory
        ↓
Password change required at next logon
        ↓
Requirement validated from Client-1
        ↓
Authentication validated after reset
        ↓
Work documented
        ↓
Incident resolved
```

## What I Did

I created the ServiceNow password-reset incident and verified that the user account was active and not locked out. I reset the user's password in Active Directory and configured the account to require a password change at the next logon.

From `Client-1`, Windows confirmed that the temporary password could not be used for normal authentication until the password-change requirement was satisfied. After validation, I confirmed successful domain authentication as `STRATOVAINC\cbruen`.

I documented the work in ServiceNow and resolved the incident.

<details>
<summary><strong>Evidence: Password Reset Incident</strong></summary>

### ServiceNow incident intake
The incident records the caller, Password Reset category, priority, and reported sign-in problem.

<p align="center">
  <img width="900" alt="ServiceNow password reset incident intake" src="https://github.com/user-attachments/assets/3fca4023-f5cb-4d90-a73c-31b93868fcae" />
</p>

### First-logon password-change requirement enforced
Windows returned error `1907`, confirming that the temporary password required a password change before sign-in.

<p align="center">
  <img width="900" alt="Windows password change required before sign-in validation" src="https://github.com/user-attachments/assets/60b7d11b-e77d-4589-8683-5d49d9e6c494" />
</p>

### Successful authentication after reset
A new command session running as `STRATOVAINC\cbruen` confirms successful domain authentication after the reset validation.

<p align="center">
  <img width="900" alt="Successful domain authentication after password reset" src="https://github.com/user-attachments/assets/563f81e6-0004-476d-bbbc-f453bf463e71" />
</p>

### ServiceNow incident resolved
The activity history shows the ticket moving from **New → In Progress → Resolved**, along with the work notes and final resolution notes.

<p align="center">
  <img width="900" alt="ServiceNow password reset incident resolved with work notes" src="https://github.com/user-attachments/assets/02bf4fd0-efe5-42de-9c8a-9f0c90dfda6e" />
</p>

</details>

### Result

The password-reset workflow was documented from intake through validation and resolution. The final ServiceNow record shows the work performed and confirms successful domain authentication from `Client-1`.

---

# Skills Demonstrated

### ServiceNow / ITSM

- Incident creation and categorization
- Caller and issue documentation
- Impact, urgency, and priority awareness
- Incident state progression
- Work notes and activity history
- Resolution codes and resolution notes
- Service Catalog requests
- REQ and RITM tracking
- Request fulfillment and completion
- Difference between incidents and service requests

### Active Directory Support

- Active Directory Users and Computers
- Organizational Units
- Security groups
- Department-based user organization
- Account status verification
- Account lockout and unlock
- Password reset
- First-logon password-change requirements
- Group membership changes
- Domain authentication validation

### Windows / Networking

- Windows Server 2025
- Windows 11 domain join
- DNS configuration for Active Directory
- Basic IP connectivity testing
- Domain membership verification
- Group Policy application verification
- Shared-folder access testing
- Command-line authentication validation

### PowerShell

- Bulk user provisioning
- Department assignment
- Security-group assignment
- Active Directory account verification
- Lockout-status verification
- Membership validation

---

# Key Takeaways

This environment reinforced a repeatable support process:

```text
Understand the issue or request
        ↓
Verify the user and current state
        ↓
Check the relevant account or access setting
        ↓
Make the authorized change
        ↓
Retest from the user's perspective
        ↓
Document the result
        ↓
Resolve or complete the ticket
```

The four workflows also reinforced an important ITSM distinction:

- **Incident:** something that should already work is broken.
- **Service Request:** the user needs IT to provide something new.

The technical change is only part of the support process. The change should also be **validated and documented** before the work is considered complete.

---

# Environment Scope

This is an **independent hands-on environment** built for IT support and cybersecurity training. It uses synthetic users, test systems, and a controlled Azure environment.

The environment is intended to demonstrate practical familiarity with:

- ServiceNow ticketing workflows
- Foundational Active Directory support
- User account administration
- Basic access troubleshooting
- PowerShell automation
- Clear technical documentation
- Validation of changes before ticket resolution

It is not presented as production Systems Administrator experience.

