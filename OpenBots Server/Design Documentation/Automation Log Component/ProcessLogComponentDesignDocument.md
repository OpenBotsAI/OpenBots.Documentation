Author: Dairon Hernandez
Creation Date: 9/15/2020

Updated On: 10/15/2020
Updated By: Nicole Carrero

**Process Log Component**

**Context**

- Problem: User cannot see process logs of an executed process.  Users should be able to view the logs of a process triggered through the Server. User should be able to view and download all logs for a particular job id to a csv file.  Agent and Studio should be able to create or edit a process log.
- Requirements: Ensure that administrators and users are able to view a list of triggered jobs and view the logs pertaining to that job.  Ensure that administrators and users can view more details about a log and export all logs to a CSV file.  Ensure that Agent and Studio can create or edit a process log.

**Component Scope**

- In-Scope:
  - Users can view process logs pertaining to a triggered job.
  - Users can view more details about a log.
  - Users can export all process logs by job id to a CSV file.
  - Agent/Studio can create or edit a process log pertaining to a triggered job.
- Out-of-Scope:
  - Agent/Studio can delete a process log pertaining to a triggered job.
  - Users can create, edit, or delete logs pertaining to a triggered job.
  - Users can filter logs by their properties, such as level and message.

**Design**

- Overview: The overall structure uses the ProcessLogsController to allow the user to make a web request for the list of logs or individual details.  The ProcessLogsController gets called, which in turn accesses the methods in the ProcessLogManager, and then uses the ProcessLogRepository to retrieve the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.  The same structure will be utilized to create, edit, or export process logs.
- Proposed Solution:
  - User Interface:
    - On the Dashboard, there will be a link to Process Logs.  The process logs will be displayed in a table.
      - The headers will be: Process Name, Machine Name, Message, Message Template, Level, Logger, Jobs, and View.
      - The appropriate data or link will be displayed underneath each heading for all process logs.
        - When a user clicks on the link under Jobs, the user will be redirected to a list of jobs for that process.
        - The user can click on each process log view, where the user will be redirected to a view of that specific process execution log's details based on its id.
          - On the details page, the user can click on a link to redirect to view details of the process itself.
          - All appropriate information will be displayed about each process log.
          - There will be an option to go back to the view of of all process logs for the specific job.
        - There will be a button to export all process logs into a CSV file.
  - ProcessLogsController:
    - The ProcessLogsController will make an API request and use the ProcessLogManager to access the ProcessLogRepository to retrieve all the logs of a specific job from the Server and will return that information back to the view.  The controller will utilize this same structure to export the list of logs to a file, retrieve individual log details, or create/edit a process log.
    - Routes:
      - All process logs: [HttpGet("api/v1/processlogs")]
        - Payloads
          - Input : None
          - Output : JSON file listing all process logs
      - Count of process logs: [HttpGet("api/v1/processlogs/count")]
        - Payloads
          - Input : None
          - Output : Count of all process logs
      - Get individual process log details for triggered job: [HttpGet("api/v1/processlogs/{id}")]
        - Payloads
          - Input : Process log id
          - Output : JSON file with details about the process log for the given id
      - Export process log: [HttpGet("api/v1/processlogs/export/{filetype?}")]
        - Payloads
          - Input : Job id and (optional) file type
          - Output : CSV, Zip, or JSON file listing all process log details
      - Create process log: [HttpPost("api/v1/processlogs")]
        - Payloads
          - Input : ProcessLog model
          - Output : JSON file with the created log
      - Edit process log: [HttpPut("api/v1/processlogs/{id}")]
        - Payloads
          - Input : Process log id
          - Output : JSON file with updated process log information
  - ProcessLogManager:
   - The ProcessLogManager will inherit BaseManager, which inherits IManager, and IProcessLogManager.
      - Beyond the base class and interfaces, ProcessLogManager will implement GetJobLogs() and ZipCsv() to assist the ProcessLogsController, which will be used when exporting process logs.
  - ProcessLogRepository:
   - The ProcessLogRepository will inherit EntityRepository<ProcessLog>, which inherits ReadOnlyEntityRepository, and IProcessLogRepository
     - In addition to the methods inherited from the base class, the ProcessLogRepository will retrieve process logs by job id, or the details of an individual process log.
  - ProcessLog Data Model:
    - The processLog data model will be used to view details of each job's process logs.  It will inherit from Entity.
      - In addition to the properties inherited by the base class, ProcessLog will have string Message, string MessageTemplate, string Level, DateTime ProcessLogTimestamp, string Exception, string Properties, Guid JobId, Guid ProcessId, Guid AgentId, string MachineName, string ProcessName, and string Logger.

**Sequence Diagrams**

- [Process Logs Component Sequence Diagram](Sequence Diagrams/ProcessLogComponentSequenceDiagram.png)
- [Process Log Count Sequence Diagram](Sequence Diagrams/CountProcessLogSequenceDiagram.png)
- [Process Log Details Sequence Diagram](Sequence Diagrams/ProcessLogDetailsSequenceDiagram.png)
- [Export Process Logs Sequence Diagram](Sequence Diagrams/ExportProcessLogsSequenceDiagram.png)
- [Edit Process Log Sequence Diagram](Sequence Diagrams/EditProcessLogSequenceDiagram.png)
- [Create Process Log Sequence Diagram](Sequence Diagrams/CreateProcessLogSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Get list of logs for specified jobId and convert to appropriate file type.
- Negative Test Cases:
  - Does not find logs if the specified id doesn't exist and does not convert to appropriate file type.
