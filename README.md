# 🏢 Employee Leave Request Approval System

![Camunda 8](https://img.shields.io/badge/Camunda-8.x-blue?style=for-the-badge&logo=camunda)
![BPMN 2.0](https://img.shields.io/badge/BPMN-2.0-orange?style=for-the-badge)
![DMN 1.3](https://img.shields.io/badge/DMN-1.3-yellow?style=for-the-badge)

Welcome to the **Employee Leave Request Approval System** repository! This project models an automated HR business process using **Camunda 8 (Zeebe)**, adhering to BPMN 2.0 and DMN 1.3 standards.

---

## 📝 Scenario Overview

XYZ Corp requires an automated solution for handling employee leave requests. The system routes leave requests dynamically based on the employee's available balance, the leave type, and the requested duration.

### 💼 Business Rules
1. **Leave Types**: Casual, Sick, or Earned.
2. **Auto-Approval**: If Sick leave is requested and `days ≤ 3`, it is automatically approved without managerial intervention.
3. **Manager Review**: If the leave type is Casual/Earned, or Sick leave exceeds 3 days, it routes to the direct manager.
4. **Rejection Workflows**: If the manager rejects the request, the employee is notified, and the workflow ends.
5. **HR Integration**: Approved requests are forwarded to HR, updating the employee's available leave balance.
6. **Hard Constraints**: If requested days exceed the available balance at any point, it is **immediately rejected** with an "Insufficient Balance" notification.

---

## 🛠️ Project Components

### 1. BPMN Process Model (`Leave_Request_Process.bpmn`)
The core orchestrator designed in Camunda Desktop Modeler. 
- Integrates Automated **Service Tasks** (Fetching balance, Notifying Employees, Updating HR system).
- Routes logic using **Exclusive Gateways**.
- Implements a **User Task** for human-in-the-loop Manager Approval.
- Invokes a **Business Rule Task** utilizing the DMN decision engine.

### 2. DMN Decision Table (`Leave_Approval_Decision.dmn`)
A dynamically mapped decision table encapsulating the routing logic.
- **Inputs**: `requestedDays` (Integer), `availableBalance` (Integer), `leaveType` (String).
- **Outputs**: `routingDecision` ("Auto-Approved", "Manager-Review", or "Rejected").

### 3. Detailed Analysis Report (`Employee_Leave_Request_Analysis.md`)
A thorough breakdown of the technical decisions made while designing this system, including:
- Differentiation of Service Tasks vs. User Tasks.
- Expected Data Payloads (Variables).
- Architectural justification for using Multiple Hit-Policy tables (DRG) vs. a Single Monolithic table.

---

## 🚀 Getting Started

To explore or deploy this workflow:

1. Clone the repository:
   ```bash
   git clone https://github.com/Shudhanshu9122/WorkFlow-Automation-EX-3.git
   ```
2. Open the `.bpmn` and `.dmn` files using the [Camunda Desktop Modeler](https://camunda.com/download/modeler/).
3. Deploy to a Camunda 8 cluster (SaaS or Self-Managed) using the deployment tool in the Modeler.

---

## 💡 System Architecture Highlights

*   **Scalability**: The system cleanly separates hard HR constraints (balances) from subjective routing rules (approvals).
*   **Automation-First**: Minimizes managerial overhead by auto-approving minor sick leaves.
*   **Transparency**: Employees are actively notified at all rejection endpoints.

---
*Developed as part of Workflow Automation coursework (Scenario 3).*
