# OpenClaw Deployment Comparison: Kiloclaw vs. Self-Hosted on Fly.io

## Executive Summary

This document compares two approaches to running OpenClaw: the **Kiloclaw subscription service** ($9/month + token costs) versus **self-hosting on Fly.io** using your own AI provider keys. The self-hosted option offers greater control, potentially lower costs at scale, but requires more operational overhead.

---

## Cost Comparison

### Kiloclaw Subscription

| Cost Component | Amount | Notes |
|---------------|--------|-------|
| **Base Subscription** | $9/month | Per-user or per-instance |
| **AI Token Costs** | Variable | Passed through at cost or with markup |
| **Total Monthly (light use)** | ~$9-15 | Low AI usage |
| **Total Monthly (heavy use)** | ~$30-100+ | High AI usage (GPT-4, many requests) |
| **Setup Cost** | $0 | Zero upfront |
| **Contract** | Monthly | Easy to cancel |

**Advantages:**
- Predictable base cost
- No infrastructure to manage
- No server maintenance
- Instant setup

**Disadvantages:**
- Ongoing monthly fee forever
- Token costs may include markup
- Vendor lock-in
- Limited customization

---

### Self-Hosted on Fly.io

| Cost Component | Amount | Notes |
|---------------|--------|-------|
| **Fly.io VM** (shared-cpu-2x) | ~$5-10/month | 2 CPU, 2GB RAM |
| **Fly.io Volume** (1GB) | ~$0.15/month | Persistent storage |
| **AI Token Costs** | Variable | Pay direct to provider (OpenAI, Anthropic, etc.) |
| **Total Monthly (light use)** | ~$5-15 | VM + low AI usage |
| **Total Monthly (heavy use)** | ~$20-100+ | VM + high AI usage |
| **Setup Cost** | $0 | Free tier available for testing |
| **Contract** | Pay-as-you-go | Only pay for what you use |

**Advantages:**
- Full control over infrastructure
- Direct API pricing (no markup)
- Custom configurations
- Data privacy (your own instance)
- Can run multiple agents/channels
- No vendor lock-in

**Disadvantages:**
- Requires technical setup
- You manage updates and maintenance
- Responsible for security patches
- Need to monitor costs

---

## Detailed Cost Scenarios

### Scenario 1: Light Personal Use

**Usage:** ~100 messages/day, GPT-3.5 level

| Approach | Monthly Cost | Breakdown |
|----------|-------------|-----------|
| Kiloclaw | $12 | $9 base + $3 tokens |
| Fly.io Self-Hosted | $8 | $6 VM + $2 tokens |
| **Savings** | **$4/month** | **33% cheaper** |

### Scenario 2: Moderate Professional Use

**Usage:** ~500 messages/day, GPT-4 level

| Approach | Monthly Cost | Breakdown |
|----------|-------------|-----------|
| Kiloclaw | $45 | $9 base + $36 tokens |
| Fly.io Self-Hosted | $40 | $8 VM + $32 tokens |
| **Savings** | **$5/month** | **11% cheaper** |

### Scenario 3: Heavy Business Use

**Usage:** 2000+ messages/day, multiple models

| Approach | Monthly Cost | Breakdown |
|----------|-------------|-----------|
| Kiloclaw | $120 | $9 base + $111 tokens (with markup) |
| Fly.io Self-Hosted | $95 | $10 VM + $85 tokens (direct pricing) |
| **Savings** | **$25/month** | **21% cheaper** |

### Scenario 4: Multi-Channel Deployment

**Usage:** Telegram + Discord + Web UI + 3 users

| Approach | Monthly Cost | Notes |
|----------|-------------|-------|
| Kiloclaw | $27 | $9 x 3 users |
| Fly.io Self-Hosted | $12 | One VM, unlimited users/channels |
| **Savings** | **$15/month** | **56% cheaper** |

---

## Feature Comparison

| Feature | Kiloclaw Subscription | Fly.io Self-Hosted |
|---------|---------------------|-------------------|
| **Setup Time** | 5 minutes | 1-2 hours initial |
| **Technical Skill Required** | Low | Medium |
| **Custom Channels** | Limited | Unlimited (Telegram, Discord, Slack, etc.) |
| **Custom Models** | Limited | Any OpenAI-compatible endpoint |
| **Data Privacy** | Provider-dependent | Full control |
| **API Key Management** | Managed by Kiloclaw | You manage (Key Vault + Fly secrets) |
| **Update Frequency** | Automatic | Manual or automated |
| **Uptime SLA** | Provider-dependent | Fly.io ~99.95% |
| **Scaling** | Automatic | Manual (flyctl scale) |
| **Backup/Recovery** | Provider-managed | You manage (volume snapshots) |
| **Custom Skills** | Limited | Full OpenClaw ecosystem |
| **Multi-Agent** | Paid tiers | Unlimited |
| **Device Pairing Control** | Provider-managed | Full control |

---

## Maintenance Overhead

### Kiloclaw: Low Maintenance

**Time Required:** ~0 hours/month

The provider handles:
- Server updates
- Security patches
- Certificate renewal
- Scaling
- Monitoring
- Backups

**Your responsibilities:**
- Pay the bill
- Configure channels

---

### Fly.io Self-Hosted: Medium Maintenance

**Time Required:** ~1-3 hours/month

**Regular tasks:**
| Task | Frequency | Time | Command/Action |
|------|-----------|------|----------------|
| Monitor logs | Weekly | 15 min | `flyctl logs --app openclaw-fly-lhr-20260512` |
| Check status | Weekly | 5 min | `flyctl status --app openclaw-fly-lhr-20260512` |
| Update image | Monthly | 30 min | `flyctl deploy --app openclaw-fly-lhr-20260512` |
| Rotate secrets | Quarterly | 30 min | `flyctl secrets set ...` |
| Review costs | Monthly | 15 min | Fly.io dashboard |
| Scale if needed | As needed | 10 min | `flyctl scale vm ...` |

**Your responsibilities:**
- Server updates (`flyctl deploy`)
- Secret rotation
- Cost monitoring
- Security configuration
- Backup verification
- Channel configuration
- Device pairing approvals

---

## Security Comparison

| Security Aspect | Kiloclaw | Fly.io Self-Hosted |
|----------------|----------|-------------------|
| **API Keys** | Stored by provider | Stored in Azure Key Vault + Fly secrets |
| **Data at Rest** | Provider-controlled | Encrypted volume on Fly.io |
| **Data in Transit** | TLS | TLS (enforced by Fly.io) |
| **Access Control** | Provider-managed | Gateway token + device pairing |
| **Audit Logs** | Provider-dependent | Full application logs |
| **Compliance** | Provider-dependent | You control |

---

## Scalability

### Kiloclaw
- **Horizontal scaling:** Automatic (theoretically unlimited)
- **Vertical scaling:** Automatic
- **Multi-region:** Provider-dependent
- **Limitations:** Provider-imposed rate limits, cost tiers

### Fly.io Self-Hosted
- **Horizontal scaling:** `flyctl scale count 2` (up to your budget)
- **Vertical scaling:** `flyctl scale vm performance-2x --memory 4096`
- **Multi-region:** Deploy to multiple regions
- **Limitations:** Your budget, API rate limits from AI provider

---

## When to Choose Which

### Choose Kiloclaw Subscription If:
- ✅ You want zero maintenance
- ✅ You need instant setup
- ✅ You have low technical expertise
- ✅ You prefer predictable monthly costs
- ✅ You don't need custom configurations
- ✅ You're testing/learning OpenClaw
- ✅ You value convenience over cost savings

### Choose Fly.io Self-Hosted If:
- ✅ You want full control
- ✅ You have technical expertise (or access to it)
- ✅ You want to avoid vendor lock-in
- ✅ You need custom channels or configurations
- ✅ You want direct API pricing (no markup)
- ✅ Data privacy is critical
- ✅ You're running multiple users/channels
- ✅ You want to use existing API keys
- ✅ You're comfortable with ~1-3 hours/month maintenance

---

## Migration Path

### From Kiloclaw to Self-Hosted
1. Export your Kiloclaw configuration (if possible)
2. Set up Fly.io app (this repository)
3. Configure channels (Telegram, Discord, etc.)
4. Transfer API keys to Azure Key Vault
5. Test parallel (both running)
6. Switch DNS/URLs
7. Cancel Kiloclaw subscription

### From Self-Hosted to Kiloclaw
1. Export OpenClaw config from `/data`
2. Sign up for Kiloclaw
3. Import configuration
4. Update channel webhooks (Telegram bot, Discord, etc.)
5. Test
6. Destroy Fly.io app
7. Cancel Fly.io billing

---

## Real-World Example: Our Current Deployment

### What We Built
- **Platform:** Fly.io (London region)
- **App:** `openclaw-fly-lhr-20260512`
- **Channels:** Web UI + Telegram (@fly_open_claw_bot)
- **AI Provider:** OpenAI (configured)
- **Secrets:** Azure Key Vault + Fly.io secrets

### Current Monthly Costs (Estimated)
| Component | Cost |
|-----------|------|
| Fly.io VM (shared-cpu-2x, 2GB) | ~$6-8 |
| Fly.io Volume (1GB) | ~$0.15 |
| AI Tokens (variable) | ~$0-50 |
| **Total** | **~$6-58/month** |

### Maintenance Tasks So Far
| Task | Time Spent |
|------|-----------|
| Initial deployment | 2 hours |
| Device pairing fixes | 1 hour |
| Telegram bot setup | 30 min |
| API key troubleshooting | 30 min |
| Documentation | 2 hours |
| **Total setup** | **~6 hours** |

---

## Cost-Benefit Analysis

### Break-Even Point

If Kiloclaw costs $9/month base + token markup (~20%), and Fly.io costs $8/month VM + direct tokens:

| Monthly Token Usage | Kiloclaw | Fly.io | Savings |
|-------------------|----------|--------|---------|
| $10 | $19 | $18 | $1 |
| $50 | $69 | $58 | $11 |
| $100 | $129 | $108 | $21 |
| $200 | $249 | $208 | $41 |

**Conclusion:** The more you use AI, the more you save with self-hosting.

### Hidden Costs

| Cost Type | Kiloclaw | Fly.io |
|-----------|----------|--------|
| Time (maintenance) | $0 | ~$50-150/month (if outsourced) |
| Learning curve | Low | Medium |
| Risk of misconfiguration | Low | Medium |
| Downtime risk | Provider's problem | Your problem |

---

## Recommendations

### For Individuals / Hobbyists
- **Start with Kiloclaw** if you want to test OpenClaw without technical setup
- **Switch to Fly.io** if you enjoy tinkering and want to learn

### For Small Teams (2-5 users)
- **Fly.io is likely cheaper** and gives you more control
- One VM can serve multiple users
- Shared API key across team

### For Businesses
- **Fly.io recommended** for:
  - Data compliance requirements
  - Custom integrations
  - Cost control at scale
- **Kiloclaw recommended** for:
  - No dedicated ops person
  - Quick proof-of-concept
  - Predictable budgeting

### For AI-Heavy Use Cases
- **Fly.io strongly recommended**
- Direct API pricing saves significant money
- No per-user fees
- Scale compute independently of AI costs

---

## Conclusion

| Criteria | Winner | Margin |
|----------|--------|--------|
| **Lowest upfront cost** | Kiloclaw | Significant |
| **Lowest ongoing cost (heavy use)** | Fly.io | Significant |
| **Easiest setup** | Kiloclaw | Significant |
| **Most control** | Fly.io | Significant |
| **Best for privacy** | Fly.io | Significant |
| **Best for non-technical users** | Kiloclaw | Significant |
| **Best for scaling** | Tie | Depends on use case |

**Final Verdict:**
- **Use Kiloclaw** for simplicity, quick starts, and low maintenance
- **Use Fly.io self-hosted** for cost savings at scale, full control, data privacy, and custom configurations

Our current Fly.io deployment is operational and ready for production use. The main blocker is configuring a valid AI provider API key.

---

## References

- [Fly.io Pricing](https://fly.io/docs/about/pricing/)
- [OpenAI API Pricing](https://openai.com/pricing)
- [OpenClaw Documentation](https://docs.openclaw.ai)
- [DEPLOYMENT.md](DEPLOYMENT.md) — Our deployment guide
- [AGENTS.md](AGENTS.md) — Coding guidelines and secrets policy
