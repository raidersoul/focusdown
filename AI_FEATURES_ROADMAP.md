# Focus Flip AI Features Roadmap 🤖

> **Status**: Future Implementation (v2.x+)  
> **Created**: December 12, 2025  
> **Last Updated**: December 12, 2025

This document outlines the AI integration strategy for Focus Flip. These features are designed to enhance user engagement, provide personalized insights, and differentiate the app from competitors.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Feature Tiers](#feature-tiers)
3. [Tier 1: Smart Session Insights](#tier-1-smart-session-insights)
4. [Tier 2: AI Focus Coach](#tier-2-ai-focus-coach)
5. [Tier 3: Intelligent Recommendations](#tier-3-intelligent-recommendations)
6. [Tier 4: Social AI Features](#tier-4-social-ai-features)
7. [Technical Architecture](#technical-architecture)
8. [Privacy & Data Considerations](#privacy--data-considerations)
9. [Monetization Strategy](#monetization-strategy)
10. [Implementation Timeline](#implementation-timeline)

---

## Executive Summary

AI integration in Focus Flip focuses on three core principles:

1. **Personalization** — Tailor the experience based on individual usage patterns
2. **Motivation** — Provide contextual encouragement at the right moments
3. **Intelligence** — Surface insights users couldn't discover on their own

All AI features leverage existing session data (time, duration, mood, tags, streaks) to generate value without requiring additional user input.

---

## Feature Tiers

| Tier | Priority | Complexity | Premium? | Target Release |
|------|----------|------------|----------|----------------|
| 1. Smart Session Insights | 🔴 High | Medium | Yes | v2.0 |
| 2. AI Focus Coach | 🟠 Medium | Low-Medium | Partial | v2.1 |
| 3. Intelligent Recommendations | 🟡 Medium | Medium | Yes | v2.2 |
| 4. Social AI Features | 🟢 Low | High | Yes | v3.0 |

---

## Tier 1: Smart Session Insights

### Overview
Weekly AI-generated insights that analyze user patterns and provide actionable recommendations.

### User Stories
- *As a user, I want to understand my productivity patterns so I can optimize my schedule*
- *As a user, I want to see my progress trends over time*
- *As a user, I want personalized tips based on my actual behavior*

### Feature Breakdown

#### 1.1 Weekly Insight Cards
Display on Stats screen every Monday:

```
┌─────────────────────────────────────┐
│  ✨ Your Week in Focus              │
├─────────────────────────────────────┤
│  🏆 Best Day: Tuesday (2h 15m)      │
│  ⏰ Peak Hours: 9am - 11am          │
│  📈 Trend: +23% vs last week        │
│                                     │
│  💡 "Your afternoon sessions are    │
│     32% shorter. Try morning blocks │
│     for deep work."                 │
└─────────────────────────────────────┘
```

#### 1.2 Pattern Detection
Analyze and surface:
- **Time patterns**: Best day of week, optimal time of day
- **Duration patterns**: Ideal session length for completion
- **Streak patterns**: What causes streak breaks
- **Tag patterns**: Which categories get most focus time
- **Mood correlations**: How mood affects session outcomes

#### 1.3 Smart Comparisons
```
"You've focused 4.5 hours this week — that's 1.2 hours more than 
your average and puts you in the top 15% of Focus Flip users."
```

### Data Required
- `sessions` table (timestamps, duration, mood, tags)
- `users` table (streak data, total focus time)
- Aggregated anonymized benchmarks (optional)

### UI Placement
- **Primary**: New "Insights" tab or card on ExperimentalStatsScreen
- **Secondary**: Push notification summary on Mondays

### API Design

```typescript
// Request
POST /api/insights/weekly
{
  "user_id": "abc123",
  "week_start": "2025-12-09"
}

// Response
{
  "insights": [
    {
      "type": "peak_hours",
      "title": "Your Peak Hours",
      "message": "You focus best between 9am-11am",
      "data": { "peak_start": 9, "peak_end": 11 }
    },
    {
      "type": "recommendation",
      "title": "Pro Tip",
      "message": "Your Tuesday sessions are 40% longer than average. Block more time on Tuesdays.",
      "confidence": 0.85
    }
  ],
  "summary": {
    "total_time": 270,
    "best_day": "Tuesday",
    "trend": "+23%"
  }
}
```

---

## Tier 2: AI Focus Coach

### Overview
Contextual, personalized messages that appear at key moments to motivate and guide users.

### User Stories
- *As a user, I want encouragement before starting a session*
- *As a user, I want to celebrate my achievements after a session*
- *As a user, I want to be reminded when I'm at risk of breaking my streak*

### Feature Breakdown

#### 2.1 Pre-Session Motivation
Display on HomeScreen before session starts:

```
┌─────────────────────────────────────┐
│  "You've completed 3 sessions       │
│   today already. Ready for #4? 💪"  │
└─────────────────────────────────────┘
```

Message variants based on context:
- **Streak context**: "Day 12 of your streak! Keep the momentum going."
- **Time of day**: "Morning sessions are your strongest. Great timing!"
- **Recent activity**: "First session in 3 days — welcome back! Start small."
- **Challenge context**: "You're 15 minutes behind Sarah in the challenge. Time to catch up!"

#### 2.2 Post-Session Celebration
Display in SessionReviewModal:

```
┌─────────────────────────────────────┐
│  🎉 "47 minutes! That's your 2nd    │
│      longest session this week."    │
│                                     │
│  Keep going — you're on fire! 🔥    │
└─────────────────────────────────────┘
```

Celebration tiers:
- Personal bests (longest session, most in a day)
- Milestone completions (10th session, 100 hours total)
- Streak achievements
- Challenge progress

#### 2.3 Streak Protection Nudges
Push notification when streak is at risk:

```
"⚠️ Your 15-day streak expires at midnight! 
Even a 5-minute session counts. Flip to focus?"
```

Trigger conditions:
- No session today AND it's past 6pm
- Within 2 hours of streak expiration
- User has opened app but not started session

#### 2.4 Re-engagement Messages
For users who haven't opened the app:

```
Day 3: "Your focus streak was building momentum. Ready to restart?"
Day 7: "We miss you! A single 10-minute session can spark a new habit."
Day 14: "Start fresh — no pressure. Just flip when you're ready."
```

### Implementation Notes
- Use local context (AsyncStorage) for simple messages
- Call API for personalized/comparative messages
- Cache responses to minimize API calls
- Fallback to generic messages if offline

### Message Generation Prompt Template

```
You are a supportive focus coach for a productivity app. Generate a short, 
encouraging message (max 100 characters) based on this user context:

- Current streak: {streak_days} days
- Sessions today: {sessions_today}
- Last session: {last_session_ago} ago
- Time of day: {time_of_day}
- Personal best: {personal_best_minutes} minutes

Be warm, specific, and action-oriented. Avoid generic platitudes.
```

---

## Tier 3: Intelligent Recommendations

### Overview
AI-powered suggestions for session duration, optimal timing, and goal setting.

### Feature Breakdown

#### 3.1 Smart Duration Suggestions
Before starting a session:

```
┌─────────────────────────────────────┐
│  💡 Suggested: 35 minutes           │
│                                     │
│  "Based on your morning patterns    │
│   and energy level, 35 min is your  │
│   sweet spot."                      │
│                                     │
│  [Use Suggestion] [Choose Custom]   │
└─────────────────────────────────────┘
```

Factors considered:
- Time of day + historical completion rates
- Typical session length for selected tag
- Energy pattern (morning vs afternoon)
- Time since last session

#### 3.2 Optimal Scheduling
Integration with calendar/notifications:

```
"📅 Your calendar shows 10am-12pm is free tomorrow. 
Based on your patterns, that's your most productive window. 
Schedule a focus block?"
```

#### 3.3 Goal Setting Assistant
Help users set realistic goals:

```
User: "I want to focus 2 hours every day"

AI: "That's ambitious! Looking at your history, you average 
45 minutes on weekdays. How about we start with 1 hour daily 
for the first week, then build up?"

[Adjust Goal] [Keep Original]
```

#### 3.4 Tag/Mood Predictions
Pre-fill based on patterns:

```
"It's Monday morning — starting a Work session?"
[Work ✓] [Study] [Personal] [Other]
```

---

## Tier 4: Social AI Features

### Overview
AI-enhanced social features for challenges, leaderboards, and community.

### Feature Breakdown

#### 4.1 Smart Challenge Suggestions
```
"🎯 Challenge Idea: You and @sarah both average 45 mins/day. 
Race to 5 hours this week?"

[Send Challenge] [Modify]
```

Matching criteria:
- Similar activity levels
- Mutual friends/connections
- Compatible time zones
- Historical engagement

#### 4.2 Comparative Insights
```
"You're in the top 10% of users for morning focus. 
But @alex just passed you in total streak days! 👀"
```

#### 4.3 Community Trends
```
"🌍 Focus Flip users logged 50,000 hours yesterday. 
Your 2 hours contributed to a record-breaking day!"
```

#### 4.4 AI-Generated Challenges
Automatically create themed challenges:
- "No-Phone-Zone Week" 
- "Deep Work December" 
- "Morning Ritual Challenge"

---

## Technical Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT (React Native)                │
├─────────────────────────────────────────────────────────────┤
│  - Insight Cards Component                                  │
│  - Coach Message Component                                  │
│  - Recommendation UI                                        │
│  - Local caching (AsyncStorage)                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    SUPABASE EDGE FUNCTIONS                  │
├─────────────────────────────────────────────────────────────┤
│  POST /insights/weekly     → Generate weekly insights       │
│  POST /coach/message       → Get contextual coach message   │
│  POST /recommend/duration  → Get session recommendation     │
│  POST /challenges/suggest  → Get challenge suggestions      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        AI PROVIDER                          │
├─────────────────────────────────────────────────────────────┤
│  Option A: OpenAI GPT-4o-mini (cost-effective)              │
│  Option B: Claude 3 Haiku (fast, good for coaching)         │
│  Option C: Google Gemini Flash (good free tier)             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      SUPABASE DATABASE                      │
├─────────────────────────────────────────────────────────────┤
│  - sessions (existing)                                      │
│  - users (existing)                                         │
│  - ai_insights (new - cache generated insights)             │
│  - ai_usage (new - track API usage per user)                │
└─────────────────────────────────────────────────────────────┘
```

### New Database Tables

```sql
-- Cache AI-generated insights
CREATE TABLE ai_insights (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(64) REFERENCES users(id) ON DELETE CASCADE,
    insight_type VARCHAR(50) NOT NULL, -- 'weekly', 'monthly', 'coach'
    content JSONB NOT NULL,
    generated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    
    UNIQUE(user_id, insight_type, generated_at::date)
);

-- Track AI API usage for billing/limits
CREATE TABLE ai_usage (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(64) REFERENCES users(id) ON DELETE CASCADE,
    feature VARCHAR(50) NOT NULL, -- 'insights', 'coach', 'recommend'
    tokens_used INTEGER DEFAULT 0,
    cost_usd DECIMAL(10, 6) DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Index for quick lookups
CREATE INDEX idx_ai_insights_user ON ai_insights(user_id, insight_type);
CREATE INDEX idx_ai_usage_user ON ai_usage(user_id, created_at);
```

### Edge Function Example

```typescript
// supabase/functions/generate-insights/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const { user_id } = await req.json()
  
  // Fetch user's session data
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )
  
  const { data: sessions } = await supabase
    .from('sessions')
    .select('*')
    .eq('user_id', user_id)
    .gte('created_at', new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString())
    .order('created_at', { ascending: false })
  
  // Analyze patterns
  const patterns = analyzePatterns(sessions)
  
  // Generate insight via AI
  const insight = await generateInsight(patterns)
  
  // Cache result
  await supabase.from('ai_insights').upsert({
    user_id,
    insight_type: 'weekly',
    content: insight,
    expires_at: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString()
  })
  
  return new Response(JSON.stringify(insight), {
    headers: { 'Content-Type': 'application/json' }
  })
})

function analyzePatterns(sessions: any[]) {
  // Calculate peak hours, best days, trends, etc.
  // Returns structured data for prompt
}

async function generateInsight(patterns: any) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'gpt-4o-mini',
      messages: [
        { role: 'system', content: INSIGHT_SYSTEM_PROMPT },
        { role: 'user', content: JSON.stringify(patterns) }
      ],
      max_tokens: 300
    })
  })
  
  return response.json()
}
```

### Cost Estimation

| Feature | API Calls/User/Month | Tokens/Call | Cost/User/Month |
|---------|---------------------|-------------|-----------------|
| Weekly Insights | 4 | ~500 | $0.01 |
| Coach Messages | 30 | ~100 | $0.02 |
| Recommendations | 20 | ~150 | $0.01 |
| **Total** | **54** | — | **~$0.04** |

Using GPT-4o-mini at $0.15/1M input tokens, $0.60/1M output tokens.

---

## Privacy & Data Considerations

### Data Used for AI
- Session timestamps and durations ✓
- Tags and moods ✓
- Streak data ✓
- Anonymized benchmarks ✓

### Data NOT Used
- Personal identifiers (name, email)
- Exact location data
- Content of focus sessions
- Third-party app data

### User Controls
1. **Opt-out**: Toggle to disable AI features entirely
2. **Data visibility**: Show exactly what data AI uses
3. **Delete AI data**: Clear all AI-generated insights

### Compliance
- GDPR: AI data included in data export/deletion
- CCPA: Disclose AI data usage in privacy policy
- App Store: Follow Apple's AI disclosure guidelines

---

## Monetization Strategy

### Free Tier
- Basic coach messages (pre-defined, not AI-generated)
- Simple streak reminders
- Generic tips

### Premium Tier ($4.99/month or $39.99/year)
- Weekly AI insights
- Personalized coach messages
- Smart duration recommendations
- Priority AI processing

### Upsell Moments
```
"🔒 Unlock personalized insights by upgrading to Pro"
[See What You're Missing] [Maybe Later]
```

Show preview of locked insight:
```
"Your peak focus hours are ██:██ - ██:██"
```

---

## Implementation Timeline

### Phase 1: Foundation (v2.0) — 2-3 weeks
- [ ] Set up Supabase Edge Functions
- [ ] Create `ai_insights` and `ai_usage` tables
- [ ] Implement basic Weekly Insights
- [ ] Design Insight Cards UI component
- [ ] Add to ExperimentalStatsScreen

### Phase 2: Coach (v2.1) — 2 weeks
- [ ] Implement Coach Message component
- [ ] Add pre-session motivation to HomeScreen
- [ ] Add post-session celebration to SessionReviewModal
- [ ] Set up streak protection notifications

### Phase 3: Recommendations (v2.2) — 2 weeks
- [ ] Build duration recommendation engine
- [ ] Add suggestion UI to session start flow
- [ ] Implement tag/mood predictions
- [ ] A/B test recommendation adoption

### Phase 4: Social AI (v3.0) — 4 weeks
- [ ] Challenge suggestion algorithm
- [ ] Comparative insights feature
- [ ] Community trends dashboard
- [ ] AI-generated themed challenges

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Insight engagement | 60% view rate | Analytics: "Insight: Viewed" |
| Coach message clicks | 30% CTR | Analytics: "Coach: Tapped" |
| Recommendation adoption | 40% use suggested duration | Analytics: "Suggestion: Accepted" |
| Premium conversion | +15% from AI features | RevenueCat cohort analysis |
| Retention impact | +10% D30 retention | Compare AI users vs non-AI |

---

## Open Questions

1. **Which AI provider?** OpenAI vs Claude vs Gemini — need to evaluate latency and cost
2. **Offline fallback?** Should we cache AI responses for offline use?
3. **Localization?** Do we need multilingual AI responses?
4. **A/B testing?** How do we test AI features vs control group?

---

## Appendix: Prompt Library

### Weekly Insight System Prompt
```
You are an AI assistant for Focus Flip, a productivity app. Analyze the user's 
focus session data and generate encouraging, actionable insights.

Rules:
- Be specific (use actual numbers from their data)
- Be positive but honest
- Keep messages under 100 characters each
- Focus on patterns they might not notice
- Suggest one concrete action

Output JSON with: summary, insights[], recommendation
```

### Coach Message System Prompt
```
You are a warm, supportive focus coach. Generate a short motivational message 
(max 80 characters) based on the user's current context.

Tone: Friendly, encouraging, slightly playful
Avoid: Generic platitudes, pressure, comparison shaming

Context provided: streak, sessions today, time of day, recent activity
```

---

## References

- [OpenAI GPT-4o-mini Pricing](https://openai.com/pricing)
- [Supabase Edge Functions](https://supabase.com/docs/guides/functions)
- [Apple App Store AI Guidelines](https://developer.apple.com/app-store/review/guidelines/#artificial-intelligence)
- Competitor Analysis: Forest, Centered, Opal, One Sec
