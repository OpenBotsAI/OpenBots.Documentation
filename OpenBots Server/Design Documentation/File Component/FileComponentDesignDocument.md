Author: Nicole Carrero
Creation Date: 2/9/2021

Updated On: 3/5/2021
Updated By: Nicole Carrero

**File Component**

**Context**

- Problem: Users are not able to save files in a file manager.  Users should be able to upload a file as well as create folders to store those files.  A user should be able to view a list of folders and files as well as rename, move, copy, and delete them.  Users should be able to export a file.
- Requirements: Ensure a user is able to use the file manager to upload a file, create a folder, view a list of folders and files, and rename, move, copy, and delete files or folders.  Ensure a user is able to export a file.

**Component Scope**

- In-Scope:
  - User can view a list of files and folders in the file manager.
  - User can upload a file and create a folder.
  - User can edit and delete existing files by renaming, copying, or moving a file.
  - User can edit folders by renaming, copying, or moving a folder.
  - User can view details of files and folders.
  - User can export a file.
- Out-of-Scope:
  - User can export a folder and its contents.
  - User can update a file with a new version of the file.

**Design**

- Overview: The overall structure uses the FilesController to allow the user to make a web request for either the entire list of files/folders within a drive or folder or details of an individual file or folder.  If the user requests access to the entire list of files, folders, or drives the request goes to the FilesController, which may use the FileManager (which can call the LocalFileStorageAdapter), and then calls the ServerFileRepository, ServerFolderRepository, or ServerDriveRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the files, folders, or drives could not be found.  The same process will occur when the user requests the file, folder, or server details as well as add, edit, delete, or export a file or folder.
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link named "Files Manager" that redirects the user to view the files and folders at the top of the selected drive.
      - The existing files and folders will be displayed in a table.
        - The headers will be: Name, Type.
        - The appropriate data will be displayed underneath each heading.
          - The user can click on each file or folder, where the user will be shown the specific details.
            - There will be a button to redirect the user to add, delete, rename, move, or copy a file or folder.  There will also be a button to download a file.
            - When a user double clicks a folder, it will redirect to show the files and folders within it.
            - The user can refresh the page using the refresh button.
  - Files Controller:
    - The FilesController will make an API request to the FileManager, which in turn will call the appropriate adapter.  The ServerFileRepository, ServerFolderRepository, and ServerDriveRepository will be used to retrieve all the files, folders, and/or drives from the Server and will return that information back to the view.  The user can view all files, folders, and/or drives, view file, folder, or drive details, and add, edit (rename, move, or copy), or delete a file or folder.  The user can also export a file.
    - NOTE: Most APIs listed below use an additional optional query parameter if the user requests access to a file, folder, or drive that is not the default "Files" drive.  For instance, get all files and folders would look like this: [HttpGet("api/v1/files?driveName={driveName}")].
    - Routes:
      - All files and folders: [HttpGet("api/v1/files")]
        - Payloads
          - Input : Drive name (optional)
          - Output : JSON file listing all file and folder information
      - All child files and folders within a parent folder: [HttpGet("api/v1/files?$filter=ParentId+eq+guid'{parentId}'")]
        - Payloads 
          - Input : Drive name (optional), parent id
          - Output : JSON file listing all child file and folder information
      - All files: [HttpGet("api/v1/files")]
        - Payloads
          - Input : Drive name (optional), file query parameter (?file=true - optional)
          - Output : JSON file listing all file information
      - All folders: [HttpGet("api/v1/files?file=false")]
        - Payloads
          - Input : Drive name (optional)
          - Output : JSON file listing all folder information
      - File count: [HttpGet("api/v1/files/count")]
        - Payloads
          - Input : Drive name (optional)
          - Output : Count of all files
      - Folder count: [HttpGet("api/v1/files/count/folder")]
        - Payloads
          - Input : Drive name (optional)
          - Output : Count of all folders
      - Folder or file details: [HttpGet("api/v1/files/{id}")]
        - Payloads
          - Input : Folder or file id, drive name (optional)
          - Output : JSON file listing specific file or folder information
      - Drive details: [HttpGet("api/v1/files/drive")]
        - Payloads
          - Input : Drive name (optional)
          - Output : JSON file listing all drive information
      - Upload files: [HttpPost("api/v1/files")]
        - Payloads
          - Input : Drive name (optional), storage path, files, is file (optional - true)
          - Output : JSON file listing newly created file information
      - Create folder: [HttpPost("api/v1/files")]
        - Payloads
          - Input : Drive name (optional), folder name, storage path, is file (set to false)
          - Output : JSON file listing newly created folder information
      - Export file: [HttpGet("api/v1/files/{id}/download"]
          - Input : File id, drive name (optional)
          - Output : File in its original format
      - Delete folder or file: [HttpDelete("api/v1/files/{id}")]
        - Payloads
          - Input : Folder or file id, drive name (optional)
          - Output : 200 OK response
      - Rename folder or file: [HttpPut("api/v1/files/{id}/rename")]
        - Payloads
          - Input : Folder or file id, folder or file name, drive name (optional)
          - Output : JSON file listing updated folder or file information
      - Move folder or file: [HttpPut("api/v1/files/{id}/move/{parentId}")]
        - Payloads
          - Input : Folder or file id, parent folder id, drive name (optional)
          - Output : JSON file listing updated folder or file information
      - Copy folder or file: [HttpPost("api/v1/files/{id}/copy/{parentId}")]
        - Payloads
          - Input : Folder or file id, parent folder id, drive name (optional)
          - Output : JSON file listing new folder or file information
      - Create drive: [HttpPost("api/v1/files/drive?driveName={driveName}")]
        - Payloads
          - Input : Drive name
          - Output : JSON file listing newly created server drive information
      - Drive names: [HttpGet("api/v1/files/driveNames?adapterType={adapterType}")]
        - Payloads
          - Input: Adapter type (default: LocalFileStorage)
          - Output : JSON file listing drive names and their entity ids
- Managers/Adapters:
  - File Manager:
    - The FileManager will inherit BaseManager, which inherits IManager, and IFileManager.
      - Beyond the base class and interfaces, FileManager will implement the appropriate methods to assist FilesController.
    - Local File Storage Adapter:
      - The LocalFileStorageAdapter will perform all drive, folder, and file functions when the configuration is set up for local file storage.  It will inherit IFileStorageAdapter.
        - Beyond the interface, the LocalFileStorageAdapter will implement the appropriate methods to assist the FileManager.
    - Directory Manager:
      - The DirectoryManager will be used in the LocalFileStorageAdapter.  It will inherit IDirectoryManager.
        - Beyond the interface, the DirectoryManager will implement the appropriate System.IO.File methods to assist the LocalFileStorageAdapter.
- Repositories:
  - Server Drive Repository:
    - ServerDriveRepository will inherit EntityRepository<ServerDrive>, which inherits ReadOnlyEntityRepository, and IServerDriveRepository.
      - The ServerDriveRepository will retrieve all drive entities, add a new drive, or retrieve, edit, or delete a drive entity by id.
  - Server Folder Repository:
    - ServerFolderRepository will inherit EntityRepository<ServerFolder>, which inherits ReadOnlyEntityRepository, and IServerFolderRepository.
      - The ServerFolderRepository will retrieve all folder entities, add a new folder, or retrieve,, move, rename, copy, or delete a folder entity and its folder by id.
        - Beyond the interface and base class, ServerFolderRepository will implement the FindAllView and FindAllFilesFoldersView methods to assist LocalFileStorageAdapter.
  - Server File Repository:
    - ServerFileRepository will inherit EntityRepository<ServerFile>, which inherits ReadOnlyEntityRepository, and IServerFileRepository.
      - The ServerFileRepository will retrieve all file entities, add a new file, or retrieve, move, rename, copy, or delete a file entity along with its file by id.
        - Beyond the interface and base class, ServerFileRepository will implement the FindAllView method to assist LocalFileStorageAdapter.
  - File Attribute Repository:
    - The FileAttributeRepository will inherit EntityRepository<FileAttribute>, which inherits ReadOnlyEntityRepository, and IFileAttributeEntity.
      - The FileAttributeEntity will retrieve all file attribute entities, add a new file attribute, or retrieve, edit, or delete a file attribute entity by id.
- Data Models/View Models:
  - Server Drive Data Model:
    - The ServerDrive data model will be used to store details of each server drive.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, ServerDrive will have string FileStorageAdapterType, Guid OrganizationId, long StorageSizeInBytes, and string StoragePath.
  - Server Folder Data Model:
    - The ServerFolder data model will be used to store details of each server folder.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, ServerFolder will have Guid StorageDriveId, Guid ParentFolderId, Guid OrganizationId, string StoragePath, and long SizeInBytes.
  - Server File Data Model:
    - The ServerFile data model will be used to store details of each server file.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, ServerFile will have Guid StorageFolderId, string ContentType, string StoragePath, string StorageProvider, long SizeInBytes, string HashCode, Guid OrganizationId, Guid ServerDriveId, and List<FileAttribute> FileAttributes.
  - File Attribute Data Model:
    - The FileAttribute data model will be used to store details of each attribute of a file.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, FileAttribute will have ServerFolderId, int AttributeValue, string DataType, Guid OrganizationId, and Guid ServerDriveId.
  - File Folder View Model:
    - The FileFolderViewModel view model will be used to request and display details of each file or folder.
      - It will have Guid Id, string Name, long Size, string StoragePath, string FullStoragePath, bool HasChild, string ContentType, string CreatedBy, DateTime CreatedOn, DateTime UpdatedOn, bool IsFile, Guid ParentId, Guid StorageDriveId, FileStream Content, IFormFile[] Files, and string Hash.
      - It also includes two Map methods to map an existing ServerFile or ServerFolder entity to FileFolderViewModel.

**Sequence Diagrams**

- [File Component Sequence Diagram](Sequence Diagrams/FileComponentSequenceDiagram.pdf)
- [File Folder Count Sequence Diagram](Sequence Diagrams/FileFolderCountSequenceDiagram.pdf)
- [Details Sequence Diagram (File, Folder, Drive)](Sequence Diagrams/FileFolderDriveDetailsSequenceDiagram.png)
- [Add File Folder Sequence Diagram](Sequence Diagrams/AddFileFolderSequenceDiagram.png)
- [Export File Sequence Diagram](Sequence Diagrams/ExportFileSequenceDiagram.png)
- [Delete File Folder Sequence Diagram](Sequence Diagrams/DeleteFileFolderSequenceDiagram.png)
- [Edit File Folder Sequence Diagram (Rename, Move)](Sequence Diagrams/EditFileFolderSequenceDiagram.png)
- [Copy File Folder Sequence Diagram](Sequence Diagrams/CopyFileFolderSequenceDiagram.png)
- [Add Drive Sequence Diagram](Sequence Diagrams/AddDriveSequenceDiagram.png)
- [Drive Names Sequence Diagram](Sequence Diagrams/DriveNamesSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A