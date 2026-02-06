# Simulation 006 — Network Share Access & Active Directory Permissions Troubleshooting

## Summary
In this simulation, I diagnosed and resolved a user access issue involving a mapped network drive that stores shared company resources. The problem was caused by missing Active Directory security group membership controlling access to the network share.

A structured, layered troubleshooting approach was used to isolate the issue, verify connectivity, identify the root cause, and restore proper access.

---

## Ticket / Scenario

**User Impact:**  
User unable to access mapped network drive containing shared resources.

**Reported Issue:**  
Mapped drive fails to open and displays access denied.

**Environment:**
- Windows endpoint (domain-joined)
- Windows Server Domain Controller
- Active Directory security groups
- Network file share

---

## Symptoms

- Mapped network drive inaccessible  
- Access denied when attempting to open shared folder  
- UNC path access also restricted  

---

## Troubleshooting Process

1. Verified workstation network connectivity  
2. Tested access via mapped drive  
3. Tested access using UNC path  
4. Confirmed network share existed and was online  
5. Reviewed Active Directory group membership  
6. Identified user was not part of required security group  

---

## Root Cause

The user account was missing membership in the Active Directory security group responsible for granting access to the network share.

---

## Fix Implemented

- Added user to the appropriate AD security group controlling share access  
- Allowed group membership to propagate  
- Retested access to the mapped drive and UNC path  

---

## Verification

- Successfully accessed the network share after group reassignment  
- Confirmed permissions applied correctly  

---

## LinkedIn Post & Video

https://www.linkedin.com/posts/caleb-mallek-4aa10b390_itsupport-helpdesk-activedirectory-activity-7424507909675339776-ngRF

---

## Key Takeaways

- Always verify basic connectivity before diving into permissions  
- Test both mapped drives and UNC paths during access issues  
- Active Directory group membership is commonly used to control file access  
- A layered troubleshooting approach leads to faster root cause identification  

