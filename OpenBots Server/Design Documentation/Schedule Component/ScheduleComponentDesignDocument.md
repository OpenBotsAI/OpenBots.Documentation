Author: Nicole Carrero
Creation Date:  8/19/2020

Updated On: 01/28/2021
Updated By: Dairon Hernandez

**Schedule Component**

**Context**

- Problem: Users cannot schedule when a Robot Job is performed.  An administrator should be able to define a trigger that will be utilized for the scheduling of Robot Jobs.  An administrator should be able to view a list of existing triggers and edit or delete them.  An administrator should be able to utilize time-based triggering through CRON expressions and disable triggers as needed.
- Requirements: Ensure than an administrator is able to create schedules with a set name and type and utilize CRON expressions to perform schedule-based triggering.  Ensure that an administrator and user are able to view and edit a schedule from a list of all active schedules and disable them as needed.

**Component Scope**

- In-Scope:
  - Administrator can create schedules with a set name and type.
  - Administrator can utilize CRON expressions to perform shedule-based triggering.
  - Administrator and user can view/edit from a list of all active schedules and disable them as needed.
  - Administrator can delete a schedule from a list. 
- Out-of-Scope:
  - User can create schedules with a set name and type.
  - User can utilize CRON expressions to perform schedule-based triggering.

**Design**

- Overview:  The overall structure uses the SchedulesController to allow the user to make a web request for the list of schedules, schedule details, to create a new schedule, or edit/delete an existing shedule.  The SchedulesController gets called, which in turn accesses the methods in the ScheduleManager, and then uses the ScheduleRepository to retrieve, add, edit, or delete the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of Schedules.
    - The schedules will be displayed in a table.
      - There will be a button to add a new schedule to the list.
      - The headers will be: Name, Agent Name, Automation Name, Status, View, Edit, Delete.
        - The appropriate data will be displayed underneath each heading for all schedules.
        - The user can click on each schedule to view, edit, or delete it.
  - SchedulesController:
    - The SchedulesController will make an API request and use the ScheduleManager to access the ScheduleRepository to retrieve all the schedules from the Server and will return that information back to the view.  The controller will utilize this same structure to view, add, edit, or remove a schedule.
    - Routes:
      - All schedules: [HttpGet("api/v1/schedules")]
        - Payloads
          - Input : None
          - Output : JSON file listing all schedule information
      - Count schedules: [HttpGet("api/v1/schedules/count")]
        - Payloads
          - Input : None
          - Output : Count of all schedules in the Server
      - Schedule Details: [HttpGet("api/v1/schedules/{id}")]
        - Payloads
          - Input : Schedule id
          - Output : JSON file listing schedule information
      - Schedule Details: [HttpGet("api/v1/schedules/{id}/View")]
        - Payloads
          - Input : Schedule id
          - Output : JSON file listing schedule information with additional viewmodel details
      - Create a schedule: [HttpPost("api/v1/schedules")]
        - Payloads
          - Input : Schedule data model
          - Output : 200 OK response
      - Edit a schedule: [HttpPut("api/v1/schedules/{id}")]
        - Payloads
          - Input : Schedule data model
          - Output : 200 OK response
      - Delete a schedule: [HttpDelete("api/v1/scheduled/{id}")]
        - Payloads
          - Input : Schedule id
          - Output : 200 OK response
      - Edit a schedule property: [HttpPatch("api/v1/schedules/{id}")]
        - Payloads
          - Input : JSONPatchDocument in request body with changes
          - Output : 200 OK response
      - API to run a Job "NOW": [HttpPatch("api/v1/schedules/{id}/runnow")]
        - Payloads
          - Input : RunNowViewModel containing automationId and agentId
          - Output : 200 OK response if run to attempt was successful
  - ScheduleManager:
    - The ScheduleManager will inherit BaseManager and IScheduleManager, which both inherit IManager.
      - Beyond the base class and interfaces, ScheduleManager will implement the StartNewRecurringJob(), CreateCommonJob(), CreateJob(), and GetScheduleAgentsandAutomations() methods to assist SchedulesController.
  - ScheduleRepository:
    - The ScheduleRepository will inherit EntityRepository, which inherits ReadOnlyEntityRepository and IScheduleRepository, which inherits IEntityRepository.
      - The ScheduleRepository will retrieve all schedules, add a new schedule, or retrieve/edit/delete a schedule by id.
  - Schedule Data Model:
    - The Schedule data model will be used to view details of each schedule.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Schedule will have Guid AgentId, string AgentName, string CronExpression, DateTime LastExecution, DateTime NextExecution, bool IsDisabled, Guid ProjectId, string TriggerName, bool Recurrence, string StartingType, DateTime StartJobOn, DateTime RecurrenceUnit, DateTime JobRecurEveryUnit,  DateTime EndJobOn, DateTime EndJobAtOccurence, DateTime NoJobEndDate, string Status, DateTime ExpiryDate, DateTime StartDate, and Guid AutomationId.

**Sequence Diagrams**

- [Schedule Component Sequence Diagram](Sequence Diagrams/ScheduleComponentSequenceDiagram.png)
- [Count Schedules Sequence Diagram](Sequence Diagrams/CountSchedulesSequenceDiagram.png)
- [Schedule Details Sequence Diagram](Sequence Diagrams/ScheduleDetailsSequenceDiagram.png)
- [Create Schedule Sequence Diagram](Sequence Diagrams/CreateScheduleSequenceDiagram.png)
- [Edit Schedule Sequence Diagram](Sequence Diagrams/EditScheduleSequenceDiagram.png)
- [Delete Schedule Sequence Diagram](Sequence Diagrams/DeleteScheduleSequenceDiagram.png)
- [Edit Schedule Property Sequence Diagram](Sequence Diagrams/EditScheduleSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - N/A
- Negative Test Cases:
  - N/A