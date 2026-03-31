# Managed Disk Policies

### DSK01_Allowed_Disk_SKUs_for_environment

Azure Managed Disks come in various SKUs with dramatically different performance characteristics and costs. Choosing premium or ultra disks when standard disks would suffice leads to significant overspend, especially when multiplied across many VMs.

| SKU | Storage Type | IOPS | Throughput | Cost (256 GB) |
| :------------- |:------------- |:------------- |:------------- |:------------- |
| Standard_LRS | HDD | 500 | 60 MB/s | $19.20/month |
| StandardSSD_LRS | SSD | 500 | 60 MB/s | $30.72/month |
| Premium_LRS | SSD | 1,100 | 125 MB/s | $49.15/month |
| PremiumV2_LRS | SSD | Configurable | Configurable | $61.44/month + IOPS/throughput |
| UltraSSD_LRS | SSD | Configurable | Configurable | $122.88/month + IOPS/throughput |

This policy helps prevent over-provisioning by restricting which [disk SKUs](https://azure.microsoft.com/en-us/pricing/details/managed-disks/) can be used. Define allowed SKUs based on workload performance requirements and environment type.

#### Use Cases

- Restrict development and test environments to Standard_LRS to minimize costs.
- Allow StandardSSD_LRS for non-production workloads requiring better performance than HDD.
- Reserve Premium_LRS for production databases and performance-sensitive applications.
- Block UltraSSD_LRS unless specifically required for extreme IOPS workloads (100,000+ IOPS).

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedDiskSKUs | Standard_LRS, StandardSSD_LRS, Premium_LRS, PremiumV2_LRS, UltraSSD_LRS, StandardSSD_ZRS, Premium_ZRS | Standard_LRS |

#### Assignment Behaviors

| effect | AllowedDiskSKUs | Behavior |
| :------------- |:------------- |:------------- |
| Audit | Standard_LRS | Marks all disks using premium SKUs as non-compliant |
| Deny | Standard_LRS, StandardSSD_LRS | Prevents creation of premium, ultra, or zone-redundant disks |
| Deny | Premium_LRS | Forces Premium SSD only, blocking all other tiers |
| Disabled | Any | Does nothing. |

---

### DSK02_Max_Disk_Size_for_Environment

Disk costs scale with size, and over-provisioning disk capacity is a common source of waste. Many workloads don't require large disks, especially in non-production environments. Limiting disk sizes prevents unnecessary spending.

| Disk Size | Standard_LRS | StandardSSD_LRS | Premium_LRS |
| :------------- |:------------- |:------------- |:------------- |
| 128 GB | $9.60/month | $19.20/month | $30.72/month |
| 256 GB | $19.20/month | $30.72/month | $49.15/month |
| 512 GB | $38.40/month | $61.44/month | $81.92/month |
| 1 TB | $76.80/month | $122.88/month | $122.88/month |
| 4 TB | $307.20/month | $491.52/month | $491.52/month |

This policy sets a maximum disk size threshold to prevent over-provisioning. Costs multiply quickly across multiple VMs - 10 VMs with 1TB disks cost $768-$1,229/month more than 256GB disks.

#### Use Cases

- Limit sandbox environments to 128 GB to keep costs minimal for testing.
- Cap non-production disks at 256 GB unless larger sizes are justified.
- Prevent accidental creation of multi-terabyte disks in development.
- Enforce disk sizing standards that align with actual data requirements.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| MaxDiskSize | Integer (GB) | 256 |

#### Assignment Behaviors

| effect | MaxDiskSize | Behavior |
| :------------- |:------------- |:------------- |
| Audit | 256 | Marks disks larger than 256 GB as non-compliant |
| Deny | 128 | Prevents creation of disks larger than 128 GB |
| Deny | 512 | Blocks disks over 512 GB, allowing moderate sizes |
| Disabled | Any | Does nothing. |
