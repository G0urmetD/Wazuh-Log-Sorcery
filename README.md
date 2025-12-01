# Wazuh Log Sorcery

Custom Wazuh decoders and rules for enterprise security monitoring. This collection includes ready-to-use configurations for Palo Alto Networks firewalls and Passwork password manager.

## Quick Start

1. Copy decoder and rule files to your Wazuh manager:
```bash
# Install decoders
cp decoders/<vendor>/*.xml /var/ossec/etc/decoders/

# Install rules
cp rules/<vendor>/*.xml /var/ossec/etc/rules/
```

2. Restart Wazuh manager:
```bash
systemctl restart wazuh-manager
```

## What's Included

### 🔥 Palo Alto Networks
**Rules:** 7 configuration monitoring rules (IDs 100501-100507)
- Configuration change detection (commit, set, edit, delete, rename, move)
- Requires Wazuh's built-in Palo Alto decoder (parent rule 64500)

### 🔐 Passwork Password Manager  
**Decoders:** CEF format parser with program_name matching (php-fpm)  
**Rules:** 148 monitoring rules (IDs 100700-100960)

Monitors 15 categories:
- Password operations (view, create, edit, delete, copy, share, export)
- Shortcut management
- Vault operations and access control
- Folder operations
- User authentication (login, logout, 2FA, sessions)
- User and group administration
- Role management
- System settings (SSO, LDAP sync, licensing)
- Background tasks and audit logs

**Documentation:**
- [Passwork Syslog Setup](https://passwork.pro/tech-guides/administration/action-history/syslog-and-eventviewer/)
- [Event Reference](https://passwork.pro/tech-guides/administration/action-history/action-history-events-list/)

## Repository Structure

```
decoders/
└── passwork/
    └── passwork_decoder-v2.xml

rules/
├── palo/
│   └── paloalto_rules.xml
└── passwork/
    └── passwork_rules.xml
```

## Contributing

Contributions welcome! Please maintain the existing structure:
- Place decoders in `decoders/<vendor>/`
- Place rules in `rules/<vendor>/`
- Include vendor-specific README where helpful

## License

MIT License - see [LICENSE](LICENSE) for details.
