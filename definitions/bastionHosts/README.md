# Azure Bastion Policies

### BAS01_Allowed_Azure_Bastion_SKUs

Azure Bastion provides secure RDP/SSH connectivity to virtual machines without exposing public IP addresses. However, the service has significant cost variations between SKUs, with premium tiers adding features that may not be necessary for all environments.

| SKU | Hourly Cost | Monthly Cost | Key Features |
| :------------- |:------------- |:------------- |:------------- |
| Developer | Free | Free | Basic connectivity, single VM at a time |
| Basic | $0.19 | $138.70 | Standard features, 2 instances |
| Standard | $0.29 | $211.70+ | Scaling, file transfer, shareable links |
| Premium | $0.45 | $328.50+ | Private-only mode, session recording |

This policy helps control costs by restricting which [Azure Bastion SKUs](https://azure.microsoft.com/en-us/pricing/details/azure-bastion/) can be deployed. You can define allowed SKUs based on security requirements and budget constraints.

#### Use Cases

- Use Developer SKU for sandbox and personal development environments where concurrent sessions aren't needed.
- Restrict non-production environments to Basic SKU to avoid unnecessary scaling costs.
- Reserve Premium SKU for production environments with strict compliance requirements for session recording.
- Block Standard/Premium SKUs when advanced features like file transfer aren't required.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| allowedBastionSkus | Developer, Basic, Standard, Premium | Developer |

#### Assignment Behaviors

| effect | allowedBastionSkus | Behavior |
| :------------- |:------------- |:------------- |
| Audit | Developer | Marks Bastions using any SKU other than Developer as non-compliant |
| Deny | Developer, Basic | Prevents creation of Standard or Premium Bastions |
| Deny | Basic | Forces Basic SKU only, blocking both lower and higher tiers |
| Disabled | Any | Does nothing. |

---

### BAS02_Maximum_Bastion_Instance_Count

Azure Bastion Standard and Premium SKUs support scaling beyond the default 2 instances (scale units) to handle more concurrent sessions. Each additional scale unit increases costs proportionally, with each unit supporting approximately 20-25 concurrent sessions.

| SKU | Base Instances | Cost per Additional Instance | Max Instances |
| :------------- |:------------- |:------------- |:------------- |
| Developer | 1 (fixed) | N/A | 1 |
| Basic | 2 (fixed) | N/A | 2 |
| Standard | 2 | $0.14/hour ($102.20/month) | 50 |
| Premium | 2 | $0.22 ($160.60/month) | 50 |

This policy prevents excessive scaling by setting a maximum instance count threshold.

#### Use Cases

- Cap instance counts for production environments with predictable usage patterns.
- Prevent runaway costs from over-provisioning.
- Ensure instance counts align with actual concurrent user requirements.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| MaxInstanceCount | Integer (2-50) | 2 |

#### Assignment Behaviors

| effect | MaxInstanceCount | Behavior |
| :------------- |:------------- |:------------- |
| Audit | 2 | Marks Bastions with more than 2 instances as non-compliant |
| Deny | 5 | Prevents creation or scaling of Bastions beyond 5 instances |
| Deny | 2 | Forces the minimum instance count only, blocking all scaling |
| Disabled | Any | Does nothing. |
