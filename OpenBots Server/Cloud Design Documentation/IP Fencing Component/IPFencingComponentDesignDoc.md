Author: Dairon Hernandez
Creation Date: 11/25/2020

Updated On: 3/18/2021
Updated By: Nicole Carrero

**IPFencing Component**

**Context**

- Problem: Users are not able to restrict what IPs are able to access the API endpoints. Administrators should be able to set which IPs are able to make requests to the API.
- Requirements: Ensure that an Administrator is able to set an AllowList or a DenyList of APIs that will be prohibited or permitted depending on the desired behavior.

**Component Scope**

- In-Scope:
  - Administrators can set which IPs are allowed access to the API.
  - Administrators can allow certain IPRanges to have access to the API.
  - Administators can allow only requests which contain certain headers and header values.
- Out-of-Scope:
  - Allow Users to set IPRanges and Headers

**Design**

- Overview: The IPFencing values can be set and updated using the IPFencing controller. Once an organization's rules have been created, they are then applied using the IPFilter middleware. The IPFilter middleware uses the IPFencing manager to call the necessary methods that verify if the IP matches and of the IP ranges and or headers. If an IP is listed on one of the organization's rules, depending on what the rule's usage is, this request will succeed or be restricted form access.
- Proposed Solution:
  - User Interface:
    - In the Organization Settings there will be a section to update the values of the current Organization's setting.
    - The settings will request that a one of allow or deny modes be selected.
      - If allow mode is selected, then any request except the ones that are listed as deny in the IPFencing table will be allowed.
      - If the deny mode is selected, then no requests will be allowed except the ones that match on one of the IPFencing rules.
    - Once an IPFencing mode has been selected, it becomes possible to add new rules to the IPFencing table.
        - Rules may contain specific IPV4/IPV6 addresses, IPV4/IPV6 ranges, or Header/Header value pairs
  - IPFencingController:
    - The IPFencing controller will be responsible for performing the necessary CRUD operations on the IPFencing table. 
    - NOTE: The current API version is 1.
    - Routes:
      - All IPFencing: [HttpGet("api/apiVersion/IPFencing")]
        - Payloads
          - Input : None
          - Output : JSON file contataining all IPFencing rules for the specified Organization
      - Individual IPFencing details: [HttpGet("api/apiVersion/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : Organization id, IP fencing id
          - Output : JSON file of all individual IPFencing information for the specified Organization
      - Create an IPFencing: [HttpPost("api/apiVersion/Organizations/{organizationId}/IPFencing")]
        - Payloads
          - Input : Organization id, CreateIPFencingViewModel data
          - Output : JSON file listing new IPFencing information
      - Edit IPFencing: [HttpPut("api/apiVersion/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : Organization id, IP fencing model data
          - Output : JSON file listing updated IPFencing information
      - Delete IPFencing: [HttpDelete("api/apiVersion/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : Organization id, IP fencing id
          - Output : 200 OK response
      - Edit IPFencing property: [HttpPatch("api/apiVersion/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : Organization id, IP fencing id, JsonPatchDocument in request body with changes
          - Output : 200 OK response
  - IPFencing Manager:
    - The IPFencingManager will inherit BaseManager, which inherits IManager, and IIPFencingManager.
      - Beyond the base class and interfaces, IPFencingManager will implement the appropriate methods to assist IPFencingController and the IPFilter MiddleWare.
  - IPFencingRepository:
    - The IPFencingRepository will inherit EntityRepository<IPFencing>, which inherits EntityRepository, and IIPFencingRepository.
      - Beyond the base classes and interface, IPFencingRepository will retrieve all IPFencing, find an individual IPFencing's details, and support getting by paretnID.
  - IPFencing Data Model:
    - The IPFencing data model will be used to view details of each created IPFencing rule.  It will inherit the Entity clas.
      - Beyond the base class, IPFencing will have UsageType Usage, RuleType Rule, string IPAddress, string IPRange, string HeaderName, string HeaderValue, Guid OrganizationId.

**Sequence Diagrams**

- [IPFencing Component Sequence Diagram](Sequence Diagrams/IPFencingComponent.png)
- [IPFilter MiddleWare Sequence Diagram](Sequence Diagrams/IPFiltersMiddleware.png)

**Unit Tests**

- Positive Test Cases:

- Negative Test Cases:
