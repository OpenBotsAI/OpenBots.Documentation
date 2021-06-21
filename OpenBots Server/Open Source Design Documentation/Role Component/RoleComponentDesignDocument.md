Author: NC
Creation Date: 8/20/2020

Updated By: NC
Updated On: 6/16/2021

**Role Component**

**Context**

- Problem: Users are not assigned roles.  An administrator should be able to assign roles to existing users to regulate their access.
- Requirements: Ensure that an administrator is able to assign roles to a user to dictate their access.

**Component Scope**

- In-Scope:
  - An administrator can assign a user an administrator role.
  - An administrator can reassign an administrator to a user role.
- Out-of-Scope:
  - A user can assign or reassign roles to users and administrators.
  - An administrator and/or user can add, edit, or delete roles from a list of all roles.
  - An administrator and/or user can define the access levels provided for key components by a role.

**Design**

- Overview: The overall structure uses the MembershipController to allow the user to make a web request to change a user to administrator or vice versa.  Once the given user exists in an organization (see User Component Design Document for more information), the MembershipController can get called, which in turn accesses the methods in the MembershipManager, and then uses the OrganizationMemberRepository to edit the appropriate data from the Server if the request is valid.  If the request is invalid, the user will receive an error message stating the error that occurred while trying to process the request.
- Proposed Solution:
  - User Interface:
    - In the Dashboard under Teams, administrators will be able to click the "All Team Members" link to view all team members.
      - In the table of team members, administrators will have the option to grant users administrative rights or remove them.
  - MembershipController:
    - The MembershipController inherits from EntityController<Organization>.  It will make an API request and use the MembershipManager to access the OrganizationMemberRepository to update admin rights to the Server and will return that information back to the view.
    - NOTE: The current API version is 1.
    - Routes:
      - Grant administrator permission: [HttpPut("/api​/v{apiVersion}​/People​/{personId}​/Organizations​/{id}​/GrantAdmin")]
        - Payloads
          - Input : Person id, Organization id, IsAdministrator = true
          - Output : 200 OK response
      - Revoke administrator permission: [HttpPut("/api/v{apiVersion}/People/{personId}/Organizations/{id}/RevokeAdmin")]
        - Payloads
          - Input : Person id, Organization id, IsAdministrator = false
          - Output : 200 OK response
  - MembershipManager:
    - The MembershipManager will inherit BaseManager, which inherits IManager, and IMemebershipManager.
      - Beyond the base class and interfaces, MembershipManager will implement the appropriate methods to assist MembershipController.
  - OrganizationMemberRepository:
   - The OrganizationMemberRepository will inherit TenantEntityRepository<OrganizationMember>, which inherits EntityRepository and ITenantRepository, and IOrganizationMemberRepository.
     - Beyond the base classes and interfaces, OrganizationMemberRepository will find the appropriate person by organization id and person id, then update the entity in the Server.
  - OrganizationMember Data Model:
    - The OrganizationMember data model will be used to view details of each member in an organization.  It will inherit the Entity class and ITenanted Interface.
      - Beyond the base class and interface, OrganizationMember will have Guid OrganizationId, Guid PersonId, bool IsAdministrator, string ApprovedBy, DateTime ApprovedOn, bool IsInvited, string InvitedBy, DateTime InvitedOn, bool InviteAccepted, DateTime InviteAcceptedOn, bool IsAutoApprovedByEmailAddress, and Person Person.

**Sequence Diagrams**

- [Role Component Sequence Diagram](Sequence Diagrams/RoleComponentSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A