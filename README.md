# Accounts

A **base Accounts API** for the [API Commons](https://apicommons.org) — the account
lifecycle every service reinvents, described once.

We need an account for everything these days. Every new service builds the same account
resource, the same onboarding flow, and the same errors — each one slightly different,
which is exactly what makes integration expensive.

## What's here

- **[openapi.yml](openapi.yml)** — OpenAPI 3.1 covering list, create, read, update, and
  close.
- **[apis.yml](apis.yml)** — the APIs.json index for this base.

## Shape

| Operation | Method | Path |
| --- | --- | --- |
| `listAccounts` | GET | `/accounts` |
| `createAccount` | POST | `/accounts` |
| `getAccount` | GET | `/accounts/{accountId}` |
| `updateAccount` | PATCH | `/accounts/{accountId}` |
| `closeAccount` | DELETE | `/accounts/{accountId}` |

Two choices worth keeping when you copy it:

**Updates are [RFC 7396](https://www.rfc-editor.org/rfc/rfc7396) JSON Merge Patch**, sent
as `application/merge-patch+json`. A client changes one field without sending the whole
account back. The trap that comes with it is documented on `AccountPatch`: **a member set
to `null` is removed**, so a client that serializes absent optional fields as null will
silently erase data.

**Writes are conditional.** Reads return an `ETag`; `PATCH` accepts `If-Match` and answers
`412` when the account changed underneath you. Closing an account is `DELETE`, and the
description is explicit that closing is not erasing — whatever service adopts this owes
its users a retention policy.

## Errors

Every API Commons base errors the same way: [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)
problem details, `application/problem+json`, with the same `Problem` schema and the same
set of named responses lifted from
[problem-details-for-http-apis](https://github.com/api-commons/problem-details-for-http-apis).

That block is byte-identical across the bases on purpose. If you adopt more than one,
your clients parse one error format.

Conformance is checked by the
[Problem Details Spectral ruleset](https://github.com/api-commons/spectral-problem-details-ruleset):

```
spectral lint openapi.yml \
  -r https://raw.githubusercontent.com/api-commons/spectral-problem-details-ruleset/main/problem-details.yaml
```

This file lints **clean** under that ruleset, and under `spectral:oas` apart from one
deliberate warning: `oas3-api-servers`. A base template has no server, and adding a
placeholder would only trip `oas3-server-not-example.com`. Add your own `servers` when
you adopt it.

## Using it

Copy `openapi.yml` into your own repo and change it. This is a starting point, not a
dependency — there is no hosted API behind it and nothing to install. Keep the error
components as they are and you inherit a standard error contract for free.

`apis.yml` is the [APIs.json](https://apisjson.org) index for this base, pointing at the
OpenAPI, this repository, the ruleset, and the documentation.

## License

[Apache-2.0](LICENSE).
