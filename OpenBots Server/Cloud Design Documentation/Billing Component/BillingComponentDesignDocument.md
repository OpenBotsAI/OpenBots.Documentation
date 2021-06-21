Author: DH
Creation Date: 5/13/2021

Updated By: NC
Updated On: 6/18/2021

**Billing Component**

**Context**

- Problem: Users are not able easily track the status of different billing tables.
- Requirements: Ensure that users are able to access information about the organization's billing records.

**Component Scope**

- In-Scope:
  - Users can view and export their current organization's daily balances.
  - Users can purchase additional credits for their organization.
  - Users' organizations will be blocked in cases where credit usage has been exceeded.
  - Users can view their current storage usage.
  - Users can purchase additional storage.
- Out-of-Scope:
  - Allow users to manually block and unblock their organization.

**Design**

- Overview: The Billing component is responsible for tracking organization usage of minutes and storage. User organizations will be blocked in case where the credits are not sufficient to continue Cloud Server usage at the end of the day. Users are able to add credits at any time to prevent an organization block.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the different Billing tabs.
        - Orchestration: Displays a paginated list of all daily balances for the current organization. Additionally, users are able to export all of the records for the current month and add credits to their organization by click on the "Buy Orchestration Credits" button.
        - Storage: Displays details about the current organization's storage and its different drives. Users are also able to buy additional storage by clicking on the "Buy Storage" button.
  - Billing Controller:
    - The BillingController will allow users to access information on the billing status of their current organizations. 
    - NOTE: The current API version is 1.
    - Routes:
      - Get all DailyOrchestrationDebits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyOrchestrationDebits")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyOrchestrationDebits
      - Get all DailyCredits: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyCredits")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyCredits
      - Get all DailyBalances: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyBalances")]
        - Payloads
          - Input : None
          - Output : JSON file containing all DailyBalances
      - Export DailyBalances: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyBalances/export/{fileType?}")]
        - Payloads
          - Input : None
          - Output : Downloadable file containing DailyBalances
      - DailyOrchestrationDebits sum: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyOrchestrationDebits/Sum")]
        - Payloads
          - Input : None
          - Output : Sum of TotalMinutes 
      - DailyCredits sum: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/Billing/DailyCredits/Sum")]
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
  - Manager:
    - The TransactionManager will inherit BaseManager and ITransactionManager.
      - Beyond the base class and interface, TransactionManager will implement the appropriate methods to assist the billingController. In addition the TransactionManager will contain methods for handling updates of the different billing tables.
  - Repositories:
    - The MonthlyBalanceRepository will inherit TenantEntityRepository<MonthlyBalance> and IMonthlyBalanceRepository.
      - The MonthlyBalanceRepository will track the monthly balance of each organization.
    - The DailyOrchestrationDebitRepository will inherit TenantEntityRepository<DailyOrchestrationDebit> and IDailyOrchestrationDebitRepository.
      - The DailyOrchestrationDebitRepository will track an organization's credit usage per day.
    - The DailyCreditRepository will inherit TenantEntityRepostitory<DailyCreditRepository> and IDailyCreditRepository.
      - The DailyCreditRepository will track an organization's available credits per day.
  - The DailyBalanceRepository will inherit TenantEntityRepository<DailyBalanceRepository> and IDailyBalanceRepository.
    - The DailyBalanceRepository will track an organization's daily balance.
- Models:
  - Monthly Balances Data Model:
    - The MonthlyBalances data model will be used to view details of each organization's monthly balance.  It will inherit the Entity class and ITenanted.
      - Beyond the base class, MonthlyBalances will have Guid OrganizationId, int MonthNumber, int Year, int Month, int DaysInMonth, long OpeningBalanceMinutes, long CreditOrchestrationInMinutes, long DebitOrchestrationInMinutes, long BalanceOrchestrationInMinutes, long AdditionalStorageInMB, long BaseStorageInMB, long TotalStorageInMB, long StorageUsageInMB, bool HasError, string ErrorCode, and string ErrorMessage.
    - Daily Orchestration Debit Data Model:
      - The DailyOrchestrationDebit data model will be used to view details of each organization's daily credit usage.  It will inherit the Entity class and ITenanted.
        - Beyond the base class, it will have Guid OrganizationId, int DayNumber, DateTime EntryDate, int Year, int Month, int Day, long TotalMinutes, int CompleteExecutionTime, and int IncompleteExecutionTime.
    - Daily Credit Data Model:
      - The DailyCredit data model will be used to view available daily credits.  It will inherit the Entity class and ITenanted.
        - Beyond the base class, it will have Guid OrganizationId, int DayNumber, DateTime EntryDate, int Year, int Month, int Day, long AdditionalMinutes, long AdditionalStorageInMB, long BaseStorageInMB, long TotalStorageInMB, string TransactionID, and string TransactionMessage.
    - Daily Balance Data Model:
      - The DailyBalance data model will be used to view an organization's daily balance.  It will inherit Entity and ITenanted.
        - Beyond the base class, it will have Guid OrganizationId, int DayNumber, DateTime EntryDate, int Year, int Month, int Day, long OpeningOrchestrationBalanceMinutes, long CreditOrchestrationMinutes, long DebitOrchestrationMinutes, long ForfeitedOrchestrationMinutes, long OrchestrationMinutesOverage, long AdditionalStorageInMB, long BaseStorageInMB, long TotalStorageInMB, and long StorageUsageInMB.

**Sequence Diagrams**

- [Organization Billing Sequence Diagram](Sequence Diagrams/OrganizationBillingSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A
