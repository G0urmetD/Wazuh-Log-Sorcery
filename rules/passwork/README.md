# Passwork Rules

Custom Wazuh rules for the Passwork password manager.

Covers all **168 documented action history events** of Passwork **7.7.0**.

## Files

- `passwork_rules.xml` - 173 rules (IDs 100700-100972)

## Requirements

Requires the CEF decoder `decoders/passwork/passwork_decoder.xml`, which matches
on `program_name` = `php-fpm`. Without it the rules never fire.

In Passwork, enable **Settings and Users → Action History → Settings →
Record action history to syslog or Windows Event Log**, and make sure the Wazuh
agent reads the resulting log file:

| Platform | Log file |
|----------|----------|
| DEB (Ubuntu, Debian) | `/var/log/syslog` |
| RPM (CentOS, RedHat) | `/var/log/messages` |
| Docker | `/<passwork>/log/php/syslog` |
| Windows Server | Event Viewer |

## Rule Groups

| ID range | Rules | Category |
|----------|-------|----------|
| 100700 | 1 | Base rule (level 0, parent for all others) |
| 100701-100726 | 26 | Password / item actions, incl. offline mode |
| 100730-100741 | 12 | Shortcut actions, incl. offline mode |
| 100750-100770 | 21 | Vault actions and access control |
| 100780-100792 | 13 | Folder actions |
| 100800-100829 | 30 | User account: sessions, 2FA, passkeys, WebAuthn, API tokens |
| 100830-100853 | 24 | User management: admin resets, service accounts, LDAP |
| 100860-100865 | 6 | Group actions |
| 100870-100872 | 3 | Invitations |
| 100880-100884 | 5 | Roles |
| 100890-100892 | 3 | System settings |
| 100900 | 1 | SSO settings |
| 100910 | 1 | License |
| 100920-100938 | 19 | LDAP settings and synchronization |
| 100950-100952 | 3 | Background tasks |
| 100960-100961 | 2 | Action history settings |
| 100970-100972 | 3 | Correlation rules |

Free IDs inside each range are reserved for future Passwork events of that category.
New correlation rules belong in 100970-100979.

## High Severity Rules

| ID | Level | Event |
|----|-------|-------|
| 100845 | 12 | Organization owner changed |
| 100847 | 12 | Emergency password reset |
| 100890 | 12 | System settings changed |
| 100900 | 12 | SSO settings changed |
| 100923 | 12 | LDAP server settings changed |
| 100928 | 12 | LDAP server deleted |
| 100710 | 10 | Password exported - potential data exfiltration |
| 100817 | 10 | Authentication blocked - possible brute force |
| 100837 | 10 | User password reset by admin |
| 100849 | 10 | User master password reset by admin |
| 100924 | 10 | LDAP user list settings changed |
| 100925 | 10 | LDAP group list settings changed |
| 100926 | 10 | LDAP synchronization settings changed |

## Correlation Rules

| ID | Level | Trigger | Detects |
|----|-------|---------|---------|
| 100818 | 10 | 5x `100814` / 120s, same `srcip` | Brute force against the sign-in form |
| 100970 | 10 | 10x `100724` / 300s, same `srcuser` | Mass offline save of passwords - possible exfiltration to an endpoint |
| 100971 | 10 | 5x `100853` / 300s, same `srcip` | Repeated sign-ins with valid credentials but no LDAP group authorization |
| 100972 | 10 | 5x `100829` / 300s, same `srcuser` | Repeated failed attempts to unlock the authentication settings page |

## Rule Groups for Filtering

Beyond the file-level group `passwork_rules`, rules are tagged by subject and by
behaviour, so alerts can be filtered either way:

**Subject:** `passwork_password`, `passwork_shortcut`, `passwork_vault`,
`passwork_folder`, `passwork_user_account`, `passwork_user_management`,
`passwork_group`, `passwork_invite`, `passwork_role`, `passwork_system`,
`passwork_sso`, `passwork_license`, `passwork_ldap`, `passwork_task`,
`passwork_activity_log`, `passwork_offline`, `passwork_passkey`,
`passwork_webauthn`, `passwork_service_account`

**Behaviour:** `authentication`, `authentication_success`, `authentication_failed`,
`access_control`, `data_access`, `data_modification`, `data_sharing`, `data_import`,
`data_export`, `configuration_change`, `session`, `api`, `ldap`, `security_warning`,
`critical`, `attack`

## Decoded Fields

Available in rule conditions and alert output:

`passwork.event_code`, `passwork.event_name`, `passwork.severity`,
`passwork.product`, `passwork.device`, `passwork.device_version`,
`passwork.device_action`, `passwork.action`, `passwork.outcome`,
`passwork.message`, `passwork.reason`, `passwork.suid`, `passwork.duid`,
`passwork.item_id`, `passwork.item_name`, `passwork.folder_id`,
`passwork.folder_name`, `passwork.vault_id`, `passwork.vault_name`,
`passwork.group_name`, `passwork.role`, `passwork.target_user`,
`passwork.file_name`, `passwork.old_name`, `passwork.device_external_id`,
`passwork.start_time`, `passwork.end_time`, `passwork.extension`,
`passwork.cs1_label`, `passwork.cs2_label`, `passwork.cs3_label`,
`passwork.cs4_label`, `cef.version`

Mapped to Wazuh standard fields: `srcip`, `srcuser`, `dstuser`, `url`.

## Testing

Verify decoding and rule matching against a real log line:

```bash
/var/ossec/bin/wazuh-logtest
```

In Passwork, **Action History → Settings → Send test event** generates a
`test_event_sent` event (rule 100961) to confirm the pipeline end to end.

## Keeping the Ruleset Current

Passwork adds action history events with new features. To check for gaps, compare
the event codes in `passwork_rules.xml` against the official event list:

```bash
grep -oE 'passwork\.event_code">\^[a-zA-Z0-9_]+\$' passwork_rules.xml \
  | sed 's/.*\^//; s/\$$//' | sort -u
```

Unmatched events fall through the base rule 100700 (level 0) and produce **no
alert**, so gaps are silent. See [CHANGELOG.md](../../CHANGELOG.md) for the
Passwork release this ruleset is aligned with.

## Documentation

- [Event forwarding overview](https://passwork.pro/tech-guides/administration/action-history/syslog-and-eventviewer/)
- [Event list](https://passwork.pro/tech-guides/administration/action-history/action-history-events-list/)
- [Passwork release notes](https://passwork.pro/blog/release-notes/)
