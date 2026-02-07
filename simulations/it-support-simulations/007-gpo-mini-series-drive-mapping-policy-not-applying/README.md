# Simulation 007 — GPO Mini Series (Part 1): Drive Mapping Policy Not Applying

## Summary
In this simulation, I diagnosed and resolved a Group Policy issue where a Finance user was no longer receiving a mapped network drive. The issue was isolated to a single user and traced back to missing Active Directory security group membership used for Group Policy drive mapping.

A structured troubleshooting approach was used to validate connectivity, confirm scope, review Group Policy application, and restore proper access through security group reassignment.

---

## Ticket / Scenario

**User Impact:**  
Finance user unable to access mapped Finance network drive required for daily work.

**Reported Issue:**  
Mapped Finance drive missing and inaccessible.

**Environment:**
- Windows endpoint (domain-joined)
- Windows Server Domain Controller
- Active Directory security groups
- Group Policy drive mapping
- NTFS permissions
- Network file share

---

## Symptoms

- Finance drive no longer mapped on user workstation  
- UNC path to Finance share inaccessible under affected user account  
- Issue isolated to a single user  

---

## Troubleshooting Process

1. Verified basic network connectivity  
2. Tested access to the Finance share using UNC path under the affected user account  
3. Confirmed mapped drive was missing from the workstation  
4. Tested access using another Finance user account to determine scope  
5. Verified Group Policy application using `gpresult`  
6. Reviewed Active Directory security group membership for affected user  
7. Identified missing membership in the security group used for drive mapping  

---

## Root Cause

The affected user was no longer a member of the Active Directory security group required for the Finance drive Group Policy mapping, causing the drive to be automatically removed.

---

## Fix Implemented

- Re-added the user to the appropriate Active Directory security group  
- Ran `gpupdate /force`  
- Signed out and signed back in to refresh Group Policy  

---

## Verification

- Confirmed Group Policy drive mapping was applied  
- Verified Finance drive was successfully remapped  
- Confirmed access to the Finance share was restored  

---

## LinkedIn Post & Video

https://www.linkedin.com/posts/caleb-mallek-4aa10b390_itsupport-helpdesk-activedirectory-activity-7425710175350738944-ByBJ?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGAf2QIBpfh8fvg-z4gxhvhFLqdO3YSSshg

---

## Key Takeaways

- Group Policy drive mappings often rely on security group–based item-level targeting  
- Least-privilege design can include automatic cleanup when group membership changes  
- Testing with another user account helps quickly isolate scope  
- Understanding Group Policy application and cleanup behavior is critical for real-world troubleshooting  
