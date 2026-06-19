````markdown
# AWS Service Quota Increase Centralized Monitoring

Centralized monitoring solution for AWS Service Quota increase requests across a multi-account AWS Organization. Automatically detects quota increase requests in any member account and sends formatted email alerts to a central operations team.

## Architecture

```
Member Accounts (200+)                    Central/Monitoring Account
┌─────────────────────────┐              ┌──────────────────────────────────────┐
│                         │              │                                      │
│  Default Event Bus      │   Forward    │  Custom Event Bus                    │
│  ┌───────────────────┐  │   Raw Event  │  ┌───────────────────┐              │
│  │  EB Rule           │──┼────────────►│  │  EB Rule           │              │
│  │  (match quota      │  │             │  │  (match quota      │              │
│  │   increase)        │  │             │  │   increase)        │              │
│  └───────────────────┘  │             │  └────────┬──────────┘              │
│  ┌───────────────────┐  │             │           │                          │
│  │  IAM Role          │  │             │           ▼                          │
│  │  (EB Forward)      │  │             │  ┌──────────────────┐               │
│  └───────────────────┘  │             │  │    Lambda          │               │
│                         │              │  │  (format + enrich) │               │
└─────────────────────────┘              │  └────────┬──────────┘               │
                                         │           │                          │
                                         │           ▼                          │
                                         │  ┌──────────────────┐               │
                                         │  │    SNS Topic      │               │
                                         │  │  (email alerts)   │               │
                                         │  └──────────────────┘               │
                                         └──────────────────────────────────────┘
```

## Features

- **Cross-account event forwarding** via EventBridge with Organization-level policy (no need to list individual account IDs)
- **Account name resolution** via AWS Organizations API for human-readable alerts
- **Formatted email alerts** with all relevant quota details, tracking IDs, and console deep-links
- **Optional Support Case ID lookup** for customers with Business/Enterprise Support
- **StackSet deployment** for automatic rollout to all current and future member accounts
- **Least-privilege IAM** with scoped permissions

## Prerequisites

- AWS Organizations with CloudFormation StackSets enabled
- CloudTrail enabled in all member accounts (management events)
- AWS CLI configured with appropriate permissions

## Quick Start

1. Deploy the central account stack
2. Confirm SNS email subscription
3. Retrieve the Central Event Bus ARN
4. Deploy member account template via StackSet

See [docs/deployment-guide.md](docs/deployment-guide.md) for detailed step-by-step instructions.

## Repository Structure

```
├── README.md
├── templates/
│   ├── member-account-template.yaml      # Deploy via StackSet to all member accounts
│   └── central-account-template.yaml     # Deploy to the central monitoring account
├── docs/
│   └── deployment-guide.md               # Step-by-step deployment instructions
└── LICENSE
```

## Services Used

- **Amazon EventBridge** — Cross-account event routing
- **AWS Lambda** — Event processing and enrichment
- **Amazon SNS** — Email alert delivery
- **AWS CloudFormation / StackSets** — Infrastructure as Code deployment
- **AWS Organizations** — Account name resolution
- **AWS CloudTrail** — API call event capture
- **AWS IAM** — Least-privilege access control
- **AWS Service Quotas** — Source of quota increase events

## How It Works

1. A user in any member account requests a Service Quota increase (via Console, CLI, or API)
2. CloudTrail captures the `RequestServiceQuotaIncrease` API call
3. EventBridge rule on the default bus matches the event and forwards it to the central account's custom event bus
4. The central EventBridge rule triggers a Lambda function
5. Lambda enriches the event (resolves account name, optionally looks up support case) and formats a readable email
6. SNS delivers the formatted alert to the subscribed email address

## Security

- Cross-account access is scoped to `aws:PrincipalOrgID` — only accounts within your Organization can put events
- Lambda execution role follows least-privilege (only SNS publish, Organizations read, optional Support read)
- No hardcoded account IDs or credentials

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
````
