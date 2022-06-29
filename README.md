# pgAdmin with OAUTH2 via Azure AD

This repo is intended to help setup pgadmin4 with OAUTH2 using Azure Active Directory

## Pre-Requisites

- Linux host (this example assume's a RHEL-based distro)
- Access to an Azure Active Directory tenant with sufficient permissions to create App Registrations

## Identity Provider

### Create App Registration

Login to the Azure Portal and navigate to Azure Active Directory.

Click `+Add` --> `App registration`

- Name: `pgadmin`
- Supported account types: `Accounts in this organizational directory only`

### Create Secret

Navigate to the **Cerficates & secrets** blade.

Click `New client secret`

- Description: `pgadmin4 secret`
- Expires: `12 months`

Store the secret in a vault, as it will be required when configuring pgadmin.

### Authentication Redirect URIs

Select the **Authentication** blade.

Under **Platform configurations**, click `Add a platform`.

Select `Web`.

Add the URI of the pgadmin server

*- Note: The URI should match the pattern `https://<host>:<port>/pgadmin4/oauth2/authorize`*

Click `Configure`.

## pgAdmin App

### Install pgAdmin4

#### Ensure the system is up-to-date: 

    sudo dnf update -y

#### Add the pgAdmin4 RPM

- For Fedora systems:

        sudo rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-fedora-repo-2-1.noarch.rpm

- For CentOS, Rocky, RedHat, Alma systems:

        sudo rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-redhat-repo-2-1.noarch.rpm

#### Install pgAdmin4

    sudo dnf install pgadmin4-web

#### Run Setup Script

    sudo /usr/pgadmin4/bin/setup-web.sh

### Configure OAUTH

Create a copy of the default configuration file:

    sudo cp /usr/pgadmin4/web/config.py /usr/pgadmin4/web/config_local.py

    sudo chmod 644 /usr/pgadmin4/web/config_local.py

Edit `config_local.py` and define OAUTH2 as the authentication source:

- Locate the line `AUTHENTICATION_SOURCES` and modify as per below:

        AUTHENTICATION_SOURCES = ['oauth2']

Locate the configuration block labeled `OAUTH2_CONFIG` and make changes per the example below:

At minimum, ensure you replace the placeholder values for `<registered_app_id>`, `<registered_app_secret>`, and `<tenant_id>`.

These values can be obtained from the Azure Active Directory portal where the Registered App was created.

#### Sample OAUTH2 config:

    OAUTH2_CONFIG = [
        {
            # The name of the of the oauth provider, ex: github, google
            'OAUTH2_NAME': 'azure',
            # The display name, ex: Google
            'OAUTH2_DISPLAY_NAME': 'Azure AD',
            # Oauth client id
            'OAUTH2_CLIENT_ID': '<registered_app_id>,
            # Oauth secret
            'OAUTH2_CLIENT_SECRET': '<registered_app_secret>',
            # URL to generate a token,
            # Ex: https://github.com/login/oauth/access_token
            'OAUTH2_TOKEN_URL': 'https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/token',
            # URL is used for authentication,
            # Ex: https://github.com/login/oauth/authorize
            'OAUTH2_AUTHORIZATION_URL': 'https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/authorize',
            # Oauth base url, ex: https://api.github.com/
            'OAUTH2_API_BASE_URL': 'https://graph.microsoft.com/v1.0/',
            # Name of the Endpoint, ex: user
            'OAUTH2_USERINFO_ENDPOINT': 'me',
            # Oauth scope, ex: 'openid email profile'
            # Note that an 'email' claim is required in the resulting profile
            'OAUTH2_SCOPE': 'User.Read email openid profile',
            # Font-awesome icon, ex: fa-github
            'OAUTH2_ICON': None,
            # UI button colour, ex: #0000ff
            'OAUTH2_BUTTON_COLOR': None,
        }
    ]

Execute the setup script again so pgadmin4 can pick up the new `config_local.py`:

    sudo /usr/pgadmin4/bin/setup-web.sh --yes

Login to the pgAdmin4 web UI and confirm that there is now a button in the login dialogue labeled `Login with Azure AD` (depending on how the `OAUTH2_DISPLAY_NAME` was customized).
