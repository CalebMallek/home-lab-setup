# Simulation 005 — Network Connectivity Troubleshooting & DNS Resolution

## Summary
In this simulation, I diagnosed and resolved a network-related access issue affecting internal company resources. The walkthrough included reviewing workstation network configuration, verifying connectivity, and identifying a DNS misconfiguration as the root cause preventing proper name resolution in a domain environment.

To reflect realistic service desk troubleshooting practices, layered diagnostic techniques such as connectivity testing and DNS query tools were used to isolate the issue before applying a corrective fix at the endpoint level. The DNS settings were then restored to the domain controller to reestablish proper internal resource access.

---

## Ticket / Scenario

**User Impact:**  
User unable to access internal company resources due to network-related connectivity issues.

**Reported Issue:**  
Network-related access issue affecting internal resources.

**Environment:**
- Windows endpoint (domain-joined)
- Windows Server Domain Controller (DNS)
- Active Directory environment
- Internal network resources

---

## Symptoms

- Inability to access internal company resources  
- Network-related connectivity issues  
- Name resolution failures within the domain environment  

---

## Troubleshooting Process

1. Reviewed workstation network configuration  
2. Verified network connectivity  
3. Performed layered connectivity testing  
4. Used DNS query tools to test name resolution  
5. Identified DNS misconfiguration at the endpoint level  

---

## Root Cause

The workstation was configured with incorrect DNS settings, preventing proper name resolution to the domain controller and internal resources.

---

## Fix Implemented

- Restored DNS settings to the domain controller  
- Applied corrective changes at the endpoint level  

---

## Verification

- Confirmed proper name resolution within the domain environment  
- Restored access to internal company resources  
- Verified stable network connectivity  

---

## LinkedIn Post & Video

https://www.linkedin.com/posts/caleb-mallek-4aa10b390_itsupport-helpdesk-tier1support-activity-7421986495609085953-B21-?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGAf2QIBpfh8fvg-z4gxhvhFLqdO3YSSshg

---

## Key Takeaways

- DNS misconfiguration can directly impact access to internal resources  
- Layered diagnostics improve troubleshooting accuracy  
- Verifying network configuration is a critical early step  
- Proper DNS integration is essential in domain environments  

