Author: Nicole Carrero
Creation Date: 8/20/2020

Updated On: 9/15/2020; 10/15/2020
Updated By: Dairon Hernandez; Nicole Carrero

**Process Execution Log Component**

**Context**

- Problem: User cannot see process execution logs of a triggered job.  Users should be able to view the execution logs of a job triggered through the Server, select which job to view the logs for, and filter the logs by level.  Users should be able to view more details of an individual log, where they are presented with the full information provided to the Server.
- Requirements: Ensure that administrators and users are able to view a list of triggered jobs and view the logs pertaining to that job.  Ensure that administrators and users can filter the presented logs by level, and view more details about a log.  Ensure that the Studio and Agent can create, edit, or delete logs pertaining to a triggered job.

**Component Scope**

- In-Scope:
  - Users can view process execution logs pertaining to a triggered job.
  - Users can filter the list of process execution logs by level.
  - Users can view more details about a specific process execution.
  - Agent/Studio can create, edit, or delete logs pertaining to a triggered job.
- Out-of-Scope:
  - Users can create, edit, or delete logs pertaining to a triggered job.
  - Users can filter the list of logs by job id, timestamp, computer, process name, rendered message, or properties.

**Design**

- Overview: The overall structure uses the ProcessExecutionLogsController to allow the user to make a web request for the list of logs or individual execution details.  The ProcessExecutionLogsController gets called, which in turn accesses the methods in the ProcessExecutionLogManager, and then uses the ProcessExecutionLogRepository to retrieve the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.  The same process will be utilized when adding, editing, or deleting a process execution log.
- Proposed Solution:
  - User Interface:
    - On the Jobs page, there will be a list of jobs (see Job Component Design Document for more information).  For each triggered job, there will be a link to redirect the user to the list of corresponding process execution logs.
    - The execution logs will be displayed in a table.  Users will be able to see the table after clicking on the corresponding job.
      - The headers will be: Job Name, Process Name, Agent Name, Started On, Completed On, Status, and Has Errors.
      - The appropriate data will be displayed underneath each heading for all execution logs.
        - There will be a feature to filter process execution logs by level.  There will be a way to go back to view all process execution logs for the triggered job.
        - The user can click on each process execution log, where the user will be redirected to a view of that specific process execution log's details based on its id.
          - On the page showing individual process execution log details, all appropriate information will be displayed about each process execution log.
          - There will be an option to go back to the view of of all process execution logs for the specific job.
  - ProcessExecutionLogsController:
    - The ProcessExecutionLogsController will make an API request and use the ProcessExecutionLogManager to access the ProcessExecutionLogRepository to retrieve all the process execution logs of a specific job from the Server and will return that information back to the view.  The controller will utilize this same structure to export the list of logs to a file or retrieve individual process execution log details.
    - Routes:
      - All process execution logs for triggered job: [HttpGet("api/v1/processexecutionlogs]
        - Payloads
          - Input : None
          - Output : JSON file listing process execution logs
      - All process execution logs for triggered job (using view model): [HttpGet("api/v1/processexecutionlogs/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing process execution logs
      - Count of process execution logs: [HttpGet("api/v1/processexecutionlogs/count")]
        - Payloads
          - Input : None
          - Output : Count of all process execution logs for a triggered job
      - Get process execution log details for triggered job: [HttpGet("api/v1/processexecutionlogs/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing the process execution log details for a specific job
      - Get process execution log details for triggered job (using view model): [HttpGet("api/v1/processexecutionlogs/view/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing the process execution log details for a specific job
      - Add a process execution log for a job: [HttpPost("api/v1/processexecutionlogs]
        - Payloads
          - Input : ProcessExecutionLog
          - Output : JSON file with the created log
      - Add process execution log for a started job: [HttpPost("api/v1/processexecutionlogs/startprocess]
        - Payloads
          - Input : ProcessExecutionLog
          - Output : JSON file with the created log
      - Edit process execution log for a triggered job: [HttpPut("api/v1/processexecutionlogs/{id}]
        - Payloads
          - Input : ProcessExecutionLog
          - Output : JSON file with updated values
       - Edit process execution log for a completed job: [HttpPut("api/v1/processexecutionlogs/{id}/endprocess]
        - Payloads
          - Input : ProcessExecutionLog
          - Output : JSON file with updated values
       - Delete process execution log: [HttpDelete("api/v1/processexecutionlogs/{id}")]
         - Payloads
           - Input : Process Execution Log id
           - Output : 200 OK response
       - Edit process execution log property: [HttpPatch("api/v1/processexecutionlogs/{id}")]
         - Payloads
           - Input : Process Execution Log id
           - Output : 200 OK response
  - ProcessExecutionLogManager:
   - The ProcessExecutionLogManager will inherit BaseManager, which inherits IManager, and IProcessExecutionLogManager.
      - Beyond the base class and interface, ExecutionLogManager will implement GetExecutionView() and GetProcessAndAgentNames() methods to assist ProcessExecutionLogsController.
  - ProcessExecutionLogRepository:
    - ProcessExecutionLogRepository will inherit EntityRepository<ProcessExecutionLog>, which inherits ReadOnlyEntityRepository, and IProcessExecutionLogRepository.
    - In addition to the methods in the base class, the ProcessExecutionLogRepository will retrieve process execution logs using a view model.
  - ProcessExecutionLog Data Model:
    - The ProcessExecutionLog data model will be used to view details of each job's execution logs.  It will inerit the Entity base class.
      - In addition to the properties from the base Entity class, ProcessExecutionLog will have Guid JobId, Guid ProcessId, Guid AgentId, DateTime StartedOn, DateTime CompletedOn, string Trigger, string TriggerDetails, string Status, string HasErrors, string ErrorMessage, and string ErrorDetails.

**Sequence Diagrams**

- [Process Execution Log Component Sequence Diagram](Sequence Diagrams/ExecutionLogComponentSequenceDiagram.png)
- [Count Process Execution Log Sequence Diagram](Sequence Diagrams/CountProcessExecutionLogSequenceDiagram.png)
- [Process Execution Log Details Sequence Diagram](Sequence Diagrams/CountProcessExecutionLogSequenceDiagram.png)
- [Create Process Execution Log Sequence Diagram](Sequence Diagrams/CreateProcessExecutionLogSequenceDiagram.png)
- [Edit Process Execution Log Sequence Diagram](Sequence Diagrams/EditProcessExecutionLogPropertySequenceDiagram.png)
- [Edit Process Execution Log Property Sequence Diagram](Sequence Diagrams/EditProcessExecutionLogPropertySequenceDiagram.png)
- [Delete Process Execution Log Sequence Diagram](Sequence Diagrams/DeleteProcessExecutionLogSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - N/A
- Negative Test Cases:
  - N/A
