[PS script to perform enterprise DNS health check](https://gallery.technet.microsoft.com/scriptcenter/PS-script-to-perform-DNS-bf8eee66 "Link Microsoft Gallery")

`https://gallery.technet.microsoft.com/scriptcenter/PS-script-to-perform-DNS-bf8eee66 `

# EnterpriseDNS_HealthCheck

# Introduction

A zone can be hosted on multiple DNS servers, a single DNS server can host multiple zones, and a DNS server can act as resolver for external zones. Thus, an organization’s DNS infrastructure can consist of multiple DNS zones hosted and/or getting resolved across multiple DNS servers. Each DNS client in the enterprise needs to be provisioned with single or multiple DNS servers capable of resolving all required zones. DNS clients generally get the necessary DNS sever list from a DHCP scope or server options configured on enterprise DHCP servers. The designated DNS servers needs to be configured with appropriate settings (Forwarders, RootHints, Zone Delegations etc.) to redirect DNS queries from clients to corresponding name servers or other resolvers. If a zone is being hosted across multiple DNS servers, zone aging settings should be configured appropriately on each zone hosting server so that cleanup of stale records will take place at regular intervals.

With the evolution of DNS Server, DHCP Server, and Active Directory Windows PowerShell modules in Windows8, it’s now easy to gather information about DNS building blocks across the entire enterprise and validate their health.

# Problem statement

Currently there is no tool to visualize the integrated entities (configuration parameters or resource records) and thereby enable management (monitoring, troubleshooting, configuring) of these entities to ensure all clients are able to resolve names across all zones in an enterprise. As a result, mistakes during initial configuration or changes in configuration of DHCP and DNS servers can lead to issues where a particular client application fails. Administrators must then spend resources navigating the labyrinth of client application settings, application servers, DNS client settings and multiple DNS servers to troubleshoot and fix the problem.

# Solution

In a typical DNS deployment, a couple of DNS servers are intended to resolve all the zones in the enterprise, and on the Internet. These servers are configured in DHCP scopes as option 6 (IPv4) or option 23 (IPv6) so that all the clients point to these DNS servers. These DNS servers resolve all the names by forwarding queries to DNS servers that host the appropriate zones. Since DNS is hierarchical, a DNS server may have to communicate with other DNS servers before it is able to successfully resolve a DNS query. Not only should all DNS servers in the path be responsive, but the necessary configuration on all DNS servers should be accurate and up to date. While individual DNS servers might be configured properly and be responding to queries, unless the records and configuration parameters that define the relationship between multiple DNS servers are valid and accurate, some DNS servers might not be successful in name resolution.  Ensuring that the DNS servers are able to resolve all the required names is a crucial part of ensuring that the naming infrastructure in an enterprise is healthy.

This script will help a DNS administrator to ensure that all the DNS servers that are configured on clients as DNS servers are able to resolve all required zones (corpnet & Internet). Additionally, it will verify the health of other DNS resources (Forwarders, RootHints, Zone Delegations, Zone Aging settings, etc.) and generate consolidated reports, which can be used for further investigation.

# Technical Information

# How this script gathers information about DNS resources?

The user can specify information about DNS resources either through CmdLine inputs or write single or multiple entries in a text file (each entry in the list separated by new line) with the same name as the parameter name (ex: DNSServerList.txt, DHCPServerList.txt). CmdLine values will take precedence over text file entries if both exist. If both are not present, then the script will try to fetch DNS resource information automatically, as described below:

DNSServerList: List of name resolvers on which a health check needs to be performed. If this list isn’t specified through CmdLine or text file, then this script will prepare it from configured DNS servers in DHCP scope and server options across all enterprise DHCP servers, then dump it in a cache file for the next run.

DHCPServerList: List of DHCP servers across the enterprise (required only if the DNS server cache file is unavailable). If this list isn’t specified through CmdLine or text file, then this script will work with all DHCP servers registered in the current user’s domain and dump it in a cache file for next the run.

DomainList: List of all available root domains across the enterprise. If this list isn’t specified through CmdLine or text file, then this script will work with all available domains in the current user’s forest and dump it in a cache file for the next run.

ZoneList: List of zones to be verified. If this list isn’t specified through CmdLine or text file, then this script will enumerate all the zones hosted on authoritative DNS servers.

ZoneHostingServerList: List of authoritative DNS servers (which host one or more zones). If this list isn’t specified through CmdLine or text file then all DNS servers obtained above will be considered as zone hosting servers.

# How this script works?

# Test-RootDomainHealthAcrossAllDnsServers:
Prepares a table of domain (from DomainList) & respective hosting servers based on received NS records (from default DNS server of test launcher machine) and dumps it into ZoneAndHostingServersHash.html.
Sends a name resolution & Test-DnsServer query for each domain and its _msdcs delegation to each DNS server in DNSServerList.
Validates that each DNS server contains SRV record for _ldap._tcp.dc._msdcs for each domain.
Generates Test-RootDomainHealthAcrossAllDnsServers.html report based on the findings.

# Test-ZoneHealthAcrossAllDnsServers:
Prepares a table of zone (from ZoneList) & respective hosting servers (from ZoneHostingServerList) and dumps it into ZoneAndHostingServersHash.html.
Sends a name resolution & Test-DnsServer query for each zone to each DNS server in DNSServerList.
Generates Test-ZoneHealthAcrossAllDnsServers.html report based on the findings.

# Test-ZoneAgingHealth:
Gathers zone aging settings for each zone in ZoneList across all the server hosting this zone.
Throw warning (visible on PS console & output log file Test-EnterpriseDnsHealth.txt) if aging is enabled on any server with non-default interval (refresh or no-refresh) values or without any scavenge server.
Throw error if aging isn’t enabled on any or more than one server.
Generates Test-ZoneAgingHealth.html report based on the findings.

# Test-ZoneDelegationHealth:
Enumerate all the delegations (As per underlying NS records) under each zone in ZoneList across all the server hosting this zone.
Sends a zone delegation name resolution query to all configured name servers inside above delegations.
Generates Test-ZoneDelegationHealth.html report based on the findings.

# Test-ConfiguredForwarderHealth:
Fetches all configured Forwarders from input DNSServerList and checks that all the Forwarders are responsive.
Generates Test-ConfiguredForwarderHealth.html report based on the findings.

# Test-ConfiguredRootHintsHealth:
Fetches all configured RootHints from input DNSServerList and checks that all the RootHints servers are responsive.
Generates Test-ConfiguredRootHintsHealth.html report based on the findings.

# System Requirements

- Operating System: This PS script can be launched from a machine with Windows Server 2012 or Windows 8 with Remote Server Administration Tools (RSAT) for Windows 8 installed.

- Prerequisites: All the DNS & DHCP servers deployed in the must be running Windows Server 2008 or later.
Required Windows PowerShell modules: DnsServer, DhcpServer & ActiveDirectory.

- Access Permissions: The user should have read access to the enterprise resources such as Active Directory, DHCP, and DNS servers.

# Usage and examples

```powershell
Test-EnterpriseDnsHealth -ValidationType All –DhcpServerList Dhcp1.contoso.com, Dhcp2.contoso.com –Verbose
```
This example performs a health check of all the resources and prints verbose messages on PS console. It fetches information about DNS servers from the DHCP servers Dhcp1.contoso.com and Dhcp2.contoso.com (if DnsServerList.txt is unavailable).

```powershell
Test-EnterpriseDnsHealth @(“Zone”, “Domain”) –ZoneHostingServerList Srv1.contoso.com,Srv2.contoso.com
```

This example performs a health check of all the zones hosted on Srv1.contoso.com and Srv2.contoso.com (if ZoneList.txt is unavailable) or all the zones listed in ZoneList.txt (if ZoneList.txt is available). It’ll also perform a health check of all available root domains across the enterprise (if DomainList.txt is unavailable) or all the root domains listed in DomainList.txt (if DomainList.txt is available).

```powershell
Test-EnterpriseDnsHealth ZoneAging,ZoneDelegation -CleanUpOldCacheFiles -ZoneList Zone1.contoso.com,Zone2.contoso.com
```

This example performs a health check of ZoneAging & ZoneDelegation inside the zones Zone1.contoso.com & Zone2.contoso.com and deletes the old caches with user confirmation.

```powershell
Test-EnterpriseDnsHealth Forwarder,RootHints –CleanUpOldCacheFiles   –CleanUpOldReports –Force –DnsServerList Dns1.contoso.com,Dns2.contoso.com
```

This example performs a health check of Forwarder & RootHints configured on the DNS servers Dns1.contoso.com & Dns2.contoso.com and deletes the old caches & reports without user confirmation.
