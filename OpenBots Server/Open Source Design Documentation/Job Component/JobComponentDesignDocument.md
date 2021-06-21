Author: NC
Creation Date: 8/18/2020

Updated By: NC
Updated On: 6/16/2021

**Job Component**

**Context**

- Problem: Users are not able to monitor triggered jobs of a particular automation.  Administrators should be able to view a list of triggered jobs and access their logs, view the details of a job, and see that execution of jobs are delegated to the Agents at the time of the Heartbeat.
- Requirements: Ensure that an Administrator is able to view a list of all triggered jobs and their statuses and be redirected in order to access the related automation and agent details.

**Component Scope**

- In-Scope:
  - Administrator can trigger a job of an automation onto a Robot or Environment by creating a new schedule (See Schedule Component).
  - Administrator can view a list of all triggered jobs and their details.
  - Administrator is redirected in order to access the automation or agent related to the job.
  - Administrator can delete a job.
- Out-of-Scope:
  - Administrator and/or user can edit a job of an automation.
  - User can trigger a job of a automation onto a Robot or Environment.
  - User is redirected in order to access the logs of a job through the logging interface.

**Design**

- Overview: The overall structure uses the JobsController to allow the user to make a web request for either the entire list of triggered jobs or an individual job's details.  If the user makes a request, the web request goes to the JobsController, which in turn accesses the JobManager, and then calls the JobRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the jobs could not be found.  The same process will take place when creating, editing, or deleting a job.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of triggered jobs.
    - The jobs will be displayed in a table with the most recently triggered job at the top.
      - The headers will be: Agent Name, Automation Name, Start Time, End Time, Job Status, Successful, and View.
      - The appropriate data will be displayed underneath each heading for all jobs.
      - The user can click on each job, where the user will be redirected to a view of that specific job's details.
        - The user can click on links to redirect to either the Automation or Agent details.
  - JobsController:
    - The JobsController will make an API request, access the JobRepository to retrieve all the jobs from the Server, and will return that information back to the view.  The controller will utilize this same structure to create, edit, or delete a job.
    - NOTE: The current API version is 1.
    - Routes:
      - All jobs: [HttpGet("api/v{apiVersion}/jobs")]
        - Payloads
          - Input : None
          - Output : JSON file listing all triggered job information
      - All jobs view: [HttpGet("api/v{apiVersion}/jobs/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing triggered job information
      - Count all jobs: [HttpGet("api/v{apiVersion}/jobs/count")]
        - Payloads
          - Input : None
          - Output : Count of all jobs
      - Count all jobs by status: [HttpGet("api/v{apiVersion}/jobs/countbystatus")]
        - Payloads
          - Input : None
          - Output : Count of all jobs grouped by status
      - Job/Agent lookup: [HttpGet("api/v{apiVersion}/jobs/jobagentslookup")]
        - Payloads
          - Input : None
          - Output : JSON file of related automation and agent information
      - Job details: [HttpGet("api/v{apiVersion}/jobs/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file of all individual job information
      - Job details view: [HttpGet("api/v{apiVersion}/jobs/view/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing individual job information view
      - Export all jobs: [HttpGet("api/v{apiVersion}/jobs/export/{fileType?}")]
        - Payloads
          - Input : File type specifying the if the response should be zip, CSV, or Json file
          - Output : Zip, CSV, or Json file containing all retrieved jobs
      - Sum of all jobs: [HttpGet("api/v{apiVersion}/jobs/sum")]
        - Payloads
          - Input : None
          - Output : Sum of executionTimeInMinutes, automationExecutionLogCount and automationLogCount for the current jobs
      - Create a job: [HttpPost("api/v{apiVersion}/jobs")]
        - Payloads
          - Input : Job data model
          - Output : JSON file listing new job information
      - Edit a job: [HttpPut("api/v{apiVersion}/jobs/{id}")]
        - Payloads
          - Input : Job data model
          - Output : JSON file listing updated job information
      - Edit job with specified status: [HttpPut("api/v{apiVersion}/jobs/{id}/status/{status}")]
        - Payloads
          - Input : Agent id, JobErrorViewModel
          - Output : JSON file listing updated job information
      - Delete a job: [HttpDelete("api/v{apiVersion}/jobs/{id}")]
        - Payloads
          - Input : Job id
          - Output : 200 OK response
      - Edit job property: [HttpPatch("api/v{apiVersion}/jobs/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Create a job checkpoint: [HttpPost("api/v{apiVersion}/jobs/{id}/AddCheckpoint")]
        - Payloads
          - Input : JobCheckpoint model and job id
          - Output : JSON file listing new job checkpoint information
      - Job checkpoints: [HttpGet("api/v{apiVersion}/jobs/JobId/JobCheckpoint")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing checkpoints for the specified job id
  - Job Manager:
    - The JobManager will inherit BaseManager, which inherits IManager, and IJobManager.
      - Beyond the base class and interfaces, JobManager will implement appropriate methods to assist JobsController.
  - Repositories:
    - The JobRepository will inherit EntityRepository<Job>, which inherits EntityRepository, and IJobRepository.
      - Beyond the base classes and interface, JobRepository will retrieve all jobs, find an individual job's details, and find related properties from the Agents and Automations data tables.
    - The JobParameterRepository will be responsible for updating and retreiving records from the JobParameters table.  It will inherit EntityRepository<JobParameter> and IJobParameterRepository.
    - The JobCheckpointRepository will be responsible for updating and retreiving records from the JobCheckpoint table.  It will inherit EntityRepository<JobCheckpoint> and IJobCheckpointRepository.
  - Models:
    - Job Data Model:
      - The Job data model will be used to view details of each triggered job.  It will inherit the Entity class and INonAuditable interface.
        - Beyond the base class and interface, Job will have Guid ScheduleId, Guid AgentId, Guid AgentGroupId, DateTime StartTime, DateTime EndTime, long ExecutionTimeInMinutes, DateTime EnqueueTime, DateTime Dequeue Time, Guid Automation Id (required), int AutomationVersion (required), Guid AutomationVersionId (required), JobStatusType JobStatus, string Message, bool IsSuccessful, string ErrorReason, string ErrorCode, string SerializedErrorString, int AutomationLogCount, and int AutomationExecutionLogCount.
          - JobStatusType is an emun of type int with values Unknown (0), New (1), Assigned (2), InProgress (3), Failed (4), Completed (5), Stopping (6), and Abandoned (9).
    - The JobParameter data model contains details about the parameters that belong to a particular job. JobParameter inherits from NamedEntity, which inherits Entity.
      - Beond the base classes, it will have string Datatype, string Value, and Guid JobId.
    - The JobCheckpoint data model contains information about the current state of a job's checkpoint. It inherits from NamedEntity, which inherits Entity, and INonAuditable.
      - Beyond the base classes, it will have string Message, string Iterator, string IteratorValue, string IteratorPosition, int IteratorCount, and Guid JobId.
    - The AllJobsViewModel will be used to view additional details of all jobs.  It inherits IViewModel<Job, AllJobsViewModel>.
      - It will have Guid Id, string AgentName, string AgentGroupName, string AutomationName, Guid ScheduleId, Guid AgentId, Guid AgentGroupId, DateTime StartTime, DateTime EndTime, long ExecutionTimeInMinutes, DateTime EnqueueTime, DateTime DequeueTime, Guid AutomationId (required), int AutomationVersion (required), Guid AutomationVersionId (required), JobStatusType JobStatus, string Message, bool IsSuccessful, DateTime CreatedOn, string CreatedBy, string ErrorReason, string ErrorCode, string SerializedErrorString, int AutomationLogCount, and int AutomationExecutionLogCount.
  - The JobViewModel will be used to view additional details of a job.  It inherits IViewModel<Job, JobViewModel>.
    - It will have Guid Id, string AgentName, string AgentGroupName, string AutomationName, Guid ScheduleId, Guid AgentId, Guid AgentGroupId, DateTime StartTime, DateTime EndTime, long ExecutionTimeInMinutes, DateTime EnqueueTime, DateTime DequeueTime, Guid AutomationId (required), int AutomationVersion (required), Guid AutomationVersion (required), JobStatusType JobStatus, string Message, bool IsSuccessful, DateTime CreatedOn, string CreatedBy, string ErrorReason, string ErrorCode, string SerializedErrorString, int AutomationLogCount, int AutomationExecutionLoCount, and IEnumerable<JobParameter> JobParameters.
  - The CreateJobViewModel will be used to create new jobs.  It inherits IViewModel<CreateJobViewModel, Job>.
    - It will have Guid Id, Guid ScheduleId, Guid AgentId, Guid AgentGroupId, DateTime StartTime, DateTime EndTime, DateTime EnqueueTime, DateTime DequeueTime, Guid AutomationId (required), JobStatusType JobStatus, string Message, bool IsSuccessful, IEnumerable<JobParameter> JobParameters.
  - The JobsLookupViewModel will be used to view a list of agents and automations for jobs.
    - It will have List<JobAgentsLookup> AgentsLookup and List<JobAutomationLookup>.
      - JobAgentsLookup will have Guid AgentId and string AgentName.
      - JobAutomationLookup will have Guid AutomationId, string AutomationName, and string AutomationNameWithVersion.
  - The JobErrorViewModel will be used to view errors that occur during a job.
    - It will have string ErrorReason, string ErrorCode, and string SerializedErrorString.

**Sequence Diagrams**

- [Job Component Sequence Diagram](Sequence Diagrams/JobComponentSequenceDiagram.png)
- [Count Jobs Sequence Diagram](Sequence Diagrams/CountJobsSequenceDiagram.png)
- [Count Jobs by Status Sequence Diagram](Sequence Diagrams/CountJobsByStatusSequenceDiagram.png)
- [Job & Agent Lookup Sequence Diagram](Sequence Diagrams/JobAgentLookupSequenceDiagram.png)
- [Job Details Sequence Diagram](Sequence Diagrams/JobDetailsSequenceDiagram.png)
- [Get Next Job for Agent Sequence Diagram](Sequence Diagrams/GetNextJobForAgentSequenceDiagram.png)
- [Create Job Sequence Diagram](Sequence Diagrams/CreateJobSequenceDiagram.png)
- [Edit Job Sequence Diagram](Sequence Diagrams/EditJobSequenceDiagram.png)
- [Edit Job Property Sequence Diagram](Sequence Diagrams/EditJobPropertySequenceDiagram.png)
- [Delete Job Sequence Diagram](Sequence Diagrams/DeleteJobSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
    - GetNextJob method retreives the next available Job for the specified agent ID.
    - We are able to get all JobParameters for the specified Job Id.
- Negative Test Cases:
    - GetNextJob method returns an empty enumerable if the agentId provided does not have any available jobs.
    - We are not able to get any JobParameters if the parameters for the speicifed id have been deleted.