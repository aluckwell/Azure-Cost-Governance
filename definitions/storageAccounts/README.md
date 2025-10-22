# Storage Account Policies

### STO01_Allowed_Storage_Account_Redundancy_Tiers

Storage Account redundancy determines how many copies of your data Azure maintains and where they're located. Higher redundancy tiers provide better durability and availability but cost significantly more. Choosing the wrong tier leads to overspend, especially for large data volumes.

| Redundancy Tier | Copies | Location | Relative Cost |
| :------------- |:------------- |:------------- |:------------- |
| Standard_LRS | 3 | Single datacenter | 1x |
| Standard_ZRS | 3 | Across availability zones | ~1.25x |
| Standard_GRS | 6 | Two regions (async) | ~2x |
| Standard_GZRS | 6 | Zones + regions | ~2.5x |
| Standard_RAGRS | 6 | Two regions (read access) | ~2.5x |
| Standard_RAGZRS | 6 | Zones + regions (read access) | ~3x |
| Premium_LRS | 3 | Single datacenter | ~2-20x (varies by type) |
| Premium_ZRS | 3 | Across availability zones | ~2.5-25x (varies by type) |

For 1TB of blob storage:
- Standard_LRS: $18/month
- Standard_GRS: $36/month
- Standard_RAGZRS: $54/month

This policy helps control costs by restricting which [redundancy tiers](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy) can be used for storage accounts.

#### Use Cases

- Restrict development and test environments to Standard_LRS to minimize costs.
- Use Standard_ZRS for production data requiring zone resilience without geo-redundancy.
- Reserve GRS/GZRS for critical data requiring disaster recovery across regions.
- Block Premium tiers unless specifically required for high-performance workloads.
- Prevent RAGRS/RAGZRS unless read access to secondary region is genuinely needed.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedStorageAccountRedundancyTiers | Standard_LRS, Standard_ZRS, Standard_GRS, Standard_GZRS, Standard_RAGRS, Standard_RAGZRS, Premium_LRS, Premium_ZRS | Standard_LRS |

#### Assignment Behaviors

| effect | AllowedStorageAccountRedundancyTiers | Behavior |
| :------------- |:------------- |:------------- |
| Audit | Standard_LRS | Marks storage accounts using any redundancy other than LRS as non-compliant |
| Deny | Standard_LRS, Standard_ZRS | Prevents creation of geo-redundant or premium storage accounts |
| Deny | Standard_LRS | Forces locally redundant storage only |
| Disabled | Any | Does nothing. |

---

### STO02_Storage_Account_Has_No_Lifecycle_Management

Storage costs accumulate over time as data grows. Without lifecycle management policies, data remains in expensive hot storage tiers indefinitely, even when it's rarely accessed. Lifecycle management automatically moves or deletes data based on age, dramatically reducing costs.

Cost comparison for 1TB of blob storage:
- Hot tier: $18/month
- Cool tier: $10/month (after 30 days)
- Cold tier: $4.50/month (after 90 days)
- Archive tier: $1.80/month (after 180 days)

A lifecycle policy moving data from Hot → Cool → Archive can reduce storage costs by 90% for infrequently accessed data.

This policy identifies storage accounts without lifecycle management policies, indicating potential cost optimization opportunities.

#### Use Cases

- Ensure all storage accounts have policies to move old data to cooler tiers.
- Identify accounts storing logs or backups in hot tier that should be in archive.
- Prevent creation of storage accounts without lifecycle management in production.
- Enforce automatic deletion of temporary data after retention periods.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | AuditIfNotExists, Disabled | AuditIfNotExists |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| AuditIfNotExists | Marks storage accounts without lifecycle management policies as non-compliant |
| Disabled | Does nothing. |

---

### STO03_Allowed_Storage_Account_Blob_Access_Tiers

Storage accounts have a default access tier that determines the base cost for blob storage. The Hot tier is optimized for frequently accessed data but costs more than Cool or Cold tiers. Setting the wrong default tier leads to overspend.

| Access Tier | Storage Cost (per GB) | Best For |
| :------------- |:------------- |:------------- |
| Hot | $0.018/month | Frequently accessed data |
| Cool | $0.010/month | Infrequently accessed, 30+ day retention |
| Cold | $0.0045/month | Rarely accessed, 90+ day retention |

For 1TB of storage:
- Hot: $18/month
- Cool: $10/month (44% savings)
- Cold: $4.50/month (75% savings)

Note: Cool and Cold tiers have higher transaction costs and early deletion fees, so they're only cost-effective for data accessed infrequently.

This policy restricts which default access tiers can be used, helping ensure storage accounts are configured appropriately for their data access patterns.

#### Use Cases

- Force Cool or Cold tier for backup and archive storage accounts.
- Restrict development environments to Hot tier only for simplicity.
- Prevent Hot tier for log storage accounts where data is rarely accessed.
- Ensure appropriate tier selection based on data access frequency.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedStorageAccountAccessTiers | Hot, Cool, Cold | Hot |

#### Assignment Behaviors

| effect | AllowedStorageAccountAccessTiers | Behavior |
| :------------- |:------------- |:------------- |
| Audit | Cool | Marks storage accounts using Hot or Cold tier as non-compliant |
| Deny | Hot | Prevents creation of storage accounts with Cool or Cold default tier |
| Deny | Cool, Cold | Forces cooler tiers only, blocking Hot tier |
| Disabled | Any | Does nothing. |

---

### STO04_Storage_Account_has_SFTP_enabled

Azure Storage SFTP support enables secure file transfer protocol access to blob storage. However, this feature has a significant fixed monthly cost of over $200, regardless of usage. If SFTP isn't actively used, this is pure waste.

| Feature | Monthly Cost | Notes |
| :------------- |:------------- |:------------- |
| SFTP Enabled | ~$200-250 | Fixed cost regardless of usage |
| SFTP Disabled | $0 | Use alternative access methods |

Alternative access methods with no additional cost:
- Azure Storage Explorer
- AzCopy command-line tool
- Azure Portal upload/download
- REST API / SDK access
- Azure File Shares with SMB

This policy identifies storage accounts with SFTP enabled, helping you evaluate whether the feature justifies the monthly cost or if alternative access methods would suffice.

#### Use Cases

- Disable SFTP in non-production environments where it's not needed.
- Identify storage accounts with SFTP enabled but no active SFTP users.
- Prevent accidental enablement of SFTP during storage account creation.
- Use alternative access methods (AzCopy, Storage Explorer) to avoid SFTP costs.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks storage accounts with SFTP enabled as non-compliant |
| Deny | Prevents enabling SFTP on storage accounts |
| Disabled | Does nothing. |
