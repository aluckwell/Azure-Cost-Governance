# Application Gateway Policies


### AGW01_Allowed_Application_Gateway_Tiers

Application Gateway is a fixed cost resource. You pay pax x-amount per hour, regardless of its actual usage. It could be bone idle and you'll still pay up to $263 per month for the privilege:

| SKU | Hourly Fixed Cost | Monthly Cost |
| :------------- |:------------- |:------------- |
| Basic | $0.0225 | $16.425 |
| Standard_v2 | $0.20 | $146 |
| WAF_v2 | $0.36 | $262.8 |



This policy helps prevent over-provisioning by restricting the [SKUs](https://learn.microsoft.com/en-gb/azure/application-gateway/overview-v2?WT.mc_id=Portal-fx#sku-types) that Application Gateways can use. During Policy Assignment you can define which SKUs are allowed based on application requirements, and block the others to avoid over-provisioning. If requirements change, the parameters can be easily changed to allow the newly required SKU.

#### Use Cases

- Only allow the basic SKU in sandbox and non-production environments to keep costs minimal.
- Ensure the basic SKU is used for lightweight, low traffic Applications that require only basic features.
- Avoid the WAFv2 SKU when WAF capabilities are provided by other services such as Cloudflare.


#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedAppGatewaySKUs | Basic, Standard_v2, WAF_v2 | Basic |

#### Assignment Behaviors

| effect | AllowedAppGatewaySKUs | Behavior |
| :------------- |:------------- |:------------- |
| Audit | WAF_v2 | Marks App Gateways with WAFv2 SKU as non-compliant |
| Deny | WAF_v2 | Denies the creation of Application Gateways with WAFv2 SKU |
| Deny | Standard_v2, WAF_v | Denies all but the basic SKU, forcing only basic SKU usage |
| Disabled | Any | Does nothing. |


---

### AGW02_Application_Gateway_WAF_State_Disabled

The WAF_v2 SKU costs $262.80/month compared to $146/month for Standard_v2. If you're paying for the WAF tier but have the Web Application Firewall disabled, you're wasting money on a feature you're not using.

This policy identifies Application Gateways using the WAF_v2 SKU with the WAF policy disabled, indicating you should either enable WAF or downgrade to Standard_v2 to save costs.

#### Use Cases

- Identify WAF-tier gateways with disabled WAF policies that should be downgraded to Standard_v2.
- Prevent deployment of WAF-tier gateways without active WAF protection.
- Ensure WAF capabilities are actually utilized when paying the premium.
- Audit environments where WAF protection is provided by other services (like Cloudflare).

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks WAF policies with disabled state as non-compliant |
| Deny | Prevents creation/update of disabled WAF policies attached to gateways |
| Disabled | Does nothing. |

---

### AGW03_Application_Gateway_has_no_Backend_Targets

Application Gateways cost $146-$263/month regardless of usage. If a gateway has no backend targets configured, it's not routing any traffic and is pure waste. This often happens when applications are decommissioned but the gateway is left running.

This policy identifies Application Gateways with empty backend pools, indicating they're unused and can likely be deleted to save costs.

#### Use Cases

- Identify orphaned gateways left running after application decommissioning.
- Prevent deployment of gateways without configured backends.
- Audit environments for unused infrastructure that can be deleted.
- Catch configuration errors where backends weren't properly configured.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks Application Gateways with no backend targets as non-compliant |
| Deny | Prevents creation of Application Gateways without backend targets |
| Disabled | Does nothing. |

---

### AGW04_Application_Gateway_Not_Using_Auto-Scaling

Application Gateway v2 SKUs support auto-scaling, which can significantly reduce costs during low-traffic periods. Without auto-scaling, you pay for a fixed number of instances 24/7, even when traffic is minimal.

This policy identifies v2 Application Gateways with fixed capacity instead of auto-scaling, indicating potential cost optimization opportunities.

#### Use Cases

- Ensure all v2 gateways use auto-scaling to reduce costs during off-peak hours.
- Identify gateways with fixed capacity that could benefit from auto-scaling.
- Prevent deployment of v2 gateways without auto-scaling enabled.
- Optimize costs for applications with variable traffic patterns.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks v2 Application Gateways with fixed capacity as non-compliant |
| Deny | Prevents creation of v2 gateways without auto-scaling |
| Disabled | Does nothing. |

---

### AGW05_Application_Gateway_Max_Auto_Scaling_Instances

While auto-scaling reduces costs during low-traffic periods, misconfigured maximum instance counts can lead to runaway costs during traffic spikes. Each instance adds $146/month (Standard_v2) or $262.80/month (WAF_v2).

This policy sets maximum thresholds for both minimum and maximum auto-scaling instance counts to prevent excessive costs while maintaining appropriate capacity.

#### Use Cases

- Limit non-production gateways to 2 instances maximum to control costs.
- Cap production gateway scaling at reasonable levels based on expected traffic.
- Prevent runaway costs from misconfigured auto-scaling or traffic attacks.
- Ensure minimum instance counts don't exceed necessary baseline capacity.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AppGatewayMinInstanceThreshold | Integer | 1 |
| AppGatewayMaxInstanceThreshold | Integer | 2 |

#### Assignment Behaviors

| effect | AppGatewayMinInstanceThreshold | AppGatewayMaxInstanceThreshold | Behavior |
| :------------- |:------------- |:------------- |:------------- |
| Audit | 1 | 2 | Marks gateways with min > 1 or max > 2 as non-compliant |
| Deny | 2 | 5 | Prevents gateways with min > 2 or max > 5 instances |
| Deny | 1 | 10 | Allows up to 10 instances but blocks higher scaling |
| Disabled | Any | Any | Does nothing. |

### AGW06_Application_Gateway_Should_Use_Keyvault_Integration

While this policy is categorized under cost optimization, it primarily addresses security and operational best practices. However, there are indirect cost benefits: Key Vault integration enables automated certificate renewal, preventing outages from expired certificates that could result in lost revenue or emergency remediation costs.

This policy identifies Application Gateways with locally installed certificates instead of Key Vault integration, indicating a security and operational improvement opportunity.

#### Use Cases

- Ensure all gateways use Key Vault for centralized certificate management.
- Prevent manual certificate uploads that require manual renewal processes.
- Avoid outages from expired certificates through automated renewal.
- Improve security posture by removing certificates from gateway configuration.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks Application Gateways with local certificates as non-compliant |
| Deny | Prevents creation/update of gateways with locally installed certificates |
| Disabled | Does nothing. |

---