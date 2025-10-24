# Resume-Admin-API

**Administrative Backend Service** for the Resume Builder SaaS platform.

## Overview

Resume-Admin-API provides backend operations for system administrators, including user management, financial operations, content management, support ticket handling, and system monitoring. Built on Cloudflare Workers with shared database access to Resume-API.

## Technology Stack

- **Runtime**: Cloudflare Workers (serverless)
- **Database**: Cloudflare D1 (shared with Resume-API)
- **Cache**: Cloudflare KV (admin sessions, feature flags)
- **Analytics**: Cloudflare Analytics Engine
- **Queue**: Cloudflare Queues (email, reports)
- **Framework**: Hono (lightweight web framework)
- **Language**: TypeScript

## Key Features

### MVP-1: Admin Foundation
- Admin role-based authentication (Super Admin, Admin, Support)
- Dashboard analytics (users, revenue, system health)
- User management (view, edit, suspend, delete)
- Activity logging and audit trails

### MVP-2: Financial Operations & Support
- Subscription management (upgrade, downgrade, refund)
- Payment history and analytics
- Revenue reporting and forecasting
- Support ticket system
- Email notification management

### MVP-3: Content & System Management
- Template management (create, edit, publish)
- Feature flag management (gradual rollout)
- System health monitoring
- Database management tools
- Content moderation

### MVP-4: Advanced Analytics & Automation
- Custom report builder
- Cohort analysis
- Churn prediction
- Automated workflows (onboarding, retention)
- Integration management

## Service Dependencies

**Depends On**:
- **Resume-SSO** (MVP-4.3: MFA) - For admin authentication with MFA enforcement
- **Resume-API Database** - Shared D1 database for read/write access

**Services that depend on Resume-Admin-API**:
- Resume-Admin (admin dashboard UI)

## Admin Roles & Permissions

### Super Admin
- Full access to all features
- User management (all operations)
- Financial operations (refunds, adjustments)
- System configuration
- Feature flag management
- Database access

### Admin
- User management (view, suspend)
- Content management
- Support tickets (all)
- Analytics (read-only)
- Template management

### Support
- View users
- Manage support tickets
- Basic user actions (password reset)
- No financial access
- Limited analytics

## API Endpoints

### Authentication (Protected with MFA)
All endpoints require JWT token from Resume-SSO with admin role + MFA verification.

### Dashboard Analytics
- `GET /api/admin/dashboard` - Overview metrics
- `GET /api/admin/analytics/users` - User analytics
- `GET /api/admin/analytics/revenue` - Revenue analytics
- `GET /api/admin/analytics/system` - System health metrics

### User Management
- `GET /api/admin/users` - List users (paginated, searchable)
- `GET /api/admin/users/:id` - Get user details
- `PUT /api/admin/users/:id` - Update user
- `DELETE /api/admin/users/:id` - Delete user
- `POST /api/admin/users/:id/suspend` - Suspend user
- `POST /api/admin/users/:id/unsuspend` - Unsuspend user
- `POST /api/admin/users/:id/impersonate` - Impersonate user
- `GET /api/admin/users/:id/activity` - User activity log

### Subscription Management
- `GET /api/admin/subscriptions` - List subscriptions
- `GET /api/admin/subscriptions/:id` - Get subscription details
- `PUT /api/admin/subscriptions/:id` - Update subscription
- `POST /api/admin/subscriptions/:id/cancel` - Cancel subscription
- `POST /api/admin/subscriptions/:id/refund` - Issue refund
- `GET /api/admin/payments` - Payment history
- `POST /api/admin/payments/:id/refund` - Refund payment

### Support Tickets
- `GET /api/admin/tickets` - List support tickets
- `GET /api/admin/tickets/:id` - Get ticket details
- `POST /api/admin/tickets/:id/reply` - Reply to ticket
- `PUT /api/admin/tickets/:id/status` - Update ticket status
- `PUT /api/admin/tickets/:id/assign` - Assign ticket to admin
- `POST /api/admin/tickets/:id/close` - Close ticket

### Template Management
- `GET /api/admin/templates` - List templates
- `POST /api/admin/templates` - Create template
- `PUT /api/admin/templates/:id` - Update template
- `DELETE /api/admin/templates/:id` - Delete template
- `POST /api/admin/templates/:id/publish` - Publish template
- `POST /api/admin/templates/:id/unpublish` - Unpublish template

### Feature Flags
- `GET /api/admin/features` - List feature flags
- `POST /api/admin/features` - Create feature flag
- `PUT /api/admin/features/:id` - Update feature flag
- `POST /api/admin/features/:id/rollout` - Gradual rollout config
- `DELETE /api/admin/features/:id` - Delete feature flag

### System Management
- `GET /api/admin/system/health` - System health check
- `GET /api/admin/system/metrics` - System metrics
- `GET /api/admin/system/logs` - Application logs
- `POST /api/admin/system/cache/clear` - Clear cache
- `GET /api/admin/database/stats` - Database statistics

### Reports
- `GET /api/admin/reports` - List saved reports
- `POST /api/admin/reports` - Create custom report
- `GET /api/admin/reports/:id` - Get report data
- `POST /api/admin/reports/:id/schedule` - Schedule report
- `GET /api/admin/reports/:id/export` - Export report (CSV, PDF)

### Automation
- `GET /api/admin/workflows` - List automated workflows
- `POST /api/admin/workflows` - Create workflow
- `PUT /api/admin/workflows/:id` - Update workflow
- `POST /api/admin/workflows/:id/enable` - Enable workflow
- `POST /api/admin/workflows/:id/disable` - Disable workflow
- `GET /api/admin/workflows/:id/history` - Workflow execution history

## Development Setup

### Prerequisites
- Node.js 18+
- Wrangler CLI (`npm install -g wrangler`)
- Cloudflare account
- Access to Resume-API database

### Environment Variables

Create a `.dev.vars` file:

```bash
# Database (Shared with Resume-API)
DATABASE_URL=<d1-database-url>

# Cloudflare
KV_NAMESPACE=admin-cache
ANALYTICS_ENGINE=admin-analytics
QUEUE_NAME=admin-jobs

# Admin Configuration
ADMIN_SESSION_DURATION=8h
REQUIRE_MFA_FOR_ADMIN=true

# Stripe (for refunds)
STRIPE_SECRET_KEY=sk_test_...

# Email
EMAIL_FROM=admin@resumebuilder.com
SUPPORT_EMAIL=support@resumebuilder.com
```

### Quick Start

```bash
# Install dependencies
npm install

# Run database migrations (admin tables)
npm run db:migrate

# Start development server
npm run dev

# Run tests
npm test

# Deploy to production
npm run deploy
```

## Database Schema Extensions

The service extends the Resume-API database with admin-specific tables:

```sql
-- Admin users
CREATE TABLE admin_users (
  id TEXT PRIMARY KEY,
  user_id TEXT UNIQUE NOT NULL,
  role TEXT NOT NULL, -- 'super_admin', 'admin', 'support'
  created_at INTEGER NOT NULL,
  created_by TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Admin activity log
CREATE TABLE admin_activity_log (
  id TEXT PRIMARY KEY,
  admin_id TEXT NOT NULL,
  action TEXT NOT NULL,
  target_type TEXT,
  target_id TEXT,
  metadata TEXT,
  created_at INTEGER NOT NULL,
  FOREIGN KEY (admin_id) REFERENCES admin_users(id)
);

-- Support tickets
CREATE TABLE support_tickets (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  subject TEXT NOT NULL,
  status TEXT NOT NULL, -- 'open', 'in_progress', 'resolved', 'closed'
  priority TEXT NOT NULL, -- 'low', 'medium', 'high', 'urgent'
  assigned_to TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (assigned_to) REFERENCES admin_users(id)
);

-- Feature flags
CREATE TABLE feature_flags (
  id TEXT PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  description TEXT,
  enabled INTEGER DEFAULT 0,
  rollout_percentage INTEGER DEFAULT 0,
  target_tiers TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

-- Automated workflows
CREATE TABLE automation_workflows (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  trigger_type TEXT NOT NULL,
  trigger_config TEXT NOT NULL,
  actions TEXT NOT NULL,
  enabled INTEGER DEFAULT 1,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);
```

## Security Features

- MFA enforcement for all admin accounts
- Role-based access control (RBAC)
- Admin activity logging (audit trail)
- IP whitelisting for admin access
- Rate limiting on sensitive operations
- User impersonation with audit trail

## Performance Targets

- API response time: **< 500ms** (95th percentile)
- Dashboard load: **< 2s**
- User search: **< 300ms**
- Report generation: **< 10s**
- Analytics queries: **< 1s**

## Testing

```bash
# Run all tests
npm test

# Run unit tests
npm run test:unit

# Run integration tests
npm run test:integration

# Test admin permissions
npm run test:permissions

# Test coverage
npm run test:coverage
```

## Development Plan

See [HIGH_LEVEL_PLAN.md](./HIGH_LEVEL_PLAN.md) for the complete dependency-tree based development plan with MVPs, epics, and critical path.

## GitHub Issues

Track development progress in [GitHub Issues](https://github.com/MEYWD-Labs/Resume-Admin-API/issues):
- [MVP-1 Epics](https://github.com/MEYWD-Labs/Resume-Admin-API/issues?q=is%3Aissue+is%3Aopen+MVP-1) - Admin Foundation
- [MVP-2 Epics](https://github.com/MEYWD-Labs/Resume-Admin-API/issues?q=is%3Aissue+is%3Aopen+MVP-2) - Financial Operations & Support
- [MVP-3 Epics](https://github.com/MEYWD-Labs/Resume-Admin-API/issues?q=is%3Aissue+is%3Aopen+MVP-3) - Content & System Management
- [MVP-4 Epics](https://github.com/MEYWD-Labs/Resume-Admin-API/issues?q=is%3Aissue+is%3Aopen+MVP-4) - Advanced Analytics & Automation

## Deployment

```bash
# Deploy to production
wrangler deploy

# Deploy with migrations
npm run deploy:migrate

# View logs
wrangler tail

# Monitor admin activity
wrangler analytics
```

## Monitoring & Alerts

- Real-time system health monitoring
- Admin activity audit logs
- Performance metrics tracking
- Error alerting via email/Slack
- Database query performance monitoring

## Support

For issues or questions, create a GitHub issue in this repository.

## License

Proprietary - All rights reserved
