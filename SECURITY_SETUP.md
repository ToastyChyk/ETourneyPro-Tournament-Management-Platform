# Local configuration and credential rotation

## Important: rotate exposed credentials

This repository previously contained a Gmail SMTP app password and a fixed organizer password. **Removing them from the current source does not remove them from Git history.** Revoke the Gmail app password in the Google Account that owns it and issue a new one only if email delivery is needed. Change/remove any seeded organizer account created with the former password, particularly in shared or accessible databases.

Do not put replacement credentials in source code, commits, screenshots, or issue comments. If you need complete removal from Git history, coordinate history rewriting with all collaborators and understand that forks/caches may still retain old copies. Rotation is required regardless.

## Development setup

The project expects local SQL Server databases configured through `ConnectionStrings:ETourneyProDB` and `ConnectionStrings:HangfireConnectionDB` in `appsettings.json` (replace with environment-specific values as needed).

Use .NET user secrets from the web project folder for credentials:

```powershell
dotnet user-secrets set "Smtp:FromAddress" "your-sender@example.com"
dotnet user-secrets set "Smtp:AppPassword" "<NEW_APP_PASSWORD>"
dotnet user-secrets set "SeedOrganizer:Username" "your-local-organizer"
dotnet user-secrets set "SeedOrganizer:Password" "<STRONG_UNIQUE_PASSWORD>"
```

The SMTP password must be the **new** password obtained after revoking the exposed one. Never reuse the previous credential. SMTP settings are optional until email is sent; a missing setting causes an explicit exception rather than using a shared fallback secret.

The organizer account is provisioned **only in Development**, and only when both `SeedOrganizer` settings exist. Existing organizer users are not modified automatically: manually rotate or disable them as needed.

In non-development environments, supply secrets through your deployment's environment variables or secret store, for example `Smtp__FromAddress` and `Smtp__AppPassword`. Avoid checking any credential-bearing appsettings files into Git.

## Python API client

```powershell
python -m pip install requests
$env:ETOURNEY_API_URL = "https://localhost:5001/tournaments-api"
# If your local ASP.NET certificate is not trusted, configure an appropriate CA certificate bundle:
$env:REQUESTS_CA_BUNDLE = "C:\\path\\to\\trusted-ca-bundle.pem"
python Group8_Enterprise_FinalProject/PythonClient/tournament_api_client.py
```

The client now uses normal TLS certificate verification. It no longer disables validation or suppresses HTTPS warnings. Locally issued certificates may need to be trusted/exported first. Do not work around this by setting `verify=False`.

## Scope of this PR

This PR removes credentials from active source, changes organizer seeding to explicit development-only configuration, and restores Python HTTPS certificate verification. It does **not** rewrite repository history, automatically revoke credentials, validate email delivery, or test SQL Server startup.
