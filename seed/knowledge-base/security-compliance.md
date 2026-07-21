# Flowdesk — Security & Compliance

## Data storage & residency
- Primary hosting: AWS. Customers choose region at signup: **US (us-east-1)** or **EU (eu-central-1, Frankfurt)**. Data does not leave the chosen region.
- Backups: encrypted, same-region, 30-day retention.

## Encryption
- In transit: TLS 1.2+.
- At rest: AES-256 (database, file storage, and backups).

## Compliance
- **SOC 2 Type II** certified (report available under NDA).
- **GDPR:** DPA available for all customers; we act as processor.
- **HIPAA:** we sign **BAAs for Team plan customers**. PHI must be stored only in designated encrypted fields/attachments per the BAA terms.

## Access & authentication
- SSO/SAML 2.0 (Okta, Azure AD, Google) on Team plan.
- 2FA available on all plans.
- Role-based access control; full audit logs on Team plan.

## Security questionnaires
Written answers to security questionnaires are provided for prospects evaluating 20+ seats — typical turnaround 3 business days.

## Incident reporting
security@flowdesk.example — 24h acknowledgment SLA. Status page: status.flowdesk.example.
