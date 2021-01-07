Author: Dairon Hernandez
Creation Date: 08/06/2020

Updated On: 11/18/2020
Updated By: Dairon Hernandez

**Agent Component**

**Context**

- Problem: Administrators are not able to create associations between accounts and declared machines. Administrators should be able to associate machines and accounts and store them in an agent entity. Additionally, administrators and users should be able to view the existing agents and monitor their heartbeat/execution status. Finally, administrators should be able to edit/delete agents from this view.

- Requirements: The Administrator should be able to view a list of existing agents and their current heartbeat/execution status, declare new agents as an association between machines and accounts, and edit/delete existing agents. Users should be able to view a list of existing agents and their current heatbeat/execution status. Machines should be able to connect with and receive an agent id.

**Component Scope**

- In-Scope:
  - Administrator and/or user can view a list of agents and their heartbeats.
  - Administrator can declare new agents.****
  - Administrator can edit and delete existing agents.
  - Administrator can connect machines and receive an agent id.
- Out-of-Scope:
  - User can edit and delete existing agents.
  - User can declare new agents.
  - User can connect machines and receive an agent id.

**Design**

- Overview:  The overall structure uses the AgentsController to allow the user to make a web request for the list of agents, agent details, creating a new agent, or editing/deleting an existing agent.  In addition, there will be a connect method to handshake the machine and the agent entity on the Server.  Once the machine and agent are connected, a heartbeat method can be called inside the controller which will be responsible for validating agents and updating the heartbeat status.
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link that redirects the user to the list of Agents.
      - The agents will be displayed in a table.
        - There will be a button to add a new agent to the list.
        - The headers will be: Name, Status, Health, Is Enabled, Jobs, View, Edit, Delete.
          - The appropriate data will be displayed underneath each heading for all agents.
          - The user can click on each agent to view, connect, edit, or delete it.
          - Hovering over one of the agent shows the last heartbeat time.
  - Agent Controller:
    - The AgentsController will make an API request and use the AgentManager to access the AgentRepository to retrieve all the agents from the Server and will return that information back to the view. The controller will utilize this same structure to view, add, edit, or remove an agent.  Additionally, the AgentController can receive a connect, disconnect, or a heartbeat request, which will be handled by the AgentManager.
    - Routes:
      - All agents: [HttpGet("api/v1/agents")]
        - Payloads
          - Input : None
          - Output : JSON file listing all agent information
      - Count agents: [HttpGet("api/v1/agents/count")]
        - Payloads
          - Input : None
          - Output : Count of all agents
      - Agent details: [HttpGet("api/v1/agents/{id}")]
        - Payloads
          - Input : Agent id
          - Output : JSON file listing agent information
      - Create an agent: [HttpPost("api/v1/agents")]
        - Payloads
          - Input : Name, MachineName, MacAddresses, IPAddresses, MachineCredentials, and IsEnabled
          - Output : 200 OK response
      - Edit an agent: [HttpPut("api/v1/agents/{id}")]
        - Payloads
          - Input : Agent id, Name, MachineName, MacAddresses, IPAddresses, IsEnabled, LastReportedOn, LastReportedStatus, LastReportedWork, LastReportedMessage, IsHealthy, and CredentialId
          - Output : 200 OK response
      - Delete an agent: [HttpDelete("api/v1/agents/{id}")]
        - Payloads
          - Input : Agent id
          - Output : 200 OK response
      - Edit agent property: [HttpPatch("api/v1/agents/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK reposnse
      - Connect verified machine: [HttpPatch("api/v1/agents/connect")]
        - Payloads
          - Input : JsonPatchDocument in request body with MacAddresses and MachineName changes
          - Output : AgentId, AgentName
      - Disconnect verified machine [HttpPatch("api/v1/agents/disconnect")]
        - Payloads
          - Input : JsonPatchDocument in request body with AgentId, MachineName, and MacAdresses changes
          - Output : 200 OK response
      - Agent heartbeat: [HttpPost("api/v1/agents/{AgentId}/AddHeartbeat")]
        - Payloads
          - Input : Heartbeat viewmodel containing lastUpdated fields
          - Output : 200 OK containing the created AgentHeartbeat      
      - Get Agent heartbeats: [HttpGet("api/v1/agents/{AgentId}/AgentHeartbeats")]
        - Payloads
          - Input : AgentId
          - Output : 200 OK containing the created AgentHeartbeat  
      - Agents lookup: [HttpGet("api/v1/agents/getlookup")]
        - Payloads
          - Input : None
          - Output : List of AgentIds and AgentNames
  - Agent Manager(s):
    - The AgentManager will inherit BaseManager and IAgentManager, which both inherit IManager.
      - Beyond the base class and interfaces, AgentManager will implement the ValidateAgent() method to assist AgentsController.  It will be responsible for performing the heartbeat functionalities of validating the Agent id.
  - Agent Repository(s):
    - The AgentRepository will inherit EntityRepository, which inherits ReadOnlyEntityRepository, and IAgentRepository, which inherits IEntityRepository.
      - Beyond the base classes and interfaces, AgentRepository will retrieve all agents, add a new agent, retrieve/edit/delete an agent by id, connect/disconnect an agent, and find an agent by machine name, mac address, and IP address.
    - AgentHeartbeatRepository will be responsible for updating and retreiving information about an agent's hearbeat
  - Agent Data Model(s):
    - The Agent data model will be used to view details of each agent.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, Agent will have string MachineName, Guid MachineCredentials, string MacAddresses, string IPAddresses, bool IsEnabled, bool isConnected, and Guid CredentialId.
    - The AgentHeartbeat model will be used to view details of each agent's heatbeat status. It will inherit Entity and INonAuditable.      
      - Heartbeats will contain the fields DateTime LastReportedOn, string LastReportedStatus, string LastReportedWork, string LastReportedMessage, bool IsHealthy

**Sequence Diagrams**

- [Agent Component Sequence Diagram](Sequence Diagrams/AgentComponentSequenceDiagram.png)
- [Count Agents Sequence Diagram](Sequence Diagrams/CountAgentsSequenceDiagram.png)
- [Agent Details Sequence Diagram](Sequence Diagrams/AgentDetailsSequenceDiagram.png)
- [Create Agent Sequence Diagram](Sequence Diagrams/CreateAgentSequenceDiagram.png)
- [Edit Agent Sequence Diagram](Sequence Diagrams/EditAgentSequenceDiagram.png)
- [Delete Agent Sequence Diagram](Sequence Diagrams/DeleteAgentSequenceDiagram.png)
- [Edit Agent Property Sequence Diagram](Sequence Diagrams/EditAgentPropertySequenceDiagram.png)
- [Connect Agent Sequence Diagram](Sequence Diagrams/ConnectAgentSequenceDiagram.png)
- [Disconnect Agent Sequence Diagram](Sequence Diagrams/DisconnectAgentSequenceDiagram.png)
- [Agent Heartbeat Sequence Diagram](Sequence Diagrams/AgentHeartbeatSequenceDiagram.png)
- [Agents Lookup Sequence Diagram](Sequence Diagrams/AgentsLookupSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases:
  - Verify we can correctly fetch the agent's credential name from the credentials table.
  - Verify that we can get the agent's username from the aspNetUsers table
  - Verify referential integrity for Agents returns true when an agent vialoates referential integrity.
- Negative Test Cases:
  - Referential integrity returns false when it is not violated by the provided Agent ID.