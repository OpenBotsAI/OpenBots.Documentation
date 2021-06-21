Author: Nicole Carrero
Creation Date: 8/20/2020

Updated By: NC
Updated On: 6/15/2021

**Automation Execution Log Component**

**Context**

- Problem: User cannot see automation execution logs of a triggered job.  Users should be able to view the execution logs of a job triggered through the Server, select which job to view the logs for, and filter the logs by level.  Users should be able to view more details of an individual log, where they are presented with the full information provided to the Server.
- Requirements: Ensure that administrators and users are able to view a list of triggered jobs and view the logs pertaining to that job.  Ensure that administrators and users can filter the presented logs by level, and view more details about a log.  Ensure that the Studio and Agent can create, edit, or delete logs pertaining to a triggered job.

**Component Scope**

- In-Scope:
  - Users can view automation execution logs pertaining to a triggered job.
  - Users can filter the list of automation execution logs by level.
  - Users can view more details about a specific automation execution.
  - Agent/Studio can create, edit, or delete logs pertaining to a triggered job.
- Out-of-Scope:
  - Users can create, edit, or delete logs pertaining to a triggered job.
  - Users can filter the list of logs by job id, timestamp, computer, automation name, rendered message, or properties.

**Design**

- Overview: The overall structure uses the AutomationExecutionLogsController to allow the user to make a web request for the list of logs or individual execution details.  The AutomationExecutionLogsController gets called, which in turn accesses the methods in the AutomationExecutionLogManager, and then uses the AutomationExecutionLogRepository to retrieve the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.  The same process will be utilized when adding, editing, or deleting an automation execution log.
- Proposed Solution:
  - User Interface:
    - On the Jobs page, there will be a list of jobs (see Job Component Design Document for more information).  For each triggered job, there will be a link to redirect the user to the list of corresponding automation execution logs.
    - The execution logs will be displayed in a table.  Users will be able to see the table after clicking on the corresponding job.
      - The headers will be: Job Name, Automation Name, Agent Name, Started On, Completed On, Status, and Has Errors.
      - The appropriate data will be displayed underneath each heading for all execution logs.
        - There will be a feature to filter automation execution logs by level.  There will be a way to go back to view all automation execution logs for the triggered job.
        - The user can click on each automation execution log, where the user will be redirected to a view of that specific automation execution log's details based on its id.
          - On the page showing individual automation execution log details, all appropriate information will be displayed about each automation execution log.
          - There will be an option to go back to the view of of all automation execution logs for the specific job.
  - AutomationExecutionLogsController:
    - The AutomationExecutionLogsController will make an API request and use the AutomationExecutionLogManager to access the AutomationExecutionLogRepository to retrieve all the automation execution logs of a specific job from the Server and will return that information back to the view.  The controller will utilize this same structure to export the list of logs to a file or retrieve individual automation execution log details.
    - Routes:
    - NOTE: The current API version is 1.
      - All automation execution logs for triggered job: [HttpGet("api/v{apiVersion}/automationexecutionlogs]
        - Payloads
          - Input : None
          - Output : JSON file listing automation execution logs
      - All automation execution logs for triggered job (view model): [HttpGet("api/v{apiVersion}/automationexecutionlogs/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing automation execution logs
      - Count of automation execution logs: [HttpGet("api/v{apiVersion}/automationexecutionlogs/count")]
        - Payloads
          - Input : None
          - Output : Count of all automation execution logs for a triggered job
      - Get automation execution log details for triggered job: [HttpGet("api/v{apiVersion}/automationexecutionlogs/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing the automation execution log details for a specific job
      - Get automation execution log details for triggered job (view model): [HttpGet("api/v{apiVersion}/automationexecutionlogs/view/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing the automation execution log details for a specific job
      - Add an automation execution log for a job: [HttpPost("api/v{apiVersion}/automationexecutionlogs]
        - Payloads
          - Input : AutomationExecutionLog data model
          - Output : JSON file with the created log
      - Add automation execution log for a started job: [HttpPost("api/v{apiVersion}/automationexecutionlogs/startautomation]
        - Payloads
          - Input : AutomationExecutionLog data model
          - Output : JSON file with the created log
      - Edit automation execution log for a triggered job: [HttpPut("api/v{apiVersion}/automationexecutionlogs/{id}]
        - Payloads
          - Input : AutomationExecutionLog data model
          - Output : JSON file with updated values
       - Edit automation execution log for a completed job: [HttpPut("api/v{apiVersion}/automationexecutionlogs/{id}/endautomation]
        - Payloads
          - Input : AutomationExecutionLog data model
          - Output : JSON file with updated values
       - Delete automation execution log: [HttpDelete("api/v{apiVersion}/automationexecutionlogs/{id}")]
         - Payloads
           - Input : Automation execution log id
           - Output : 200 OK response
       - Edit automation execution log property: [HttpPatch("api/v{apiVersion}/automationexecutionlogs/{id}")]
         - Payloads
           - Input : Automation execution log id
           - Output : 200 OK response
  - AutomationExecutionLogManager:
   - The AutomationExecutionLogManager will inherit BaseManager, which inherits IManager, and IAutomationExecutionLogManager.
      - Beyond the base class and interface, AutomationExecutionLogManager will implement the appropriate methods to assist AutomationExecutionLogsController.
  - AutomationExecutionLogRepository:
    - AutomationExecutionLogRepository will inherit EntityRepository<AutomationExecutionLog>, which inherits ReadOnlyEntityRepository, and IProcessExecutionLogRepository.
    - In addition to the methods in the base class, the AutomationExecutionLogRepository will retrieve automation execution logs using a view model.
  - AutomationExecutionLog Data Model:
    - The AutomationExecutionLog data model will be used to view details of each job's execution logs.  It will inerit the Entity base class.
      - In addition to the properties from the base Entity class, AutomationExecutionLog will have Guid JobId, Guid AutomationId, Guid AgentId, DateTime StartedOn, DateTime CompletedOn, string Trigger, string TriggerDetails, string Status, string HasErrors, string ErrorMessage, and string ErrorDetails.

**Sequence Diagrams**

- [Automation Execution Log Component Sequence Diagram](Sequence Diagrams/AutomationExecutionLogComponentSequenceDiagram.png)
- [Count Automation Execution Log Sequence Diagram](Sequence Diagrams/AutomationExecutionLogsCountSequenceDiagram.png)
- [Automation Execution Log Details Sequence Diagram](Sequence Diagrams/AutomationExecutionLogDetailsSequenceDiagram.png)
- [Create Automation Execution Log Sequence Diagram](Sequence Diagrams/CreateAutomationExecutionLogSequenceDiagram.png)
- [Edit Automation Execution Log Sequence Diagram](Sequence Diagrams/EditAutomationExecutionLogSequenceDiagram.png)
- [Edit Automation Execution Log Property Sequence Diagram](Sequence Diagrams/EditAutomationExecutionLogPropertySequenceDiagram.png)
- [Delete Automation Execution Log Sequence Diagram](Sequence Diagrams/DeleteAutomationExecutionLogSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - N/A
- Negative Test Cases:
  - N/A
