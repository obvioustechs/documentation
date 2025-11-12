# Remote Desktop Gateway with Microsoft Entra Multi-Factor Authentication (RDG with Entra MFA)

## Domain Controller

### Network Policy Server

- Server Manager > Tools > Network Policy Server
- RADIUS Clients and Servers
  - Right-click _RADIUS Clients_ and click _New_
    - Enter `RDS-SERVER` as the name
    - Enter the RDS server's IP address
    - Enter the shared secret
- Policies
    - Connection Request Policies > Use Windows authentication for all users
        - Set _Type of network access server_ to _Remote Desktop Gateway_
    - Right-click _Network Policies_
    - Click _New_
        - Name the policy `RDG_CAP`
        - Set _Type of network access server_ to _Remote Desktop Gateway_
        - Add a day and time restriction and select _Permitted_
        - Use the default access permission
        - Uncheck MS-CHAP-v2 and MS-CHAP
        - Check _Allow clients to connect without negotiating an authentication method_
        - Use the defaults for everything else and click _Finish_

### Network Policy Server Extension for Entra Multi-Factor Authentication

- Run `Invoke-WebRequest -Uri "https://download.microsoft.com/download/b/f/f/bffb4f12-9c09-4dbc-a4af-08e51875eea9/NpsExtnForAzureMfaInstaller.exe" -OutFile "NpsExtnForAzureMfaInstaller.exe"`
- Run `.\NpsExtnForAzureMfaInstaller.exe`
- Go through the installation steps
- Run `& "C:\Program Files\Microsoft\AzureMfa\Config\AzureMfaNpsExtnConfigSetup.ps1"`
- Use your Microsoft Admin credentials
- Get the tenant ID from https://entra.microsoft.com
    - Go to _Entra ID_ > _Overview_ in the sidebar

## Remote Desktop Services Gateway Server

### Roles and Features Installation

- Server Manager > Dashboard > Add roles and features
    - Installation Type
        - Role-based or feature-based installation
    - Server Roles
        - Remote Desktop Services
    - Role Services
        - Select all except _Remote Desktop Virtualization Host_

### Domain Configuration

- Settings > Network & internet > Ethernet > DNS server assignment > Edit
    - Switch to manual
    - Enable IPv4
    - Set Preferred DNS to the domain controller's IP address
    - Set Alternate DNS to `1.1.1.1`
- Settings > System > About > Related links > Domain or workgroup
    - Select _Change_
    - Set the computer name
    - Select Domain and enter the domain name
    - Enter a domain admin's credentials
- Close out and click _Restart now_

### Remote Desktop Services Installation

- Sign in with a domain admin account
- Server Manager > Dashboard > Add roles and features
    - Installation Type
        - Remote Desktop Services installation
    - Accept the defaults until _RD Web Access_
    - RD Web Access
        - Select this server from the server pool
    - Confirmation
        - Select _Restart the destination server automatically if required_
    - Click Deploy

### Remote Desktop Services Configuration

- Server Manager > Remote Desktop Services > Overview > Deployment Overview
    - RD Gateway
        - Select this server
        - Enter this server's FQDN
        - Click _Add_
    - RD Licensing
        - Select this server
        - Click _Add_
    - Tasks > Edit Deployment Properties
        - (Only for testing?): Uncheck _Bypass RD Gateway server for local addresses_
        - Click _Apply_
        - Certificates
            - Select _RD Gateway_
            - Import the certificate for each role service
            - Or for a self-signed certificate
                - Click _Create new certificate_
                - Use this server's FQDN for the name
                - Store the certificate
                - Click allow the certificate...
                - Click OK and Apply
                - Select existing certificate... and browse for the one that was just stored for each role service

### Remote Desktop Gateway Manager Configuration

- Server Manager > Tools > Remote Desktop Services > Remote Desktop Gateway Manager
- Computer name > Policies
    - Right-click _Connection Authorization Policies_
    - Create New Policy > Wizard
        - Select _Create a RD CAP and a RD RAP_
        - Name it `RD_CAP`
        - Add User group membership `Domain Admins`
        - Defaults for the next 2
        - Name it `RD_RAP`
        - Select _Allow users to connect to any network resource_
        - Select defaults for the rest
- Right-click the computer name
- Properties
    - Server Farm
        - Type the computer name and click add (an error pops up, that's fine)
    - RD CAP Store
        - Select _Central server_
        - Add the DC's IP

### Network Policy Server Configuration

- Server Manager > Tools > Network Policy Server
    - Radius Clients and Servers > Remote RADIUS Server > TS GATEWAY SERVER GROUP
        - Address
            - Add the domain controller's server name
        - Authentication/Accounting
            - Enter the shared secret
        - Load Balancing
            - Replace the 3 and 30 values both with 60
        - Click _Apply_
    - Policies > Connection Request Policies > TS GATEWAY AUTHORIZATION POLICY > Settings
        - Authentication
            - Forward requests
        - Accounting
            - Forward requests
