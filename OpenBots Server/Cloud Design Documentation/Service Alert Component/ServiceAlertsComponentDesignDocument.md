Author: DH
Creation Date: 6/14/2021

Updated By: NC
Updated On: 6/18/2021

**Service Alerts Component**

**Context**

- Problem: Organizations are not able to receive notifications from CloudServer.

- Requirements: Ensure service alerts are clearly visible in the organization's dashboard. Organization users should receive service alerts specific to their organization as well as alerts intended for all organizations. 

**Component Scope**

- In-Scope:
  - User in each organization can view their own service alerts on the dashboard.
  - User in each organization can view global alerts on the dashboard.
- Out-of-Scope:
  - User can interact with service alerts.
  - User can create, update and delete service alerts.

**Design**

- Overview: The overall structure uses the ServiceAlertsController to allow the front end to access a list of active service alerts for the current organizations. The dashboard will display a list of all active service alerts for the current user's organization as well as all active global alerts (intended for all organizations). 

- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a list of all active service alerts for the current organization.
        - The service alerts will display the title and description of the current organization's and any global alerts.
  - Service Alerts Controller:
    - The ServiceAlertsController will be used to retreive data from the ServiceAlerts data table. 
    - NOTE: The current API version is 1.
    - Routes:
      - Get all ServiceAlerts: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/ServiceAlerts")]
        - Payloads
          - Input : Organization id
          - Output : JSON file containing a list of all ServiceAlerts
      - Get all 'Active' ServiceAlerts: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/ServiceAlerts/Active")]
        - Payloads
          - Input : Organization id
          - Output : JSON file containing a list of all 'Active' (current date falls within start and end dates) ServiceAlerts

  - Service Alert Manager:
    - The ServiceAlertManager will inherit BaseManager, which inherits IManager, and IServiceAlertManager.
      - Beyond the base class and interfaces, ServiceAlertManager will implement appropriate methods to assist the ServiceAlertsController.
  - ServiceAlert Repository:
    - The ServiceAlertRepository will query data from the ServiceAlerts table.
  - ServiceAlert Data Model:
    - The ServiceAlert data model will be used to store details of each service alert.  It will inherit the Entity class and ITenanted interface.
      - Beyond the base entity properties, ServiceAlert will contain string ServiceAlertKey, string ServiceTag, Guid OrganizationId, DateTime StartOnUTC, DateTime EndOnUTC, string AlertCode, string AlertType, string Message, string Description, and bool IsUserActionDisabled.

**Sequence Diagrams**

- [ServiceAlert Component Sequence Diagram](Sequence Diagrams/ServiceAlertsSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A