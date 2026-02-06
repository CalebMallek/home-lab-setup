# Simulation 004 — Multi-Ticket Prioritization & Access Handling

## Summary
In this simulation, I demonstrate working within a real ticketing system to manage and prioritize multiple incoming incidents based on business impact and urgency. The walkthrough includes addressing a high-priority VPN access issue through Active Directory account review and credential reset, as well as resolving a new hire domain login problem to restore onboarding access.

To better reflect real-world IT support workflows, I’ve implemented a dedicated help desk ticketing platform within my home lab environment. This allows me to practice realistic service desk processes such as queue management, prioritization, documentation, and status tracking—mirroring how MSPs and internal IT teams operate in production environments.

In addition to resolving critical access-related tickets, I also triage additional incoming incidents to demonstrate typical Tier 1 workload management and workflow continuity.

---

## Ticket / Scenario

**User Impact:**  
Multiple users impacted by access-related issues affecting remote access and onboarding.

**Reported Issues:**  
High-priority VPN access issue  
New hire domain login issue  
Additional incoming incidents requiring triage  

**Environment:**
- Help desk ticketing system
- Windows endpoint(s) (domain-joined)
- Windows Server Domain Controller
- Active Directory account review and credential reset workflow
- VPN access

---

## Symptoms

- User unable to access VPN
- New hire unable to log into domain account
- Multiple tickets entering the queue requiring prioritization and triage

---

## Troubleshooting Process

1. Managed an active ticket queue and prioritized incidents based on impact and urgency  
2. Addressed high-priority VPN access issue through Active Directory account review  
3. Performed credential reset and proceeded with access restoration steps  
4. Resolved a new hire domain login issue to restore onboarding access  
5. Triaged additional incoming incidents to demonstrate Tier 1 workload handling  
6. Documented updates and maintained status tracking within the ticketing system  

---

## Root Cause

Access-related incidents requiring Active Directory account review and credential handling, along with a new hire domain login problem impacting onboarding access.

---

## Fix Implemented

- Reviewed the affected user account in Active Directory  
- Performed credential reset for VPN access restoration  
- Resolved new hire domain login issue to restore onboarding access  
- Continued triage and status tracking for additional incoming incidents  

---

## Verification

- Confirmed VPN access issue was addressed following credential reset workflow  
- Confirmed new hire domain login issue was resolved and onboarding access restored  
- Verified tickets were documented and updated appropriately in the ticketing system  

---

## LinkedIn Post & Video

https://www.linkedin.com/posts/caleb-mallek-4aa10b390_itsupport-helpdesk-tier1support-activity-7420997128019218432-D6mH?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGAf2QIBpfh8fvg-z4gxhvhFLqdO3YSSshg

---

## Key Takeaways

- Ticket prioritization should be driven by business impact and urgency  
- A ticketing platform improves realism by enforcing documentation and status tracking  
- Access issues often require Active Directory account review and credential handling  
- Tier 1 support requires balancing resolution work with continuous triage  


