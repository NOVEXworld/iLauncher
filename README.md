# iLauncher – Microsoft/Minecraft Authentication and Security Overview

*iLauncher* is an independent third-party launcher for **Minecraft: Java Edition** (with planned future support for Bedrock). It is _not affiliated_ with Microsoft, Mojang, Xbox, or any official Minecraft product. Minecraft, Mojang, and Xbox Game Studios are trademarks of their respective owners. iLauncher’s source code is currently not publicly available (closed-source).

This document outlines iLauncher’s account authentication, data handling, and security design. It is intended as supporting material for Microsoft/Minecraft Java Edition Game Service API AppID reviews, to clarify iLauncher’s functionality and compliance.

## Current Features and Scope

**Implemented features (Java Edition only):**
- Version management (official releases, snapshots, old alpha/beta, April Fools)
- Java runtime detection/installation
- Minecraft file integrity verification and repair
- Download progress logs and optional mirror support
- Microsoft account sign-in (OAuth 2.0 device code flow)
- LittleSkin (Yggdrasil) account sign-in (third-party Minecraft auth)
- Offline (non-authenticated) accounts for local testing
- Light/dark themes; Chinese and English UI; skin/avatar display for authenticated accounts

**Planned features (future releases, *not yet implemented*):** Java+Bedrock unified account management, Bedrock Edition support, enhanced mod/modpack management, import/export of settings, etc. These are under development and are not active in the current release.

**Scope boundaries:** iLauncher does *not* include any server-side components or user data collection. It does *not* provide unofficial “free” login bypass or other hacks. Offline accounts are purely local identities; they do **not** perform any online authentication or license checks, and they cannot access online protected content (see below).

## Microsoft Account Sign-In (OAuth Device Code Flow)

iLauncher uses the **Microsoft OAuth 2.0 Device Authorization Grant** (device code) for Microsoft account login. This is a public client (desktop) flow:

1. iLauncher (using its registered Microsoft *Client ID*) requests a device/user code from Microsoft (`/oauth2/v2.0/devicecode` endpoint).
2. iLauncher shows the user a verification URL and code. The user opens that URL in a browser on any device and signs in to their Microsoft account.
3. The user consents, and Microsoft displays a code on the browser screen.
4. iLauncher polls the token endpoint (`/oauth2/v2.0/token`) with the device code until Microsoft returns the access token and refresh token.
5. iLauncher proceeds with Xbox Live/XSTS authentication (below) and then stores the tokens for launching the game.

Because iLauncher runs on a desktop, it is classified as a **public client application**. Public clients **cannot securely hold a client secret**, so iLauncher’s Azure App Registration does not use a secret. Instead, *“Allow public client flows”* is enabled in the app registration to permit this flow. Microsoft documentation notes that the device code flow is intended for “input-constrained devices” (TVs, IoT, etc.), and suggests using more secure flows where possible. We have chosen device code flow for simplicity (no embedded browser), but we ensure it is configured correctly and safely. (In the future, if iLauncher uses an interactive auth flow, it will implement the recommended Authorization Code flow with **PKCE** for public clients.)

**App Registration details:** The iLauncher app registration uses the recommended redirect URI for native apps (`https://login.microsoftonline.com/common/oauth2/nativeclient`). It requests only the scopes needed: e.g. `XboxLive.signin`, `offline_access`, `openid`, `profile`, etc. No unnecessary permissions (such as email, calendar, or directory scopes) are requested, following least-privilege principles.

### Tokens and Data from Microsoft Sign-In

After successful login, the following data is obtained from Microsoft (via the OAuth and Xbox flows) and stored *locally*:

- **Microsoft refresh token:** to obtain new tokens without re-prompting the user (expires as specified by Microsoft; used to call `/token` again).
- **Minecraft Profile ID (UUID)** and **player name**: obtained via `api.minecraftservices.com` profile endpoints after XSTS auth.
- **Xbox User ID (XUID)** and **player hash**: obtained from Xbox Live (these link the user to Xbox services).
- **Access tokens / ID tokens** (Microsoft Graph tokens, Xbox Live tokens, etc.): held in memory or local cache for as long as needed.
- **Skin/texture profiles**: URLs or data fetched from Minecraft profiles.

These values are stored **only on the user’s device**. iLauncher does *not* upload them anywhere, does not send them to the developers, and does not log them in crash reports. In practice, we use Microsoft’s recommended **cross-platform token cache** for desktop apps (via MSAL extensions). On Windows this cache is protected by DPAPI; on macOS by Keychain; on Linux by the system keyring. The user can remove all stored account data via the launcher UI (or by deleting the launcher’s data folder), which erases these tokens.

All network communication is via HTTPS to official Microsoft/Xbox/Minecraft endpoints. iLauncher never proxies or inspects user credentials. At no point does iLauncher see or store the user’s Microsoft password. The Microsoft login happens on Microsoft’s site; only tokens are returned to the app. Similarly, Xbox Live authentication is done with standard APIs.

After obtaining Microsoft OAuth tokens, iLauncher follows the **Xbox Live/XSTS flow**:
1. Call `https://user.auth.xboxlive.com/user/authenticate` with the Microsoft access token to get an Xbox Live user token.
2. Call `https://xsts.auth.xboxlive.com/xsts/authorize` with the Xbox Live token (and app’s Xbox Services configuration) to get an XSTS token.
3. Call `https://api.minecraftservices.com/authentication/login_with_xbox` with the XSTS token to get the final Minecraft access token.

Every request to Xbox/Minecraft services includes the required XSTS token and signature header as per Microsoft’s specifications. If the XSTS token is invalid or expired, the calls fail with HTTP 401. We handle refreshing tokens appropriately.

## LittleSkin (Yggdrasil) Authentication

iLauncher also supports **LittleSkin** account login (an independent Yggdrasil auth service). In this mode, the user enters their LittleSkin username/password into iLauncher, which then sends these credentials *only to LittleSkin’s official API endpoint* for authentication. iLauncher does not store the password after use, does not forward it to anyone else, and does not mix LittleSkin data with Microsoft/Xbox data.

LittleSkin is not affiliated with Microsoft or Mojang. Using LittleSkin simply provides an alternate login method for Minecraft accounts. iLauncher clearly separates the flows: if the user logs in with LittleSkin, no Microsoft/Xbox tokens are used and vice versa.

## Offline Accounts

iLauncher allows creating **Offline Accounts** for single-player or offline servers. Offline accounts consist of a player name (and optional UUID) set by the user. **No network calls are made** for offline accounts. They do not contact Microsoft/Xbox and cannot access any online-only features. Importantly, offline accounts cannot play on official servers that require authentication; they are purely for local testing or private servers explicitly allowing offline mode.

Offline accounts are stored locally like normal profiles. iLauncher ensures these are kept distinct from authenticated accounts. (The Minecraft launcher will treat these as unverified accounts with only local validity.)

## Data Privacy and Storage

iLauncher is designed to **keep user data on the local device**. We do *not* collect or transmit personal data beyond what is necessary for authentication. In particular:

- **No password collection:** iLauncher never receives or stores Microsoft passwords. Microsoft credentials are entered only on official Microsoft web pages. LittleSkin passwords are sent directly to LittleSkin’s servers. Offline account info (just a name) is local only.
- **No selling or sharing:** iLauncher does not sell or share account data. No tokens or credentials are sent to the developer or third parties. We do not collect telemetry or usage data about the user’s accounts.
- **Local account data:** The stored values (profile ID, tokens, etc.) reside on disk in the user’s launcher directory, typically encrypted by the OS. They are not included in any public repository or transmitted by iLauncher.
- **No cloud backup:** We do not upload game saves or user data to any cloud service. By default, iLauncher interacts only with Mojang/Microsoft endpoints for version files and authentication.
- **User control:** The user can remove stored accounts or data via the launcher, which deletes these tokens immediately. Uninstalling iLauncher (or deleting its data folder) similarly removes all saved credentials.

## Permissions and Least-Privilege

iLauncher requests only the minimum permissions needed for its operation. In the Microsoft App Registration, the **scopes** granted are limited to those required for login (user profile and Xbox sign-in). We do not request any broad Graph API scopes, write permissions, or organizational data. On the Xbox/Minecraft side, we only request the XSTS login (user context) – we do not access friends lists, achievements, or other personal data. All OAuth requests clearly indicate the purpose to the user via consent screens.

Microsoft and Xbox/XSTS services require using an **allowlisted** application client ID. New app registrations are *not* allowed by default. As confirmed by Microsoft, the Minecraft Services API enforces an allowlist of client IDs. This means that even with correct OAuth configuration, the final `login_with_xbox` call will fail with 403 unless the AppID is approved.

The only approved path is via the **ID@Xbox (Xbox Developer)** program. We will submit iLauncher’s Client ID through the official Xbox developer channels for review. The support response indicates: *“Third-party Minecraft Java launcher developers are directed to register through the Xbox Developer / ID@Xbox program to have their application's client ID allowlisted for Minecraft API access.”*. We will follow that process and attach this documentation.

## Security Measures

- **Token storage:** We use secure local storage (MSAL’s cross-platform token cache) to protect tokens at rest. This ensures that, on Windows, tokens are encrypted via DPAPI under the user’s profile. On macOS, the Keychain is used; on Linux, a secure keyring is used.
- **Secret management:** As a public client, iLauncher has **no client secret**. Thus there is no risk of leaking a secret key (and Microsoft’s guidance is that public clients cannot hold secrets anyway).
- **Data in transit:** All communication uses HTTPS. We do not implement any custom SSL bypass or certificate pinning, but use Microsoft/Xbox endpoints’ standard certificates.
- **No additional telemetry:** We do not include analytics or tracking SDKs. Crash or bug reports do not contain user tokens or PII.

## Microsoft/Minecraft API Allowlist Review

**Important:** A regular Microsoft app registration is insufficient for Minecraft API access. The rejection notice received was generic and did not specify iLauncher’s fault. We will request clarification and seek a new review through the ID@Xbox program, as advised by Microsoft. We note:
- No self-service configuration (redirect URI, permissions, etc.) can override the Minecraft allowlist, per Microsoft’s answer.
- Launchers like MultiMC and Prism obtained API access by applying through ID@Xbox. We will follow the same path.
- We will present this document (with all declarations and references) to the ID@Xbox team.

## Review Checklist (Pre-Submission)

Before reapplying, we will ensure:
- **Complete documentation:** This file, README, privacy policy, and screenshots are included in the repo for reviewer reference.
- **Transparent policies:** All statements in this document reflect the actual implementation and have been verified.
- **Support contact:** We provide a valid contact (e.g. GitHub issue link or developer email) for questions.
- **Legal:** Trademark and affiliation disclaimers are clear (see top). Terms of use / privacy statements are drafted for users.

If any changes are made to the launcher (e.g. adding PKCE flow), we will update this document accordingly.

**Sources:** Official Microsoft documentation and communications have been cited throughout. Key references include:
- Microsoft identity platform docs (OAuth flows, client app types)
- MSAL.NET token cache guidelines (desktop apps)
- Microsoft Q&A (Minecraft/Xbox allowlist guidance).

This file is intended to be public and may be updated with new guidance from Microsoft or code audits. Please review carefully before submission.
