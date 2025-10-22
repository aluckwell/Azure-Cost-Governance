# Azure Firewall Policies

### FW01_Allowed_Azure_Firewall_SKUs

Azure Firewall is a managed network security service with significant cost differences between SKUs. The Premium tier costs double the Standard tier but includes advanced threat protection features that may not be necessary for all environments.

| SKU | Hourly Cost | Monthly Cost | Key Features |
| :------------- |:------------- |:------------- |:------------- |
| Basic | $0.125 | $91.25 | Small deployments, 250 Mbps throughput |
| Standard | $1.25 | $912.50 | Standard features, threat intelligence |
| Premium | $2.50 | $1,825 | IDPS, TLS inspection, URL filtering |

This policy helps control costs by restricting which [Azure Firewall SKUs](https://azure.microsoft.com/en-us/pricing/details/azure-firewall/) can be deployed. Define allowed SKUs based on security requirements and environment type.

#### Use Cases

- Use Basic SKU for development and test environments with simple networking needs.
- Restrict non-production to Standard SKU to avoid premium feature costs.
- Reserve Premium SKU for production environments requiring advanced threat protection.
- Block Premium SKU when IDPS and TLS inspection are provided by other security tools.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| AllowedFirewallSKUs | Basic, Standard, Premium | Basic |

#### Assignment Behaviors

| effect | AllowedFirewallSKUs | Behavior |
| :------------- |:------------- |:------------- |
| Audit | Basic | Marks firewalls using Standard or Premium as non-compliant |
| Deny | Basic, Standard | Prevents creation of Premium firewalls |
| Deny | Standard | Forces Standard SKU only, blocking Basic and Premium |
| Disabled | Any | Does nothing. |

---

### FW02_Premium_Firewall_Has_Premium_Features_Disabled

Azure Firewall Premium costs $1,825/month compared to $912.50 for Standard. If you're paying for Premium but have premium features like IDPS (Intrusion Detection and Prevention) or TLS inspection disabled, you're wasting money on capabilities you're not using.

Premium features include:
- **IDPS**: Intrusion Detection and Prevention System for advanced threat protection
- **TLS Inspection**: Decrypt and inspect encrypted traffic for threats
- **URL Filtering**: Advanced web filtering with category-based blocking
- **Web Categories**: Granular control over web access

This policy identifies Premium firewalls with premium features disabled, indicating potential cost optimization by downgrading to Standard SKU.

#### Use Cases

- Identify Premium firewalls where IDPS is turned off, indicating Standard SKU would suffice.
- Flag firewalls without TLS inspection configured, suggesting premium tier isn't needed.
- Audit Premium deployments to ensure premium features justify the 2x cost increase.
- Prevent deployment of Premium firewalls without premium features enabled.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks Premium firewall policies with disabled premium features as non-compliant |
| Deny | Prevents creation/update of Premium policies without premium features enabled |
| Disabled | Does nothing. |

---

### FW03_Firewall_Policy_Assigned_To_Multiple_Firewalls

Azure Firewall Policies can be shared across multiple firewalls. While the first firewall using a policy incurs no additional policy cost, each additional firewall using the same policy incurs a charge. This can add up quickly in hub-and-spoke architectures.

| Firewalls Using Policy | Additional Monthly Cost |
| :------------- |:------------- |
| 1 | $0 |
| 2 | ~$100 |
| 5 | ~$400 |
| 10 | ~$900 |

This policy highlights when a firewall policy is assigned to multiple firewalls, helping you understand the additional costs and consider whether separate policies or policy inheritance would be more cost-effective.

#### Use Cases

- Identify policies shared across many firewalls that are driving up costs.
- Audit hub-and-spoke architectures to understand policy sharing costs.
- Prevent accidental assignment of policies to multiple firewalls in non-production.
- Evaluate whether policy inheritance or separate policies would reduce costs.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks firewall policies assigned to more than 1 firewall as non-compliant |
| Deny | Prevents assignment of policies to multiple firewalls |
| Disabled | Does nothing. |

---

### FW04_Firewall_Policy_Analytics_Is_Enabled

Azure Firewall Policy Analytics provides insights into firewall traffic patterns and rule effectiveness. However, this feature costs approximately £199 ($250) per month per firewall. If you're not actively using the analytics data, this is pure waste.

Policy Analytics includes:
- Traffic flow visualization
- Rule hit count analysis
- Top talkers and applications
- Geo-location mapping

This policy identifies firewalls with Policy Analytics enabled, helping you evaluate whether the insights justify the additional monthly cost.

#### Use Cases

- Disable Policy Analytics in non-production environments where traffic analysis isn't needed.
- Identify firewalls with Analytics enabled but not being actively monitored.
- Enable Analytics temporarily for troubleshooting, then disable to avoid ongoing costs.
- Reserve Analytics for production firewalls where traffic insights provide value.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| Audit | Marks firewall policies with Analytics enabled as non-compliant |
| Deny | Prevents enabling Policy Analytics on firewall policies |
| Disabled | Does nothing. |
