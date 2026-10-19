# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased] - 2026-10-19

Brings the Passwork ruleset up to date with **Passwork 7.7.0** (released 2026-08-03).
The previous state covered Passwork 7.2.x; the 22 action history events introduced
between 7.3.0 and 7.7.0 were passing through rule `100700` (level 0) and therefore
produced no alerts.

Verified against the official
[event list](https://passwork.pro/tech-guides/administration/action-history/action-history-events-list/):
**168 of 168 documented events are now covered**, no stale event codes remain.

### Added — Passwork rules (22 event rules, 3 correlation rules)

Offline mode — new in Passwork 7.7.0:

| ID | Level | Event code | Description |
|----|-------|------------|-------------|
| 100723 | 3 | `item_viewed_offline` | Password opened offline |
| 100724 | 6 | `offlineItem_created` | Password saved offline to device |
| 100725 | 5 | `offlineItem_synced` | Offline password synchronized |
| 100726 | 3 | `offlineItem_removed` | Offline password removed from device |
| 100741 | 3 | `shortcut_viewed_offline` | Shortcut viewed offline |

Passkeys, WebAuthn and master password — new in Passwork 7.3.0–7.6.1:

| ID | Level | Event code | Description |
|----|-------|------------|-------------|
| 100819 | 3 | `user_email_confirmed` | Email confirmed |
| 100820 | 5 | `user_master_key_changed` | Master password changed |
| 100821 | 7 | `user_passkey_added` | Passkey added |
| 100822 | 5 | `user_passkey_deleted` | Passkey deleted |
| 100823 | 5 | `user_passkey_auth_enabled` | Passkey authentication enabled |
| 100824 | 7 | `user_passkey_auth_disabled` | Passkey authentication disabled |
| 100825 | 5 | `user_webauthn_2fa_added` | 2FA biometrics or security key added |
| 100826 | 7 | `user_webauthn_2fa_deleted` | 2FA biometrics or security key deleted |
| 100827 | 3 | `user_webauthn_2fa_enabled` | 2FA with biometrics or security key enabled |
| 100828 | 7 | `user_webauthn_2fa_disabled` | 2FA with biometrics or security key disabled |
| 100829 | 5 | `user_auth_check_failed` | Failed attempt to unlock authentication settings |
| 100849 | 10 | `user_master_key_reset` | User master password reset by admin |
| 100850 | 8 | `user_passkey_reset` | User passkey reset by admin |

Service accounts — new in Passwork 7.6.0:

| ID | Level | Event code | Description |
|----|-------|------------|-------------|
| 100851 | 8 | `service_account_api_tokens_generated` | Service account API token generated |
| 100852 | 5 | `service_account_api_tokens_revoked` | Service account API token revoked |

LDAP restriction and test event — new in Passwork 7.6.2 / 7.7.0:

| ID | Level | Event code | Description |
|----|-------|------------|-------------|
| 100853 | 7 | `user_auth_failed_ldap_group_restriction` | Sign-in denied by LDAP group restriction |
| 100961 | 3 | `test_event_sent` | Test event sent (log pipeline verification) |

Correlation rules in the new range 100970–100979:

| ID | Level | Trigger | Detects |
|----|-------|---------|---------|
| 100970 | 10 | 10× `100724` / 300s, same `srcuser` | Mass offline save of passwords — possible exfiltration to an endpoint |
| 100971 | 10 | 5× `100853` / 300s, same `srcip` | Repeated sign-ins with valid credentials but no LDAP group authorization |
| 100972 | 10 | 5× `100829` / 300s, same `srcuser` | Repeated failed attempts to unlock the authentication settings page |

Rule levels follow the conventions already used in `passwork_rules.xml` and
intentionally do not mirror Passwork's own "Priority" column, which the existing
ruleset does not follow either.

### Added — Passwork decoder

- `passwork.duid` — the `duid` CEF extension ("ID of the user on whom the action
  was performed") is officially documented but was not decoded. It identifies the
  target of every admin action on a foreign account (`user_password_reset`,
  `user_master_key_reset`, `user_passkey_reset`, `user_2fa_reset_by_admin`,
  `service_account_api_tokens_generated`, …).
- `passwork.device` — the client type Passwork reports per event
  (`web`, `desktop`, `browser addon`, `mobile`, `api`, `internal`), which makes it
  possible to distinguish e.g. an export issued via the API from one issued in the
  web interface.

  **Verify before relying on this field.** The position of the device value inside
  the CEF header is not specified in the Passwork documentation. The decoder only
  matches when the value sits in the Device Product field (position 3) and is
  inert otherwise. Confirm with `wazuh-logtest` against a real log line. If it does
  match, the existing `passwork.product` field carries the same value and
  `passwork.device` is a correctly named alias for it.

### Changed

- `rules/passwork/passwork_rules.xml`: rule ID range in the file header extended
  from `100700 - 100960` to `100700 - 100979`.
- `README.md`: Passwork rule count and ID range updated.

### Known issues (not addressed in this change)

- `msg=([^=]+)` also captures the name of the following CEF key into
  `passwork.message`.
- `cs1`–`cs4` use `(\S+)` and therefore truncate values containing spaces, unlike
  the `passworkVaultName` / `passworkFolderName` / `passworkItemName` decoders,
  which have dedicated quoted variants.
- Dots are unescaped in the `srcip` regexes (`\d+.\d+.\d+.\d+`). Cosmetic — the
  patterns still match valid addresses.

## [0.1.0] - 2025-12-01

Initial release.

### Added

- Passwork CEF decoder (`decoders/passwork/passwork_decoder-v2.xml`) using
  `program_name` matching on `php-fpm`, with 30 extension field decoders.
- Passwork ruleset (`rules/passwork/passwork_rules.xml`) with 146 event rules and
  one brute force correlation rule, covering the full Passwork 7.2.x event list.
- Palo Alto Networks configuration change rules
  (`rules/palo/paloalto_rules.xml`, IDs 100501–100507) on top of Wazuh's built-in
  Palo Alto decoder.
