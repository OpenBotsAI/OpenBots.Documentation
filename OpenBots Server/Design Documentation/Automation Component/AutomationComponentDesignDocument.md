Author: Nicole Carrero
Creation Date: 8/14/2020

Updated On: 3/9/2021
Updated By: Dairon Hernandez

**Automation Component**

**Context**

- Problem: Users are unable to view automated processes created through the Studio.  An admisistrator should be able to view, edit, upload, export, and delete automations on the Server.  Published operations from Studio should be uploaded to this location.  Uploaded automations should be stored locally to the Server.  The currently stored version should be exported in its original uploaded format.  A user should be able to view business automations that are stored in the Server.
- Requirements: Ensure that an administrator is able to upload automations either manually or through publishing from a Studio instance, view/edit/delete existing automations on the Server, and export a stored automation for reuse.

**Component Scope**

- In-Scope:
  - Administrator can upload an automation manually.
  - Administrator can upload an automation through publishing a Studio instance.
  - Administrator can edit or delete an existing automation.
  - Administrator can export a stored automation for reuse.
  - Administrator and user can view existing automations on the Server.
- Out-of-Scope:
  - User can upload an automation manually.
  - User can upload an automation through publishing a Studio instance.
  - User can edit or delete an existing automation.
  - User can export a stored automation for reuse.
  - Administrator and/or user can create an automation on the Server.

**Design**

- Overview: The overall structure uses the AutomationsController to allow the user to make a web request for the list of automations, to upload/export an automation, or view/edit/delete an existing automation.  The AutomationsController gets called, which in turn accesses the methods in the AutomationManager, and then uses the AutomationRepository to retrieve, add, edit, or delete the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of Automations.
      - The automations will be displayed in a table.
      - The headers will be: Automation Name (including version number), Created On, Created By, Status, Jobs, View, Edit, and Delete.
        - The appropriate data will be displayed underneath each heading for all automations.
        - The administrator can click on each automation to view corresponding jobs, edit, delete, or export it for reuse.
      - Automations that are uploaded through Studio will automatically populate into the table.
    - On the same page, administrators will have the option to add/upload an automation manually.
  - AutomationsController:
    - The AutomationsController will make an API request and use the AutomationManager to access the AutomationRepository to retrieve all the automations from the Server and will return that information back to the view.  The controller will utilize this same structure to upload, export, edit, or delete an automation.
    - Routes:
      - All automations: [HttpGet("api/v1/automations")]
        - Payloads
          - Input : None
          - Output : JSON file listing all automation information
      - All automations (view model) [HttpGet("api/v1/automations/view")]
        - Payloads
          - Input : None
          - Output : JSON file listing all automation view model information
      - Count automations: [HttpGet("api/v1/automations")]
        - Payloads
          - Input : None
          - Output : Number of automations in Server
      - Automation details: [HttpGet("api/v1/automations/{id}")]
        - Payloads
          - Input : Automation id
          - Output : JSON file listing specific automation information
      - Automation details (view model): [HttpGet("api/v1/automations/view/{id}")]
        - Payloads
          - Input : Automation id
          - Output : JSON file listing specific automation view model information
      - Add an automation: [HttpPost("api/v1/automations")]
        - Payloads
          - Input : Automation model data (name, automation engine, file, and optional drive name)
          - Output : JSON file listing new automation information
      - Upload automation file: [HttpPost("api/v1/automations/{id}/upload")]
        - Payloads
          - Input : Automation file
          - Output : JSON file listing updated automation information
      - Update automation with file (create new automation with updated version): [HttpPost("api/v1/automations/{id}/update")]
        - Payloads
          - Input : Automation id, file, name, drive name (optional)
          - Output : JSON file listing updated automation information
      - Update automation: [HttpPut("api/v1/automations/{id}")]
        - Payloads
          - Input : Automation id, Automation name
          - Output : JSON file listing updated automation information
      - Export an automation: [HttpGet("api/v1/automations/{id}")]
        - Payloads
          - Input : Automation id
          - Output : Automation file
      - Edit automation property: [HttpPatch("api/v1/automations/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete an automation: [HttpDelete("api/v1/automations/{id}")]
        - Payloads
          - Input : Automation id, drive name (optional)
          - Output : 200 OK response
      - Automation lookup: [HttpGet("api/v1/automations/getlookup")]
        - Payloads
          - Input : None
          - Output : JSON file listing automation id, name, and name with version
      - Update Automation Parameters: [HttpPost("api/v1/automations/automationId/UpdateParameters")]
        - Payloads
          - Input : JSON body containing a list of automation parameters
          - Output : JSON file listing new automation parameters
  - AutomationManager:
    - The AutomationManager will inherit BaseManager, which inherits IManager, and IAutomationManager.
      - Beyond the base class and interfaces, AutomationManager will implement appropriate methods to assist AutomationController
- Repositories:
  - AutomationRepository:
    - AutomationRepository will inherit EntityRepository<Automation>, which inherits ReadOnlyEntityRepository<Automation>, and IAutomationRepository.
      - The AutomationRepository will retrieve all automations, add/edit a new automation, or retrieve/delete an automation by id.  It will use FindAllView method to find the view model properties to display in the return of the get automations request.
  - AutomationVersionRepository:
    - The AutomationVersionRepository will inherit EntityRepository<AutomationVersion>, which inherits ReadOnlyEntityRepository<AutomationVersion>, and IAutomationVersionRepository.
      - The AutomationVersionRepository will retrieve all automation versions, add/edit a new automation version, or retrieve/delete an automation version by id.
- Models:
  - Automation Data Model:
    - The Automation data model will be used to view details of each automation.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Automation will have Guid FileId, string OriginalPackageName, string AutomationEngine, double AverageSuccessful ExecutionInMinutes, and double AverageUnSuccessfulExecutionInMinutes.
  - Automation Version Data Model:
    - The AutomationVersion data model will be used to view details about the automation version of each automation.  It will inherit the Entity class.
      - Beyond the base class, AutomationVersion will have Guid AutomationId, int VersionNumber, string PublishedBy, DateTime PublishedOnUTC, and string Status.
  - AutomationParameter Data Model:
    - The AutomationParameters will be used to store parameter information associated with a particular Automation, which will be utilized when executing a job.
      - Beyond the base class, AutomationParameter will have Guid AutomationId, string Name, string Value, and string DataType.
  - AllAutomationsViewModel View Model:
    - The AllAutomationsVieWModel view model will be used to view details of each automation in addition to the details of each automation version.  It will inherit IViewModel<Automation, AllAutomationsViewModel>.
      - Beyond the interface, AllAutomationsViewModel will have Guid Id, string Name, DateTime CreatedOn, string CreatedBy, string Status, and int VersionNumber.
      - It will have a Map method to map a specific automation to the view model.
  - AutomationViewModel View Model:
    - The AutomationViewModel view model will be used to view details of an individual automation in addition to the details of its automation version.  It will inherit NamedEntity, which inherits Entity, and IViewModel<Automation, AutomationViewModel>.
      - Beyond the base classes and interface, AutomationViewModel will have int VersionNumber, Guid VersionId, string Status, IFormFile File, Guid FileId, string OriginalPackageName, string PublishedBy, DateTime PublishedOnUTC, string AutomationEngine, and string DriveName.
      - It will have a Map method to map a specific automation to the view model.

**Sequence Diagrams**

- [Automation Component Sequence Diagram](Sequence Diagrams/AutomationComponentSequenceDiagram.png)
- [Count Automations Sequence Diagram](Sequence Diagrams/CountAutomationsSequenceDiagram.png)
- [Automation Details Sequence Diagram](Sequence Diagrams/AutomationDetailsSequenceDiagram.png)
- [Add Automation Sequence Diagram](Sequence Diagrams/AddAutomationSequenceDiagram.png)
- [Upload Automation Sequence Diagram](Sequence Diagrams/UploadAutomationSequenceDiagram.png)
- [Edit Automation File Sequence Diagram](Sequence Diagrams/CountAutomationsSequenceDiagram.png)
- [Edit Automation Sequence Diagram](Sequence Diagrams/EditAutomationSequenceDiagram.png)
- [Export Automation Sequence Diagram](Sequence Diagrams/ExportAutomationComponentSequenceDiagram.png)
- [Edit Automation Property Sequence Diagram](Sequence Diagrams/CountAutomationsSequenceDiagram.png)
- [Delete Automation Sequence Diagram](Sequence Diagrams/CountAutomationsSequenceDiagram.png)
- [Automations Lookup Sequence Diagram](Sequence Diagrams/AutomationsLookupSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A