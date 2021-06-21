Author: DH
Creation Date: 11/25/2020

Updated By: NC
Updated On: 6/16/2021

**IPFencing Component**

**Context**

- Problem: Users are not able to restrict what IPs are able to access the API endpoints. Administrators should be able to set which IPs are able to make requests to the API.
- Requirements: Ensure that an Administrator is able to set an AllowList or a DenyList of APIs that will be prohibited or permitted depending on the desired behavior.

**Component Scope**

- In-Scope:
  - Administrators can set which IPs are allowed access to the API.
  - Administrators can allow certain IP ranges to have access to the API.
  - Administators can allow only requests which contain certain headers and header values.
- Out-of-Scope:
  - Users can set IP ranges and headers.

**Design**

- Overview: The IP fencing values can be set and updated using the IPFencingController. Once an organization's rules have been created, they are then applied using the IP filter middleware. The IP filter middleware uses the IPFencingManager to call the necessary methods that verify if the IP matches and checks the IP ranges and/or headers. If an IP is listed on one of the organization's rules, depending on what the rule's usage is, this request will succeed or be restricted from access.
- Proposed Solution:
  - User Interface:
    - In the organization's Settings section, there will be a page to update the values of the current organization's settings.
    - The settings will request that allow or deny modes be selected.
      - If allow mode is selected, then any request except the ones that are listed as deny in the IP fencing table will be allowed.
      - If the deny mode is selected, then no requests will be allowed except the ones that match on one of the IP fencing rules.
    - Once an IP fencing mode has been selected, it becomes possible to add new rules to the IP fencing table.
        - Rules may contain specific IPV4/IPV6 addresses, IPV4/IPV6 ranges, or header/header value pairs.
  - IP Fencing Controller:
    - The IPFencingController will be responsible for performing the necessary operations on the IPFencing data table.
    - NOTE: The current API version is 1.
      - All IP fencing: [HttpGet("api/v{apiVersion}/IPFencing")]
        - Payloads
          - Input : None
          - Output : JSON file containing all IP fencing rules for the specified organization
      - IP fencing details: [HttpGet("api/v{apiVersion}/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : IP fencing id
          - Output : JSON file of all individual IP fencing information for the specified organization
      - Create IP fencing: [HttpPost("api/v{apiVersion}/Organizations/{organizationId}/IPFencing")]
        - Payloads
          - Input : CreateIPFencingViewModel 
          - Output : JSON file listing new IP fencing information
      - Edit IP fencing: [HttpPut("api/v{apiVersion}/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : IP fencing data model
          - Output : JSON file listing updated IP fencing information
      - Delete IP fencing: [HttpDelete("api/v{apiVersion}/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : IP fencing id
          - Output : 200 OK response
      - Edit IP fencing property: [HttpPatch("api/v{apiVersion}/Organizations/{organizationId}/IPFencing/{id}")]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
  - IP Fencing Manager:
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

- Positive Test Cases: N/A
- Negative Test Cases: N/A
