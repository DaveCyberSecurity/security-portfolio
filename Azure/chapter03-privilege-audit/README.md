<h1 align="center">The Privilege Audit:</h1> 


<h2 align="center">explanation yada yada</h2>

---


## Scenario:
Privilege analysis was conducted using five independent collection methods: Azure IAM review, Azure CLI enumeration, Azure Resource Graph KQL analysis, Privileged Identity Management exports, and manual validation. Combining these methods provided visibility into standing privileges, eligible privileges, orphaned assignments, service principal permissions, and privilege escalation opportunities that would not have been observable through any single Azure interface. This approach provides a high-confidence assessment of effective privileged access across the tenant.


| Method               | Sees                                        | Misses                              |
|----------------------|---------------------------------------------|-------------------------------------|
| IAM blade / export   | Active assignments at scope, inherited      | Group members, orphaned principals  |
| Azure CLI            | Same, plus null principalName (orphans)     | One scope per run                   |
| Resource Graph (KQL) | Whole tenant in one query                   | Eligible assignments                |
| PIM export           | Eligible vs active, activation history      | Assignments outside PIM             |
