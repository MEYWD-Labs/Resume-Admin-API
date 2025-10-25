# Content Moderation & NSFW Detection

## Service Dependencies

**Implements**: Resume-API (content moderation pipeline), Resume-Admin-API (moderation queue)
**Depends On**: Workers AI (detection models), Resume-SSO (user context)
**Used By**: All services creating user content (resumes, profiles, comments)

## 5-Layer Moderation System

```
┌─────────────────────────────────────────────────────────┐
│         Multi-Layer Content Moderation Stack            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Layer 1: AI Detection (Workers AI)                    │
│  ├─ Vision Models (NSFW images)                        │
│  ├─ Text Analysis (toxic language)                      │
│  └─ PDF Content Scanning                               │
│  Latency: <100ms | Confidence: 0-100                   │
│                                                          │
│  Layer 2: Rule-Based Filters                           │
│  ├─ Keyword blocklists (RegEx)                         │
│  ├─ Domain blacklists                                   │
│  └─ Pattern matching                                    │
│  Latency: <10ms | High precision                       │
│                                                          │
│  Layer 3: User Reporting                               │
│  ├─ Community flagging                                  │
│  ├─ Report categories                                   │
│  └─ Abuse prevention                                    │
│                                                          │
│  Layer 4: Human Review                                 │
│  ├─ Moderation queue                                    │
│  ├─ Admin dashboard                                     │
│  └─ Bulk actions                                        │
│  SLA: 24-48 hours                                      │
│                                                          │
│  Layer 5: Appeal Process                               │
│  ├─ User contest submission                             │
│  ├─ Auto-review (obvious false positives)              │
│  └─ Final human decision                               │
│  SLA: 48 hours                                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Detection Categories

**Content Types Scanned**:
- ✅ **NSFW/Adult Content**: Images, text, PDF content
- ✅ **Hate Speech**: Discrimination, slurs, targeted harassment
- ✅ **Violence**: Threats, graphic content
- ✅ **Spam**: Automated content, scams, phishing
- ✅ **PII Leakage**: SSN, credit cards, private info
- ✅ **Malicious Links**: Phishing, malware domains
- ✅ **Copyright**: Plagiarized content

## Workers AI Integration

**Vision Models** (Image Analysis):
```typescript
// NSFW image detection using Llama Vision
const result = await env.AI.run('@cf/llava-hf/llava-1.5-7b-hf', {
  image: imageBuffer,
  prompt: 'Analyze this image for inappropriate content. Is it safe for work?'
});

if (result.confidence > 0.9 && result.isNSFW) {
  await blockContent(contentId, 'nsfw_image');
}
```

**Text Analysis**:
```typescript
// Toxicity detection
const analysis = await env.AI.run('@cf/huggingface/distilbert-sst-2-int8', {
  text: contentText
});

if (analysis.toxicity > 0.7) {
  await queueForReview(contentId, 'toxic_language', analysis);
}
```

## Moderation Actions

**Automatic Actions** (High Confidence >90%):
- **Block**: Immediate content removal, user notified
- **Quarantine**: Content hidden, queued for review
- **Warning**: User notified, no action yet

**Progressive Discipline**:
```
Strike System:
├─ 3 strikes: 24-hour suspension
├─ 5 strikes: 7-day suspension
├─ 10 strikes: Permanent ban
└─ Reset: 90 days no violations
```

## Database Schema

**Moderation Tables** (15 total):
- `moderation_events` - All detection events
- `flagged_content` - Content awaiting review
- `moderation_queue` - Work items for admins
- `user_strikes` - Violation tracking
- `appeal_requests` - User contests
- `moderation_rules` - Dynamic rule engine
- `blocklists` - Keywords, domains, patterns
- `moderator_actions` - Admin action audit trail

**Key Indexes**:
```sql
CREATE INDEX idx_flagged_content_status ON flagged_content(status, priority DESC);
CREATE INDEX idx_moderation_queue_assigned ON moderation_queue(assigned_to, status);
CREATE INDEX idx_user_strikes_active ON user_strikes(user_id, expires_at);
```

## Content Upload Flow

```
User Upload → Hash Check (KV Cache)
              │
              ├─ Known Bad → Block Immediately
              │
              └─ Unknown → AI Scan Pipeline
                            │
                            ├─ Confidence >90% → Auto Decision
                            ├─ Confidence 60-90% → Queue for Review
                            └─ Confidence <60% → Allow with Monitoring
```

## Performance Targets

- **Detection Latency**: <100ms (automated), <5s (comprehensive)
- **Processing Capacity**: 1M+ moderations/day
- **False Positive Rate**: <2%
- **Review SLA**: 24 hours critical, 48 hours standard
- **Appeal Response**: 48 hours

## Implementation Dependencies

**Foundation**:
- Workers AI models configured
- D1 database with moderation tables
- KV for hash-based known content cache

**Core Detection**:
- Vision model integration
- Text analysis pipeline
- PDF content extraction
- Hash fingerprinting system

**Review System**:
- Admin moderation queue
- Bulk action tools
- Appeal workflow
- Strike management

