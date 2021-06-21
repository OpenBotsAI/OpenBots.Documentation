Author: DH
Creation Date: 02/19/2021

Updated By: NC
Updated On: 6/15/2021

**AgentGroups Component**

**Context**

- Problem: Users are not able to synchronize mutiple agents at once. Users should be able to associate agents in a group in order to execute jobs.

- Requirements: Users should be able to add existing agents to an existing group. If a schedule is created with an agent group id, then the new job can be executed by any of the agents in the agent group.

**Component Scope**

- In-Scope:
  - Administrator and/or user can view a list of agent groups and their agents.
  - Users can create new groups and add existing agents to them.
  - Users can edit or delete existing groups.
  - Users can add agents to multiple groups.
- Out-of-Scope:
  - User can add agents that have not yet been created to a group.

**Design**

- Overview:  The overall structure uses the AgentGroupsController to allow the user to make a web request for the list of agent groups, agent agent group details, creating a new agent group, or editing/deleting an existing agent group.  In addition, there will be a get all all group members method which can be used to get a list of all agents associated with a particular agent group id.
- Proposed Solution:
  - User Interface:
    - Dashboard
      - The Dashboard will display a link that redirects the user to the list of agent groups.
      - The agents groups will be displayed in a table.
        - There will be a button to add a new agent group to the list.
        - The headers will be: Name, Is Enabled, View, Edit, Delete.
          - The appropriate data will be displayed underneath each heading for all agent groups.
          - The user can click on each agent to view, connect, edit, or delete it.
          - Navigating to the view page of one of the agent groups shows all agents that are part of this group.
  - Agent Groups Controller:
    - The AgentGroupsController will use the AgentGroupsManager to handle business logic for the different endpoints in addition to the existing entity controllers. The controller will contain the basic operations necessary for the agent groups entity, in addition to allowing users to insert new records for agent group members as well as retreiving records from agent group members.
    - Routes:
    - NOTE: The current API version is 1.
      - All agent groups: [HttpGet("api/v{apiVersion}/agentGroups")]
        - Payloads
          - Input : None
          - Output : JSON file listing all agent group information
      - Count agent groups: [HttpGet("api/v{apiVersion}/agentGroups/count")]
        - Payloads
          - Input : None
          - Output : Count of all agent groups
      - Agent group details: [HttpGet("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : Agent group id
          - Output : JSON file listing agent group information
      - Agentn group details view: [HttpGet("api/v{apiVersion}/agentGroups/view/id")]
        - Payloads
          - Input : Agent group id
          - Output : JSON file listing agent group view information (includes the group members)
      - Create an agent group: [HttpPost("api/v{apiVersion}/agentGroups")]
        - Payloads
          - Input : Name, Description, IsEnabled
          - Output : 200 OK response
      - Create an agent group: [HttpPut("api/v{apiVersion}/agentGroups/{id}/UpdateGroupMembers")]
        - Payloads
          - Input : Agent id and agent group id
          - Output : Newly created agent group members
      - Edit an agent group: [HttpPut("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : Name, Description, IsEnabled
          - Output : 200 OK response
      - Delete an agent group: [HttpDelete("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : Agent group id
          - Output : 200 OK response
      - Edit agent groups property: [HttpPatch("api/v{apiVersion}/agentGroups/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Agent groups lookup: [HttpGet("api/v{apiVersion}/agentGroups/GetLookup")]
        - Payloads
          - Input : None
          - Output : Json file listing AgentGroupsLookupViewModel information
  - Agent Group Manager:
    - The AgentGroupManager will be responsible for executing business logic assocated with the agent group entity. The manager will include the appropriate methods to assist the AgentGroupsController.
  - Agent Group Repository:
    - The AgentGroup Repository will inherit EntityRepository, which inherits ReadOnlyEntityRepository, and IAgentGroupRepository, which inherits IEntityRepository.
      - Beyond the base classes and interfaces, AgentRepository will retrieve all agent groups, and perform basic operations on the agent groups table.
    - AgentGroupMemberRepository will be responsible for associating agent groups with Agents.
  - Agent Groups Data Models:
    - The AgentGroups data model will be used to view details of each agent group.  It will inherit the NamedEntity class, which inherits the Entity class.
      - Beyond the base classes, AgentGroups will have string Name, string Description, and boolean IsEnabled.
    - The AgentGroupMember model will be used to view details of each group's members. AgentGroupMember will inherit Entity.
      - AgentGroupMember will contain Guid AgentGroupId and Guid AgentId.
    - The AgentGroupViewModel will be used to view details of each agent group.  It inherits NamedEntity, which inherits Entity, and IViewModel<AgentGroup, AgentGroupViewModel>.
      - Beyond the base classes and interface, AgentGroupViewModel will contain bool IsEnabled, string Description, and IEnumerable<AgentGroupMemerViewModel> AgentGroupMembers.
    - The AgentGroupMemberViewModel will be used to view details of each agent group.
      - It will contain Guid Id, Guid AgentGroupId, Guid AgentId, string AgentGroupName, and string AgentName.
    - The AgentGroupsLookupViewModel will be used to view a list of agent group names.
      - It will contain Guid AgentGroupId and string AgentGroupName.

**Sequence Diagrams**

- [Agent Component Sequence Diagram](Sequence Diagrams/AgentGroupsSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A