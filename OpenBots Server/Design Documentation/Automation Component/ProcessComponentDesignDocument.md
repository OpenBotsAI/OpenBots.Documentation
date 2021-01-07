Author: Nicole Carrero
Creation Date: 8/14/2020

Updated On: 11/24/2020
Updated By: Nicole Carrero

**Process Component**

**Context**

- Problem: Users are unable to view automated processes created through the Studio.  An admisistrator should be able to view, edit, upload, export, and delete processes on the Server.  Published operations from Studios should be uploaded to this location.  Uploaded processes should be stored locally to the Server.  The currently stored version should be exported in its original uploaded format.  A user should be able to view business processes that are stored in the Server.
- Requirements: Ensure that an administrator is able to upload processes either manually or through publishing from a Studio instance, view/edit/delete existing processes on the Server, and export a stored process for reuse.

**Component Scope**

- In-Scope:
  - Administrator can upload a process manually.
  - Administrator can upload a process through publishing a Studio instance.
  - Administrator can edit or delete an existing process.
  - Administrator can export a stored process for reuse.
  - Administrator and user can view existing processes on the Server.
- Out-of-Scope:
  - User can upload a process manually.
  - User can upload a process through publishing a Studio instance.
  - User can edit or delete an existing process.
  - User can export a stored process for reuse.
  - Administrator and/or user can create an automated process on the Server.

**Design**

- Overview: The overall structure uses the ProcessesController to allow the user to make a web request for the list of processes, to upload/export a process, or view/edit/delete an existing process.  The ProcessesController gets called, which in turn accesses the methods in the ProcessManager, and then uses the ProcessesRepository to retrieve, add, edit, or delete the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of Processes.
      - The processes will be displayed in a table.
      - The headers will be: Process Name (including version number), Created On, Created By, Status, Jobs, View, Edit, and Delete.
        - The appropriate data will be displayed underneath each heading for all processes.
        - The administrator can click on each process to view corresponding jobs, edit, delete, or export it for reuse.
      - Processes that are uploaded through Studio will automatically populate into the table.
    - On the same page, administrators will have the option to add/upload a process manually.
  - ProcessesController:
    - The ProcessesController will make an API request and use the ProcessManager to access the ProcessRepository to retrieve all the processes from the Server and will return that information back to the view.  The controller will utilize this same structure to upload, export, edit, or delete a process.
    - Routes:
      - All processes: [HttpGet("api/v1/processes")]
        - Payloads
          - Input : None
          - Output : JSON file listing all process information
      - All processes (view model) [HttpGet("api/v1/processes/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing all process view model information
      - Count processes: [HttpGet("api/v1/processes")]
        - Payloads
          - Input : None
          - Output : Number of processes in Server
      - Process details: [HttpGet("api/v1/processes/{id}")]
        - Payloads
          - Input : Process id
          - Output : JSON file listing specific process information
      - Process details (view model): [HttpGet("api/v1/processes/view/{id}")]
        - Payloads
          - Input : Process id
          - Output : JSON file listing specific process view model information
      - Add a process: [HttpPost("api/v1/processes")]
        - Payloads
          - Input : Process model data
          - Output : JSON file listing new process information
      - Upload process file: [HttpPost("api/v1/processes/{id}/upload")]
        - Payloads
          - Input : Process file
          - Output : JSON file listing updated process information
      - Update process with file (create new process with updated version): [HttpPost("api/v1/processes/{id}/update")]
        - Payloads
          - Input : Process id, Process file
          - Output : JSON file listing updated process information
      - Update process: [HttpPut("api/v1/processes/{id}")]
        - Payloads
          - Input : Process id, Process name
          - Output : JSON file listing updated process information
      - Export a process: [HttpGet("api/v1/processes/{id}")]
        - Payloads
          - Input : Process id
          - Output : Process file
      - Edit process property: [HttpPatch("api/v1/processes/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete a process: [HttpDelete("api/v1/processes/{id}")]
        - Payloads
          - Input : Process id
          - Output : 200 OK response
      - Process lookup: [HttpGet("api/v1/processes/getlookup")]
        - Payloads
          - Input : None
          - Output : JSON file listing process id, name , and name with version
  - ProcessManager:
    - The ProcessManager will inherit BaseManager, which inherits IManager, and IProcessManager.
      - Beyond the base class and interfaces, ProcessManager will implement Export(), DeleteProcess(), UpdateProcess(), Update(), GetOrganizationId(), AddProcessVersion(), and GetProcessesAndProcessVersions() methods to assist ProcessesController.
- Repositories:
  - ProcessRepository:
    - ProcessRepository will inherit EntityRepository<Process>, which inherits ReadOnlyEntityRepository<Process>, and IProcessRepository.
      - The ProcessRepository will retrieve all processes, add/edit a new process, or retrieve/delete a process by id.  It will use FindAllView() to find the view model properties to display in the return of the get processes request.
  - ProcessVersionRepository:
    - The ProcessVersionRepository will inherit EntityRepository<ProcessVersion>, which inherits ReadOnlyEntityRepository<ProcessVersion>, and IProcessVersionRepository.
      - The ProcessVersionRepository will retrieve all process versions, add/edit a new process version, or retrieve/delete a process version by id.
- Models:
  - Process Data Model:
    - The Process data model will be used to view details of each process.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Process will have Guid BinaryObjectId and string OriginalPackageName.
  - Process Version Data Model:
    - The ProcessVersion data model will be used to view details about the process version of each process.  It will inherit the Entity class.
      - Beyond the base class, ProcessVersion will have Guid ProcessId, int VersionNumber, string PublishedBy, DateTime PublishedOnUTC, and string Status.
  - AllProcessesViewModel View Model:
    - The AllProcessessVieWModel view model will be used to view details of each process in addition to the details of each process version.  It will inherit IViewModel<Process, AllProcessesViewModel>.
      - Beyond the interface, AllProcessesViewModel will have Guid Id, string Name, DateTime CreatedOn, string CreatedBy, string Status, and int VersionNumber.
  - ProcessViewModel View Model:
    - The ProcessViewModel view model will be used to view details of an individual process in addition to the details of its process version.  It will inherit NamedEntity, which inherits Entity, and IViewModel<Process, ProcessViewModel>.
      - Beyond the base classes and interface, ProcessViewModel will have int VersionNumber, Guid VersionId, string Status, IFormFile File, Guid BinaryObjectId, string OriginalPackageName, string PublishedBy, and DateTime PublishedOnUTC.

**Sequence Diagrams**

- [Process Component Sequence Diagram](Sequence Diagrams/ProcessComponentSequenceDiagram.png)
- [Count Processes Sequence Diagram](Sequence Diagrams/CountProcessesSequenceDiagram.png)
- [Process Details Sequence Diagram](Sequence Diagrams/ProcessDetailsSequenceDiagram.png)
- [Add Process Sequence Diagram](Sequence Diagrams/AddProcessSequenceDiagram.png)
- [Upload Process Sequence Diagram](Sequence Diagrams/UploadProcessSequenceDiagram.png)
- [Edit Process File Sequence Diagram](Sequence Diagrams/EditProcessFileSequenceDiagram.png)
- [Edit Process Sequence Diagram](Sequence Diagrams/EditProcessSequenceDiagram.png)
- [Export Process Sequence Diagram](Sequence Diagrams/ExportProcessSequenceDiagram.png)
- [Edit Process Property Sequence Diagram](Sequence Diagrams/EditProcessSequenceDiagram.png)
- [Delete Process Sequence Diagram](Sequence Diagrams/DeleteProcessSequenceDiagram.png)
- [Processes Lookup Sequence Diagram](Sequence Diagrams/ProcessesLookupSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Export a process with a valid id and downloads the file to the appropriate path.
  - Upload the process to the Server.
- Negative Test Cases:
  - Export the process with an invalid id and returns an error.
  - Process does not upload to the Server and returns an error.