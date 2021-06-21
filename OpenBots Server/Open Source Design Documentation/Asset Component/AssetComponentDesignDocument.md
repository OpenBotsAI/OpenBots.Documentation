Author: DH
Creation Date: 08/13/2020

Updated By: NC
Updated On: 6/15/2021

**Asset Component**

**Context**

- Problem: Administrators are not able to create assets.  Administrators should be able to create an asset by defining its name, data type, and value. An administrator should be able to view a list of assets, as well as edit and delete them.  Administrators should be able to export an asset file if necessary.  Administrators should be able to create assets associated with an agent.

- Requirements: Ensure an administrator is able to create new assets by defining their key values and edit or delete them. Ensure an administrator is able to view a list of assets, request the values of individual assets, and export an individual asset's file. Additionally, agent assets can be created and used by the Agent and Studio.

**Component Scope**

- In-Scope:
  - Administrator can view a list of assets.
  - Administrator can declare new assets.
  - Administrator can edit and delete existing assets.
  - Administrator can view details of individual assets.
  - Administrator can export an asset's file.
  - Administrator can create an agent asset.
- Out-of-Scope:
  - User can view and declare assets.
  - User can edit and delete existing assets.
  - User can view details of individual assets.
  - User can export an asset's file.

**Design**

- Overview: The overall structure uses the AssetsController to allow the user to make a web request for either the entire list of assets or details of an individual asset.  If the user requests access to the entire list of assets, the request goes to the AssetsController, which may use the AssetsManager, and then calls the AssetsRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the assets could not be found.  The same process will occur when the user requests the asset details as well as add, edit, delete, or export an asset.

- Types of Assets
  - Global asset: The first time an asset name is used it will create a new global asset. Global assets cannot be deleted until all child assets are deleted.
  - Agent asset: Assets that contain an agent id and share a name with a global asset. Agent assets can only be created if a global asset of the same name already exists. 

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
    - Routes:
    - NOTE: The current API version is 1.
      - Get all assets: [HttpGet("api/v{apiVersion}/assets")]
        - Payloads
          - Input : None
          - Output : JSON file containing a list of all assets
      - Get all assets view: [HttpGet("api/v{apiVersion}/assets/view")]
        - Payloads
          - Input : None
          - Output : JSON file containing a list of AssetViewModel data
      - Count all assets: [HttpGet("api/v{apiVersion}/assets/count")]
        - Payloads
          - Input : None
          - Output : Total count of assets from the Server
      - Get asset details: [HttpGet("api/v{apiVersion}/assets/{id}")]
        - Payloads
          - Input : Asset id
          - Output : JSON file containing details for the provided asset id
      - Get asset view details: [HttpGet("api/v{apiVersion}/assets/view/{id}")]
        - Payloads
          - Input : Asset id
          - Output : JSON file containing AssetViewModel information
      - Get asset by name: [HttpGet("api/v{apiVersion}/assets/GetAssetByName/{AssetName}")]
        - Payloads
          - Input : Asset name and (optional) asset type
          - Output : global asset file, unless called by an agent with an existing agent-asset
      - Add asset: [HttpPost("api/v{apiVersion}/assets")]
        - Payloads
          - Input : GlobalAssetViewModel data
          - Output : JSON file listing new asset information
      - Add agent asset: [HttpPost("api/v{apiVersion}/assets/AddAgentAsset")]
        - Payloads
          - Input : AgentAssetViewModel data
          - Output : JSON file listing created asset information
      - Export asset file: [HttpGet("api/v{apiVersion}/assets/{id}/export")]
        - Payloads
          - Input : Asset id
          - Output : Asset file as Memory Stream
      - Update asset: [HttpPut("api/v{apiVersion}/assets/{id}")]
        - Payloads
          - Input : Name, value (Number, Text, or JSON)
          - Output : 200 OK response
      - Update asset with file: [HttpPut("api/v{apiVersion}/assets/{id}/update")]
        - Payloads
          - Input : Name, file
          - Output : 200 OK response
      - Update asset property: [HttpPatch("api/v{apiVersion}/assets/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete asset: [HttpDelete("api/v{apiVersion}/assets/{id}")]
        - Payloads
          - Input : Asset id
          - Output : 200 OK response   
      - Increment asset: [HttpPut("api/v{apiVersion}/assets/{id}/increment")]
        - Payloads
          - Input : Asset id
          - Output : 200 OK response
      - Decrement asset: [HttpPut("api/v{apiVersion}/assets/{id}/decrement")]
         - Payloads
           - Input : Asset id
           - Output : 200 OK response
      - Add to asset: [HttpPut("api/v{apiVersion}/assets/{id}/add?value={number}")]
        - Payloads
          - Input : Asset id, number to add
          - Output : 200 OK response
      - Subtract from asset: [HttpPut("api/v{apiVersion}/assets/{id}/subtract?value={number}")]
        - Payloads
          - Input : Asset id, number to subtract
          - Output : 200 OK response
      - Append text to asset: [HttpPut("api/v{apiVersion}/assets/{id}/append?value={text}")]
        - Payloads
          - Input : Asset id, text to append
          - Output : 200 OK response
  - Asset Manager:
    - The AssetManager will inherit BaseManager, which inherits IManager, and IAssetManager.
      - Beyond the base class and interfaces, AssetManager will implement appropriate methods to assist AssetsController.     
  - Asset Manager:
    - The AssetManager will inherit BaseManager, which inherits IManager, and IAssetManager.
      - Beyond the base class and interfaces, AssetManager will implement appropriate methods to assist AssetsController.
  - Asset Repository:
    - The AssetRepository will retrieve all assets or details of an asset as well as add, edit, delete, or export assets.
  - Asset Data Models:
    - The Asset data model will be used to store details of each asset.  It will inherit NamedEntity, which inherits Entity.
      - Beyond the base classes, Asset will have string Type, string TextValue, double NumberValue, string JsonValue, Guid FileId, long SizeInBytes, and Guid AgentId.
    - The UpdateAssetViewModel will be used to update a text, number, file, or JSON asset.
      - UpdateAssetViewModel will have Guid Id, string Name, string Type, string TextValue, double NumberValue, string JsonValue, Guid FileId, IFormFile File, and string DriveId.
    - The GlobalAssetViewModel will be used to create a new global asset.  It will inherit IViewModel<GlobalAssetViewModel, Asset>.
      - GlobalAssetViewModel will have Guid Id, string Name, string Type, string TextValue, double NumberValue, string JsonValue, Guid FileId, string DriveId, and IFormFile File.
    - The AgentAssetViewModel will be used to create a new agent asset.
      - AgentAssetViewModel will have string Name, string TextValue, double NumberValue, string JsonValue, Guid FileId, IFormFile File, Guid DriveId, and Guid AgentId.

**Sequence Diagrams**

- [Asset Component Sequence Diagram](Sequence Diagrams/AssetComponentSequenceDiagram.png)
- [Count Assets Sequence Diagram](Sequence Diagrams/CountAssetsSequenceDiagram.png)
- [Asset Details Sequence Diagram](Sequence Diagrams/AssetDetailsSequenceDiagram.png)
- [Add Asset Sequence Diagram](Sequence Diagrams/AddAssetSequenceDiagram.png)
- [Export Asset File Sequence Diagram](Sequence Diagrams/ExportAssetSequenceDiagram.png)
- [Edit Asset Sequence Diagram](Sequence Diagrams/EditAssetSequenceDiagram.png)
- [Edit Asset File Sequence Diagram](Sequence Diagrams/EditAssetFileSequenceDiagram.png)
- [Edit Asset Property Sequence Diagram](Sequence Diagrams/EditAssetPropertySequenceDiagram.png)
- [Delete Asset Sequence Diagram](Sequence Diagrams/DeleteAssetSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A