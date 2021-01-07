Author: Nicole Carrero
Creation Date: 8/19/2020

Updated On: 11/30/2020
Updated By: Nicole Carrero

**System Configuration Component**

**Context**

- Problem: Users cannot view and modify core system configuration settings.  Administrators should be able to view and modify core system configuration settings from a provided user interface.  Administrators should be able to view a list of all accessible values and modify key features.
- Requirements: Ensure that an administrator is able to modify key system configuration settings.

**Component Scope**

- In-Scope:
  - Administrator can view configuration settings for general settings.
  - Administrator can view system configuration setting details for each item in the provided list.
  - Administrator can edit configuration settings for general settings.
- Out-of-Scope:
  - Administrator and/or user can add or remove default key system configuration settings.

**Design**

- Overview: The overall structure uses the ConfigurationValuesController to allow the user to make a web request for the system configuration settings.  The ConfigurationValuesController gets called and then uses the ConfigurationValueRepository to retrieve, add, edit, or delete the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the Settings page.
      - The user will be able to view and modify default configuration settings, such as enabling Swagger, maximum amount of records to export into a file, maximum amount of records to return on a page, file adapter, file storage path, file storage provider, and maximum amount of retries for processing a queue item.
  - Configuration Values Controller:
    - The ConfigurationValuesController will make an API request to access the ConfigurationValueRepository to modify the appporpiate settings.
      - Routes:
        - All configuration values: [HttpGet("api/v1/configurationvalues")]
          - Payloads
            - Input : None
            - Output : JSON file listing all configuration value information
        - Configuration values count: [HttpGet("api/v1/configurationvalues/count")]
          - Payloads
            - Input : None
            - Output : Count of all configuration values
        - Configuration value details: [HttpGet("api/v1/configurationvalues/{id}")]
          - Payloads
            - Input : Configuration value id
            - Output : JSON file listing specific configuration value details
        - Add configuration value: [HttpPost("api/v1/configurationvalues")]
          - Payloads
            - Input : Configuration value data model
            - Output : JSON file listing newly created configuration value details
        - Edit configuration value: [HttpPut("api/v1/configurationvalues/{id}")]
          - Payloads
            - Input : Configuration value id, Configuration value data model
            - Output : JSON file listing updated configuration value details
        - Edit configuration value property: [HttpPatch("api/v1/configurationvalues/{id}")]
          - Payloads
            - Input : Configuration value id, JSONPatchDocument in request body
            - Output : 200 OK response
        - Delete configuration value: [HttpDelete("api/v1/configurationvalues/{id}")]
          - Payloads
            - Input : Configuration value id
            - Output : 200 OK response
  - EFConfigurationProvider & EFConfigurationSource:
    - These classes will allow the configuration values to be used as default configuration settings. 
  - ConfigurationValueRepository:
    - The ConfigurationValueRepository will inherit EntityRepository<ConfigurationValue>, which inherits ReadOnlyEntityRepository<ConfigurationValue>, and IConfigurationValueRepository.
      - Beyond the base classes and interface, ConfigurationValueRepository will access the ConfigurationValue data table to retrieve, add, edit, or delete the entity in the Server.
- Data Models:
  - Configuration Value Data Model:
    - The ConfigurationValue data model will be used to store configuration values for default settings.  It will inherit the NamedEntity class, which inherits Entity and INamedEntity.  Entity inherits the IEntity interface.
      - Beyond the base classes and interface, ConfigurationValue will have string Name string Value.

**Sequence Diagrams**

- ConfigurationValuesController:
  - [All Configuration Values Sequence Diagram](Sequence Diagrams/AllConfigurationValuesSequenceDiagram.png)
  - [Configuration Values Count Sequence Diagram](Sequence Diagrams/ConfigurationValueCountSequenceDiagram.png)
  - [Configuration Value Details Sequence Diagram](Sequence Diagrams/ConfigurationValueDetailsSequenceDiagram.png)
  - [Add Configuration Value Sequence Diagram](Sequence Diagrams/AddConfigurationValueSequenceDiagram.png)
  - [Edit Configuration Value Sequence Diagram](Sequence Diagrams/EditConfigurationValueSequenceDiagram.png)
  - [Edit Configuration Value Property Sequence Diagram](Sequence Diagrams/EditConfigurationValuePropertySequenceDiagram.png)
  - [Delete Configuration Value Sequence Diagram](Sequence Diagrams/DeleteConfigurationValueSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A