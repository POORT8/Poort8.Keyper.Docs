# Changelog

All notable customer-visible changes to the Poort8 NoodleBar, Keyper and the API's are listed in this weekly changelog.

## 2026-09-08

**✨ Highlights:** Keyper approval-link writes now reject empty transaction payloads, and the NoodleBar API reference now points to the configured Keycloak realm and scope instead of test values.

### NoodleBar

#### Changed

- The OpenAPI security definition for NoodleBar now publishes the configured Keycloak client-credentials token endpoint and requires the `noodlebar-api` scope instead of the old test-only values. Regenerate OpenAPI-based clients or update token acquisition examples to request tokens from your dataspace realm. [#1313](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1313)

### Keyper

#### Changed

- `POST /v1/api/approval-links` now rejects requests that do not contain any approval transaction payload for normal orchestration flows. Provide at least one of `addPolicyTransactions`, `policyValidityUpdates`, `policyRevokes`, `addResourceGroupTransactions`, `addEmployeeToOrganizationTransactions`, or `addOROrganizationTransaction`. The DVU building flows (`dvu.voeg-gebouwen-toe@v1`, `dvu.voeg-gebouw-toe@v1`) remain exempt. [#1307](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1307)
- `PUT /v1/api/approval-links/{id}` now rejects updates that would leave a non-DVU approval link without any transactions after the patch is applied. [#1307](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1307)

## 2026-09-01

**✨ Highlights:** Validation-error responses are now documented with a concrete problem-details schema across several NoodleBar and Keyper operations.

### NoodleBar

#### Fixed

- The OpenAPI reference now documents `400 Bad Request` responses as `application/problem+json` with the `FastEndpointsErrorResponse` schema for `GET /v1/api/organization-registry`, `POST|PUT /v1/api/policies`, `POST|PUT /v1/api/resourcegroups`, and `POST /v1/api/resources`. OpenAPI-generated clients can now model validation failures consistently for these operations. [#1279](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1279)

### Keyper

#### Fixed

- `POST /v1/api/approval-links` now documents validation failures as `application/problem+json` with the `FastEndpointsErrorResponse` schema, aligning the published contract with the existing runtime behavior. [#1279](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1279)

## 2026-08-28

**✨ Highlights:** Keyper approval links can now include policy revocations and validity updates alongside new policy grants, enabling richer consent workflows in a single approval step.

### Keyper

#### Added

- `POST /v1/api/approval-links` and `GET /v1/api/approval-links/{id}` now support two new optional arrays in the request and response bodies: `policyRevokes` (revoke an existing Authorization Registry policy by ID and reason) and `policyValidityUpdates` (adjust the `notBefore` and/or `expiration` timestamps of an existing policy). Consumers can include these alongside the existing `addPolicyTransactions` to orchestrate revocations or renewals as part of a single approval flow. Clients using OpenAPI-generated code should regenerate. [#1217](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1217)

## 2026-08-12

**✨ Highlights:** Policies can now have their validity window adjusted after creation, and approval links can be partially updated without recreating them.

### NoodleBar

#### Added

- `PUT /v1/api/policies/{id}/validity` overwrites the `NotBefore` and/or `Expiration` fields of an existing policy. At least one must be provided. A field can only be changed while its current value still lies in the future, and the new value must be now or later. No other policy field is touched. [#1201](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1201)

### Keyper

#### Added

- `PUT /v1/api/approval-links/{id}` partially updates an approval link. Only the fields included in the request body are changed; omitted fields keep their existing value. Updates are only accepted while the approval link is in an updatable state for its configured orchestration flow. [#1201](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1201)

## 2026-07-24

**✨ Highlights:** GIR authorization enforcement now uses the correct `dsgo.gir` use case key, fixing access control for installation data; Keyper gains a new GIR maintenance-data approval flow and TSL deployment support.

### NoodleBar

#### Fixed

- The NoodleBar API reference now documents OAuth2 client credentials as the authentication scheme instead of a generic `Bearer` HTTP scheme. Consumers using OpenAPI-generated clients should regenerate to pick up the updated security definition. [#1131](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1131)

### NoodleBar (GIR)

#### Changed

- **BREAKING:** GIR endpoints (`POST /v1/api/GIRBasisdataMessage`, `GET /v1/api/GIRBasisdataMessage`, `GET /v1/api/GIRBasisdataMessage/{guid}`) now enforce access against Authorization Registry policies with `useCase: "dsgo.gir"` (previously used `"GIR"`). Update any existing AR policies from `useCase: "GIR"` to `useCase: "dsgo.gir"`. [#1125](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1125)

#### Fixed

- `GET /v1/api/GIRBasisdataMessage` and `GET /v1/api/GIRBasisdataMessage/{guid}` no longer list `400 Bad Request` or `415 Unsupported Media Type` in the OpenAPI reference; these codes are not applicable to GET endpoints and were never returned at runtime. [#1158](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1158)

### Keyper

#### Added

- A new GIR digital maintenance book approval workflow `dsgo.gir-onderhoudsboekje@v1` is now available for the GIR dataspace. Pass `dsgo.gir-onderhoudsboekje@v1` as `orchestration.flow` in `POST /v1/api/approval-links` to start a building installation and maintenance data access approval flow. [#1125](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1125)
- Keyper is now available for **NoodleBar Topsector Logistiek** (`TSL`). Authenticate with `authority: https://auth.poort8.nl/realms/tsl` and `audience: keyper-api`. [#1139](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1139)

#### Changed

- `addPolicyTransactions` entries in the approval link request and response now use a dedicated `ApprovalLinkPolicy` schema instead of `PolicyCreateRequestDto`. Required fields (`subjectId`, `action`, `resourceId`) are unchanged. A new optional `policyId` field (nullable string) is included; Keyper populates it with the Authorization Registry policy ID after the approval is processed, making it visible in `GET /v1/api/approval-links/{id}` responses. Consumers using OpenAPI-generated clients should regenerate. [#1138](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1138) [#1140](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1140)

## 2026-07-14

**✨ Highlights:** GIR API consumers can now filter building installation data by component ID when querying the GIRBasisdataMessage endpoint.

### NoodleBar (GIR)

#### Added

- `GET /v1/api/GIRBasisdataMessage` now accepts an optional `componentID` query parameter to filter results to a specific component. [#1123](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1123)

## 2026-07-10

**✨ Highlights:** The GIR registrar Keyper workflow is renamed to `dsgo.gir-supplier-delegation@v1`, a new Datastekker workflow is available for building installation data access, and approval links now auto-refresh on access instead of expiring after 7 days.

### NoodleBar

#### Added

- `GET /v1/api/systems` responses now include a `url` field (nullable string) for each system, containing a link to the system's API documentation or reference page when available. [#1098](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1098)

#### Fixed

- `GET /v1/api/authorization/explained-enforce` now documents `400 Bad Request` in the OpenAPI reference. The endpoint was already returning `400` for invalid or missing parameter combinations; the spec now matches the runtime behavior. [#1057](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1057)
- `PUT /v1/api/resourcegroups/{resourceGroupId}/resources/{resourceId}` and `DELETE /v1/api/resourcegroups/{resourceGroupId}/resources/{resourceId}` were returning `200 OK` at runtime while the spec documented `204 No Content`; both now return `204 No Content` as documented. The `PUT` endpoint also now documents `400 Bad Request` for use-case mismatch between the resource and its group. Clients that checked for `200` on these calls should update to expect `204`. [#1109](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1109)

### Keyper

#### Added

- A new GIR Datastekker approval workflow `dsgo.gir-datastekker@v1` is now available for the GIR dataspace. Pass `dsgo.gir-datastekker@v1` as `orchestration.flow` in `POST /v1/api/approval-links` to start a building installation data access approval flow via the Datastekker platform. [#1102](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1102)

#### Changed

- **BREAKING:** The GIR registrar approval workflow has been renamed from `gir.registrar@v1` to `dsgo.gir-supplier-delegation@v1`. Update the `orchestration.flow` value in `POST /v1/api/approval-links` calls from `gir.registrar@v1` to `dsgo.gir-supplier-delegation@v1`. [#1104](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1104) [#1105](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1105)
- Approval links now expire after 1 hour (previously 7 days). When an approver opens an expired link within 7 days of expiry, Keyper automatically creates a new link with the same configuration and sends a fresh notification. Integrations that display or act on `ExpiresInSeconds` or the calculated `expiresAt` timestamp in API responses will see the new shorter value. [#1097](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1097)

## 2026-07-03

**✨ Highlights:** NoodleBar now provides catalog endpoints to list dataspace systems and reusable tag filters for API-driven discovery.

### NoodleBar

#### Added

- `GET /v1/api/systems` lists APIs and apps across the dataspace, with optional `type` and repeatable `tag` query filters to narrow discovery results. [#1065](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1065)
- `GET /v1/api/systems/tags` returns the flat controlled tag list (`id`, `name`) used to filter `/v1/api/systems` results consistently. [#1065](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1065)

### Keyper

#### Changed

- The GIR registrar approval workflow now shows installer-focused approval text and eHerkenning authentication messaging in verification prompts, reducing ambiguity for approvers. [#1072](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1072) [#1079](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1079)

## 2026-06-26

**✨ Highlights:** GIR API consumers can now request signed delegation evidence directly through a dedicated endpoint for authorization checks.

### NoodleBar (GIR)

#### Added

- `POST /v1/api/delegation` is now available to request signed delegation evidence for GIR authorization checks, returning a `delegation_token` that contains permit/deny evidence for the requested policy context. [#1026](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1026)

## 2026-06-19

**✨ Highlights:** Token endpoint OpenAPI documentation now explicitly defines successful response payload fields for both NoodleBar and GIR consumers.

### NoodleBar

#### Fixed

- `POST /connect/token` now documents the `200 OK` JSON response body with `access_token`, `token_type`, and `expires_in`, so generated clients and API consumers can reliably parse successful token responses. [#1014](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1014)

### NoodleBar (GIR)

#### Fixed

- `POST /connect/token` now documents the `200 OK` JSON response body with `access_token`, `token_type`, and `expires_in`, aligning GIR token endpoint behavior with the published OpenAPI contract for client generation and validation. [#1014](https://github.com/POORT8/Poort8.Dataspace.Private/pull/1014)

## 2026-06-12

**✨ Highlights:** Resource and resource-group mutation endpoints now enforce strict field validation, and all Keyper approval-link endpoints now require authentication

### NoodleBar

#### Fixed

- `GET /v1/api/authorization/explained-enforce` query parameters (`useCase`, `issuer`, `serviceProvider`, `type`, `attribute`, `context`) were incorrectly documented as required; they are now correctly marked as optional. Calls that omit any of these parameters continue to work as before. [#974](https://github.com/POORT8/Poort8.Dataspace.Private/pull/974)
- `POST /v1/api/policies` now correctly documents a `201 Created` response. The API was already returning `201`; the OpenAPI reference was incorrectly showing `200 Success`. Update client code that matches on the documented `200` status. [#993](https://github.com/POORT8/Poort8.Dataspace.Private/pull/993)

#### Changed

- `POST` and `PUT` endpoints for resource groups (`/v1/api/resourcegroups`) and resources (`/v1/api/resources`, `/v1/api/resourcegroups/{resourceGroupId}/resources`) now validate that `resourceGroupId`, `useCase`, `name`, and `description` are non-null and non-empty; requests with missing or blank values now return `400 Bad Request` instead of being accepted silently. Additionally, each resource's `useCase` must match the parent resource group's `useCase`; mismatches also return `400`. Clients that always supply complete, non-empty values for these fields are unaffected. [#980](https://github.com/POORT8/Poort8.Dataspace.Private/pull/980)

### Keyper

#### Changed

- All Keyper approval-link endpoints now require an API token; unauthenticated requests are rejected. The previous notice that authentication "will be mandatory soon" has been removed from the API reference. See the Authentication section of the Keyper API docs for how to obtain a token for your dataspace. [#982](https://github.com/POORT8/Poort8.Dataspace.Private/pull/982)
- `PUT /v1/api/approval-links/{id}` is now documented as a partial-update operation: only the fields included in the request body are updated; omitted or null fields retain their existing value. The endpoint returns `403 Forbidden` when the approval link is not in an updatable state for its current orchestration flow. [#982](https://github.com/POORT8/Poort8.Dataspace.Private/pull/982)

## 2026-06-05

**✨ Highlights:** Keyper request validation is now stricter for approval-link payloads, and the API reference now documents Keycloak as the default token flow.

### Keyper

#### Changed

- `POST /v1/api/approval-links` now enforces non-empty values for `requester.name`, `addEmployeeToOrganizationTransactions[].employee.employeeId`, and `addEmployeeToOrganizationTransactions[].employee.useCase`; requests with null or empty values for these fields now fail validation. [#976](https://github.com/POORT8/Poort8.Dataspace.Private/pull/976)
- The Keyper API authentication documentation now lists per-dataspace Keycloak token endpoints and keeps Auth0 documented as legacy migration support. [#976](https://github.com/POORT8/Poort8.Dataspace.Private/pull/976)

## 2026-06-04

**✨ Highlights:** Core NoodleBar API endpoints are now versioned at `/v1/api/`, and GIR installations no longer return a `deletedAt` field in metadata.

### NoodleBar

#### Added

- Core API endpoints are now available at versioned `/v1/api/` paths: `GET /v1/api/authorization/enforce`, `GET /v1/api/authorization/explained-enforce`, `GET|POST|PUT /v1/api/policies`, `GET|DELETE /v1/api/policies/{id}`, `GET|POST|PUT /v1/api/resourcegroups`, `GET|DELETE /v1/api/resourcegroups/{id}`, `POST /v1/api/resourcegroups/{resourceGroupId}/resources`, `PUT|DELETE /v1/api/resourcegroups/{resourceGroupId}/resources/{resourceId}`, `GET|POST|PUT /v1/api/resources`, and `GET|DELETE /v1/api/resources/{id}`. The announced 90-day deprecation window for unversioned `/api/` routes ends on **2026-09-02**; migrate to `/v1/api/` routes as soon as possible. [#965](https://github.com/POORT8/Poort8.Dataspace.Private/pull/965)

#### Fixed

- `POST /v1/api/resourcegroups` now correctly assigns each resource's own `useCase` when creating a resource group. Previously, the resource group's `useCase` was propagated to child resources, causing policies to target the wrong use case. [#939](https://github.com/POORT8/Poort8.Dataspace.Private/pull/939)

### NoodleBar (GIR)

#### Changed

- **BREAKING:** The `deletedAt` field has been removed from all GIR registration metadata responses. GIR now permanently removes records (hard delete) rather than marking them as deleted. Clients that rely on `deletedAt` to detect removed registrations should instead treat absent records as deleted.

### Keyper

#### Added

- The Keyper API now accepts Keycloak JWT bearer tokens alongside Auth0 tokens, allowing Keycloak-managed clients to authenticate directly. Contact Poort8 to configure your Keycloak client for Keyper access. [#966](https://github.com/POORT8/Poort8.Dataspace.Private/pull/966)

#### Changed

- The `useCase` field in resource group requests within `POST /v1/api/approval-links` is now optional. Existing payloads that include `useCase` continue to work without changes. [#970](https://github.com/POORT8/Poort8.Dataspace.Private/pull/970)

## 2026-04-28

**✨ Highlights:** Organization validation is now available through dedicated API endpoints, and organization search responses are capped for safer, more predictable integrations.

### NoodleBar

#### Added

- `GET /api/organization-registry/{id}/validate` checks whether an organization is currently active in the registry. The response contains `{ "isValid": boolean, "reason": string }`, so clients can show a clear validation result without fetching the full organization record. [#870](https://github.com/POORT8/Poort8.Dataspace.Private/pull/870)
- `GET /api/organization-registry/{id}/validate-approver` checks whether a person is authorized to act as approver for an organization. Pass the approver email address as `?email=` and use the `{ "isValid": boolean, "reason": string }` response to guide approval flows. [#870](https://github.com/POORT8/Poort8.Dataspace.Private/pull/870)
- `POST /api/organization-registry/names` resolves multiple organization identifiers to display names in a single request. Send `{ "identifiers": ["..."] }`; the response maps each identifier to its display name. [#870](https://github.com/POORT8/Poort8.Dataspace.Private/pull/870)

#### Changed

- Organization search results are now capped at 50 results per request. Clients that previously relied on larger result sets should narrow the search query or add client-side follow-up selection. [#851](https://github.com/POORT8/Poort8.Dataspace.Private/pull/851)

### Keyper

#### Changed

- Approval pages now show the requester's display name and optional message to the approver, giving approvers more context before accepting or rejecting access. [#840](https://github.com/POORT8/Poort8.Dataspace.Private/pull/840)
- GDS and sensor optimization approval flows now show a portal link on the confirmation screen after approval or rejection, making the next step clearer for end users. [#853](https://github.com/POORT8/Poort8.Dataspace.Private/pull/853)

## 2026-04-17

**✨ Highlights:** GIR moved to a versioned v1 API contract, and NoodleBar API clients can now use Keycloak bearer tokens.

This release contains several breaking changes for GIR API consumers. Update endpoint URLs, payload fields, authentication, and organization identifiers before upgrading. Docs: [GIR API versioning](../gir/api-versioning.md), [DSGO token endpoint](../gir/connect-token.md).

### NoodleBar (GIR)

#### Changed

- **BREAKING:** GIRBasisdataMessage endpoints now use the `/v1/` path prefix. Migration: update calls from `/api/GIRBasisdataMessage` to `/v1/api/GIRBasisdataMessage`. [#850](https://github.com/POORT8/Poort8.Dataspace.Private/pull/850)
- **BREAKING:** The `installation` field in GIRBasisdataMessage requests and responses has been renamed to `installationBaseData`. Additional v1 schema updates include changed operational and lifecycle status enums, ETIM feature field naming updates, and stricter validation rules. Migration: update payloads and generated clients to match the v1 schema. [#842](https://github.com/POORT8/Poort8.Dataspace.Private/pull/842)
- **BREAKING:** `GET` and `POST /v1/api/GIRBasisdataMessage` now only accept DSGO bearer tokens obtained via `POST /connect/token`. Auth0 and other bearer tokens are rejected with a 401. Migration: switch GIR clients to DSGO token acquisition before calling the v1 endpoints. [#848](https://github.com/POORT8/Poort8.Dataspace.Private/pull/848)
- **BREAKING:** GIR organization identifiers now use the DID format `did:ishare:EU.NL.NTRNL-<kvkNumber>` instead of `NL.KVK.<kvkNumber>`. Migration: update hardcoded identifiers, configuration, and test data to the DID format. [#843](https://github.com/POORT8/Poort8.Dataspace.Private/pull/843)

#### Security

- `POST /connect/token` now validates that the requesting party is a member of the `EU.DS.NL.DSGO` dataspace. Requests from parties without this membership are rejected, preventing tokens from being issued to unauthorized GIR participants. [#841](https://github.com/POORT8/Poort8.Dataspace.Private/pull/841)

### NoodleBar

#### Added

- The NoodleBar API now accepts Keycloak JWT bearer tokens alongside existing Auth0 tokens. Mutation endpoints for resources, resource groups, policies, employees, and organizations enforce that the token organization matches the resource owner; cross-organization mutations require a delegated scope. Docs: [requesting API access](requesting-api-access.md), [validating API tokens](validating-api-tokens.md). [#827](https://github.com/POORT8/Poort8.Dataspace.Private/pull/827) [#828](https://github.com/POORT8/Poort8.Dataspace.Private/pull/828)

#### Fixed

- `GET /v1/api/policies` now correctly returns policies owned by the requesting organization when called with a standard, non-delegated scope. Previously, ownership filtering could produce an empty or incorrect result set. [#833](https://github.com/POORT8/Poort8.Dataspace.Private/pull/833)

## 2026-04-03

**✨ Highlights:** GIR gained DSGO token-based authentication, and Keyper gained a sensor optimization approval workflow.

### NoodleBar (GIR)

#### Added

- `POST /connect/token` is now available on the GIR instance for DSGO (DigiGO) authentication using JWT client assertions. Submit `grant_type`, `scope`, `client_id`, `client_assertion_type`, and `client_assertion` as `application/x-www-form-urlencoded`; the assertion is validated against the DSGO satellite trusted list using certificate chain verification. Docs: [DSGO token endpoint](../gir/connect-token.md). [#618](https://github.com/POORT8/Poort8.Dataspace.Private/pull/618) [#826](https://github.com/POORT8/Poort8.Dataspace.Private/pull/826)

### Keyper

#### Added

- A Dutch-language sensor optimization workflow `keyper.sensor-optimization@v1` is now available. Pass `keyper.sensor-optimization@v1` as `orchestration.flow` in `POST /v1/api/approval-links`; unlike the default workflow, this flow does not require the approver to be a member of the requesting organization. Docs: [Keyper](../keyper/README.md). [#820](https://github.com/POORT8/Poort8.Dataspace.Private/pull/820)

## 2026-03-27

**✨ Highlights:** Keyper gained a generic default approval workflow, and onboarding PDF validation now handles valid uploads reliably.

### NoodleBar

#### Fixed

- `POST /api/onboarding` now correctly validates PDF files submitted as `BusinessRegisterExtract`. Previously, valid PDF files could be rejected because the uploaded stream was not rewound after header validation. [#801](https://github.com/POORT8/Poort8.Dataspace.Private/pull/801)

### Keyper

#### Added

- A generic English-language workflow `keyper.default@v1` is now available. Pass `keyper.default@v1` as `orchestration.flow` in `POST /v1/api/approval-links` to use a standard dataspace approval flow without dataspace-specific copy. Docs: [Keyper](../keyper/README.md). [#817](https://github.com/POORT8/Poort8.Dataspace.Private/pull/817)

## 2026-03-20

**✨ Highlights:** Onboarding now supports Dutch, Belgian, and German organizations, with live registry checks for each country.

This release contains breaking changes to `POST /api/onboarding`. Existing integrations must update request encoding, field names, and phone number formatting before upgrading.

### NoodleBar

#### Changed

- **BREAKING:** `POST /api/onboarding` now requires `multipart/form-data` instead of JSON, adds the required `CountryCode` field (`NL`, `BE`, or `DE`), and renames `KvkNumber` to `BusinessRegisterNumber`. Migration: change the `Content-Type`, send `CountryCode: NL` for existing Dutch registrations, and rename the field in your payload. [#762](https://github.com/POORT8/Poort8.Dataspace.Private/pull/762)
- **BREAKING:** `POST /api/onboarding` now requires phone numbers in international E.164 format, for example `+31612345678`. Migration: replace local phone numbers such as `0612345678` with the international form including the country dialing prefix. [#786](https://github.com/POORT8/Poort8.Dataspace.Private/pull/786)
- `POST /api/onboarding` now returns 400 when the submitted `CountryCode` is not accepted by the dataspace instance, giving callers a clear validation failure instead of an ambiguous registration outcome. [#770](https://github.com/POORT8/Poort8.Dataspace.Private/pull/770)
- Belgian organizations onboarded through `POST /api/onboarding` now receive a `VatCheck` verification record during provisioning. The VAT number is derived from the KBO number and verified against the EU VIES service. [#785](https://github.com/POORT8/Poort8.Dataspace.Private/pull/785)

#### Added

- `POST /api/onboarding` now accepts Belgian organization registrations. Set `CountryCode` to `BE` and provide a KBO number as `BusinessRegisterNumber`; the KBO number is verified against the official Belgian business registry and recorded as a `KboCheck`. [#762](https://github.com/POORT8/Poort8.Dataspace.Private/pull/762) [#768](https://github.com/POORT8/Poort8.Dataspace.Private/pull/768)
- `POST /api/onboarding` now accepts German organization registrations. Set `CountryCode` to `DE`, provide a commercial register number as `BusinessRegisterNumber` (HRB format, e.g. `HRB12345`), and optionally supply `RegistrationCourt`; the organization's LEI is looked up automatically through the GLEIF registry. [#789](https://github.com/POORT8/Poort8.Dataspace.Private/pull/789)
- `POST /api/onboarding` now accepts an optional `BusinessRegisterExtract` field containing a PDF upload of the business register extract. [#762](https://github.com/POORT8/Poort8.Dataspace.Private/pull/762)
- `POST /api/onboarding` now accepts an optional `Vat` field for the organization's VAT number. [#785](https://github.com/POORT8/Poort8.Dataspace.Private/pull/785) [#789](https://github.com/POORT8/Poort8.Dataspace.Private/pull/789)

### Keyper

#### Changed

- Users who authenticate successfully but are not authorized to approve for their organization now see a dedicated error screen instead of being redirected back to authentication. This makes invalid-approver cases easier to diagnose for end users and support teams. [#715](https://github.com/POORT8/Poort8.Dataspace.Private/pull/715) [#763](https://github.com/POORT8/Poort8.Dataspace.Private/pull/763)
