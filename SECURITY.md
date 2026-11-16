# Security Policy

## Disclaimer

This repository contains sample code for demonstration and learning. It is **not production-ready**.
Review and test it against your own security, compliance and operational requirements before any
production use. The authors and AWS make no warranty regarding its fitness for production workloads.

## Reporting Vulnerabilities

If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security
via our [vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/).
Please do **not** create a public GitHub issue.

## AWS Services Used

Amazon EventBridge, Amazon SNS (email subscription), AWS KMS, AWS CloudTrail (source of the KMS API events)
and AWS CloudFormation. There is no Lambda function and no application code.

## Prerequisites and Permissions

Deploying the stack requires permissions to create a KMS key, an SNS topic with a topic policy and
subscription, and three EventBridge rules. The stack does not create IAM roles, so it does not need the
`CAPABILITY_IAM` capability.

- The **topic policy** lets only the three rules in this stack publish (`sns:Publish` with `aws:SourceArn` set
  to the rule ARNs and `aws:SourceAccount` set to the deploying account). It denies any publish that does not use TLS.
- The **topic key policy** lets the account administer the key and lets Amazon EventBridge rules in this
  account and Region call `kms:GenerateDataKey*` and `kms:Decrypt` (`aws:SourceAccount` and an `aws:SourceArn`
  pattern of `rule/*`). The key has automatic rotation turned on.

## Known Security Considerations

The following items were reviewed and accepted for this sample. Revisit them before any production use.

| Item | Why it is acceptable here | Production recommendation |
|------|---------------------------|---------------------------|
| `kms:*` to the account root in the key policy | Standard default that keeps the key manageable if other administrators are removed | Scope key administration to a named deployment role |
| The key policy allows any EventBridge rule in the account (`rule/*`), not only the three rules | The key policy cannot reference the rules without a circular dependency in CloudFormation. The topic policy limits who can publish to the three rules | Create the rules first and reference their ARNs in the key policy, or use a fixed rule name prefix |
| One email address and no escalation | Keeps the sample small | Subscribe a distribution list or an incident tool and add a second endpoint |
| Alerts depend on the topic key | Required for an encrypted topic | Alarm on the `NumberOfNotificationsFailed` metric of the topic, and keep the key out of reach of routine administrators |
| Stack and topic key are deleted with the stack | Clean-up is expected for a sample | Set `DeletionPolicy: Retain` on the key if you need to read old messages |
| Alerts use CloudTrail management events | Works without extra setup in most accounts | Use an organization trail and a central event bus for many accounts |

## Operational Notes

- The email contains the account ID, Region, caller ARN and key ID. Use a distribution list that is limited to people who may see those details.
- The caller ARN is the IAM role session or user that made the call. It identifies who disabled or scheduled
  the deletion of the key.
- A failed request (for example, access denied) also produces an email. The last line of the message shows the error code.
- Anyone with `sns:Subscribe` permission can add another subscriber to the topic. Restrict it with IAM policies.
- Deploy the stack in every Region where you use KMS keys. For many accounts, use AWS CloudFormation StackSets.

## Resource Cleanup

Delete the CloudFormation stack. The topic key goes into a pending deletion period (30 days by default).
Delete or cancel any test KMS keys you created.

## Dependencies

There are no third-party packages. The template uses only AWS-managed services.
