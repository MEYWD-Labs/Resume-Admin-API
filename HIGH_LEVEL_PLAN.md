# High-Level Development Plan - Resume-Admin-API

**Service Type**: Admin Backend API (Cloudflare Workers)
**Build Approach**: AI-Automated Development
**Priority**: MEDIUM - Required for platform management

---

## Dependency Tree Overview

```
Resume-Admin-API (Admin Backend Service)
├── DEPENDS ON:
│   ├── Resume-SSO (admin authentication/authorization with MFA)
│   └── Resume-API Database (shared D1 database)
│
├── BLOCKS:
│   ├── Resume-Admin (admin dashboard - requires all APIs)
│   ├── Resume-Mobile-Admin (mobile admin - requires all APIs)
│   └── Resume-Processor (provides analytics processing, cleanup jobs)

Services that DEPEND on Resume-Admin-API:
├── Resume-Admin (MVP-1: Dashboard functionality)
├── Resume-Mobile-Admin (MVP-1: Mobile admin functionality)
└── Resume-Processor (MVP-3: Analytics jobs, cleanup tasks)

MVP Breakdown:
├── MVP-1: Admin Foundation ⟶ Blocks: Resume-Admin, Resume-Mobile-Admin dashboards
│   ├── Admin Authentication & Authorization
│   ├── Dashboard Analytics APIs
│   ├── User Management APIs
│   └── Template Management APIs
│
├── MVP-2: Financial Operations & Support ⟶ Blocks: Payment features in Resume-Admin, Resume-Mobile-Admin
│   ├── Subscription Management APIs
│   ├── Revenue Analytics
│   ├── Coupon Management
│   └── Support Ticket System
│
├── MVP-3: Content & System Management ⟶ Blocks: System management in Resume-Admin, Resume-Processor jobs
│   ├── Content Management
│   ├── Website Management APIs
│   ├── Feature Flag Management
│   └── System Health Monitoring
│
└── MVP-4: Advanced Analytics & Automation ⟶ Blocks: Advanced features in Resume-Admin, Resume-Processor
    ├── Custom Report Builder
    ├── A/B Testing Analytics
    ├── AI Usage Analytics
    └── Automated Workflows
```

---

## MVP-1: Admin Foundation (Foundation)

**Status**: MUST BUILD AFTER Resume-SSO admin roles are ready
**Dependencies**: Resume-SSO (MVP-4.3: MFA for admin access), Resume-API (MVP-1.1: Database)
**Blocks**: Resume-Admin, Resume-Mobile-Admin (all admin features)

### Epic 1.1: Admin Authentication & Authorization
**What**: Validate admin users and enforce role-based access
**Complexity**: Large
**Dependencies**: Resume-SSO (admin role management)

**Tasks**:
- [ ] Admin JWT validation middleware
- [ ] Admin role extraction from JWT (super_admin, admin, support)
- [ ] Permission-based route access control
- [ ] Admin action logging to D1
- [ ] Create admin_users table in D1
- [ ] Create audit_log table in D1
- [ ] `GET /admin/admins` - List all admins
- [ ] `POST /admin/admins` - Create new admin (Super Admin only)
- [ ] `PUT /admin/admins/:id` - Update admin role (Super Admin only)
- [ ] `DELETE /admin/admins/:id` - Remove admin access (Super Admin only)

**Acceptance Criteria**:
- [ ] Only users with admin roles can access endpoints
- [ ] Role-based permissions enforced (Super Admin, Admin, Support)
- [ ] All admin actions logged to audit_log
- [ ] Super Admins can manage other admins
- [ ] Non-admin users rejected with 403

---

### Epic 1.2: Dashboard Analytics APIs
**What**: Core metrics for admin dashboard
**Complexity**: Large
**Dependencies**: Epic 1.1 (Admin Auth), Resume-API (MVP-1: Database)

**Tasks**:
- [ ] `GET /admin/metrics/overview` - Key platform metrics
  - [ ] Total users count
  - [ ] Active users (DAU/MAU)
  - [ ] Total resumes count
  - [ ] Total exports count
  - [ ] Conversion rate (free → paid)
- [ ] `GET /admin/metrics/users` - User growth time-series
- [ ] `GET /admin/metrics/resumes` - Resume creation time-series
- [ ] `GET /admin/metrics/exports` - Export time-series
- [ ] Granularity support (hourly, daily, weekly, monthly)
- [ ] Use Cloudflare Analytics Engine for aggregations
- [ ] Cache aggregated metrics in KV (TTL: 5-15 minutes)
- [ ] Query optimization for large datasets

**Acceptance Criteria**:
- [ ] Overview metrics accurate and real-time
- [ ] Time-series data returned for specified granularity
- [ ] Metrics cached for performance
- [ ] Response time < 500ms for cached data
- [ ] Handles large datasets efficiently

---

### Epic 1.3: User Management APIs
**What**: CRUD and management operations for users
**Complexity**: Large
**Dependencies**: Epic 1.1 (Admin Auth), Resume-API (MVP-1: Database)

**Tasks**:
- [ ] `GET /admin/users` - List users (paginated, 50 per page)
- [ ] `GET /admin/users/:id` - Get user details
- [ ] `PUT /admin/users/:id` - Update user
- [ ] `DELETE /admin/users/:id` - Soft delete user
- [ ] `POST /admin/users/:id/suspend` - Suspend account
- [ ] `POST /admin/users/:id/unsuspend` - Unsuspend account
- [ ] User search functionality (email, name, ID)
- [ ] Filter by subscription tier (free, pro, premium)
- [ ] Filter by status (active, suspended, deleted)
- [ ] Filter by date range (registration date)
- [ ] Sort by various fields (created_at, email, subscription)
- [ ] `GET /admin/users/:id/resumes` - User's resumes
- [ ] `GET /admin/users/:id/activity` - Activity log
- [ ] `GET /admin/users/:id/exports` - Export history
- [ ] `POST /admin/users/:id/reset-password` - Send reset email
- [ ] `POST /admin/users/:id/verify-email` - Manually verify email
- [ ] `POST /admin/users/:id/impersonate` - Generate impersonation token

**Acceptance Criteria**:
- [ ] Pagination works correctly
- [ ] Search and filters functional
- [ ] User details include subscription and activity
- [ ] Suspend/unsuspend affects user access
- [ ] Soft delete preserves data
- [ ] Impersonation token allows admin to view as user
- [ ] All actions logged to audit_log

---

### Epic 1.4: Template Management APIs
**What**: Manage resume templates from admin panel
**Complexity**: Medium
**Dependencies**: Epic 1.1 (Admin Auth), Resume-API (MVP-1.4: Templates)

**Tasks**:
- [ ] `GET /admin/templates` - List all templates
- [ ] `GET /admin/templates/:id` - Get template details
- [ ] `POST /admin/templates` - Create new template
- [ ] `PUT /admin/templates/:id` - Update template
- [ ] `DELETE /admin/templates/:id` - Delete template
- [ ] `POST /admin/templates/:id/feature` - Mark as featured
- [ ] Store HTML templates in D1
- [ ] Store preview images in R2
- [ ] Version control for templates (save old versions)
- [ ] `GET /admin/templates/:id/stats` - Usage statistics
- [ ] Track template selections by users
- [ ] Track resume completions per template

**Acceptance Criteria**:
- [ ] Templates created with HTML and CSS
- [ ] Preview images uploaded to R2
- [ ] Version history maintained
- [ ] Usage statistics accurate
- [ ] Featured templates highlighted in Resume-UI
- [ ] Only Admins and Super Admins can manage templates

---

### Epic 1.5: Audit Logging System
**What**: Comprehensive logging of all admin actions
**Complexity**: Medium
**Dependencies**: Epic 1.1 (Admin Auth)

**Tasks**:
- [ ] Log all admin actions to audit_log table
- [ ] Fields: admin_id, action, resource_type, resource_id, changes (JSON), IP, timestamp
- [ ] `GET /admin/audit-log` - Retrieve audit logs (paginated)
- [ ] Filter by admin_id
- [ ] Filter by action type
- [ ] Filter by resource type
- [ ] Filter by date range
- [ ] Export audit log to CSV
- [ ] Audit log retention policy (1 year)

**Acceptance Criteria**:
- [ ] All admin actions logged
- [ ] Logs include full context (who, what, when, where, changes)
- [ ] Filters functional
- [ ] Audit logs tamper-proof (append-only)
- [ ] CSV export works
- [ ] Old logs auto-archived after 1 year

---

## MVP-2: Financial Operations & Support (Monetization)

**Status**: Required for revenue management
**Dependencies**: MVP-1, Resume-API (MVP-2: Stripe Integration)
**Blocks**: Payment and support management in Resume-Admin

### Epic 2.1: Subscription Management APIs
**What**: Manage user subscriptions and billing
**Complexity**: Large
**Dependencies**: Epic 1.1 (Admin Auth), Resume-API (MVP-2.1: Stripe)

**Tasks**:
- [ ] `GET /admin/subscriptions` - List all subscriptions (paginated)
- [ ] `GET /admin/subscriptions/:id` - Subscription details
- [ ] `POST /admin/subscriptions/:id/cancel` - Cancel subscription (admin action)
- [ ] `POST /admin/subscriptions/:id/refund` - Process refund via Stripe
- [ ] Subscription status tracking (active, canceled, expired, past_due)
- [ ] Filter subscriptions by status, tier, date
- [ ] Handle Stripe webhook events (update D1)
- [ ] Send notifications on subscription events

**Acceptance Criteria**:
- [ ] Subscription list accurate
- [ ] Cancel subscription works (Stripe API)
- [ ] Refunds processed correctly
- [ ] Webhooks update D1 in real-time
- [ ] Only Super Admins can refund

---

### Epic 2.2: Revenue Analytics
**What**: Financial metrics and KPIs
**Complexity**: Large
**Dependencies**: Epic 2.1 (Subscriptions)

**Tasks**:
- [ ] `GET /admin/metrics/revenue` - Revenue metrics
  - [ ] MRR (Monthly Recurring Revenue)
  - [ ] ARR (Annual Recurring Revenue)
  - [ ] Churn rate (monthly)
  - [ ] ARPU (Average Revenue Per User)
  - [ ] LTV (Lifetime Value estimate)
- [ ] Revenue time-series (daily, monthly)
- [ ] Subscription tier breakdown (revenue by tier)
- [ ] New vs returning revenue
- [ ] Failed payment tracking
- [ ] Refund tracking and impact

**Acceptance Criteria**:
- [ ] MRR and ARR calculated accurately
- [ ] Churn rate computed correctly
- [ ] ARPU and LTV estimates reasonable
- [ ] Time-series data visualizable
- [ ] Metrics updated daily
- [ ] Only Super Admins and Admins can view

---

### Epic 2.3: Transaction Management
**What**: View and manage payment transactions
**Complexity**: Medium
**Dependencies**: Epic 2.1 (Subscriptions)

**Tasks**:
- [ ] `GET /admin/transactions` - List transactions (paginated)
- [ ] `GET /admin/transactions/:id` - Transaction details
- [ ] `POST /admin/transactions/:id/refund` - Refund transaction
- [ ] Filter transactions by status (succeeded, failed, refunded)
- [ ] Filter by date range
- [ ] Filter by amount range
- [ ] Failed payment handling (retry logic)
- [ ] Export transactions to CSV

**Acceptance Criteria**:
- [ ] Transaction list complete and accurate
- [ ] Refunds processed via Stripe
- [ ] Filters functional
- [ ] Failed payments tracked
- [ ] CSV export works
- [ ] Only Super Admins can refund

---

### Epic 2.4: Coupon Management
**What**: Create and manage discount coupons
**Complexity**: Medium
**Dependencies**: Epic 2.1 (Subscriptions), Resume-API (Stripe Integration)

**Tasks**:
- [ ] Create coupons table in D1
- [ ] Create coupon_redemptions table in D1
- [ ] `POST /admin/coupons` - Create coupon
- [ ] `GET /admin/coupons` - List coupons
- [ ] `GET /admin/coupons/:id` - Coupon details
- [ ] `PUT /admin/coupons/:id` - Update coupon
- [ ] `DELETE /admin/coupons/:id` - Delete coupon
- [ ] Coupon types: percentage, fixed amount
- [ ] Max uses limit
- [ ] Expiration date
- [ ] Applicable plans (Pro, Premium, or both)
- [ ] `GET /admin/coupons/:id/usage` - Usage statistics
- [ ] Track redemptions
- [ ] Calculate revenue impact

**Acceptance Criteria**:
- [ ] Coupons created with all options
- [ ] Usage limits enforced
- [ ] Expired coupons auto-disabled
- [ ] Usage statistics accurate
- [ ] Redemptions tracked per user
- [ ] Only Admins and Super Admins can create

---

### Epic 2.5: Support Ticket System
**What**: Manage customer support tickets
**Complexity**: Large
**Dependencies**: Epic 1.3 (User Management)

**Tasks**:
- [ ] Create support_tickets table in D1
- [ ] Create ticket_messages table in D1
- [ ] `GET /admin/tickets` - List support tickets (paginated)
- [ ] `GET /admin/tickets/:id` - Ticket details with message history
- [ ] `POST /admin/tickets` - Create ticket manually
- [ ] `PUT /admin/tickets/:id` - Update ticket (assign, change status/priority)
- [ ] `POST /admin/tickets/:id/reply` - Reply to ticket (sends email to user)
- [ ] `POST /admin/tickets/:id/close` - Close ticket
- [ ] Ticket statuses: open, in_progress, resolved, closed
- [ ] Priority levels: low, medium, high, urgent
- [ ] Assign tickets to admin users
- [ ] Filter by status, priority, assigned_to
- [ ] Search tickets by user email or subject

**Acceptance Criteria**:
- [ ] Tickets created and assigned
- [ ] Admins can reply (emails sent to users)
- [ ] Status and priority updates work
- [ ] Filters and search functional
- [ ] Support role has full access to tickets
- [ ] Email notifications sent to users on replies

---

## MVP-3: Content & System Management (Operations)

**Status**: Required for system operations
**Dependencies**: MVP-2
**Blocks**: Advanced admin operations

### Epic 3.1: Content Management
**What**: Manage announcements and campaigns
**Complexity**: Medium
**Dependencies**: Epic 1.1 (Admin Auth)

**Tasks**:
- [ ] Announcement system:
  - [ ] `POST /admin/announcements` - Create announcement
  - [ ] `GET /admin/announcements` - List announcements
  - [ ] `PUT /admin/announcements/:id` - Update announcement
  - [ ] `DELETE /admin/announcements/:id` - Delete announcement
  - [ ] `POST /admin/announcements/:id/publish` - Publish announcement
- [ ] Email campaigns:
  - [ ] `POST /admin/campaigns` - Create campaign
  - [ ] `GET /admin/campaigns` - List campaigns
  - [ ] `POST /admin/campaigns/:id/send` - Send campaign
  - [ ] User segmentation (by tier, activity, signup date)
  - [ ] Schedule campaign sending
  - [ ] Track open rates and clicks
- [ ] Content library:
  - [ ] `GET /admin/content/examples` - List example content
  - [ ] `POST /admin/content/examples` - Add example
  - [ ] `PUT /admin/content/examples/:id` - Update example

**Acceptance Criteria**:
- [ ] Announcements created and published
- [ ] Email campaigns sent to segmented users
- [ ] Open and click tracking works
- [ ] Example content managed
- [ ] Only Admins and Super Admins can manage content

---

### Epic 3.2: Website Management APIs
**What**: Manage user personal websites
**Complexity**: Medium
**Dependencies**: Epic 1.3 (User Management), Resume-API (MVP-4: Website Generation)

**Tasks**:
- [ ] `GET /admin/websites` - List all user websites (paginated)
- [ ] `GET /admin/websites/:id` - Website details
- [ ] `POST /admin/websites/:id/suspend` - Suspend website (abuse/policy violation)
- [ ] `GET /admin/websites/stats` - Aggregate statistics (total, active, suspended)
- [ ] `GET /admin/domains` - List custom domains
- [ ] `POST /admin/domains/:id/verify` - Verify domain (admin manual check)
- [ ] `POST /admin/domains/:id/approve` - Approve domain
- [ ] DNS management via Cloudflare API

**Acceptance Criteria**:
- [ ] Website list shows all user sites
- [ ] Suspended websites inaccessible
- [ ] Custom domains verified and approved manually
- [ ] DNS management functional
- [ ] Only Admins and Super Admins can manage

---

### Epic 3.3: Feature Flag Management
**What**: Control feature rollouts and A/B tests
**Complexity**: Large
**Dependencies**: Epic 1.1 (Admin Auth)

**Tasks**:
- [ ] Create feature_flags table in D1
- [ ] `GET /admin/flags` - List all feature flags
- [ ] `POST /admin/flags` - Create feature flag
- [ ] `PUT /admin/flags/:id` - Update flag
- [ ] `DELETE /admin/flags/:id` - Delete flag
- [ ] `POST /admin/flags/:id/toggle` - Enable/disable flag
- [ ] Gradual rollout support (percentage-based)
- [ ] User segment targeting (by tier, location, etc.)
- [ ] A/B test configuration
- [ ] Flag evaluation API (called by other services)
- [ ] `GET /admin/flags/:id/usage` - Usage statistics
- [ ] Feature adoption metrics
- [ ] A/B test results

**Acceptance Criteria**:
- [ ] Feature flags created and toggled
- [ ] Gradual rollout works (percentage)
- [ ] User targeting rules enforced
- [ ] A/B tests configured
- [ ] Usage statistics tracked
- [ ] Only Super Admins can manage flags

---

### Epic 3.4: System Health Monitoring
**What**: Monitor system health and performance
**Complexity**: Medium
**Dependencies**: Epic 1.1 (Admin Auth)

**Tasks**:
- [ ] `GET /admin/health` - Overall system health status
- [ ] `GET /admin/health/workers` - Worker status (CPU, memory, requests)
- [ ] `GET /admin/health/database` - D1 status (size, connections, query time)
- [ ] `GET /admin/health/storage` - R2 usage (storage used, objects count)
- [ ] `GET /admin/health/queues` - Queue backlog (pending, processing, failed)
- [ ] `GET /admin/metrics/performance` - API performance
  - [ ] Response time percentiles (p50, p95, p99)
  - [ ] Error rates by endpoint
  - [ ] Request volume
- [ ] `GET /admin/errors` - Recent errors (last 24 hours)
- [ ] `GET /admin/errors/:id` - Error details
- [ ] Error frequency and patterns
- [ ] Integration with Sentry for error tracking

**Acceptance Criteria**:
- [ ] System health dashboard shows all metrics
- [ ] Performance metrics accurate
- [ ] Errors tracked and grouped
- [ ] Queue backlog monitored
- [ ] Alerts triggered for issues
- [ ] Only Super Admins and Admins can view

---

## MVP-4: Advanced Analytics & Automation (Advanced Features)

**Status**: Advanced admin capabilities
**Dependencies**: MVP-3
**Blocks**: Advanced reporting and automation

### Epic 4.1: Custom Report Builder
**What**: Build custom analytics reports
**Complexity**: Large
**Dependencies**: Epic 1.2 (Dashboard Analytics)

**Tasks**:
- [ ] `POST /admin/reports/custom` - Generate custom report
- [ ] Flexible metric selection (choose from available metrics)
- [ ] Custom date ranges and filters
- [ ] Grouping and aggregation options
- [ ] Export to CSV/PDF
- [ ] `POST /admin/reports/schedule` - Schedule recurring report
- [ ] Daily/weekly/monthly report schedules
- [ ] Email delivery to admin users
- [ ] Store generated reports in R2

**Acceptance Criteria**:
- [ ] Custom reports generated with selected metrics
- [ ] Filters and date ranges work
- [ ] CSV and PDF exports functional
- [ ] Scheduled reports sent via email
- [ ] Reports stored in R2 for later access
- [ ] Only Admins and Super Admins can create

---

### Epic 4.2: A/B Testing Analytics
**What**: Analyze A/B test results
**Complexity**: Medium
**Dependencies**: Epic 3.3 (Feature Flags)

**Tasks**:
- [ ] `GET /admin/experiments` - List A/B tests
- [ ] `GET /admin/experiments/:id/results` - Test results
- [ ] Statistical significance calculation
- [ ] Variant performance comparison (conversion rates, engagement)
- [ ] Confidence intervals
- [ ] Sample size recommendations
- [ ] Test duration tracking

**Acceptance Criteria**:
- [ ] A/B test results displayed
- [ ] Statistical significance calculated
- [ ] Variant performance compared
- [ ] Recommendations for winning variant
- [ ] Only Super Admins can view

---

### Epic 4.3: AI Usage Analytics
**What**: Track AI feature usage and costs
**Complexity**: Medium
**Dependencies**: Epic 1.2 (Analytics), Resume-API (MVP-5: AI Features)

**Tasks**:
- [ ] `GET /admin/ai/usage` - AI feature usage metrics
- [ ] AI credits consumed per user
- [ ] Most popular AI features (suggestions, analysis, rewriting)
- [ ] Cost analysis (Cloudflare AI costs)
- [ ] Usage trends over time
- [ ] Per-tier usage breakdown

**Acceptance Criteria**:
- [ ] AI usage tracked accurately
- [ ] Costs calculated (if applicable)
- [ ] Popular features identified
- [ ] Trends visualizable
- [ ] Only Super Admins and Admins can view

---

### Epic 4.4: Automated Workflows
**What**: Automate repetitive admin tasks
**Complexity**: Large
**Dependencies**: Epic 2.5 (Support), Epic 3.4 (System Health)

**Tasks**:
- [ ] Automated actions:
  - [ ] Auto-suspend abusive accounts (based on rules)
  - [ ] Auto-send payment reminders (before renewal)
  - [ ] Auto-expire coupons (at expiration date)
  - [ ] Auto-archive resolved tickets (after 30 days)
  - [ ] Auto-cleanup old sessions and tokens
- [ ] Workflow engine:
  - [ ] Define trigger conditions (time-based, event-based)
  - [ ] Action sequences (multiple steps)
  - [ ] Conditional logic (if/else)
  - [ ] Schedule actions (cron-like)
- [ ] Background job processing with Cloudflare Queues

**Acceptance Criteria**:
- [ ] Automated actions run on schedule
- [ ] Workflow engine supports conditional logic
- [ ] Actions logged to audit_log
- [ ] Queues process jobs reliably
- [ ] Only Super Admins can configure workflows

---

### Epic 4.5: GDPR Compliance & Data Management
**What**: Handle GDPR requests and bulk operations
**Complexity**: Large
**Dependencies**: Epic 1.3 (User Management)

**Tasks**:
- [ ] `POST /admin/gdpr/export/:userId` - Export all user data (JSON)
- [ ] `POST /admin/gdpr/delete/:userId` - Permanently delete user data
- [ ] Data anonymization (replace PII with anonymized values)
- [ ] Compliance audit trail
- [ ] Bulk operations:
  - [ ] `POST /admin/bulk/users` - Bulk user operations (suspend, delete, etc.)
  - [ ] `POST /admin/bulk/emails` - Bulk email sending
  - [ ] `POST /admin/bulk/refunds` - Bulk refunds
- [ ] Background job processing with Cloudflare Queues
- [ ] `POST /admin/db/backup` - Trigger database backup
- [ ] `GET /admin/db/backups` - List backups
- [ ] `POST /admin/db/restore` - Restore from backup
- [ ] `GET /admin/db/stats` - Database statistics

**Acceptance Criteria**:
- [ ] GDPR export includes all user data
- [ ] GDPR delete removes all PII
- [ ] Audit trail complete
- [ ] Bulk operations queued and processed
- [ ] Database backups created and restorable
- [ ] Only Super Admins can perform GDPR operations

---

## Critical Path (Build Order)

1. **MVP-1.1**: Admin Authentication ⟶ Foundation (blocks all admin operations)
2. **MVP-1.2**: Dashboard Analytics ⟶ Core admin metrics
3. **MVP-1.3**: User Management ⟶ Essential admin feature
4. **MVP-1.4**: Template Management ⟶ Content control
5. **MVP-1.5**: Audit Logging ⟶ Compliance requirement
6. **MVP-2.1**: Subscription Management ⟶ Revenue management
7. **MVP-2.2**: Revenue Analytics ⟶ Financial insights
8. **MVP-2.3**: Transaction Management ⟶ Payment handling
9. **MVP-2.4**: Coupon Management ⟶ Marketing tool
10. **MVP-2.5**: Support Ticket System ⟶ Customer support
11. **MVP-3.1**: Content Management ⟶ User communication
12. **MVP-3.2**: Website Management ⟶ User content control
13. **MVP-3.3**: Feature Flags ⟶ Release management
14. **MVP-3.4**: System Health ⟶ Operational visibility
15. **MVP-4.1**: Custom Reports ⟶ Advanced analytics
16. **MVP-4.2**: A/B Testing ⟶ Optimization
17. **MVP-4.3**: AI Usage Analytics ⟶ Cost tracking
18. **MVP-4.4**: Automated Workflows ⟶ Efficiency
19. **MVP-4.5**: GDPR Compliance ⟶ Legal requirement

---

## What Blocks What

| This Epic | Blocks These Services/Features |
|-----------|--------------------------------|
| MVP-1.1 (Admin Auth) | All admin operations in Resume-Admin, Resume-Mobile-Admin |
| MVP-1.2 (Analytics) | Dashboard in Resume-Admin, Resume-Mobile-Admin |
| MVP-1.3 (User Management) | User pages in Resume-Admin, Resume-Mobile-Admin |
| MVP-1.4 (Template Management) | Template control in Resume-Admin |
| MVP-2.1 (Subscriptions) | Billing pages in Resume-Admin, Resume-Mobile-Admin |
| MVP-2.2 (Revenue Analytics) | Financial dashboard in Resume-Admin |
| MVP-2.5 (Support Tickets) | Support section in Resume-Admin, Resume-Mobile-Admin |
| MVP-3.1 (Content Management) | Announcement features in Resume-Admin |
| MVP-3.2 (Website Management) | Website control in Resume-Admin |
| MVP-3.3 (Feature Flags) | Release management in Resume-Admin |
| MVP-3.4 (System Health) | Monitoring dashboard in Resume-Admin |
| MVP-4.1 (Custom Reports) | Report builder in Resume-Admin |
| MVP-4.2 (A/B Testing) | Experiments in Resume-Admin |
| MVP-4.3 (AI Analytics) | AI usage dashboard in Resume-Admin |
| MVP-4.4 (Automation) | Workflow engine in Resume-Admin, Resume-Processor jobs |

**External Dependencies**:
- Resume-SSO (MVP-4.3: MFA) ⟶ Required for Epic 1.1 (Admin Authentication)
- Resume-API (MVP-1.1: Database) ⟶ Required for Epic 1.2 (Analytics) and Epic 1.3 (User Management)
- Resume-API (MVP-2.1: Stripe) ⟶ Required for Epic 2.1 (Subscription Management)
- Resume-API (MVP-4: Website Generation) ⟶ Required for Epic 3.2 (Website Management)
- Resume-API (MVP-5: AI Features) ⟶ Required for Epic 4.3 (AI Usage Analytics)

---

## Testing Strategy

### Unit Tests
- Admin permission checks
- Analytics calculations
- Coupon validation logic
- Feature flag evaluation
- Data aggregation functions

### Integration Tests
- Full admin flow: login → view dashboard → manage user
- Subscription management: view → cancel → refund
- Support ticket flow: create → reply → close
- Feature flag: create → toggle → evaluate
- Bulk operations: queue → process → verify

### Security Tests
- Role-based access control
- Admin impersonation security
- Audit log integrity
- SQL injection attempts
- Authorization bypass attempts

---

## API Endpoints Summary

### Admin Management
- `GET /admin/admins` - List admins
- `POST /admin/admins` - Create admin
- `PUT /admin/admins/:id` - Update admin
- `DELETE /admin/admins/:id` - Delete admin

### Analytics
- `GET /admin/metrics/overview` - Platform metrics
- `GET /admin/metrics/users` - User growth
- `GET /admin/metrics/resumes` - Resume creation
- `GET /admin/metrics/revenue` - Revenue metrics
- `GET /admin/metrics/performance` - API performance

### User Management
- `GET /admin/users` - List users
- `GET /admin/users/:id` - User details
- `PUT /admin/users/:id` - Update user
- `DELETE /admin/users/:id` - Delete user
- `POST /admin/users/:id/suspend` - Suspend
- `POST /admin/users/:id/impersonate` - Impersonate

### Financial
- `GET /admin/subscriptions` - List subscriptions
- `POST /admin/subscriptions/:id/cancel` - Cancel subscription
- `POST /admin/subscriptions/:id/refund` - Refund
- `GET /admin/transactions` - List transactions
- `GET /admin/coupons` - List coupons
- `POST /admin/coupons` - Create coupon

### Support
- `GET /admin/tickets` - List tickets
- `GET /admin/tickets/:id` - Ticket details
- `POST /admin/tickets/:id/reply` - Reply to ticket
- `POST /admin/tickets/:id/close` - Close ticket

### System
- `GET /admin/health` - System health
- `GET /admin/flags` - Feature flags
- `POST /admin/flags` - Create flag
- `GET /admin/audit-log` - Audit logs

---

## Environment Variables

```bash
# Cloudflare Bindings
DB=<D1 database binding (shared with Resume-API)>
R2_BUCKET=<R2 bucket binding>
KV_CACHE=<KV namespace for caching>
ANALYTICS_ENGINE=<Analytics Engine binding>
QUEUE=<Queue binding for background jobs>

# Admin Authentication
ADMIN_JWT_SECRET=<admin-jwt-secret>

# Stripe Configuration (for refunds and management)
STRIPE_SECRET_KEY=<stripe-secret>

# Email Service
EMAIL_SERVICE_URL=<email-service-url>

# Monitoring
SENTRY_DSN=<sentry-dsn>
```

---

## Success Metrics

- [ ] 100% admin endpoint test coverage
- [ ] < 300ms response time for analytics endpoints
- [ ] All admin actions logged to audit_log
- [ ] Zero unauthorized access
- [ ] 99.9% uptime for admin APIs
- [ ] Role-based permissions enforced correctly
- [ ] Analytics accurate within 1% margin
- [ ] Background jobs process within 5 minutes
