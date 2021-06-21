Author: NC
Creation Date: 8/18/2020

Updated By: NC
Updated On: 6/15/2021

**Credentials Component**

**Context**

- Problem: Users cannot store credentials on the Server.  Administrators should be able to create a credential by defining a type, username, and password.  They should also be able to view a list of existing credentials and edit or delete them.
- Requirements: Ensure that an administrator is able to create a credential entry and provide a username and password to be stored, view a list of existing credentials and edit or delete them, and request the values stored in the credential (with the password provided as a SecureString). Additionally, credentials with agent ids can be created and used by the Agent and Studio.

**Component Scope**

- In-Scope:
  - Administrator can create a credential by defining type, username, and password.
  - Administrator can view a credential's details.
  - Administrator can view a list of existing credentials.
  - Administrator can edit or delete existing credentials.
  - Administrator can add agent credentials.
- Out-of-Scope:
  - User can create a credential by defining type, username, and password.
  - User can view a credential's details.
  - User can view a list of existing credentials.
  - User can edit or delete existing credentials.
  - User can add agent credentials.

**Design**

- Overview: The overall structure uses the CredentialsController to allow the user to make a web request for the list of credentials or to create/update/delete a credential.  The CredentialsController gets called, which in turn acesses the method in the CredentialManager, and then uses the CredentialRepository to retrieve, add, update, or delete the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Types of Credentials:
  - Global credentials: The first time a credential name is used it will create a new global credential. Global credentials cannot be deleted until all child credentials are deleted.
  - Agent credentials: Credentials that contain an agent id and share a name with a global credential. Agent credentials can only be created if a global credential of the same name already exists. 
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the list of credentials.
    - The credentials will be displayed in a table.  Users will be able to view the table after clicking the Dashboard link.
      - The headers will be: Name, Provider, Domain, Username, Start Date, End Date, View, Edit, and Delete.
        - The appropriate data will be displayed underneath each heading for all credentials.
        - The user can click on each credential to view, edit, or delete it.
          - In the edit page, users can also add an agent credential.
      - There will be a button on the page to add a new credential to the list.
  - Credentials Controller:
    - The CredentialsController will make an API request and access the CredentialManager, which in turn calls the CredentialRepository to retrieve the credentials from the Server and will return that information back to the view.  The controller will utilize this same structure to add, edit, or delete a credential.
    - Routes:
    - NOTE: The current API version is 1.
      - All credentials: [HttpGet("api/v{apiVersion}/credentials")]
        - Payloads
          - Input : none
          - Output : JSON file listing all credential information
       All credentials view: [HttpGet("api/v{apiVersion}/credentials/view")]
        - Payloads
          - Input : none
          - Output : JSON file listing all credential details (Id, Name, Provider, StartDate, EndDate, Domain, UserName, Certificate, AgentId, AgentName)
      - Count credentials: [HttpGet("api/v{apiVersion}/credentials/count")]
        - Payloads
          - Input : none
          - Output : Number of credentials in the Server
      - Credential details: [HttpGet("api/v{apiVersion}/credentials/{id}")]
        - Payloads
          - Input : Credential id
          - Output : JSON file listing credential details
      - Credential view details: [HttpGet("api/v{apiVersion}/credentials/view/{id}")]
        - Payloads
          - Input : Credential id
          - Output : JSON file listing specific credential details (Id, Name, Provider, StartDate, EndDate, Domain, UserName, Certificate, AgentId, AgentName)
      - Get credential by name: [HttpGet("api/v{apiVersion}/credentials/GetCredentialByName/{credentialName}")]
        - Payloads
          - Input : Credential name
          - Output : JSON file listing specific credential details (Id, Name, Provider, StartDate, EndDate, Domain, UserName, Certificate)
      - Get credential password: [HttpGet("api/v{apiVersion}/credentials/password/{id}")]
        - Payloads
          - Input : Credential id
          - Output : String value for the credential password
      - Create an agent credential: [HttpPost("api/v{apiVersion}/credentials/AddAgentCredential")]
        - Payloads
          - Input : Name, AgentId, Provider, StartDate, EndDate, Domain, Username, PasswordSecret (SecureString to be hashed), PasswordHash, Certificate
          - Output : 200 OK response
      - Create a credential: [HttpPost("api/v{apiVersion}/credentials")]
        - Payloads
          - Input : Name, Provider, StartDate, EndDate, Domain, Username, PasswordSecret (SecureString to be hashed), PasswordHash, Certificate
          - Output : 200 OK response
      - Update a credential: [HttpPut("api/v{apiVersion}/credentials/{id}")]
        - Payloads
          - Input : Credential id, Name, Provider, StartDate, EndDate, Domain, Username, Password string (to be hashed), Password Secret, Certificate
          - Output : 200 OK response
      - Update credential property: [HttpPatch("api/v{apiVersion}/credentials/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete a credential: [HttpDelete("api/v{apiVersion}/credentials/{id}")]
        - Payloads
          - Input : Credential id
          - Output : 200 OK response
      - Lookup list of credentials: [HttpGet("api/v{apiVersion}/credentials/getlookup")]
        - Payloads
          - Input : None
          - Output : JSON file listing all credential ids and names
  - Credential Manager:
    - The CredentialManager will inherit BaseManager, which inherits IManager.
      - Beyond the base classes, CredentialManager will implement the appropriate methods to assist CredentialsController.
  - Credential Repository:
    - The CredentialRepository will inherit EntityRepository, which inherits ReadOnlyEntityRepository, and ICredentialRepository.
    - The CredentialRepository will retrieve all credentials, add a new credential, or edit/delete a credential by id.
  - Credential Data Models:
    - The Credential data model will be used to view details of each credential.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Credential will have string Provider, DateTime StartDate, DateTime EndDate, string Domain, string Username, string PasswordSecret, string PasswordHash, string HashSalt, string Certificate, and Guid AgentId.
    - The CredentialsLookup data model will be used retrieve a list of credential names.
      - It will have Guid CredentialId and string CredentialName.
    - The CredentialViewModel will be used to view additional details of each credential.  It inherits IViewModel<Credential, CredentialViewModel>.
      - It will have Guid Id, string Name, string Provider, DateTime StartDate, DateTime EndDate, string Domain, string UserName, string PasswordHash, string Certificate, Guid AgentId, and string AgentName.
    - The GlobalCredentialViewModel will be used to create a global credential.  It inherits IViewModel GlobalCredentialViewModel, Credential>.
      - It will have string Name (required), string Provider, DateTime StartDate, DateTime EndDate, string Domain, string UserName (required), string PasswordSecret, string PasswordHash, string HashSalt, and string Certificate.
    - The AgentCredentialViewModel will be used to create an agent credential.  It inherits IViewModel<AgentCredentialViewModel, Credential>.
      - It will have string Name (required), DateTime StartDate, DateTime EndDate, string Domain, string UserName (required), string PasswordSecret, string PasswordHash, string HashSalt, string Certificate, and Guid AgentId (required).

**Sequence Diagrams**

- [Credential Component Sequence Diagram](Sequence Diagrams/CredentialComponentSequenceDiagram.png)
- [Count Credentials Sequence Diagram](Sequence Diagrams/CountCredentialsSequenceDiagram.png)
- [Credential Details Sequence Diagram](Sequence Diagrams/CredentialDetailsSequenceDiagram.png)
- [Credential View Details Sequence Diagram](Sequence Diagrams/CredentialViewDetailsSequenceDiagram.png)
- [Create Credential Sequence Diagram](Sequence Diagrams/CreateCredentialSequenceDiagram.png)
- [Edit Credential Sequence Diagram](Sequence Diagrams/EditCredentialSequenceDiagram.png)
- [Edit Credential Property Sequence Diagram](Sequence Diagrams/EditCredentialPropertySequenceDiagram.png)
- [Delete Credential Sequence](Sequence Diagrams/DeleteCredentialSequenceDiagram.png)
- [Credentials Lookup Sequence Diagram](Sequence Diagrams/CredentialsLookupSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Retrieval is attempted between StartDate or EndDate and returns true.
  - Validate Start Date is less than End Date and return true
- Negative Test Cases:
  - Retrieval is attempted before StartDate or after EndDate and returns false.
  - Invalidate if, the Start Date is not less than the End Date and return false