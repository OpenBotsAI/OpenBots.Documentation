Author: Dairon Hernandez
Creation Date: 02/19/2021

Updated On: 3/15/2021
Updated By: Dairon Hernandez

**AgentGroups Component**

**Context**

- Problem: Users are not synchronize mutiple agents at once. Users should be able to associate agents in a group in order to execute jobs.

- Requirements: Users should be able to add existing agents to an existing group. If a schedule is created with an agent group id, then the new job can be executed by any of the agents in the agent group.

**Component Scope**

- In-Scope:
  - Administrator and/or user can view a list of agentGroups and their agents
  - Users can create new groups and add existing agents to them.
  - Users can edit or delete existing groups.
  - Users can add agents to mutiple groups
- Out-of-Scope:
  - User can add agents that have not yet been created to a group

**Design**

- Overview:  The overall structure uses the AgentGroupsController to allow the user to make a web request for the list of agentGroups, agent agentGroupDetails, creating a new agentGroup, or editing/deleting an existing agentGroup.  In addition, there will be a get all AllGroupMembers() method which can be used to get a list of all agents associated with a particular agentGroup id
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link that redirects the user to the list of agentsGroups.
      - The agentsGroups will be displayed in a table.
        - There will be a button to add a new agentGroup to the list.
        - The headers will be: Name, IsEnabled, View, Edit, Delete.
          - The appropriate data will be displayed underneath each heading for all agentsGroups.
          - The user can click on each agent to view, connect, edit, or delete it.
          - Navigating to the view page of one of the agentGroups shows all agents that are part of this group.
  - AgentGroups Controller:
    - The AgentGroupsController will use the AgentGroupsManager to handle business logic for the different endpoints in addition to the existing entity controllers. The controller will contain the basic CRUD operations necessarry for the agentGroups entity, in addition to allowing users to insert new records for agentGroupMembers as well as retreiving records from agentGroupMembers
    - Routes:
    - NOTE: The current API version is 1.
      - All agentsGroups: [HttpGet("api/v{apiVersion}/agentGroups")]
        - Payloads
          - Input : None
          - Output : JSON file listing all agentGroup information
      - Get view for the specified id: [HttpGet("api/v{apiVersion}/agentGroups/view/id")]
        - Payloads
          - Input : agentGroup id
          - Output : JSON file listing AgentGroup view information. (Includes the GroupMembers)
      - Count agentsGroups: [HttpGet("api/v{apiVersion}/agentGroups/count")]
        - Payloads
          - Input : None
          - Output : Count of all agentsGroups
      - Agent details: [HttpGet("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : AgentGroup id
          - Output : JSON file listing AgentGroup information
      - Create an agentGroup: [HttpPost("api/v{apiVersion}/agentGroups")]
        - Payloads
          - Input : Name, Description, IsEnabled
          - Output : 200 OK response
      - Create an agentGroup: [HttpPut("api/v{apiVersion}/agentGroups/{id}/UpdateGroupMembers")]
        - Payloads
          - Input : Agent id and AgentGroup Id
          - Output : Newly created AgentGroupMembers
      - Edit an agentGroup: [HttpPut("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : Name, Description, IsEnabled
          - Output : 200 OK response
      - Delete an agentGroup: [HttpDelete("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : AgentGroup id
          - Output : 200 OK response
      - Edit agentGroups property: [HttpPatch("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK reposnse
  - AgentGroup Manager(s):
    - The AgentGroupManager will be responsible for executing business logic assocated with the AgentGroup entity. The manager will include the appropriate methods to assist the AgentGroupsController.
  - AgentGroup Repository(s):
    - The AgentGroup Repository will inherit EntityRepository, which inherits ReadOnlyEntityRepository, and IAgentGroupRepository, which inherits IEntityRepository.
      - Beyond the base classes and interfaces, AgentRepository will retrieve all agentsGroups, and perform CRUD operations on the agentGroups table.
    - AgentGroupMemberRepository will be responsible for associating AgentGroups with Agents.
  - AgentGroups Data Model(s):
    - The AgentGroups data model will be used to view details of each agentGroup.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, AgentGroups will have string Name, string Description and boolean IsEnabled.
    - The AgentGroupMember model will be used to view details of each groups's members. AgentGroupMember will inherit Entity. AgentGroupMember will contain Guid AgentGroupId and Guid AgentId.

**Sequence Diagrams**

- [Agent Component Sequence Diagram](https://openbots.sharepoint.com/:i:/s/OpenBotsInc/EWrHyBF29cNJva08JZDnSJABuje8nSSKbev1PePG2pxfEw?e=sP64yb)

**Unit Tests**

- Positive Test Cases:
- Negative Test Cases: