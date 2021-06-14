Author: Dairon Hernandez
Creation Date: 06/14/2021

**Service Alerts Component**

**Context**

- Problem: Organizations are not able to receive notifications from the CloudServer.

- Requirements: Ensure ServiceAlerts are clearly visible in the organization's dashboard. Organization users should receive ServiceAlerts specific to their organization as well as Alerts intended for all Organizations. 

**Component Scope**

- In-Scope:
  - Organizations can view their own ServiceAlerts on the dashboard
  - Organizations can view global alerts on their dashboard
- Out-of-Scope:
  - Organizations can interact with ServiceAlerts
  - Organizations can create, update and delete ServiceAlerts

**Design**

- Overview: The overall structure uses the ServiceAlertsController to allow the Front-End to access a list of 'Active' ServiceAlerts for the current Organizations. The dashboard will display a list of all 'Active' ServiceAlerts for the organization as well as all 'Active' alerts that are 'Global'(intended for all organizations). 

- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a list of all 'Active' ServiceAlerts for the current organization.
        - The ServiceAlerts will display the title and description of the current alerts.
  - ServiceAlerts Controller:
    - The ServiceAlertsController will be used to retreive data from the ServiceAlerts data table. 
    - NOTE: The current API version is 1.
    - Routes:
      - Get all ServiceAlerts: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/ServiceAlerts")]
        - Payloads
          - Input : Organization id.
          - Output : JSON file containing a list of all ServiceAlerts.
      - Get all 'Active' ServiceAlerts: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/ServiceAlerts/Active")]
        - Payloads
          - Input : Organization id.
          - Output : JSON file containing a list of all 'Active'(current date falls within start and end dates) ServiceAlerts.

  - ServiceAlert Manager:
    - The ServiceAlertManager will inherit BaseManager, which inherits IManager, and IServiceAlertManager.
      - Beyond the base class and interfaces, ServiceAlertManager will implement appropriate methods to assist the ServiceAlertsController.
  - ServiceAlert Repository(s):
    - The ServiceAlertRepository will query data from the ServiceAlerts table.
  - ServiceAlert Data Model(s):
    - The ServiceAlert data model will be used to store details of each ServiceAlert.  It will inherit the Entity class and ITenanted interface.
      - Beyond the base entity properties, ServiceAlert will contain the properties ServiceAlertKey, ServiceTag, OrganizationId, StartOnUTC, EndOnUTC, AlertCode, AlertType, Message, Description and IsUserActionDisabled.

**Sequence Diagrams**

- [ServiceAlert Component Sequence Diagram](Sequence Diagrams/ServiceAlertsSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A