# RACI Chart

[What is a RACI chart?](https://www.forbes.com/advisor/business/raci-chart/)

- **R** - Responsible
- **A** - Accountable
- **C** - Consulted
- **I** - Informed

| Work Item                               | CSPI QE | Product QE | Project Management | DPTP | OpenShift Install Maintainers | 
|-----------------------------------------|---------|------------|--------------------|------|-------------------------------|
| [Prerequisites](Prerequisites_Guide.md) | C       | R          | I                  | N/A  | N/A                           |
| [Kick-off](Kickoff_Guide.md)            | R       | A          | I                  | N/A  | N/A                           |
| Triggering                              | R       | I          | C                  | A    | N/A                           |
| CI Services                             | N/A     | N/A        | N/A                | R    | N/A                           |
| Provision OCP Cluster                   | A       | I          | N/A                | N/A  | R                             |
| Install Product/Operator                | R       | A          | N/A                | N/A  | N/A                           |
| [Containerized LP OCP Interop Tests](../OCP_CI_Tutorials/Containers/Container_Creation_Guide.md)           | C       | R          | N/A                | N/A  | N/A                           |
| Creation of LP Test Ref                 | R       | A          | N/A                | N/A  | N/A                           |
| Maintenance of LP Test Ref              | A       | R          | N/A                | N/A  | N/A                           |
| Creation of config/ProwJob              | R       | A          | N/A                | N/A  | N/A                           |
| Maintenance of config/ProwJob           | A       | R          | N/A                | N/A  | N/A                           |
| Reporting to Jira/Slack/Sippy           | R       | A          | I                  | N/A  | N/A                           |
| Analyzing and fixing test failures      | A       | R          | I                  | N/A  | N/A                           |
| Analyzing and fixing infra failures     | R       | I          | I                  | A    | A                             |
