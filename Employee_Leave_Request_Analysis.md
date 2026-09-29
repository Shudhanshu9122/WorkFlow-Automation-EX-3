# Employee Leave Request Approval System

## 1. Process Model Design (BPMN 2.0)
The workflow (provided in `Leave_Request_Process.bpmn`) is designed as follows:
- **Start Event**: Employee submits a leave request.
- **Service Task**: Fetch Available Leave Balance (Retrieves `availableBalance` from HR system).
- **Business Rule Task**: Evaluate Leave Rules (DMN). Evaluates the leave type, requested days, and available balance.
- **Exclusive Gateway (Routing Decision)**: Routes the flow based on the DMN outcome (`Auto-Approved`, `Manager-Review`, or `Rejected`).
  - **Path 1 (Rejected)**: Rejection due to insufficient balance. The process sends a notification and ends.
  - **Path 2 (Manager-Review)**: Routes to a User Task for Manager Approval. An exclusive gateway follows this task to check the manager's decision. If rejected, the employee is notified, and the process ends.
  - **Path 3 (Auto-Approved / Manager Approved)**: Reaches a Service Task to forward to HR and update the leave balance.
- **End Event**: Process successfully completed.

## 2. DMN Decision Table Design
The DMN (provided in `Leave_Approval_Decision.dmn`) encapsulates the business logic. It takes inputs `requestedDays`, `availableBalance`, and `leaveType` to output a `routingDecision`.

## 3. Service Tasks vs. User Tasks and Payloads
*   **Service Tasks (Automated)**:
    *   **Fetch Available Leave Balance**: Connects to the HR system API to retrieve the balance. 
        *   **Input payload**: `employeeId`
        *   **Output payload**: `availableBalance`
    *   **Forward to HR & Update Balance**: Updates the employee's leave balance in the backend HR system.
        *   **Input payload**: `employeeId`, `leaveType`, `requestedDays`
    *   **Notify Employee (Rejection)**: Sends an email/message to the employee regarding rejection.
        *   **Input payload**: `employeeId`, `rejectionReason` (e.g., "Insufficient Balance" or "Manager Rejected")
*   **User Tasks (Human-in-the-loop)**:
    *   **Manager Approval**: Requires the direct manager to manually review the request in Camunda Tasklist.
        *   **Input Variables**: `employeeId`, `employeeName`, `leaveType`, `requestedDays`, `availableBalance`
        *   **Output Variables**: `managerDecision` (Approved / Rejected), `managerComments`
*   **Business Rule Task (DMN)**:
    *   **Evaluate Leave Rules**: Executes the decision engine.
        *   **Input Variables**: `leaveType`, `requestedDays`, `availableBalance`
        *   **Output Variables**: `routingDecision`

## 4. Justification for DMN Design (Single vs. Multiple Tables)
**Recommendation: Multiple Hit-Policy Tables (Decision Requirements Graph - DRG)**

While the basic task can be achieved with a single decision table using a `FIRST` hit policy (where the balance check is placed as the topmost rule), **the optimal enterprise design utilizes multiple decision tables**.

**Why Multiple Tables?**
1.  **Separation of Concerns**: 
    *   *Table 1: Eligibility Check (Hard Rule)*: Checks if the employee has enough balance. Output: `isEligible` (boolean).
    *   *Table 2: Approval Routing (Business Rule)*: Evaluates only if `isEligible = true`. Determines if it can be auto-approved based on leave type and duration. Output: `routingDecision`.
2.  **Maintainability & Scalability**: In the future, if XYZ Corp adds more complex eligibility criteria (e.g., mandatory block-out dates, tenure-based restrictions), you only update the Eligibility Check table without risking side effects on the Approval Routing logic.
3.  **Readability and Testing**: Smaller tables with a single focus are easier for business stakeholders to read, validate, and test. A single monolithic table risks becoming convoluted as rules multiply. 

For the provided file, a unified DMN is supplied to concisely meet the prompt's base requirements, but a DRG remains the architectural best practice for such scenarios.
