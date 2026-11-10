# iLauncher

iLauncher is an independent third-party launcher for Minecraft Java Edition.

It provides a simple desktop interface for installing, managing, and launching
Minecraft Java Edition. The project is designed with future Java Edition and
Bedrock Edition support in mind.

iLauncher is not affiliated with Microsoft, Mojang, Xbox, or any official
Minecraft product.

The project source code is not publicly available. To learn more about the
product interface and current features, see the `1.png 2.png 3.png` in the
repository root.

## Current Features

- Minecraft Java Edition version management
- Release, snapshot, old alpha, old beta, and April Fools version categories
- Microsoft account sign-in
- LittleSkin account sign-in
- Offline account support
- Java runtime detection
- Automatic Java runtime installation when required
- Minecraft file verification and repair
- Download mirror support
- Download progress and logs
- Light and dark themes
- Chinese and English interface
- Skin and avatar display for authenticated accounts

## Microsoft Account Sign-in

iLauncher uses the Microsoft device authorization flow.

Users sign in with their own Microsoft account. iLauncher does not request or
store the user's Microsoft password.

The account must own the required Minecraft product before the game can be
started with an authenticated profile.

The following information may be stored locally for the selected account:

- Minecraft profile ID
- Minecraft player name
- Minecraft access token
- Microsoft refresh token
- Xbox user identifier
- Skin profile information

These values are stored locally and are not uploaded to this repository or
shared with third parties by iLauncher.

## LittleSkin Support

iLauncher supports LittleSkin accounts through the LittleSkin Yggdrasil
authentication service.

LittleSkin users must provide their own account credentials through the
launcher login window. Credentials are used only to complete authentication and
are not included in the project source code.

## Offline Accounts

Offline accounts are provided for local single-player testing and servers that
allow offline identities.

Offline accounts do not provide Microsoft authentication, ownership validation,
or access to authenticated multiplayer services.

Offline accounts do not bypass Minecraft ownership checks and cannot be used as
a replacement for Microsoft authentication.

## Privacy

iLauncher is designed to keep account information on the user's local device.

iLauncher does not:

- collect Microsoft passwords;
- collect LittleSkin passwords for analytics;
- sell account information;
- send account credentials to the project author;
- include access tokens in public repositories;
- upload local game saves by default.

Network requests may be made to the following services when the user requests
the related feature:

- Microsoft identity services;
- Xbox Live authentication services;
- Minecraft Services;
- LittleSkin authentication and profile services;
- Minecraft metadata and download mirror services.

The user may remove local accounts and launcher data from the launcher data
directory at any time.

## Security

Do not publish the following information:

- Microsoft client secrets;
- Microsoft refresh tokens;
- access tokens;
- passwords;
- private configuration files;
- personal account exports.

The project source code and local development files are kept privately and are
not publicly released.

## Roadmap

- Minecraft Bedrock Edition support
- Unified Java and Bedrock account management
- Improved skin caching
- More launcher customization
- Better game instance management
- Mod and modpack management
- Import and export of launcher settings

Bedrock Edition support is planned and is not advertised as an available
feature in the current release.

## Product Preview

Product screenshots are available in the repository root:

- `1.png`
- `2.png`
- `3.png`

The screenshots demonstrate the current product interface and feature design.
They do not contain the project source code.

## Project Status

iLauncher is currently under active development.

The current release focuses on Minecraft Java Edition account management,
version management, and game launching. Additional platform support will be
developed in future releases.

## Contact

For questions, bug reports, or account authorization issues, use the contact
information provided in the Microsoft application registration.

## License

This repository contains project documentation and public project information.

The launcher and its dependencies may be distributed under their respective
licenses. See the project license and third-party notices for details.
