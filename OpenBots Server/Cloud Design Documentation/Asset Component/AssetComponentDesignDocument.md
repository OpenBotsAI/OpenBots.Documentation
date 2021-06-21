Author: DH
Creation Date: 08/13/2020

Updated By: NC
Updated On: 6/18/2021

**Asset Component**

**Context**

- Problem: Administrators are not able to create assets.  Administrators should be able to create an asset by defining its name, data type, and value. An administrator should be able to view a list of assets, as well as edit and delete them.  Administrators should be able to export an asset file if necessary.

- Requirements: Ensure an administrator is able to create new assets by defining their key values and edit or delete them. Ensure an administrator is able to view a list of assets, request the values of individual assets, and export an individual asset's file.

**Component Scope**

- In-Scope:
  - Administrator can view a list of assets.
  - Administrator can declare new assets.
  - Administrator can edit and delete existing assets.
  - Administrator can view details of individual assets.
  - Administrator can export an asset's file.
- Out-of-Scope:
  - User can view and declare assets.
  - User can edit and delete existing assets.
  - User can view details of individual assets.
  - User can export an asset's file.

**Design**

- Overview: The overall structure uses the AssetsController to allow the user to make a web request for either the entire list of assets or details of an individual asset.  If the user requests access to the entire list of assets, the request goes to the AssetsController, which may use the AssetManager, and then calls the AssetRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the assets could not be found.  The same process will occur when the user requests the asset details as well as add, edit, delete, or export an asset.

- Types if Assets
  - Global asset: The first time an asset name is used it will create a new global asset.  Global assets cannot be deleted until all child assets are deleted.
  - Agent asset: Assets that contain an agent id and share a name with the global asset.  Agent assets can only be created if a global asset of the same name already exists.

- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link that redirects the user to the Asset component.
      - The existing assets will be displayed in a table.
        - The headers will be: Name, Type, View, Edit, and Delete.
        - The appropriate data will be displayed underneath each heading.
          - There will be a button to redirect the user to create a new asset.
          - The user can cick on each asset, where the user will be redirected to a view of that specific asset's details based on its id.
            - On the details page, a user has the option to download the asset to the local machine.
          - The user can edit each asset, where a new file can be uploaded to overwrite the old one, or add an agent asset.
          - The user can delete each asset.
  - Assets Controller:
    - The AssetsController will use the AssetManager to make an API request to the AssetRepository to retrieve all the assets from the Server and will return that information back to the view.  The user can view all assets, view asset details, and add, edit, or delete an asset.
    - NOTE: The current API version is 1.
    - Routes:
      - Get all assets: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets")]
        - Payloads
          - Input : Organization id
          - Output : JSON file containing a list of all assets
      - Get all assets view: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/view")]
        - Payloads
          - Input : Organization id
          - Output : JSON file containing a list of AssetViewModel data
      - Count all assets: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/count")]
        - Payloads
          - Input : Organization id
          - Output : Total count of assets from the Server
      - Get asset details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/{id}")]
        - Payloads
          - Input : Organization id, asset id
          - Output : JSON file containing details for the provided asset id
      - Get asset view details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/view/{id}")]
        - Payloads
          - Input : Organization id, asset id
          - Output : JSON file containing AssetViewModel information
      - Get asset by name (and type): [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/getassetbyname/{assetName}")]
        - Payloads
          - Input : Organization id, optional query parameter "?assetType={assetType}"
          - Output : JSON file listing details for the asset with a matching name (and optional type)
      - Add JSON asset: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/assets")]
        - Payloads
          - Input : Organization id, sset model data (name, type, JSON value)
          - Output : JSON file listing new asset information
      - Add text asset: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/assets")]
        - Payloads
          - Input : Organization id, asset model data (name, type, text value)
          - Output : JSON file listing new asset information
      - Add number asset: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/assets")]
        - Payloads
          - Input : Organization id, asset model data (name, type, number value)
          - Output : JSON file listing new asset information
      - Add file asset: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/assets")]
        - Payloads
          - Input : Organization id, asset model data (name, type, file, optional drive name)
          - Output : JSON file listing new asset information
      - Add agent asset: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/assets/AddAgentAsset")]
        - Payloads
          - Input : AgentAssetViewModel data
          - Output : JS?ON file listing created asset information
      - Export asset file: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/export")]
        - Payloads
          - Input : Organization id, asset id
          - Output : Asset file
      - Update asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}")]
        - Payloads
          - Input : Organization id, name, type, value (number, text, or JSON)
          - Output : 200 OK response
      - Update asset with file: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/update")]
        - Payloads
          - Input : Organization id, name, file
          - Output : 200 OK response
      - Update asset property: [HttpPatch("api/v{apiVersion}/organizations/{organizationId}/assets/{id}")]
        - Payloads
          - Input : Organization id, JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete asset: [HttpDelete("api/v{apiVersion}/organizations/{organizationId}/assets/{id}")]
        - Payloads
          - Input : Organization id, asset id
          - Output : 200 OK response   
      - Increment asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/increment")]
        - Payloads
          - Input : Organization id, asset id
          - Output : 200 OK response
      - Decrement asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/decrement")]
         - Payloads
           - Input : Organization id, asset id
           - Output : 200 OK response
      - Add to asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/add?value={number}")]
        - Payloads
          - Input : Organization id, asset id, number to add
          - Output : 200 OK response
      - Subtract from asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/subtract?value={number}")]
        - Payloads
          - Input : Organization id, asset id, number to subtract
          - Output : 200 OK response
      - Append text to asset: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/assets/{id}/append?value={text}")]
        - Payloads
          - Input : Organization id, asset id, text to append
          - Output : 200 OK response
  - Asset Manager:
    - The AssetManager will inherit BaseManager, which inherits IManager, and IAssetManager.
      - Beyond the base class and interfaces, AssetManager will implement appropriate methods to assist AssetsController.
  - Asset Repository:
    - The AssetRepository will retrieve all assets or details of an asset as well as add, edit, delete, or export assets.
  - Asset Data Models:
    - The Asset data model will be used to store details of each asset.  It will inherit the NamedEntity class, which inherits the Entity class, and ITenanted.
      - Beyond the base classes, Asset will have string Type (required), string TextValue, double NumberValue, string JsonValue, Guid FileId, long SizeInBytes, Guid AgentId, and Guid OrganizationId.
    - The UpdateAssetViewModel will be used to update a text, number, file, or JSON asset.
      - UpdateAssetViewModel will have Guid Id, string Name, string Type, string TextValue, double NumberValue, string JsonValue, Guid FileId, IFormFile File, and string DriveId.
    - The GlobalAssetViewModel will be used to create a new global asset.  It will inherit IViewModel<GlobalAssetViewModel, Asset>.
      - GlobalAssetViewModel will have Guid Id, string Name, string Type, string TextValue, double NumberValue, string JsonValue, Guid FileId, string DriveId, and IFormFile File.  It will inherit IViewModel<GlobalAssetViewModel, Asset>.
    - The AgentAssetViewModel will be used to create a new agent asset.
      - AgentAssetViewModel will have string Name, string TextValue, double NumberValue, string JsonValue, Guid FileId, IFormFile File, Guid DriveId, Guid AgentId, and Guid OrganizationId.

**Sequence Diagrams**

- [Asset Component Sequence Diagram](Sequence Diagrams/AssetComponentSequenceDiagram.png)
- [Count Assets Sequence Diagram](Sequence Diagrams/CountAssetsSequenceDiagram.png)
- [Asset Details Sequence Diagram](Sequence Diagrams/AssetDetailsSequenceDiagram.png)
- [Add Asset Sequence Diagram](Sequence Diagrams/AddAssetSequenceDiagram.png)
- [Upload Asset Sequence Diagram](Sequence Diagrams/UploadAssetSequenceDiagram.png)
- [Export Asset File Sequence Diagram](Sequence Diagrams/ExportAssetSequenceDiagram.png)
- [Edit Asset Sequence Diagram](Sequence Diagrams/EditAssetSequenceDiagram.png)
- [Edit Asset File Sequence Diagram](Sequence Diagrams/EditAssetFileSequenceDiagram.png)
- [Edit Asset Property Sequence Diagram](Sequence Diagrams/EditAssetPropertySequenceDiagram.png)
- [Delete Asset Sequence Diagram](Sequence Diagrams/DeleteAssetSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A