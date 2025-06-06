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
