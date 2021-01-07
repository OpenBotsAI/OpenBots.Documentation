Author: Nicole Carrero
Creation Date: 11/2/2020

**Binary Object Component**

**Context**

- Problem: Users are not able to save files as binary objects.  Users should be able to create a binary object by uploading a file.  A user should be able to view a list of binary objects as well as edit and delete them.  Users should be able to export the binary object file to it's original format, if necessary.
- Requirements: Ensure a user is able to create new binary objects by uploading a file and edit or delete them.  Ensure a user is able to view a list of binary objects and export an individual binary object's original file.

**Component Scope**

- In-Scope:
  - User can view a list of binary objects.
  - User can create new binary objects by uploading a file.
  - User can edit and delete existing binary objects.
  - User can view details of individual binary objects.
  - User can export a binary object's original file.
- Out-of-Scope:
  - User can view binary object file.
  - User can view binary object's original file.

**Design**

- Overview: The overall structure uses the BinaryObjectsController to allow the user to make a web request for either the entire list of binary objects or details of an individual binary object.  If the user requests access to the entire list of binary objects, the request goes to the BinaryObjectsController, which may use the BinaryObjectManager (which can call the BlobStorageAdapter or FileSystemAdapter), and then calls the BinaryObjectRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the binary objects could not be found.  The same process will occur when the user requests the binary object details as well as add, edit, delete, or export a binary object.
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link named "Files" that redirects the user to the Binary Object component.
      - The existing binary objects will be displayed in a table.
        - The headers will be: Name, Correlation Entity, Folder, Size, View, and Edit.
        - The appropriate data will be displayed underneath each heading.
          - There will be a button to redirect the user to add a new file.
          - The user can cick on each binary object, where the user will be redirected to a view of that specific binary object's details based on its id.
            - On the details page, a user has the option to download the original file to the local machine.
            - If the binary object is not related to an asset or a process (has no Correlation entity or is labeled as "BinaryObjectAPI"), the user also has the option to delete it on this page.
          - The user can edit each binary object, where a new file can be uploaded to overwrite the old one.
  - Binary Objects Controller:
    - The BinaryObjectsController will make an API request to the BinaryObjectRepository to retrieve all the binary objects from the Server and will return that information back to the view.  The user can view all binary objects, view binary object details, and add, edit, delete, or export a binary object file.
    - Routes:
      - All binary objects: [HttpGet("api/v1/binaryobjects")]
        - Payloads
          - Input : None
          - Output : JSON file listing all binary object information
      - Binary object count: [HttpGet("api/v1/binaryobjects/count")]
        - Payloads
          - Input : None
          - Output : Count of all binary objects
      - Binary object details: [HttpGet("api/v1/binaryobjects/{id}")]
        - Payloads
          - Input : Binary object id
          - Output : JSON file listing specific binary object information
      - Add binary object: [HttpPost("api/v1/binaryobjects")]
        - Payloads
          - Input : Name, Folder
          - Output : JSON file listing newly created binary object information
      - Upload binary object: [HttpPost("api/v1/binaryobjects/{id}/upload")]
        - Payloads
          - Input : Binary object id, File
          - Output : JSON file listing updated binary object information
      - Export binary object: [HttpGet("api/v1/binaryobjects/{id}/download")]
        - Payloads
          - Input : Binary object id
          - Output : File in its original format
      - Update binary object: [HttpPut("api/v1/binaryobjects/{id}")]
        - Payloads
          - Input : Binary object id, Binary object data model
          - Output : JSON file listing updated binary object information
      - Update binary object with file: [HttpPut("api/v1/binaryobjects/{id}/update")]
        - Payloads
          - Input : Binary object id, binary object view model (includes Name, Folder, and File)
          - Output : JSON file listing updated binary object information
      - Delete binary object: [HttpDelete("api/v1/binaryobjects/{id}")]
        - Payloads
          - Input : Binary object id
          - Output : 200 OK response
      - Update binary object property: [HttpPatch("api/v1/binaryobject/{id}")]
        - Payloads
          - Input : Binary object id, JsonPatchDocument in request body with changes
          - Output : 200 OK response
  - Binary Object Manager:
    - The BinaryObjectManager will inherit BaseManager, which inherits IManager, and IBinaryObjectManager.
      - Beyond the base class and interfaces, BinaryObjectManager will implement Upload(), SaveEntity(), GetHash(), Update(), UpdateEntity(), and GetOrganizationId() methods to assist BinaryObjectsController.
    - BinaryObjectManager will utilize two adapters:
      - BlobStorageAdapter will save and update the binary object entity as well as fetch the binary object file.
      - FileSystemAdapter will determine where the newly created or updated files will be saved.
        - Directory Manager:
          - The DirectoryManager will be used in the FileSystemAdapter.  It will inherit IDirectoryManager.
            - Beyond the interface, the DirectoryManager will implement CreateDirectory(), Exists(), and Delete() methods to assist the FileSystemAdapter.
  - Binary Object Repository:
    - BinaryObjectRepository will inherit EntityRepository<BinaryObject>, which inherits ReadOnlyEntityRepository, and IBinaryObjectRepository.
      - The BinaryObjectRepository will retrieve all binary object entities, add a new binary object, or retrieve, edit, or delete a binary object entity along with its file by id.
  - Binary Object Data Model:
    - The BinaryObject data model will be used to store details of each binary object file.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, BinaryObject will have Guid OrganizationId, string ContentType, Guid CorrelationEntityId, string CorrelationEntity, string Folder, string StoragePath, string StorageProvider, long SizeInBytes, and string HashCode.

**Sequence Diagrams**

- [Binary Object Component Sequence Diagram](Sequence Diagrams/BinaryObjectComponentSequenceDiagram.png)
- [Binary Object Count Sequence Diagram](Sequence Diagrams/CountBinaryObjectsSequenceDiagram.png)
- [Binary Object Details Sequence Diagram](Sequence Diagrams/BinaryObjectDetailsSequenceDiagram.png)
- [Add Binary Object Sequence Diagram](Sequence Diagrams/AddBinaryObjectSequenceDiagram.png)
- [Upload Binary Object Sequence Diagram](Sequence Diagrams/UploadBinaryObjectSequenceDiagram.png)
- [Export Binary Object Sequence Diagram](Sequence Diagrams/ExportBinaryObjectSequenceDiagram.png)
- [Edit Binary Object Sequence Diagram](Sequence Diagrams/EditBinaryObjectSequenceDiagram.png)
- [Edit Binary Object File Sequence Diagram](Sequence Diagrams/EditBinaryObjectFileSequenceDiagram.png)
- [Delete Binary Object Sequence Diagram](Sequence Diagrams/DeleteBinaryObjectSequenceDiagram.png)
- [Edit Binary Object Property Sequence Diagram](Sequence Diagrams/EditBinaryObjectPropertySequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Verify that we can upload a file and it returns the binary object id.
  - Verify that we can download a file and the storage path exists.
- Negative Test Cases:
  - Verify that we cannot upload an empty or nonexistent file and it returns a message stating the file does not exist.
  - Verify that we cannot download a file with an invalid binary object id and it returns a null storage path value.