# Security Manager Journal

This journal documents security decisions, assumptions, challenges, and
lessons learned while designing the Rigel Meridian Systems (RMS)
cybersecurity program.

The purpose is to document not only what was implemented, but why security
decisions were made and how those decisions changed as new technical and
business requirements were identified.

---

# Day 1 — Initial Security Assessment

## Executive Briefing

I joined Rigel Meridian Systems as the organization's first dedicated
Security Manager.

During my initial meeting with executive leadership and the Systems
Administrator, I was asked to identify where I believed the organization
was most exposed and where the security program should begin.

RMS employs approximately 75 people, with 22 employees regularly working
remotely.

Because remote employees routinely access company resources from home
networks, hotels, airports, and other external networks, I identified
remote access and endpoint security as an immediate priority.

## Initial Remote Workforce Findings

The initial environment included:

- Company-owned Windows laptops
- VPN access to internal resources
- VPN authentication using Active Directory usernames and passwords
- No mandatory MFA for remote access
- Windows Defender enabled locally
- Limited centralized visibility into endpoint security
- Limited visibility into patch compliance
- Several employees with local administrator accounts
- Employees regularly connecting from untrusted external networks

The combination of remote access, password-based authentication, and
limited endpoint visibility creates opportunities for credential theft,
phishing, malware, and compromised endpoints to provide an attacker with
access to RMS resources.

---

## Identity Modernization Proposal

My initial recommendation was to require MFA and investigate migrating
RMS identity management from traditional on-premises Active Directory to
Microsoft Entra ID.

The proposed architecture would use Microsoft Intune to centrally manage
company endpoints and enforce security policies.

I identified several capabilities that could improve RMS security,
including:

- Multi-factor authentication
- Conditional Access
- Phishing-resistant authentication
- Passwordless authentication
- Centralized endpoint management
- Device compliance policies
- Privileged identity controls

### Architecture Challenge

During design review, the Systems Administrator raised concerns about
completely replacing the existing Active Directory environment.

Existing servers, applications, file shares, service accounts, and
authentication mechanisms may depend on Active Directory services.

Removing Active Directory without identifying these dependencies could
cause significant business disruption.

### Revised Decision

Before retiring the existing Active Directory environment, RMS will
perform dependency discovery to determine which systems rely on it.

A hybrid identity architecture may be required during the migration
period.

On-premises Active Directory should only be retired after its technical
and business dependencies have been identified and appropriately
migrated or replaced.

### Lesson Learned

Cloud identity should not automatically be treated as a direct
replacement for traditional Active Directory.

Security modernization must account for existing infrastructure and
business dependencies before legacy systems can safely be retired.

---

# Authentication Security

## Initial MFA Policy

My initial proposal required MFA for all employees.

In a credential-compromise scenario, possession of an employee's
password should not be sufficient to access company resources.

I initially proposed automatically locking an employee account following
a failed MFA attempt and notifying security personnel.

### Architecture Challenge — Account Lockout DoS

During design review, a weakness in this policy was identified.

If an attacker possesses a valid employee password, the attacker could
intentionally generate failed MFA attempts.

If every failed MFA attempt automatically locked the employee account,
the security control itself could potentially be abused to deny service
to legitimate employees.

This would create an operational problem and provide an attacker with a
method of intentionally triggering account lockouts.

### Revised Decision

A failed MFA attempt should deny the authentication attempt but should
not automatically disable the employee's account.

Additional signals should be evaluated to determine whether security
investigation or account containment is necessary.

Examples include:

- Abnormal geographic location
- Unknown or unmanaged devices
- Device compliance status
- Repeated authentication attempts
- Unusual sign-in behavior
- Identity risk indicators

Suspicious authentication activity should generate security telemetry
and alerts for investigation.

---

# Phishing-Resistant Authentication

Another concern identified during design review was MFA fatigue.

Push-based authentication can potentially be abused by repeatedly
sending authentication requests to a user in an attempt to convince the
user to approve one.

To reduce the effectiveness of credential phishing and MFA fatigue,
I proposed adopting phishing-resistant authentication.

## Standard Employees

The proposed authentication architecture for standard employees
includes:

- Microsoft Entra ID
- Microsoft Intune-managed company devices
- Windows Hello for Business for workstation authentication
- Passkey-based authentication for protected company resources
- Conditional Access policies
- Device compliance requirements

## Privileged Employees

Privileged accounts represent a substantially greater organizational
risk because compromise could provide access to infrastructure,
security controls, or sensitive company data.

I therefore proposed stronger authentication requirements for
privileged users.

Privileged administrators will use physical FIDO2 security keys for
administrative authentication.

Administrative authentication policies will be separated from standard
employee authentication policies.

---

# Open Security Question — Account Recovery

The adoption of phishing-resistant authentication introduces an
important recovery problem.

If an employee loses access to their passkey or security key, RMS must
provide a method of restoring access without creating a weaker
authentication path that attackers can exploit.

Using SMS verification alone could undermine phishing-resistant
authentication by exposing the recovery process to risks such as social
engineering, SIM swapping, and help-desk impersonation.

A formal identity-verification and credential-recovery procedure must
therefore be designed before the authentication architecture is
considered complete.

**Status:** Open

---

# First Security Project

The initial assessment resulted in the creation of the first RMS
security project:

**SEC-001 — Remote Workforce Security**

The project will establish a security baseline for the 22 company
laptops that routinely operate outside the RMS corporate network.

The project must address:

- Data encryption
- Least privilege
- Centralized endpoint management
- Patch compliance
- Centrally monitored malware protection
- Managed-device requirements for company resources
- Phishing-resistant authentication
- Endpoint security logging
- Support for employees traveling outside the corporate network

Implementation will begin using a test Windows 11 workstation before
security policies are deployed more broadly.
