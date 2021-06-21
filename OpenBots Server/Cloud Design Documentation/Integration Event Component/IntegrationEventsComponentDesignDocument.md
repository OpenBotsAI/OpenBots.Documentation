Author: DH
Creation Date: 1/06/2021

Updated By: NC
Updated On: 6/18/2021

**IntegrationEvents Component**

**Context**

- Problem: Users are not able easily track events that occur to entities on the server.
- Requirements: Ensure that users are able to subscribe to integration events that exist within the Server.

**Component Scope**

- In-Scope:
  - Users can choose the system and/or business events that they would like to subscribe to and be notified on.
  - Users can choose to subscribe to a specific entity and be notified of all events that occur on that entity.
  - Users can choose the transport type for the integration event.
  - User can create new business events.
- Out-of-Scope:
  - Users can create new system events.

**Design**

- Overview: The integration event subscriptions can be created and updated using the IntegrationEventsSubscriptionsController. Users may only subscribe to events that exist in the Integration Events table. When an event occurs, a new integration event log will be created and if any subscriptions exist for that event, then a new integration event subscripton attempt will occur.  This all happens in the WebhookPublisher when the PublishAsync method is called.
- Proposed Solution:
  - User Interface:
    - In the Dashboard, there will be a link to redirect the user to the different integration event tabs:
        - System Events: Will contain a list of all the system events that a user can subscribe to.
        - Business Events: Will contain a list of all the Business events that a user can subscribe to.
        - Add Business Event: Will allow users to create new business events by choosing their own name, type, description and payload.
        - Raise Business Event: Will allow users to raise an existing business event which will trigger any subscription that exists for the specified event.
        - Subscriptions: Will contain a list of all existing subscriptions and additionally contains a button that allows users to create new subscriptions.
        - Add Subscriptions: Allows users to create new subscriptions.
        - Logs Tab: Displays a paginated list of all integration events that have occured in descending order.
- Controllers
  - NOTE: The current API version is 1.
  - Integration Events Controller:
    - The IntegrationEventsController will allow users to access information on existing system and business events. Additionally, the EventsController will allow user to create, edit and raise business events.
      - All integration events: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents")]
        - Payloads
          - Input : None
          - Output : JSON file contataining all integration events
      - Integration event details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/{id})]
        - Payloads
          - Input : Integration event id
          - Output : JSON file of integration event information for the specified id
      - Lookup for integration events: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/IntegrationEventLookup)]
        - Payloads
          - Input : None
          - Output : Provides a list of all integration event entity names
      - Add a business event: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/BusinessEvents)]
        - Payloads
          - Input : CreateBusinessEventViewModel data
          - Output : JSON file listing new business event information
      - Raise a business event: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/BusinessEvents/RaiseEvent/{id})]
        - Payloads
          - Input : Business event id
          - Output : 200 OK response
      - Edit business event property: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/BusinessEvents/{id})]
        - Payloads
          - Input : CreateBusinessEventViewModel data
          - Output : JSON file listing updated business event information
      - Edit business event: [HttpPatch("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/BusinessEvents/{id})]
        - Payloads
          - Input : JsonPatchDocument in request body with changes
          - Output : 200 OK response
      - Delete a business event: [HttpDelete("api/v{apiVersion}/organizations/{organizationId}/IntegrationEvents/{id}")]
        - Payloads
          - Input : Business event id
          - Output : 200 OK response
  - Inegration Event Logs Controller
    - The IntegrationEventLogsController will inherit ReadOnlyEntityController<IntegrationEventLog>, which gives users access to the logs of all integration events that have occured.
      - Routes:
        - All logs: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventLogs")]
          - Payloads
            - Input : None
            - Output : JSON file listing all log information
        - Log details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventLogs/{id}")]
          - Payloads
            - Input : Log id
            - Output : JSON file listing log information
        - Logs lookup: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventLogs/IntegrationEventLogsLookup")]
          - Payloads
            - Input : None
            - Output : IntegrationEventLogLookupViewModel information
        - Export log: [HttpGet("api/v{apiVersion}/organizations/{organzationId}/IntegrationEventLogs/ExportPayload/{id}")]
          - Payloads
            - Input : Log id
            - Output : JSON file of event's payload
  - Integration Event Subscription Attempts Controller
    - The IntegrationEventSubscriptionAttemptsController will inherit ReadOnlyEntityController<IntegrationEventSubscriptionAttempt> that will be used to view information on each transport attempt that was made on an integration subscription.
    - Routes:
        - All subscriptions attempts: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscriptionAttempts")]
          - Payloads
            - Input : None
            - Output : JSON file listing all subscription attempt information
        - All subscription attempts view: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscriptionAttempts/view")]
          - Payloads
            - Input : None
            - Output : JSON file listing all SubscriptionAttemptViewModel information
        - Subscription attempt details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscriptionAttempts/{id}")]
          - Payloads
            - Input : Subscription attempt id
            - Output : JSON file listing subscription attempt information
        - Subscription attempt details view: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscriptionAttempts/view/{id}")]
          - Payloads
            - Input : Subscription attempt id
            - Output : JSON file listing SubscriptionAttemptViewModel information
  - Integration Event Subscriptions Controller
    - IntegrationEventSubscriptionsController will inherit EntityController<IntegrationEventSubscription>, that will be used to view, edit, update, and delete integration subscriptions.
    - Routes:
      - All integration event subscriptions: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription")]
        - Payloads
          - Input : Organization id
          - Output : JSON file contataining all IntegrationEventSubscriptions
      - Integration event subscription details: [HttpGet("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription/{id})]
        - Payloads
          - Input : Organization id, IntegrationEventSubscription id
          - Output : JSON file of all IntegrationEventSubscription information for the specified id
      - Create an integration event subscription: [HttpPost("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription")]
        - Payloads
          - Input : Organization id, IntegrationEventSubscription model 
          - Output : JSON file listing new IntegrationEventSubscription information
      - Update an integration event subscription: [HttpPut("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription/{id}")]
        - Payloads
          - Input : Organization id, IntegrationEventSubscription model
          - Output : JSON file listing updated IntegrationEventSubscription information
      - Delete an integration event subscription: [HttpDelete("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription/{id}")]
        - Payloads
          - Input : Organization id, IntegrationEventSubscription id
          - Output : 200 OK response
      - Edit integration event subscription property: [HttpPatch("api/v{apiVersion}/organizations/{organizationId}/IntegrationEventSubscription/{id}")]
        - Payloads
          - Input : Organization id, IntegrationEventSubscription id, and JsonPatchDocument in request body with changes
          - Output : 200 OK response
- Managers
  - IntegrationEventSubscriptionAttempt Manager:
    - The IntegrationEventSubscriptionAttemptManager will inherit BaseManager and IIntegrationEventSubscriptionAttemptManager.
      - Beyond the base class and interface, IntegrationEventSubscriptionAttemptManager will implement the appropriate methods to assist IntegrationEventsController.
  - Webhook Publisher:
    - The WebhookPublisher will inherit IWebhookPublisher.
      - Beyond the base interface, WebhookPublisher will implement the appropriate methods to create an integration event subscription and event log.
  - IntegrationEventLog Manager:
    - The IntegrationEventLogManager will inherit BaseManager and IIntegrationEventLogManager.
      - Beyond the base class and interface, IntegrationEventLogManager will implement the appropriate methods to assist IntegrationEventLogsController.
      - If log storage is set to Azure table storage, the IntegrationEventLogManager will also use the AzureTableLoggerAdapter to add, edit, export, and delete the appropriate logs.
- Repositories
  - Integration Event Repository:
      - The IntegrationEventRepository will inherit TenantEntityRepository<IntegrationEvent>, which inherits ReadOnlyEntityRepository<IntegrationEvent>, and IIntegrationEventRepository.
      - Beyond the base classes and interface, it will access the IntegrationEvents data table to retrieve, add, edit, or delete the entity in the Server.
    - Integration Event Subscription Repository:
      - The IntegrationEventSubscriptionRepository will inherit TenantEntityRepository<IntegrationEventSubscription>, which inherits ReadOnlyEntityRepository<IntegrationEventSubscription>, and IIntegrationEventSubscriptionRepository.
      - Beyond the base classes and interface, it will access the IntegrationEventSubscriptions data table to retrieve, add, edit, or delete the entity in the Server.
    - Integration Event Subscription Attempt Repository
      - The IntegrationEventSubscriptionAttemptRepository will inherit TenantEntityRepository<IntegrationEventsubscriptionAttempt>, which inherits ReadOnlyEntityRepository<IntegrationEventSubscriptionAttempt>, and IIntegrationEventSubscriptionAttemptRepository.
      - Beyond the base classes and interface, it will access the IntegrationEventSubscriptionAttempts data table to retrieve, add, edit, or delete the entity in the Server.
    - Integration Event Log Repository:
      - The IntegrationEventLogRepository will inherit TenantEntityRepository<IntegrationEventLog>, which inherits ReadOnlyEntityRepository<IntegrationEventLog>, and IIntegrationEventLogRepository.
      - Beyond the base classes and interface, it will access the IntegrationEventLogs data table to retrieve, add, edit, or delete the entity in the Server.
  - Models:
    - Integration Event Data Model:
      - The IntegrationEvent data model will be used for system and business events.  It inherits NamedEntity, which inherits Entity, and ITenanted.
        - IntegrationEvent will have string Description, string EntityType, string PayloadSchema, bool IsSystem, and OrganizationId.
    - Integration Event Log Data Model:
      - The IntegrationEventLog data model will be used to log raised system and business events.  It inherits Entity, INonAuditable, and ITenanted.
        - IntegrationEventLog will have string IntegrationEventName, DateTime OccuredOnUTC, string EntityType, string EntityID, string PayloadJSON (required), string Message, string Status, string SHA256Hash, and Guid OrganizationId.
    - Integration Event Subscription:
      - The IntegrationEventSubscription data model will be used to manage subscriptions to system and business events.  It will inherit NamedEntity, which inherits Entity, and ITenanted.
        - IntegrationEventSubscription will have string EntityType, string IntegrationEventName, string EntityID, string EntityName, TransportType TransportType, string HTTP_URL, string HTTP_AddHeader_Key, string HTTP_AddHeader_Value, int Max_RetryCount, Guid QUEUE_QueueID, and Guid OrganizationId.
          - Transport type is a public enum including HTTPS (1), Queue (2), and SignalR (3).
    - Integration Event Subscription Attempt:
      - The IntegrationEventSubscriptionAttempt data model will be used to track subscription attempts of each integration event.  It inherits Entity, INonAuditable, and ITenanted.
        - IntegrationEventSubscriptionAttempt will have Guid EventLogID (required), Guid IntegrationEventSubscriptionID (required), string IntegrationEventName, string Status, int AttemptCounter, and Guid OrganizationId.
    - Create Business Event View Model:
      - The CreateBusinessEventViewModel will be used to add or update a business event.  It inherits IViewModel<CreateBusinessEventViewModel, IntegrationEvent>.
        - It will have string Name (required), string Description, string EntityType, and string PayloadSchema.
    - Raise Business Event View Model:
      - The RaiseBusinessEventViewModel will be used to raise a business event.
        - It will have string EntityId, string EntityName, string Message, and string PayloadJSON.
    - Integration Event Entities Lookup View Model:
      - The IntegrationEventEntitiesLookupViewModel will be used to display the list of available entities and their events.
        - It will have string EntityName and string EventName.
    - Integration Event Log Lookup View Model:
      - The IntegrationEventLogLookupViewModel will be used to display the list of entities and their events.
        - It will have List<string> IntegrationEventNameList and List<string> IntegrationEntityTypeList.
    - Subscription Attempt View Model
      - The SubscriptionAttemptViewModel will be used to display details of a subscription attempt.  It inherits IViewModel<IntegrationEventSubscriptionAttempt, SubscriptionAttemptViewModel>.
        - It will have Guid Id, string TransportType, Guid EventLogID, Guid IntegrationEventSubscriptionID, string IntegrationEventName, string Status, int AttemptCounter, DateTime CreatedOn, and string CreatedBy.

**Sequence Diagrams**
Integration Event/Event Log/Subscription Attempt:
- [Integration Event Component Sequence Diagram](Sequence Diagrams/IntegrationEventSequenceDiagram.png)
- [Integration Event Raise Business Event Sequence Diagram](Sequence Diagrams/RaiseBusinessEventSequenceDiagram.png)

Integration Event Subscription:
- [Integration Event Add Subscription Sequence Diagram](Sequence Diagrams/SubscriptionSequenceDiagram.png)

**Unit Tests**

- Positive Test Cases: N/A
- Negative Test Cases: N/A