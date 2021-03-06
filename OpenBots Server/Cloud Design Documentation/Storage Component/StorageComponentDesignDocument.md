Author: NC
Creation Date: 2/9/2021

Updated By: NC
Updated On: 6/18/2021

**Storage Component**

**Context**

- Problem: Users are not able to create storage drives and save files in a file manager.  Users should be able to create a storage drive to save files and folders in.  Users should be able to upload a file as well as create folders to store those files.  A user should be able to view a list of folders and files as well as rename, move, copy, and delete them.  Users should be able to export a file.
- Requirements: Ensure a user is able to use the file manager to upload a file, create a folder, view a list of folders and files, and rename, move, copy, and delete files or folders.  Ensure a user is able to export a file.  Ensure a user is able to create a storage drive to store folders and files in.

**Component Scope**

- In-Scope:
  - User can create a storage drive.
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

- Overview: The overall structure uses the FilesController and/or DrivesController to allow the user to make a web request for either the entire list of files/folders within a drive or folder or details of an individual file or folder, or the entire list of drives within an organization.  If the user requests access to the entire list of files, folders, or drives, the request goes to the FilesController or DrivesController, which may use the FileManager (which can call the AzureBlobStorageAdapter), and then calls the StorageFileRepository, StorageFolderRepository, or StorageDriveRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the files, folders, or drives could not be found.  The same process will occur when the user requests the file, folder, or server details as well as add, edit, delete, or export a file or folder.
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link under the Billing tab named "Storage" that redirects the user to view the available storage drives and the organization's storage informatiom.
        - The existing drives will be displayed in a table.
          - The headers will be: Drive Name, Storage Assigned, Storage Used, and Remaining.
          - The appropriate data will be displayed underneath each heading.
            - The user can click on each drive, where the user will be shown the specific details.
            - There will be a button to redirect the user to add, edit, or delete a storage drive.
      - The Dashboard will display a link in the "Files Manager" tab that redirects the user to view the files and folders at the top of the selected drive.
        - The existing files and folders will be displayed in a table.
          - The headers will be: Name, Type.
          - The appropriate data will be displayed underneath each heading.
            - The user can click on each file or folder, where the user will be shown the specific details on the right-hand side.
              - There will be a button to redirect the user to add, delete, rename, move, or copy a file or folder.  There will also be a button to download a file.
              - When a user double clicks a folder, it will redirect to show the files and folders within it.
              - The user can refresh the page using the refresh button.
  - NOTE: The current API version is 1.
  - Drives Controller:
    - The DrivesController will make an API request to the FileManager, which in turn will call the appropriate adapter as necessary.  The StorageDriveRepository will be used to retrieve all the drives from the Cloud Server and will return that information back to the view.  The user can view all drives and add, edit, or delete a drive.
    - Routes:
      - All drives: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives")]
        - Payloads
          - Input : Organization id
          - Output : JSON file listing all drives information
      - All drives view: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/view")]
        - Payloads
          - Input : Organization id
          - Output : JSON file listing drive view information
      - Drive count: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/count")]
        - Payloads
          - Input : Organization id
          - Output : Count of all drives
      - Drive details: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/DriveDetails/{id}")]
        - Payloads
          - Input : Organization id, drive id
          - Output : JSON file listing specific drive information
      - Drive details by name: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/DriveDetailsByName/{driveName}")]
        - Payloads
          - Input : Organization id, drive name
          - Output : JSON file listing specific drive information
      - Create drive: [HttpPost("api/v{apiVersion}/Storage/{organizationId}/Drives")]
        - Payloads
          - Input : Organization id, drive model
          - Ouput: JSON file listing newly created drive information
      - Update storage drive: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{id}")]
        - Payloads
          - Input : Organization id, drive id, drive model
          - Output : JSON file listing updated drive information
      - Delete drive: [HttpDelete("api/v{apiVersion}/Storage/{organizationId}/Drives/{id}")]
        - Payloads
          - Input : Organization id, drive id
          - Output : 200 OK response
      - Drive names lookup: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/driveNames")]
        - Payloads
          - Input : Organization id
          - Output : JSON file listing drive names and corresponding ids
      - Max storage of all drives: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/MaxDriveSizeCount")]
        - Payloads
          - Input : Organization id
          - Output : Count of all drives' max storage size
      - Total storage used of all drives: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/UsedDriveStorageCount")]
        - Payloads
          - Input : Organization id
          - Output : Count of all drives' currently used storage
  - Files Controller:
    - The FilesController will make an API request to the FileManager, which in turn will call the appropriate adapter as necessary.  The StorageFileRepository and StorageFolderRepository will be used to retrieve all the files and/or folders from the Cloud Server and will return that information back to the view.  The user can view all files, folders, individual file or folder details, and add, edit (rename, move, or copy), or delete a file or folder.  The user can also export a file.
    - Routes:
      - All files and folders in a drive: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}")]
        - Payloads
          - Input : Oganization id, drive id
          - Output : JSON file listing all file and folder information
      - All files in a drive: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files")]
        - Payloads
          - Input : Organization id, drive id
          - Output : JSON file listing all file information
      - All folders in a drive: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders")]
        - Payloads
          - Input : Organization id, drive id
          - Output : JSON file listing all folder information
      - All child files and folders in a drive within a parent folder: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}?$filter=ParentId+eq+guid'{parentFolderId}'")]
        - Payloads 
          - Input : Organization id, drive id, parent folder id
          - Output : JSON file listing all child file and folder information
      - File count: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/Count")]
        - Payloads
          - Input : Organization id, drive id
          - Output : Count of all files
      - Folder count: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders/Count")]
        - Payloads
          - Input : Organization id, drive id
          - Output : Count of all folders
      - Folder details: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders/{id}")]
        - Payloads
          - Input : Folder id, Organization id, drive id
          - Output : JSON file listing specific folder information
      - File details: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}")]
        - Payloads
          - Input : File id, Organization id, drive id
          - Output : JSON file listing specific file information
      - File or folder details by storage path: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/Details/{driveName}?path={storagePath}")]
        - Payloads
          - Input: Drive name, storage path of file or folder
          - Output : JSON file listing specific file or folder information
      - Upload file(s): [HttpPost("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files")]
        - Payloads
          - Input : Organization id, drive id, storage path, file(s) to upload, is file (optional - true)
          - Output : JSON file listing newly created file(s) information
      - Create folder: [HttpPost("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders")]
        - Payloads
          - Input : Organization id, drive id, folder name, storage path, is file (set to false)
          - Output : JSON file listing newly created folder information
      - Export file: [HttpGet("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}/Download"]
          - Input : File id, Organization id, drive id
          - Output : File in its original format
      - Delete folder: [HttpDelete("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders/{id}")]
        - Payloads
          - Input : Folder id, Organization id, drive id
          - Output : 200 OK response
      - Delete file: [HttpDelete("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}")]
        - Payloads
          - Input : File id, Organization id, drive id
          - Output : 200 OK response
      - Rename folder: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders/{id}/Rename")]
        - Payloads
          - Input : Folder id, folder name, Organization id, drive id
          - Output : JSON file listing updated folder information
      - Rename file: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}/Rename")]
        - Payloads
          - Input : File id, file name, Organization id, drive id
          - Output : JSON file listing updated file information
      - Move folder: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Folders/{id}/Move/{parentId}")]
        - Payloads
          - Input : Folder id, parent folder id, Organization id, drive id
          - Output : JSON file listing updated folder information
      - Move file: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}/Move/{parentId}")]
        - Payloads
          - Input : File id, parent folder id, Organization id, drive id
          - Output : JSON file listing updated file information
      - Replace a file: [HttpPut("api/v{apiVersion}/Storage/{organizationId}/Drives/{driveId}/Files/{id}/Replace")]
        - Payloads
          - Input : File id, Organization id, drive id, file to replace
          - Output : JSON file listing newly updated file information
- Managers/Adapters:
  - File Manager:
    - The FileManager will inherit BaseManager, which inherits IManager, and IFileManager.
      - Beyond the base class and interfaces, FileManager will implement the appropriate methods to assist FilesController and DrivesController.
    - Azure Blob Storage Adapter:
      - The AzureBlobStorageAdapter will perform all folder and file functions when the configuration is set up for Azure Blob storage.  It will inherit IAzureBlobStorageAdapter, which inherits IFileStorageAdapter.
        - Beyond the interfaces, the AzureBlobStorageAdapter will implement the appropriate methods to assist the FileManager.
- Repositories:
  - Storage Drive Repository:
    - StorageDriveRepository will inherit EntityRepository<StorageDrive>, which inherits TenantEntityRepository, and IStorageDriveRepository.
      - The StorageDriveRepository will retrieve all drive entities, add a new drive, or retrieve, edit, or delete a drive entity by id.
  - Storage Folder Repository:
    - StorageFolderRepository will inherit EntityRepository<StorageFolder>, which inherits TenantEntityRepository, and IStorageFolderRepository.
      - The StorageFolderRepository will retrieve all folder entities, add a new folder, or retrieve,, move, rename, copy, or delete a folder entity and its folder by id.
        - Beyond the interface and base class, StorageFolderRepository will implement methods to assist AzureBlobStorageAdapter.
  - Storage File Repository:
    - StorageFileRepository will inherit EntityRepository<StorageFile>, which inherits TenantEntityRepository, and IStorageFileRepository.
      - The StorageFileRepository will retrieve all file entities, add a new file, or retrieve, move, rename, copy, or delete a file entity along with its file by id.
        - Beyond the interface and base class, StorageFileRepository will implement methods to assist AzureBlobStorageAdapter.
  - Storage Drive Operation Repository:
    - The StorageDriveOperationRepository will inherit EntityRepository<StorageDriveOperation>, which inherits TenantEntityRepository, and IStorageDriveOperationRepository.
      - The StorageDriveOperationRepository will retrieve all file attribute entities, add a new file attribute, or retrieve, edit, or delete a file attribute entity by id.
- Data Models/View Models:
  - Storage Drive Data Model:
    - The StorageDrive data model will be used to store details of each storage drive.  It will inherit the NamedEntity class, which inherits the Entity class, and ITenanted.
      - Beyond the base  and interface, StorageDrive will have string FileStorageAdapterType, Guid OrganizationId, long StorageSizeInBytes, string StoragePath, long MaxStorageAllowedInBytes, string ConnectionString, bool IsExternal, string Description, string UserName, string Password, string EncryptionKey, bool IsDefault, bool IsUploadBlocked, bool IsDownloadBlocked, and bool IsDeletionBlocked.
  - Storage Folder Data Model:
    - The StorageFolder data model will be used to store details of each storage folder.  It will inherit the NamedEntity class, which inherits the Entity class, and ITenanted.
      - Beyond the base classes and interface, StorageFolder will have Guid StorageDriveId, Guid ParentFolderId, Guid OrganizationId, string StoragePath, and long SizeInBytes.
  - Storage File Data Model:
    - The StorageFile data model will be used to store details of each storage file.  It will inherit the NamedEntity class, which inherits the Entity class, and ITenanted.
      - Beyond the base classes and interface, StorageFile will have Guid StorageFolderId, string ContentType, string StoragePath, string StorageLocation, string StorageProvider, long SizeInBytes, string HashCode, Guid OrganizationId, Guid StorageDriveId, and List<StorageDriveOperation> StorageDriveOperations.
  - Storage Drive Operation Data Model:
    - The StorageDriveOperation data model will be used to store details of each attribute of a file.  It will inherit the NamedEntity class, which inherits the Entity class, and ITenanted.
      - Beyond the base classes and interface, StorageDriveOperation will have StorageFileId, int AttributeValue, string DataType, Guid OrganizationId, and Guid StorageDriveId.
  - File Folder View Model:
    - The FileFolderViewModel view model will be used to request and display details of each file or folder.
      - It will have Guid Id, string Name, long Size, string StoragePath, string FullStoragePath, bool HasChild, string ContentType, string CreatedBy, DateTime CreatedOn, DateTime UpdatedOn, bool IsFile, Guid ParentId, Guid StorageDriveId, FileStream Content, IFormFile[] Files, and string Hash.
      - It also includes two Map methods to map an existing StorageFile or StorageFolder entity to FileFolderViewModel.
  - Storage Drive View Model:
    - The StorageDriveViewModel will be used to request and display details of each storage drive for the billing page.
      - It will have string Name, long StorageSizeInBytes, long MaxStorageAllowedInBytes, and int PercentStorageRemaining.

**Sequence Diagrams**

DrivesController
- [Drive Component Sequence Diagram](Sequence Diagrams/DrivesController/DriveComponentSequenceDiagram.png)
- [Drive Count Sequence Diagram](Sequence Diagrams/DrivesController/DriveCountSequenceDiagram.png)
- [Drive Details Sequence Diagram](Sequence Diagrams/DrivesController/DriveDetailsSequenceDiagram.png)
- [Add Drive Sequence Diagram](Sequence Diagrams/DrivesController/AddDriveSequenceDiagram.png)
- [Update Drive Sequence Diagram](Sequence Diagrams/DrivesController/UpdateDriveSequenceDiagram.png)
- [Delete Drive Sequence Diagram](Sequence Diagrams/DrivesController/DeleteDriveSequenceDiagram.png)
- [Drive Names Sequence Diagram](Sequence Diagrams/DrivesController/DriveNamesLookupSequenceDiagram.png)
- [Drive Used Storage Count Sequence Diagram](Sequence Diagrams/DrivesController/DriveUsedStorageCountSequenceDiagram.png)
- [Drive Max Storage Count Sequence Diagram](Sequence Diagrams/DrivesController/DriveMaxStorageCountSequenceDiagram.png)

FilesController
- [File Component Sequence Diagram](Sequence Diagrams/FilesController/FileComponentSequenceDiagram.pdf)
- [File Folder Count Sequence Diagram](Sequence Diagrams/FilesController/FileFolderCountSequenceDiagram.pdf)
- [File Folder Details Sequence Diagram](Sequence Diagrams/FilesController/FileFolderDetailsSequenceDiagram.png)
- [Add File Folder Sequence Diagram](Sequence Diagrams/FilesController/AddFileFolderSequenceDiagram.png)
- [Export File Sequence Diagram](Sequence Diagrams/FilesController/ExportFileSequenceDiagram.png)
- [Delete File Folder Sequence Diagram](Sequence Diagrams/FilesController/DeleteFileFolderSequenceDiagram.png)
- [Edit File Folder Sequence Diagram (Rename, Move)](Sequence Diagrams/FilesController/EditFileFolderSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A