Author: Dairon Hernandez
Creation Date: 05/13/2021

Updated On: 
Updated By: 

**Billing Component**

**Context**

- Problem: Users are not able easily track the status of different billing tables
- Requirements: Ensure that users are able to access information about the organization billing records

**Component Scope**

- In-Scope:
  - Users can view and export their current organization's daily balances.
  - Users can purchase additional credits for their organization.
  - Users organizations will be blocked in cases where credits usage has been exceeded.
  - Users can view their current storage usage. 
  - Users can purchase additional storage. 
- Out-of-Scope:
  - Allow users to manually block and unblock their organization

**Design**

- Overview: The Billing component is responsible for tracking organization usage of minutes and storage. User organizations will be blocked in cases where their credits are not sufficient to continue server usage at the end of the day. Users are able to add credits at any time to prevent an organization block.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the different Billing tabs.
        - Orchestration Tab: Displays a paginated list of all daily balances for the current organization. Additionally users are able to export all of the records for the current month and add credits to their organization by click on the "Buy Orchestration Credits" button.
        - Storage Tab: Displays details about the current organization's storage and it's different drives. Users are also able to buy additional storage by clicking on the "Buy Storage" button.
  - BillingController:
    - The BillingController will allow users to access information on the billing status of their current organizations. 
    - NOTE: The current API version is 1.
    - Routes:
      - Get all DailyOrchestrationDebits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyOrchestrationDebits")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyOrchestrationDebits
      - Get all DailyOrchestrationDebits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyCredits")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyCredits
      - Get all DailyOrchestrationDebits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyBalances")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyBalances
      - Export DailyBalances: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyBalances/export/{fileType?}")]
        - Payloads
          - Input : None
          - Output : Downloadable file containing dailyBalances
      - DailyOrchestrationDebits Sum: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyOrchestrationDebits/Sum")]
        - Payloads
          - Input : None
          - Output : Sum of TotalMinutes 
      - DailyCredits Sum: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyCredits/Sum")]
        - Payloads
          - Input : None
          - Output : Sum of AdditionalMinutes
      - Count of DailyOrchestrationDebits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyOrchestrationDebits/Count")]
        - Payloads
          - Input : None
          - Output : Count of DailyOrchestrationDebits
      - Count of DailyCredits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyCredits/Count")]
        - Payloads
          - Input : None
          - Output : Count of DailyCredits
      - Count of DailyBalances: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyBalances/Count")]
        - Payloads
          - Input : None
          - Output : Count of DailyBalances
      - Get OrganizationBillingStatus: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/OrganizationBillingStatus")]
        - Payloads
          - Input : None
          - Output : JSON file containing the OrganizationBillingStatus
      - Get the current CreditBalance: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/CreditBalance")]
        - Payloads
          - Input : None
          - Output : JSON file containing the current CreditBalance
  - Transaction Manager:
    - The TransactionManager will inherit BaseManager and ITransactionManager.
      - Beyond the base class and interface, TransactionManager will implement the appropriate methods to assist the billingController. In addition the TransactionManager will contain methods for handling updates of the different billing tables.

**Sequence Diagrams**

- [Integration Event Sequence Diagram](Sequence Diagrams/IntegrationEventSequenceDiagram.png)
- [Integration Event Subscription Diagram](Sequence Diagrams/SubscriptionSequenceDiagram.png)
- [Integration Event Raise Business Event Diagram](Sequence Diagrams/RaiseBusinessEventSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:

- Negative Test Cases:
