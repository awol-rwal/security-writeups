# Footprinting Lab — Medium (Incomplete)

> **Source:** Personal notes and screenshots from the Hack The Box Academy Footprinting module.  
> **Status:** **Unfinished** — enumeration and several useful findings recorded, but no confirmed final compromise.  
> **Publication:** Private study record. Do not publish an Academy lab solution without confirming its sharing permissions.

## Objective

Footprint a target exposing multiple enterprise services, identify useful information through network-service enumeration, and investigate possible paths to authenticated access. The notes specify a goal of finding a user associated with the organisation and a corresponding password.

## Initial reconnaissance

The target address noted was `10.129.202.41` (an ephemeral lab address). The handwritten port list includes:

| Port | Probable service | Notes |
|---|---|---|
| 111/tcp | rpcbind | RPC enumeration attempted |
| 135/tcp | MSRPC | Windows RPC |
| 139/tcp | NetBIOS/SMB | SMB enumeration target |
| 445/tcp | SMB | Shares investigated |
| 2049/tcp | NFS | Exports investigated |
| 3389/tcp | RDP | RDP enumeration attempted |
| 5985/tcp | WinRM | Appears in the recorded port list; not confirmed as accessible |
| 47001/tcp | HTTP/Windows RPC-related endpoint | Service label uncertain in handwriting |
| 49664–49665/tcp | Dynamic RPC-style ports | Recorded among exposed ports |

These port interpretations should be checked against the original scan before treating the list as authoritative.

## RPC and NFS enumeration

An RPC query recorded a host with `Protocol-Hybrid = TRUE` and a `WINM...`-style NetBIOS name; the exact hostname is unclear from the notes.

NFS enumeration revealed an export:

```text
/TechSupport (everyone)
```

The notes identify this as an openly accessible mount and record finding a support ticket inside it. That ticket contained mail-service-related credentials/configuration information.

**Security significance:** Broadly accessible NFS shares can expose operational support data and credentials that enable access to unrelated services.

## Information exposed in the support ticket

The screenshot shows a conversation about troubleshooting an SMTP web service. A configuration extract included:

- An SMTP host name under `inlanefreight.htb`.
- A mailbox username.
- A password field.
- Web application/session settings.

The original password and other sensitive values are intentionally omitted from this reconstructed write-up. If needed for private re-testing, consult the original lab notes rather than publishing credentials.

## SMB enumeration

The notes record SMB enumeration and a share listing with the following observations:

| Share | Recorded result |
|---|---|
| `ADMIN$` | Access denied with the credentials tested |
| `C$` | Administrative share listed; access not confirmed |
| `devshare` | Accessed and found login-related information |
| `IPC$` | Listed; no substantive content recorded |
| `Users` | Listed for further investigation |

A screenshot records a `devshare` file containing what appeared to be login-related information, as well as lab flag entries. These are not reproduced here. The handwritten notes suggest continuing toward the `Users` share.

Additional SMB checks reportedly included `netshareenumall`, which returned access denied, and status information from another enumeration command; the exact command is unclear.

## RDP and MSSQL investigation

- The notes indicate that RDP login was attempted using discovered credentials, but no successful interactive session is recorded.
- NFS and SMB enumeration also surfaced other host addresses, though the relationship between hosts was not fully established.
- An attempt to connect to Microsoft SQL Server using credentials failed; a later Azure AD-related attempt returned an untrusted-certificate error.

These attempts did not establish a confirmed route to final access in the supplied notes.

## Progress and current stopping point

**Confirmed in the notes:**

1. Enumerated a broad range of Windows and UNIX-style network services.
2. Discovered an openly accessible `/TechSupport` NFS export.
3. Retrieved a support conversation containing SMTP configuration information.
4. Enumerated SMB shares and discovered login-related information in `devshare`.
5. Attempted additional RPC, RDP, SMB and MSSQL investigation.

**Not confirmed:** a successful final login, an escalation path, or completion of the lab's objective.

## What to investigate if resuming

- Revisit the exact NFS and SMB share permissions and map the files already identified.
- Clarify which account names and services the discovered configuration data apply to.
- Review the `Users` share and the reasons the other enumeration attempts failed.
- Record each attempted credential/service combination and its outcome in a private table.
- Confirm the target and host addresses because the notes include multiple lab IPs.

## Remediation themes

- Restrict NFS exports to intended hosts and minimise read access.
- Remove plaintext credentials from support tickets and configuration examples.
- Apply least-privilege SMB share permissions.
- Review cross-service credential reuse and rotate exposed secrets.
- Limit unnecessary exposure of RDP, RPC and management interfaces.

## Evidence limitations

This reconstruction follows the provided handwritten notes and screenshots. Some service labels, hostnames and addresses are hard to read; unclear details have been qualified rather than guessed. The lab remains **unfinished**.
