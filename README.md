# Azure SCIM Server for OutSystems ODC

This component connects Azure AD (Microsoft Entra) to OutSystems ODC using SCIM 2.0. It runs in the background and handles automatic user provisioning — including create, update, and deactivate — using the `/Users` and `/ServiceProviderConfig` endpoints.

No UI needed. Just plug it in, configure a few settings, and you're good to go.

---

## 🚀 Features

- Supports SCIM 2.0 operations: `POST /Users`, `PATCH /Users/{id}`, and `GET /ServiceProviderConfig`
- Handles bearer token validation for secure communication from Azure
- Automatically fetches and caches ODC API tokens with auto-refresh
- Stores synced users in `UserExtension` for easy lookup and reduced API calls
- Works with Azure's SCIM protocol, including PATCH operations and `active` flag handling

---

## ⚙️ Setup Instructions

### 1. Install the Component

- Publish the module to your ODC environment.
- Ensure it's reachable via public URL (e.g., `https://<yourenv>.outsystems.app`).

---

### 2. Configure Site Properties

#### Get API Client Credentials

1. Go to your **Users API** app in ODC.
2. Create a new API Client under **Service Center > API Clients**.
3. Copy the `ClientId` and `ClientSecret`.

#### Set the following Site Properties in Service Center:

| Property       | Description |
|----------------|-------------|
| `ClientId`     | ODC API Client ID |
| `ClientSecret` | ODC API Client Secret |
| `SCIMToken`    | Static bearer token Azure will use. Generate one at [generate-random.org](https://generate-random.org/api-token-generator) |

Azure will send the SCIMToken in the Authorization header for every request.

Official ODC API Docs: [OutSystems ODC REST APIs](https://success.outsystems.com/documentation/outsystems_developer_cloud/odc_rest_apis/)

---

### 3. Set Up Azure Enterprise App

1. In Azure AD, go to **Enterprise Applications** > **+ New Application**
2. Choose **Non-gallery application**, name it (e.g., `ODC SCIM Provisioning`)
3. Go to **Provisioning** tab
4. Set Provisioning Mode to `Automatic`
5. Tenant URL: `https://<yourenv>.outsystems.app/api/scim/v2`
6. Secret Token: Use your `SCIMToken` value

---

### 4. Test Connection

Click **Test Connection** in Azure — this should call `/ServiceProviderConfig`. If successful, Azure will allow you to configure mappings.

---

### 5. Attribute Mappings

Default attributes Azure sends:

- `userName`
- `name.givenName`
- `name.familyName`
- `emails[type eq "work"].value`
- `active`

These map to the `UserExtension` entity. No custom mapping needed unless your use case demands it.

---

### 6. Start Provisioning

1. Assign a user to the Enterprise App.
2. Use **Provision on demand** under the Provisioning tab.
3. Search the user and click **Provision** to trigger a sync.

Azure will send a `POST` or `PATCH` request to your `/Users` endpoint. If it works, you can enable full provisioning.

---

## 🧪 Testing

I’ve shared a Postman collection to help test this outside of Azure:

👉 [https://github.com/michaeldeguzman/ODC](https://github.com/michaeldeguzman/ODC)

SCIM-created users won’t appear in the ODC Portal’s Users screen, so it's important to test using Postman or similar tools. You can also check the `UserExtension` entity or call the ODC Identity API to confirm.

---

## ⚠️ Limitations

- Only `/Users` endpoint is supported for now — no `/Groups`
- Group provisioning is on the roadmap but not implemented (my Azure free tier doesn’t support it)
- Bulk operations, `/Schemas`, `/Me` are out of scope for now

---

## 💡 Need More?

Open to feedback and PRs — or reach out if you'd like to help extend this for group sync, enterprise extensions, or multi-schema support.

