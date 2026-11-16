# TwoMillion — Hack The Box Write-up (Incomplete)

**Platform:** Hack The Box  
**Machine:** TwoMillion  
**Status:** Incomplete — investigation notes; no shell or privilege escalation documented  
**Scope:** Authorised Hack The Box lab

> This write-up is based on my notes from the exercise. It records what I observed and investigated, not a confirmed end-to-end compromise. Commands below are cleaned-up examples where the original notes did not preserve exact syntax. IP addresses, tokens and credentials are omitted or replaced with placeholders.

## 1. Initial enumeration

My first notes recorded two exposed TCP services:

| Port | Service | Observation |
|---|---|---|
| 22/tcp | SSH | OpenSSH 8.9p1 (version transcribed from notes) |
| 80/tcp | HTTP | nginx |

The web service became the primary focus of investigation.

## 2. Web application and invitation workflow

I identified an API associated with the site's invitation workflow. Querying an API endpoint produced JSON containing a hint encoded with **ROT13**. Decoding the hint pointed towards a further invite-generation endpoint:

```text
/api/v1/invite/generate
```

I then queried that endpoint. The response contained invitation-related data encoded in **Base64**. Decoding the value revealed an invitation code used during registration.

### Illustrative decoding commands

These illustrate the decoding operations; the original encoded values were not retained in my notes.

```bash
# ROT13 decoding of a saved hint
printf '%s' '<rot13-hint>' | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# Base64 decoding of a saved invitation value
printf '%s' '<base64-value>' | base64 -d
```

**Observation:** The encoded responses were a useful clue in understanding the registration workflow. Encoding alone does not provide authentication or access control.

## 3. Authenticated API exploration

After working through the invitation process, I investigated authenticated application requests using **Burp Suite**. My notes identify another endpoint associated with generating a VPN configuration:

```text
/api/v1/user/vpn/generate
```

I used the structure of a valid, authenticated API request as a starting point for identifying the endpoint and understanding the application's request format.

My notes also record a host name associated with a VPN configuration. I have omitted it here because the exact transcription is uncertain and it is not essential to the investigation narrative.

**Limit of recorded evidence:** My notes do not establish whether this endpoint could be exploited, or whether a privilege boundary was crossed.

## 4. Database-related notes

I recorded database or web-panel credentials during the lab, along with a database host and port. The original notes do not clearly show how these credentials were discovered or demonstrate a successful database connection.

For that reason, I have **not** included credentials or described database access as a confirmed result.

## 5. Where the investigation stopped

The final note contains a local listener address and port, which suggests I was preparing to investigate possible command execution and obtain a reverse shell.

However, I did **not** record a successful payload execution, inbound connection, shell, user flag, or privilege escalation. The assessment is therefore marked **Incomplete**.

## 6. Security observations and possible mitigations

The following are general lessons drawn from the areas I investigated, **not confirmed vulnerabilities** in the machine:

- An API should enforce authorisation on the server for every sensitive operation, regardless of which interface invokes it.
- ROT13 and Base64 are encodings, not security controls.
- Authentication and invitation workflows deserve testing for unintended exposure or privilege changes.
- Sensitive credentials should not appear in accessible application files, logs, or responses.
- Application endpoints should validate inputs and restrict access according to user permissions.

## 7. Lessons learned and next steps

This exercise reinforced the value of following clues across API responses, decoding application data, and using Burp Suite to examine authenticated requests.

If I revisit the machine, I would document:

1. The exact requests and responses used to locate the invitation endpoints.
2. How the authenticated account was created and what permissions it received.
3. Whether the VPN-generation endpoint exposes an actual security weakness.
4. Any verified foothold, privilege escalation, and relevant mitigation.

---

*These notes reflect my own partial lab investigation. They are not intended as a complete walkthrough.*
