# Recovery Services Vault Policies

### RSV01_Allowed_Recovery_Services_Vault_Redundancy_Tiers

Recovery Services Vaults store backup data with different redundancy options that significantly impact costs. While geo-redundancy provides disaster recovery capabilities, it costs substantially more than locally redundant storage and may not be necessary for all backup scenarios.

| Redundancy Tier | Description | Relative Cost | Use Case |
| :------------- |:------------- |:------------- |:------------- |
| LocallyRedundant (LRS) | 3 copies in single region | 1x | Non-critical backups, dev/test |
| ZoneRedundant (ZRS) | 3 copies across availability zones | ~1.25x | Production with zone resilience |
| GeoRedundant (GRS) | 6 copies across regions | ~2x | Critical data requiring DR |

Backup storage costs vary by data type, but GRS costs approximately double LRS. For 1TB of backup data:
- LRS: ~$10-20/month
- ZRS: ~$12-25/month
- GRS: ~$20-40/month

This policy helps control backup costs by restricting which [redundancy tiers](https://learn.microsoft.com/en-us/azure/backup/backup-azure-recovery-services-vault-overview) can be used for Recovery Services Vaults.

#### Use Cases

- Restrict development and test environments to LocallyRedundant to minimize backup costs.
- Use ZoneRedundant for production workloads that need high availability within a region.
- Reserve GeoRedundant for critical production data requiring cross-region disaster recovery.
- Prevent unnecessary geo-redundancy for short-term backups or non-critical data.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedVaultRedundancyTiers | LocallyRedundant, ZoneRedundant, GeoRedundant | LocallyRedundant |

#### Assignment Behaviors

| effect | AllowedVaultRedundancyTiers | Behavior |
| :------------- |:------------- |:------------- |
| Audit | LocallyRedundant | Marks vaults using ZRS or GRS as non-compliant |
| Deny | LocallyRedundant | Prevents creation of vaults with zone or geo redundancy |
| Deny | LocallyRedundant, ZoneRedundant | Blocks only GeoRedundant, allowing LRS and ZRS |
| Disabled | Any | Does nothing. |
