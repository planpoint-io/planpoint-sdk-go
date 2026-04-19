# planpoint-sdk-go

Official Go SDK for the [PlanPoint](https://app.planpoint.io) API.

## Installation

```bash
go get github.com/planpoint-io/planpoint-sdk-go
```

> **Note:** Responses use `JSON200` or `JSON201` depending on the endpoint. `login`, `getFloors`, and `getLeads` return `JSON200`. All other endpoints return `JSON201`.

## Quick Start

```go
package main

import (
	"context"
	"fmt"
	"net/http"

	openapi_types "github.com/oapi-codegen/runtime/types"
	planpoint "github.com/planpoint-io/planpoint-sdk-go"
)

func main() {
	ctx := context.Background()

	// 1. Authenticate
	unauthClient, _ := planpoint.NewClientWithResponses("https://app.planpoint.io")
	pass := "yourpassword"
	loginResp, _ := unauthClient.LoginWithResponse(ctx, planpoint.LoginJSONRequestBody{
		Username: openapi_types.Email("you@example.com"),
		Password: &pass,
	})
	token := loginResp.JSON200.AccessToken

	// 2. Create an authenticated client
	client, _ := planpoint.NewClientWithResponses("https://app.planpoint.io",
		planpoint.WithRequestEditorFn(func(ctx context.Context, req *http.Request) error {
			req.Header.Set("Authorization", "Bearer "+token)
			return nil
		}),
	)

	// 3. Fetch your projects
	projectsResp, _ := client.GetMyProjectsWithResponse(ctx)
	fmt.Println(projectsResp.JSON201)
}
```

## API Reference

### Authentication

#### `LoginWithResponse(ctx, body)`

Authenticate and receive a JWT token.

| Field              | Type                  | Required | Description              |
| ------------------ | --------------------- | -------- | ------------------------ |
| `body.Username`    | `openapi_types.Email` | Yes      | User email               |
| `body.Password`    | `*string`             | No       | User password            |
| `body.Impersonate` | `*bool`               | No       | Impersonate another user |

**Returns:** `*LoginResponse` — `{ JSON200: *AuthResponse{ AccessToken, Message } }`

---

### Projects

#### `GetMyProjectsWithResponse(ctx)`

List all projects belonging to the authenticated user. Requires auth.

**Returns:** `*GetMyProjectsResponse` — `{ JSON201: *[]ProjectSummary }`

`ProjectSummary`: `{ Id, Name, Namespace?, HostName?, Status?, Paused?, CreatedAt? }`

---

#### `FindProjectWithResponse(ctx, body)`

Find a public project by namespace and hostname. No auth required.

| Field            | Type     | Required | Description       |
| ---------------- | -------- | -------- | ----------------- |
| `body.Namespace` | `string` | Yes      | Project namespace |
| `body.HostName`  | `string` | Yes      | Allowed hostname  |

**Returns:** `*FindProjectResponse` — `{ JSON201: *Project }`

---

#### `GetProjectWithResponse(ctx, id)`

Get a project by ID. Requires auth.

| Param | Type     | Required | Description      |
| ----- | -------- | -------- | ---------------- |
| `id`  | `string` | Yes      | Project ObjectId |

**Returns:** `*GetProjectResponse` — `{ JSON201: *Project }`

---

#### `UpdateProjectWithResponse(ctx, id, body)`

Update project settings. Requires auth.

| Param | Type     | Required | Description      |
| ----- | -------- | -------- | ---------------- |
| `id`  | `string` | Yes      | Project ObjectId |

**Returns:** `*UpdateProjectResponse` — `{ JSON201: *Project }`

---

### Groups

#### `GetGroupsWithResponse(ctx)`

List all groups for the authenticated user. Requires auth.

**Returns:** `*GetGroupsResponse` — `{ JSON201: *GroupsListResponse{ Records, Count } }`

---

#### `GetGroupWithResponse(ctx, id)`

Get a group by ID. Requires auth.

| Param | Type     | Required | Description    |
| ----- | -------- | -------- | -------------- |
| `id`  | `string` | Yes      | Group ObjectId |

**Returns:** `*GetGroupResponse` — `{ JSON201: *Group }`

---

#### `CreateGroupWithResponse(ctx, body)`

Create a new group. Requires auth.

**Returns:** `*CreateGroupResponse` — `{ JSON201: *Group }`

---

#### `UpdateGroupWithResponse(ctx, id, body)`

Update a group. Requires auth.

**Returns:** `*UpdateGroupResponse` — `{ JSON201: *Group }`

---

### Floors

#### `GetFloorsWithResponse(ctx, params)`

List all floors for a project. Requires auth.

| Param        | Type     | Required | Description      |
| ------------ | -------- | -------- | ---------------- |
| `params.Pid` | `string` | Yes      | Project ObjectId |

**Returns:** `*GetFloorsResponse` — `{ JSON200: *[]FloorFull }`

`FloorFull`: `{ Id, Name?, Project, Level?, Position?, Path?, Image? }`

---

#### `GetFloorWithResponse(ctx, id)`

Get a floor by ID. Requires auth.

**Returns:** `*GetFloorResponse` — `{ JSON201: *FloorFull }`

---

#### `CreateFloorWithResponse(ctx, body)`

Create a new floor. Requires auth.

**Returns:** `*CreateFloorResponse` — `{ JSON201: *FloorFull }`

---

#### `UpdateFloorWithResponse(ctx, id, body)`

Update a floor. Requires auth.

**Returns:** `*UpdateFloorResponse` — `{ JSON201: *FloorFull }`

---

### Units

#### `GetUnitsWithResponse(ctx, params)`

List all units for a project. Requires auth.

| Param        | Type     | Required | Description      |
| ------------ | -------- | -------- | ---------------- |
| `params.Pid` | `string` | Yes      | Project ObjectId |

**Returns:** `*GetUnitsResponse` — `{ JSON201: *UnitsListResponse{ Records, Count } }`

`UnitFull`: `{ Id, Floor, Name?, UnitNumber?, Status?, Price?, Bed?, Bath?, Sqft?, Model? }`

`Status` values: `Available` | `OnHold` | `Sold` | `Leased` | `Unavailable`

---

#### `GetUnitWithResponse(ctx, id)`

Get a unit by ID. Requires auth.

**Returns:** `*GetUnitResponse` — `{ JSON201: *UnitFull }`

---

#### `CreateUnitWithResponse(ctx, body)`

Create a new unit. Requires auth.

**Returns:** `*CreateUnitResponse` — `{ JSON201: *UnitFull }`

---

#### `UpdateUnitWithResponse(ctx, id, body)`

Update a unit. Requires auth.

**Returns:** `*UpdateUnitResponse` — `{ JSON201: *UnitFull }`

---

#### `DeleteUnitWithResponse(ctx, id)`

Delete a unit. Requires auth.

**Returns:** `*DeleteUnitResponse`

---

#### `BatchUpdateUnitsWithResponse(ctx, body)`

Update multiple units at once. Requires auth.

**Returns:** `*BatchUpdateUnitsResponse` — `{ JSON201: *[]UnitFull }`

---

### Leads

#### `GetLeadsWithResponse(ctx, params)`

List all leads for a project. Requires auth.

| Param        | Type     | Required | Description      |
| ------------ | -------- | -------- | ---------------- |
| `params.Pid` | `string` | Yes      | Project ObjectId |

**Returns:** `*GetLeadsResponse` — `{ JSON200: *[]Lead }`

`Lead`: `{ Id?, Name?, Email?, Phone?, Message?, Unit?, CreatedAt? }`

---

## Links

- [PlanPoint App](https://app.planpoint.io)
- [Go SDK on GitHub](https://github.com/planpoint-io/planpoint-sdk-go)
- [npm TypeScript SDK](https://www.npmjs.com/package/@planpoint/sdk)
- [PyPI Python SDK](https://pypi.org/project/planpoint-sdk)
- [PHP SDK](https://github.com/planpoint-io/planpoint-sdk-php)
- [Java SDK](https://github.com/planpoint-io/planpoint-sdk-java)
