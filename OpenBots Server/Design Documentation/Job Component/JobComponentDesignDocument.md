Author: Nicole Carrero
Creation Date: 8/18/2020

Updated On: 01/28/2021
Updated By: Dairon Hernandez

**Job Component**

**Context**

- Problem: Users are not able to monitor triggered jobs of a particular process.  Administrators should be able to view a list of triggered jobs and access their logs, view the details of a job, and see that execution of jobs are delegated to the Agents at the time of the Heartbeat.
- Requirements: Ensure that an Administrator is able to view a list of all triggered jobs and their statuses and be redirected in order to access the related process and agent details.

**Component Scope**

- In-Scope:
  - Administrator can trigger a job of a process onto a Robot or Environment by creating a new schedule (See Schedule Component).
  - Administrator can view a list of all triggered jobs and their details.
  - Administrator is redirected in order to access the process or agent related to the job.
  - Administrator can delete a job.
- Out-of-Scope:
  - Administrator and/or user can edit a job of a process.
  - User can trigger a job of a process onto a Robot or Environment.
  - User is redirected in order to access the logs of a job through the logging interface.

**Design**

- Overview: The overall structure uses the JobsController to allow the user to make a web request for either the entire list of triggered jobs or an individual job's details.  If the user makes a request, the web request goes to the JobsController, which in turn accesses the JobManager, and then calls the JobRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the jobs could not be found.  The same process will take place when creating, editing, or deleting a job.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of triggered jobs.
    - The jobs will be displayed in a table with the most recently triggered job at the top.
      - The headers will be: Agent Name, Process Name, Start Time, End Time, Job Status, Successful, and View.
      - The appropriate data will be displayed underneath each heading for all jobs.
      - The user can click on each job, where the user will be redirected to a view of that specific job's details.
        - The user can click on links to redirect to either the Process or Agent details.
  - JobsController:
    - The JobsController will make an API request, access the JobRepository to retrieve all the jobs from the Server, and will return that information back to the view.  The controller will utilize this same structure to create, edit, or delete a job.
      - All jobs: [HttpGet("api/v1/jobs")]
        - Payloads
          - Input : None
          - Output : JSON file listing all triggered job information
      - All jobs (uses view model): [HttpGet("api/v1/jobs/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing triggered job information
      - Count all jobs: [HttpGet("api/v1/jobs/count")]
        - Payloads
          - Input : None
          - Output : Count of all jobs
      - Count all jobs by status [HttpGet("api/v1/jobs/countbystatus")]
        - Payloads
          - Input : None
          - Output : Count of all jobs grouped by status
      - Job / Agent lookup: [HttpGet("api/v1/jobs/jobagentslookup")]
        - Payloads
          - Input : None
          - Output : JSON file of related process and agent information
      - Individual job details: [HttpGet("api/v1/jobs/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file of all individual job information
      - Individual job details (uses view model): [HttpGet("api/v1/jobs/view/{id}")]
        - Payloads
          - Input : Job id
          - Output : JSON file listing  individual job information
      - Get next available job for agent: [HttpGet("api/v1/jobs/next")]
        - Payloads
          - Input : Agent id
          - Output : JSON file listing next available job information
      - Create a job: [HttpPost("api/v1/jobs")]
        - Payloads
          - Input : Job model
          - Output : JSON file listing new job information
      - Edit a job: [HttpPut("api/v1/jobs/{id}")]
        - Payloads
          - Input : Job model
          - Output : JSON file listing updated job information
      - Edit job with specified status (uses view model): [HttpPut("api/v1/jobs/{id}/status/{status}")]
        - Payloads
          - Input : Job id, Job status, Agent id, job errors (error reason, error code, serailized error)
          - Output : JSON file listing updated job information
      - Delete a job: [HttpDelete("api/v1/jobs/{id}")]
        - Payloads
          - Input : Job id
          - Output : 200 OK response
      - Edit job property: [HttpPatch("api/v1/jobs/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Create a job checkpoint: [HttpPost("api/v1/jobs/JobId/,JobCheckpoint")]
      - Payloads
        - Input : JobCheckpoint model
        - Output : JSON file listing new job checkpoint information
      - All Checkpoints belonging to the specified JobId: [HttpGet("api/v1/jobs/JobId/JobCheckpoint")]
        - Payloads
          - Input : jobId 
          - Output : JSON file listing checkpoints for the specified jobId
  - JobManager:
    - The JobManager will inherit BaseManager, which inherits IManager, and IJobManager.
      - Beyond the base class and interfaces, JobManager will implement GetJobView(), GetJobAgentsLookup(), GetJobsAgentsAndProcesses(), and GetNextJob() methods to assist JobsController.
  - JobRepository:
    - The JobRepository will inherit EntityRepository<Job>, which inherits EntityRepository, and IJobRepository.
      - Beyond the base classes and interface, JobRepository will retrieve all jobs, find an individual job's details, and find related properties from the Agents and Processes data tables.
    - JobParameterRepository will be resposnible for updating and retreiving records from the JobParameters table
    - JobCheckpointRepository will be resposnible for updating and retreiving records from the JobCheckpoint table
  - Job Data Model:
    - The Job data model will be used to view details of each triggered job.  It will inherit the Entity class and INonAuditable interface.
      - Beyond the base class, Job will have string AgentId, DateTime StartTime, DateTime EndTime, double ExecutionTimeInMinutes, DateTime EnqueueTime, DateTime Dequeue Time, Guid Process Id, enum JobStatusType JobStatus, string Message, bool IsSuccessful, string ErrorReason, string ErrorCode, and string SerializedErrorString.
    - The JobParameter data model contains detials about the paramters that belong to a particular job. JobParameter inherits from NamedEnity and contains the fields string Datatype, string Value, and GUID jobId.
    - The JobCheckpoint data model contains information about the current state of a job's checkpoint. The JobCheckpoints are Non-Auditable and inherit from Named Entity. The model contains the fields: string Iterator, string IteratorValue, string IteratorPosition, string IteratorCount, GUID JobId

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
    - GetNextJob() retreives the next available Job for the specified agent ID
    - We are able to get all JobParameters for the specified Job Id
- Negative Test Cases:
    - GetNextJob() returns an empty enumerable if the agentId provided does not have any available jobs
    - We are not able to get any JobParameters if the parameters for the speicifed id have been deleted