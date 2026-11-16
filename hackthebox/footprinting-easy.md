# Footprinting Lab — Easy

> **Source:** Personal notes from the Hack The Box Academy Footprinting module.  
> **Status:** Completed according to the notes.  
> **Publication:** Private study record. Do not publish an Academy lab solution without confirming its sharing permissions.

## Objective

Enumerate the target's exposed services, investigate its internal DNS records and FTP content, and identify a path to SSH access. The notes also mention a flag file on the server; its contents are not reproduced here.

## Initial reconnaissance

The recorded target address was `10.129.142.193` (an ephemeral lab address). Initial service enumeration identified:

| Port | Service | Observation |
|---|---|---|
| 21/tcp | FTP | Accessible for further enumeration |
| 22/tcp | SSH | Potential route for authenticated access |
| 53/tcp | DNS | Internal DNS service |
| 2121/tcp | FTP | A second FTP service; authentication was required |

**Working hypothesis:** DNS records might reveal additional hosts or services, while FTP access might expose useful files. The notes caution against relying on service exploits before completing service enumeration.

## DNS enumeration

The notes record DNS queries against the lab's internal domain (`internal.inlanefreight.htb`, as transcribed). The investigation included:

- NS lookup to identify the name server.
- Version query, with a response recorded as `9.16.1 Ubuntu`.
- ANY query and subdomain exploration; the notes mention a `root` subdomain and other returned IP addresses.
- Zone transfer, which listed additional internal hosts, including apparent `app`, `internal` and `mail` subdomains.

The name server is separately recorded as `10.129.34.136`; because the screenshots include multiple changing lab addresses, this should not be assumed to be the same as the original target.

**Security significance:** DNS zone transfer or overly revealing DNS records can expose internal naming conventions and infrastructure to an unauthorised party. The precise returned record set should be checked against original terminal output before being reproduced.

## FTP enumeration

The notes show an attempt to access the service on port `2121` using credentials identified during enumeration. FTP access was then used to inspect files associated with the target's SSH configuration.

The key finding was an `authorized_keys` file on the FTP-accessible server. The notes record that this file was replaced with a new version containing the tester's public SSH key.

### Observed result

The notes conclude with a successful SSH login using the corresponding private key. They do not include full terminal output or the retrieved flag value.

**Security significance:** Exposure of a user's SSH `authorized_keys` file through a writable FTP service can enable unauthorised SSH key registration and subsequent account access. SSH itself need not be vulnerable for this to be exploitable.

## Attack path summary

1. Enumerated open services: FTP, SSH and DNS.
2. Investigated DNS records and attempted a zone transfer.
3. Identified FTP access and investigated its accessible files.
4. Found a writable route to SSH `authorized_keys`.
5. Registered an SSH public key and obtained SSH access.

## Remediation

- Restrict DNS zone transfers to explicitly authorised secondary DNS servers.
- Avoid exposing internal DNS records unnecessarily.
- Restrict FTP access and remove unnecessary FTP services, especially services reachable on nonstandard ports.
- Prevent network-accessible file services from modifying SSH authentication material.
- Audit modifications to `~/.ssh/authorized_keys` and review access permissions on user home directories.

## Lessons learned

This lab illustrates how several individually commonplace services can combine into a high-impact attack path. Service enumeration and permissions analysis were more important than finding a software exploit.

## Notes on evidence

This document is reconstructed from handwritten study notes. Exact DNS responses, credential values and command outputs are not retained. No sample output has been invented.
