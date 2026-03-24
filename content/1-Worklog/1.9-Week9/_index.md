---
title: "Week 9 Worklog"
date: 2026-03-09
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives

This week focused on test deployment, end-to-end system validation, and fixing issues discovered during implementation and testing.

- **System Deployment**:
  - Deploy the application to a real environment on EC2
  - Configure related services such as RDS, S3, and AWS Map
  - Verify connectivity between system components

- **System Testing**:
  - Test the main application flows
  - Validate the UI and user experience
  - Verify data flow between frontend and backend

- **Bug Fixing and Optimization**:
  - Fix issues found during testing
  - Improve code structure and system stability
  - Optimize performance where needed

- **System Readiness**:
  - Ensure the application runs reliably
  - Prepare for demo or pre-release validation

---

### Tasks Overview

| Day | Task | Start Date | Completion Date | References |
| :-: |------|:----------:|:---------------:|------------|
|  2  | **Deployment Setup**:<br>- Deploy app to EC2<br>- Configure environment | 09/03/2026 | 09/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  3  | **System Testing**:<br>- Test main features<br>- Check data flow | 10/03/2026 | 10/03/2026 | Internal |
|  4  | **Bug Fixing**:<br>- Identify issues<br>- Debug the system | 11/03/2026 | 11/03/2026 | Internal |
|  5  | **Optimization**:<br>- Improve code quality<br>- Tune performance | 12/03/2026 | 12/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6  | **Final Testing**:<br>- Revalidate the system<br>- Prepare demo | 13/03/2026 | 13/03/2026 | Internal |

---

### Week 9 Achievements

#### What was accomplished

- Successfully deployed the application to **Amazon EC2**
- Verified stable connections with:
  - **Amazon RDS**
  - **Amazon S3**
  - **AWS Map**
- Completed end-to-end testing of the main system flows
- Fixed issues found during testing and debugging
- Improved performance and overall user experience
- Brought the system to a **demo-ready** state

#### Architecture Summary

- **Compute**: EC2 (application deployment)
- **Database**: RDS (SQL Server)
- **Storage**: S3 (static files)
- **Map**: AWS Map (location service)
- **Flow**: Frontend → Backend → AWS Services
- **Stage**: Testing / Pre-production
