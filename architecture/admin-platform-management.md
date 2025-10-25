# Admin Platform & Management

## Service Dependencies

**Implements**: Resume-Admin-API (backend), Resume-Admin (web), Resume-Mobile-Admin (mobile)
**Depends On**: All platform services (monitoring/management)
**Used By**: Admin team (5-10 people), moderators, support

## Role-Based Access Control (RBAC)

**Admin Roles**:

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Super Admin** | Full system access, config changes | Technical leads |
| **Admin** | User/content management, no system config | Operations team |
| **Moderator** | Content moderation only | Moderation team |
| **Support** | Read-only + ticket management | Customer support |
| **Billing Admin** | Payment/subscription management | Finance team |

**Permission Granularity**:
- `users.read`, `users.write`, `users.delete`
- `content.read`, `content.moderate`, `content.delete`
- `payments.read`, `payments.write`, `payments.refund`
- `system.config`, `system.logs`, `system.metrics`
- `admin.manage` (manage other admins)

## User Management Features

**CRUD Operations**:
```
User Dashboard:
├─ Search (email, name, ID, subscription)
├─ Advanced Filters (status, plan, registration date, activity)
├─ Bulk Actions (export, suspend, email)
├─ User Details View (complete profile + activity)
└─ Edit User (admin override for all fields)
```

**Account Actions**:
- **Suspend**: Temporary account freeze (1-30 days)
- **Ban**: Permanent account termination with reason
- **Force Password Reset**: Security measure
- **Clear Sessions**: Logout user from all devices
- **Impersonate**: Support mode (with audit trail)

## Banning System

**Ban Types**:
- **Temporary**: Auto-expires after duration
- **Permanent**: Manual admin removal only
- **IP-Based**: Block specific IP addresses
- **Multi-Account**: Ban linked accounts

**Ban Workflow**:
```
Warning → Temporary Suspension → Permanent Ban
  (1-2)         (3-7 days)          (Forever)
```

**Ban Reasons** (dropdown):
- Payment Fraud
- Content Abuse
- ToS Violation
- Spam/Scam
- Legal Requirement
- User Request

**Appeal System**:
- 30-day appeal window
- Submit evidence
- Admin review queue
- Accept/Reject decision
- Strike reduction on acceptance

## Payment Administration

**Subscription Management**:
```
Admin Actions:
├─ View all subscriptions (filter by status, plan)
├─ Manual subscription creation
├─ Force plan change (with/without proration)
├─ Extend trial period
├─ Apply discount/coupon
├─ Cancel subscription (immediate or end of period)
└─ Grace period extension
```

**Payment Operations**:
- **Refund Processing**: Full/partial refunds via Stripe
- **Failed Payment Retry**: Manual retry with alternate payment method
- **Payment Method Management**: View/remove user payment methods
- **Dispute Management**: Handle chargebacks and disputes

## Invoice Management

**Invoice Operations**:
```
Invoice Dashboard:
├─ List all invoices (filter by user, status, date range)
├─ Invoice details (line items, payments, attempts)
├─ Manual invoice creation (admin charges)
├─ Credit note generation (refunds, adjustments)
├─ Void/cancel invoice
├─ Resend invoice email
├─ Regenerate PDF
└─ Bulk export (CSV, PDF archive)
```

**Billing Reports**:
- **Revenue Reports**: MRR, ARR, revenue by plan
- **Churn Analysis**: Cancellation reasons, churn rate
- **Collection Reports**: Payment success rate, failed payment recovery
- **Tax Reports**: Sales tax by jurisdiction

## Database Schema

**Admin Tables** (14 total):
- `admin_users` - Admin accounts with roles
- `admin_roles` - Role definitions
- `admin_permissions` - Granular permissions
- `admin_audit_log` - All admin actions logged
- `user_bans` - Active bans with expiry
- `ban_appeals` - User contest submissions
- `admin_invoices` - Manually created invoices
- `invoice_adjustments` - Credits, corrections
- `user_notes` - Internal admin notes
- `support_tickets` - Ticket system
- `system_config` - Feature flags, settings
- `moderation_actions` - Content moderation log

## API Endpoints (Resume-Admin-API)

**User Management** (45+ endpoints):
```
GET    /api/admin/users                List/search users
GET    /api/admin/users/:id            Get user details
PATCH  /api/admin/users/:id            Update user
DELETE /api/admin/users/:id            Delete user
POST   /api/admin/users/:id/suspend    Suspend account
POST   /api/admin/users/:id/ban        Ban account
POST   /api/admin/users/:id/unban      Unban account
POST   /api/admin/users/:id/impersonate Impersonate user
```

**Payment Administration**:
```
GET    /api/admin/subscriptions        List subscriptions
POST   /api/admin/subscriptions        Create subscription
PATCH  /api/admin/subscriptions/:id    Update subscription
POST   /api/admin/refunds              Process refund
POST   /api/admin/payments/retry       Retry failed payment
```

**Moderation**:
```
GET    /api/admin/moderation/queue     Get moderation queue
POST   /api/admin/moderation/review    Review content
GET    /api/admin/appeals              List appeals
POST   /api/admin/appeals/:id/decide   Decide appeal
```

## Admin Dashboard UI

**Key Screens**:
- **Dashboard**: Metrics overview (users, revenue, health)
- **User Management**: Search, filter, bulk actions
- **Content Moderation**: Review queue with preview
- **Payment Admin**: Subscriptions, invoices, refunds
- **Bans & Appeals**: Ban management, appeal review
- **Analytics**: Revenue, churn, user growth charts
- **System Config**: Feature flags, rate limits, settings
- **Audit Log**: Complete admin action history

## Security Controls

**Admin Security**:
- ✅ JWT with refresh token rotation
- ✅ MFA required for sensitive operations (bans, refunds)
- ✅ IP allowlisting for Super Admin role
- ✅ Session timeout (30 min inactivity)
- ✅ All actions logged to audit trail
- ✅ Rate limiting (100 req/min per admin)

**Audit Logging**:
```typescript
// Every admin action logged
{
  timestamp: '2025-01-25T00:48:12Z',
  admin_id: 'admin-123',
  action: 'user.ban',
  resource_type: 'user',
  resource_id: 'user-456',
  details: { reason: 'spam', duration: 'permanent' },
  ip_address: '192.0.2.1',
  user_agent: 'Mozilla/5.0...'
}
```

## Implementation Dependencies

**Foundation**:
- RBAC system with roles and permissions
- Admin authentication (JWT + MFA)
- Audit logging infrastructure
- Admin UI framework

**Core Features**:
- User management CRUD
- Banning system with workflows
- Payment administration
- Invoice management

**Advanced**:
- Analytics dashboards
- Bulk operations
- Advanced reporting
- System configuration panel

