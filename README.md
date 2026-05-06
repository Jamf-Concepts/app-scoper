# AppScoper

A native macOS SwiftUI application for bulk VPP app scope management in Jamf Pro. Enables Jamf Pro administrators to manage deployment type, category and scope across many apps simultaneously — replacing a one-at-a-time workflow with bulk operations.

## Features

- **Authentication** — OAuth2 client credentials flow against Jamf Pro Classic & Pro API. Bearer tokens auto-refreshed before expiry.
- **VPP App Listing** — Shows all VPP-licensed iOS/iPadOS, macOS and tvOS apps with icons
  - Filter by platform (Mac / iOS/iPadOS / tvOS), category, pricing (Free/Paid)
  - Full-text search across name, bundle ID and category
  - Recently Added detection per platform
- **Bulk Editing** — Select multiple apps and apply changes across all simultaneously
  - Deployment type (Auto Install / Self Service)
  - Jamf Pro category (assign or create)
  - Scope (targets, exclusions) with full group, device and user pickers
- **Scope Management**
  - Individual device targeting (computers and mobile devices) with serial number search
  - Smart and Static group pickers with member counts
  - User targeting with full name and email enrichment
  - User group pickers (local and LDAP)
  - Buildings, departments and network segment exclusions
  - Confirm-by-typing save guard before any bulk write operation
- **Setup Wizard** — Automatically creates the AppScoper API Role and Client in Jamf Pro using admin credentials (one-time setup)
- **Activity Log** — In-app log viewer recording every API call, authentication event and scoping operation with timestamps

## Requirements

- macOS 26 (Tahoe) or later
- Xcode 26+
- A Jamf Pro instance (cloud or on-prem)
- A Jamf Pro API Client with the AppScoper Role (see Configuration tab in-app, or use the Setup Wizard)

### Required API Role Privileges

| Privilege | Purpose |
|---|---|
| Read Mac Applications | View Mac VPP apps |
| Update Mac Applications | Set scope & deployment type for Mac apps |
| Read Mobile Device Applications | View iOS/iPadOS/tvOS VPP apps |
| Update Mobile Device Applications | Set scope & deployment type for mobile apps |
| Read Computers | Scope apps to individual Mac computers |
| Read Mobile Devices | Scope apps to individual mobile devices |
| Read Smart Computer Groups | Smart computer groups in Mac scope |
| Read Static Computer Groups | Static computer groups in Mac scope |
| Read Smart Mobile Device Groups | Smart mobile device groups in scope |
| Read Static Mobile Device Groups | Static mobile device groups in scope |
| Read User | List users for scope targeting |
| Read Smart User Groups | Smart user groups in scope |
| Read Static User Groups | Static user groups in scope |
| Read Categories | Show app categories in the list |
| Create Categories | Create new categories from AppScoper |
| Update Categories | Rename categories from AppScoper |
| Read Buildings | Buildings in scope exclusions |
| Read Departments | Departments in scope exclusions |
| Read Network Segments | Network segments in scope limitations |
| Read Volume Purchasing Locations | Show VPP license counts and token info |
| Read LDAP Servers | Search directory user groups |

## Getting Started

1. Open `AppScoper.xcodeproj` in Xcode
2. Set your **Development Team** in the target's Signing & Capabilities
3. Update `PRODUCT_BUNDLE_IDENTIFIER` if needed (placeholder: `com.yourorg.AppScoper`)
4. Build & Run (⌘R)
5. On first launch, open **Configuration** to run the Setup Wizard, or manually create an API Role and Client in Jamf Pro Settings → System → API Roles and Clients
6. Enter your Jamf Pro URL, Client ID and Client Secret on the login screen

## Project Structure

```
AppScoper/
├── AppScoperApp.swift          # App entry point
├── Models.swift                # All data models and API response types
├── JamfAPIService.swift        # Jamf Pro API client (auth, fetch, update, wizard)
├── AppViewModel.swift          # Main ObservableObject driving all UI state
├── ContentView.swift           # Root view + sidebar navigation
├── LoginView.swift             # Authentication screen
├── AppListView.swift           # VPP app list with filtering and selection
├── AppLogger.swift             # In-memory session log
├── LogViewerSheet.swift        # Log viewer UI
├── UnifiedEditSheet.swift      # Bulk edit sheet (settings, distribution, category, scope)
├── ScopeEditorSheet.swift      # Scope picker UI (groups, devices, users)
├── AppSettingsSheet.swift      # Category assignment view
└── SecurityFAQSheet.swift      # Configuration, Setup Wizard and Security & Privacy
```

## API Endpoints Used

| Endpoint | Purpose |
|---|---|
| `POST /api/oauth/token` | OAuth2 client credentials token |
| `POST /api/v1/auth/token` | Admin token (Setup Wizard only) |
| `GET /api/v1/api-roles` | Check for existing AppScoper Role (Setup Wizard) |
| `POST /api/v1/api-roles` | Create AppScoper Role (Setup Wizard) |
| `GET /api/v1/api-integrations` | Check for existing AppScoper client (Setup Wizard) |
| `POST /api/v1/api-integrations` | Create AppScoper client (Setup Wizard) |
| `GET /api/v1/volume-purchasing-locations` | VPP account listing |
| `GET /JSSResource/mobiledeviceapplications` | List mobile VPP apps |
| `GET /JSSResource/mobiledeviceapplications/id/{id}` | App detail + scope |
| `PUT /JSSResource/mobiledeviceapplications/id/{id}` | Update mobile app scope |
| `GET /JSSResource/macapplications` | List Mac VPP apps |
| `GET /JSSResource/macapplications/id/{id}` | Mac app detail + scope |
| `PUT /JSSResource/macapplications/id/{id}` | Update Mac app scope |
| `GET /JSSResource/computergroups` | List computer groups |
| `GET /JSSResource/mobiledevicegroups` | List mobile device groups |
| `GET /JSSResource/computers` | List individual computers |
| `GET /JSSResource/mobiledevices` | List individual mobile devices |
| `GET /JSSResource/users` | List users (phase 1 — names only) |
| `GET /JSSResource/users/id/{id}` | User detail (phase 2 — full name, email) |
| `GET /JSSResource/usergroups` | List local user groups |
| `GET /JSSResource/ldapservers` | Detect configured LDAP server |
| `GET /JSSResource/categories` | List Jamf Pro categories |
| `GET /JSSResource/buildings` | List buildings |
| `GET /JSSResource/departments` | List departments |
| `GET /JSSResource/networksegments` | List network segments |

## Security

- **Client secret** — Stored in macOS Keychain (`kSecClassGenericPassword`). Never written to disk in plaintext. Automatically migrated from any legacy UserDefaults storage on first launch.
- **OAuth2 tokens** — Held in memory only. Never persisted to disk. Auto-refreshed 60 seconds before expiry.
- **App sandbox** — Enabled with network client entitlement only. No file system access beyond app container.
- **Console logging** — API response bodies are logged to the in-app viewer in all builds. Console output is gated behind `#if DEBUG` to prevent sensitive scope data appearing in production system logs.
- **Admin credentials (Setup Wizard)** — Used once to create the API role and client. Never stored anywhere.
- **Principle of least privilege** — The Setup Wizard creates an API role with exactly the 21 privileges required. No administrator access, no delete permissions.

## Dependencies

AppScoper uses only Apple system frameworks (Foundation, SwiftUI, Combine, Security). No third-party libraries are required.

## Troubleshooting

AppScoper includes a built-in Activity Log accessible from the toolbar (main window) or the Activity Log button in the edit sheet footer. The log records every API call, authentication event and scoping operation with timestamps.

If you encounter unexpected behaviour:

1. Reproduce the issue
2. Open the Activity Log and review entries for errors (red) or warnings (orange)
3. Use **Copy All** in the log viewer to copy the full session log to clipboard
4. Open a GitHub Issue and include the relevant log output

**Common issues:**

- *Devices not appearing in scope pickers* — Check that your API client has **Read Computers** and **Read Mobile Devices** privileges. The log will show HTTP 401 for those fetch calls if privileges are missing.
- *401 errors on app updates* — Verify the API client has **Update Mac Applications** and **Update Mobile Device Applications** privileges.
- *Setup Wizard fails* — Ensure the admin account used has permission to create API Roles and Clients in Jamf Pro (requires Jamf Pro Administrator or equivalent).

## Support

Open an issue at https://github.com/Jamf-Concepts/app-scoper/issues

## License

This project is governed by the [Jamf Concepts Use Agreement](https://resources.jamf.com/documents/jamf-concept-projects-use-agreement.pdf).

Copyright 2026, Jamf Software LLC

For information on how Jamf handles personal data, see the [Jamf Privacy Policy](https://jamf.com/privacy).
