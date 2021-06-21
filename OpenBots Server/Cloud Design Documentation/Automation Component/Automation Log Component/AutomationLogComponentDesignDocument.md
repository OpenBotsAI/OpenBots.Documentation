Author: DH
Creation Date: 9/15/2020

Updated By: NC
Updated On: 3/18/2020

**Automation Log Component**

**Context**

- Problem: User cannot see automation logs of an executed automation.  Users should be able to view the logs of an automation triggered through the Server. Users should be able to view and download all logs for a particular job id to a csv file.  Agent and Studio should be able to create or edit an automation log.
- Requirements: Ensure that administrators and users are able to view a list of triggered jobs and view the logs pertaining to that job.  Ensure that administrators and users can view more details about a log and export all logs to a CSV file.  Ensure that Agent and Studio can create or edit an automation log.

**Component Scope**

- In-Scope:
  - Users can view automation logs pertaining to a triggered job.
  - Users can view more details about a log.
  - Users can export all automatiom logs by job id to a CSV file.
  - Agent/Studio can create or edit an automation log pertaining to a triggered job.
- Out-of-Scope:
  - Agent/Studio can delete an automation log pertaining to a triggered job.
  - Users can create, edit, or delete logs pertaining to a triggered job.
  - Users can filter logs by their properties, such as level and message.

**Design**

- Overview: The overall structure uses the AutomationLogsController to allow the user to make a web request for the list of logs or individual details.  The AutomationLogsController gets called, which in turn accesses the methods in the AutomationLogManager, and then uses the AutomationLogRepository to retrieve the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.  The same structure will be utilized to create, edit, or export automation logs.
- Proposed Solution:
  - User Interface:
    - On the Dashboard, there will be a link to Automation Logs.  The automation logs will be displayed in a table.
      - The headers will be: Automation Name, Machine Name, Message, Message Template, Level, Logger, Jobs, and View.
      - The appropriate data or link will be displayed underneath each heading for all automation logs.
        - When a user clicks on the link under Jobs, the user will be redirected to a list of jobs for that automation.
        - The user can click on each automation log view, where the user will be redirected to a view of that specific automation execution log's details based on its id.
          - On the details page, the user can click on a link to redirect to view details of the automation itself.
          - All appropriate information will be displayed about each automation log.
          - There will be an option to go back to the view of of all automation logs for the specific job.
        - There will be a button to export all automation logs into a CSV file.
  - AutomationLogsController:
    - The AutomationLogsController will make an API request and use the AutomationLogManager to access the AutomationLogRepository to retrieve all the logs of a specific job from the Server and will return that information back to the view.  The controller will utilize this same structure to export the list of logs to a file, retrieve individual log details, or create/edit an automation log.
    - NOTE: The current API version is 1.
    - Routes:
      - All automation logs: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/automationlogs")]
        - Payloads
          - Input : Organization id
          - Output : JSON file listing all automation logs
      - Count of automation logs: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/automationlogs/count")]
        - Payloads
          - Input : Organization id
          - Output : Count of all automation logs
      - Get individual automation log details for triggered job: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/automationlogs/{id}")]
        - Payloads
          - Input : Organization id, automation log id
          - Output : JSON file with details about the automation log for the given id
      - Export automation log: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/automationlogs/export/{filetype?}")]
        - Payloads
          - Input : Organization id, job id, file type (optional)
          - Output : CSV, Zip, or JSON file listing all automation log details
      - Create automation log: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/automationlogs")]
        - Payloads
          - Input : Organization id, AutomationLog model data
          - Output : JSON file with the created log
  - AutomationLog Manager:
   - The AutomationLogManager will inherit BaseManager, which inherits IManager, and IAutomationLogManager.
      - Beyond the base class and interfaces, AutomationLogManager will implement the appropriate methods to assist the AutomationLogsController, which will be used when exporting automation logs.
      - If log storage is set to Azure Table storage, the AutomationLogManager will also use the AzureTableLoggerAdapter to add, edit, export, and delete the appropriate logs.
  - AutomationLogRepository:
   - The AutomationLogRepository will inherit EntityRepository<AutomationLog>, which inherits ReadOnlyEntityRepository, and IAutomationLogRepository.
     - In addition to the methods inherited from the base class, the AutomationLogRepository will retrieve automation logs by job id, or the details of an individual automation log.
  - AutomationLog Data Model:
    - The AutomationLog data model will be used to view details of each job's automation logs.  It will inherit from Entity.
      - In addition to the properties inherited by the base class, AutomationLog will have string Message, string MessageTemplate, string Level, DateTime AutomationLogTimestamp, string Exception, string Properties, Guid JobId, Guid AutomationId, Guid AgentId, string MachineName, string AutomationName, and string Logger.

**Sequence Diagrams**

- [Automation Log Component Sequence Diagram](Sequence Diagrams/AutomationLogComponentSequenceDiagram.png)
- [Automation Log Count Sequence Diagram](Sequence Diagrams/CountAutomationLogsSequenceDiagram.png)
- [Automation Log Details Sequence Diagram](Sequence Diagrams/AutomationLogDetailsSequenceDiagram.png)
- [Export Automation Logs Sequence Diagram](Sequence Diagrams/ExportAutomationLogsSequenceDiagram.png)
- [Edit Automation Log Sequence Diagram](Sequence Diagrams/EditAutomationLogSequenceDiagram.png)
- [Create Automation Log Sequence Diagram](Sequence Diagrams/CreateAutomationLogSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Get list of logs for specified job id and convert to appropriate file type.
- Negative Test Cases:
  - Does not find logs if the specified id doesn't exist and does not convert to appropriate file type.
