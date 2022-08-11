# unbound-pfsense-ad
Export Active Directory DNS information to an unbound include file, SRV records, to use unbound / pfSense as the DNS resolver, rather than Windows AD DNS.

## Future
I'm working on a revision of this that obtains the domain GUID, domain controllers GUID, and autopopulates the entries. I need to test with multiple domain controllers. I've seen the export fail when the DNS servers are not responding appropriately with the SRV records, and avoiding this and obtaining a list of DCs to populate the configuration file would be preferable.
To obtain the domain GUID and DC GUID in PowerShell:
- Domain GUID
  - Run as Administrator
  - Get-ADDomain | Select-Object -ExpandProperty DomainSID | Select-Object -ExpandProperty Value
- Domain controller GUID
  - Get-ADDomainController | Select-Object -ExpandProperty InvocationID | Select-Object -ExpandProperty Guid
  - It will return an array, if multiple DCs are present
To obtain the domain SID in PowerShell:
- Get-ADDomain | Select-Object -ExpandProperty DomainSID | Select-Object -ExpandProperty Value

## Usage
- Run the PowerShell script as admin on a domain controller
- Configuration file is automatically generated, unbound.adinclude.conf
- Verify configuration file
  - Open with a text editor (e.g. notepad++ or something that supports UNIX format)
  - Remove bogus entries, such as for netwrok adapters with multiple IP addresses that are not accessible
- Upload configuration file
  - /var/unbound/unbound.adinclude.conf
  - chmod 644 /var/unbound/unbound.adinclude.conf
  - chown root:unbound /var/unbound/unbound.adinclude.conf
- Unbound advanced configuration:
  - server:include: /var/unbound/unbound.adinclude.conf
- Point the clients to use the firewall as DNS

## Limitations
- Active Directory uses secure dynamic DNS updates, this does not, it's likely a fit for smaller environments, but not larger environments

## Rationale
- DNS and DHCP need appliance-level availability
- Forcing Windows DNS with Active Directory impacts uptime and perception, server down == network down
- Using unbound with TYPETRANSPARENT, it is possible to use a UPN that reflects the public internet UPN, without split DNS
  - TYPETRANSPARENT will attempt to resolve the DNS record locally first, and when this fails, it will revert to the firewall's system-configured DNS
  - Tested with Azure AD Connect domains and standard Active Directory
  - Unbound should support deferring DNS resolution to an alternate DNS server as specified in the configuration file, where the firewall does not have the local records, it is on my agenda to test
- pfSense's DHCP implementation will automatically link local DHCP/DNS registrations and PTR

More information is available in the PowerShell script comments.
