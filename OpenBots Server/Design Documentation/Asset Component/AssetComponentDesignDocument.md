Author: Dairon Hernandez
Creation Date: 08/13/2020

Updated On: 9/28/2020
Updated By: Nicole Carrero

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

- Overview: The overall structure uses the AssetsController to allow the user to make a web request for either the entire list of assets or details of an individual asset.  If the user requests access to the entire list of assets, the request goes to the AssetsController, which may use the ProcessManager, and then calls the AssetsRepository for access to the appropriate data from the Server.  If the request is valid, the data will be sent back to the user in the view.  If the request is invalid, the user will receive an error message stating that the assets could not be found.  The same process will occur when the user requests the asset details as well as add, edit, delete, or export an asset.

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
          - The user can edit each asset, where a new file can be uploaded to overwrite the old one.
          - The user can delete each asset.
  - Assets Controller:
    - The AssetsController will make an API request to the AssetRepository to retrieve all the assets from the Server and will return that information back to the view.  The user can view all assets, view asset details, and add, edit, or delete an asset.
    - Routes:
      - Get all assets: [HttpGet("api/v1/assets")]
        - Payloads
          - Input : None
          - Output : JSON file containing a list of all assets
      - Count all assets: [HttpGet("api/v1/assets/count")]
        - Payloads
          - Input : None
          - Output : Total count of assets from the Server
      - Get asset details: [HttpGet("api/v1/assets/{id}")]
        - Payloads
          - Input : Asset id
          - Output : JSON file containing details for the provided asset id
      - Add an asset: [HttpPost("api/v1/assets")]
        - Payloads
          - Input : Asset model data
          - Output : JSON file listing new asset information
      - Upload asset file: [HttpPost("api/v1/assets/{id}/upload")]
        - Payloads
          - Input : Asset file
          - Output : JSON file listing updated asset information
      - Export asset file: [HttpGet("api/v1/assets/{id}/export")]
        - Payloads
          - Input : Asset id
          - Output : Asset file
      - Update asset: [HttpPut("api/v1/assets/{id}")]
        - Payloads
          - Input : Name, Value (Number Value, Text Value, or JSON Value)
          - Output : 200 OK response
      - Update asset with file: [HttpPut("api/v1/assets/{id}/update")]
        - Payloads
          - Input : Name, File (to be saved in Binary Objects Component and referenced in Assets using Binary Object id)
          - Output : 200 OK response
      - Update asset property: [HttpPatch("api/v1/assets/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete asset: [HttpDelete("api/v1/assets/{id}")]
        - Payloads
          - Input : Asset id
          - Output : 200 OK response     
  - Asset Manager(s):
    - There is no manager for the Asset Component because it utilizes methods provided in ProcessManager: Export(), Update(), and Upload().  See Process Component design documentation for more information.
  - Asset Repository(s):
    - The AssetRepository will retrieve all assets or details of an asset as well as add, edit, delete, or export assets.
  - Asset Data Model(s):
    - The Asset data model will be used to store details of each asset.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Asset will have Type (required), TextValue, NumberValue, JsonValue, and BinaryObjectId.

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