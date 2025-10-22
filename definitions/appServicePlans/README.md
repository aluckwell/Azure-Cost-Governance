# App Service Plan Policies

### ASP01_Allowed_App_Service_Plan_SKUs

App Service Plans are charged based on their SKU tier, with costs varying dramatically from free tiers to premium isolated plans. Choosing the wrong SKU for your workload can lead to significant overspend, especially in non-production environments.

| SKU Category | Example SKUs | Monthly Cost Range |
| :------------- |:------------- |:------------- |
| Free/Shared | F1, D1 | $0 - $10 |
| Basic | B1, B2, B3 | $13 - $80 |
| Premium v3 | P1v3, P2v3, P3v3 | $146 - $584 |
| Premium Memory Optimized | P1mv3, P2mv3, P3mv3 | $292 - $1,168 |
| Isolated v2 | I1v2, I2v2, I3v2 | $584 - $2,336 |

This policy helps prevent over-provisioning by restricting which [App Service Plan SKUs](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/) can be deployed. During policy assignment, you can define allowed SKUs based on environment and application requirements, blocking premium tiers where they're not needed.

#### Use Cases

- Restrict sandbox and development environments to Free (F1) or Basic (B1) tiers to minimize costs.
- Prevent the use of Isolated SKUs unless specifically required for compliance or network isolation.
- Allow only consumption-based plans (Y1) for Function Apps in non-production to avoid fixed costs.
- Block Premium Memory Optimized SKUs unless applications genuinely require the additional memory.

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | Audit, Deny, Disabled | Audit |
| allowedAppServicePlanSkus | F1, D1, B1, B2, B3, P0v3, P1v3, P2v3, P3v3, P4v3, P1mv3, P2mv3, P3mv3, P4mv3, P5mv3, I1v2, I2v2, I3v2, Y1, FC1, EP1, EP2, EP3, WS1, WS2, WS3 | F1, D1, Y1, B1 |

#### Assignment Behaviors

| effect | allowedAppServicePlanSkus | Behavior |
| :------------- |:------------- |:------------- |
| Audit | F1, B1 | Marks App Service Plans using anything other than F1 or B1 as non-compliant |
| Deny | F1, D1, B1 | Prevents creation of App Service Plans with SKUs other than free and basic tiers |
| Deny | Y1 | Forces consumption-based Function Apps only, blocking all dedicated plans |
| Disabled | Any | Does nothing. |

---

### ASP02_App_Service_Plan_Not_Using_Auto_Scaling

App Service Plans on Standard tier and above support auto-scaling, which dynamically adjusts instance counts based on demand. Without auto-scaling, you pay for a fixed number of instances 24/7, even during low-traffic periods like nights and weekends.

| Tier | Instance Cost | Fixed 3 Instances | Auto-scale 1-3 Instances | Potential Savings |
| :------------- |:------------- |:------------- |:------------- |:------------- |
| Standard (S1) | $73/month | $219/month | $73-219/month | Up to $146/month |
| Premium v3 (P1v3) | $146/month | $438/month | $146-438/month | Up to $292/month |
| Premium v3 (P2v3) | $292/month | $876/month | $292-876/month | Up to $584/month |
| Isolated v2 (I1v2) | $584/month | $1,752/month | $584-1,752/month | Up to $1,168/month |

For applications with variable traffic patterns (e.g., business hours only, weekend dips), auto-scaling can reduce costs by 40-60% by scaling down during quiet periods.

This policy identifies App Service Plans on tiers that support auto-scaling (Standard, Premium, Isolated, ElasticPremium) but don't have autoscale settings configured.

#### Use Cases

- Ensure all production App Service Plans use auto-scaling to optimize costs during off-peak hours.
- Identify plans with fixed instance counts that could benefit from dynamic scaling.
- Flag non-production plans running multiple instances 24/7 unnecessarily.
- Optimize costs for applications with predictable traffic patterns (business hours, weekday/weekend variations).

#### Parameters

| Parameters | Allowed Values | Default Value |
| :------------- |:------------- |:------------- |
| effect | AuditIfNotExists, Disabled | AuditIfNotExists |

#### Assignment Behaviors

| effect | Behavior |
| :------------- |:------------- |
| AuditIfNotExists | Marks App Service Plans without auto-scaling configured as non-compliant |
| Disabled | Does nothing. |

#### Notes

- This policy only applies to tiers that support auto-scaling (Standard, Premium, Isolated, ElasticPremium)
- Basic, Free, and Shared tiers don't support auto-scaling and won't be flagged
- Consumption plans (Y1) for Function Apps scale automatically and won't be flagged
