# OpenMemory Demo: Before and After Sleep

This walkthrough shows what happens when an agent's memories go through a sleep cycle.

## Setup

Three conversations are ingested over the course of a day:

### Conversation 1 (9:00 AM) — Deployment failure

```json
{
  "content": "The staging deploy failed again. CI passed but the app crashed on startup because DATABASE_URL wasn't set in the Fly.io secrets. I spent 45 minutes debugging before realizing it was a config issue, not a code issue.",
  "source": "user_message",
  "channel": "slack"
}
```

### Conversation 2 (11:30 AM) — Team standup

```json
{
  "content": "Alex from DevOps mentioned he's frustrated that deploys happen without a heads-up. He prefers a Slack ping in #deploys at least 10 minutes before pushing to staging. He also said the config check script exists but nobody uses it — it's at scripts/check-config.sh.",
  "source": "user_message",
  "channel": "slack"
}
```

### Conversation 3 (3:00 PM) — Process discussion

```json
{
  "content": "We agreed to create a Friday deploy checklist. The checklist should cover: run config check script, ping #deploys channel, verify secrets are set, run smoke tests after deploy. Sarah will own the doc.",
  "source": "user_message",
  "channel": "slack"
}
```

## Before Sleep: Plain Vector Retrieval

Query: *"What should I do before deploying?"*

```
Results (vector similarity only):
1. "We agreed to create a Friday deploy checklist..." (0.87 similarity)
2. "The staging deploy failed again..." (0.72 similarity)
3. "Alex from DevOps mentioned he's frustrated..." (0.68 similarity)
```

Three raw text chunks, ranked by cosine similarity. No connections. No lessons. No understanding of *why* these matter together.

## Sleep Cycle Runs

### Session sleep (~30 seconds each)

After each conversation:
- Entities extracted: `staging`, `DATABASE_URL`, `Fly.io`, `Alex`, `DevOps`, `Sarah`, `#deploys`
- Sentiment scored: Conversation 1 = frustrated (valence: -0.6), Conversation 2 = concerned (valence: -0.3), Conversation 3 = determined (valence: 0.4)
- Knowledge graph updated: new nodes + edges created
- Embeddings generated for all new content

### Nightly sleep (~3 minutes)

The LLM processes all three episodes together and:

**Lessons extracted:**
- "Always run `scripts/check-config.sh` before deploying — config issues masquerade as code bugs" (severity: important)
- "Ping #deploys channel before pushing — Alex and DevOps team need advance notice" (severity: important)

**Person model updated (Alex):**
- Role: DevOps engineer
- Communication preference: Slack, advance notice
- Known frustration: surprise deployments
- Preferred channel: #deploys

**Procedure created:**
- Name: "Pre-deployment checklist"
- Trigger: any deploy-related conversation
- Steps: (1) Run config check script, (2) Ping #deploys, (3) Verify secrets, (4) Deploy, (5) Smoke test
- Owner: Sarah
- Success rate: pending first execution

**Cross-layer edges built:**
- Lesson about config checks → links to the episode where it was learned → links to the procedure that prevents it
- Alex's frustration → links to the #deploys notification step → links to his person model
- The deploy failure episode → links to the checklist creation episode (causal: failure led to process improvement)

## After Sleep: Multi-Layer Query

Same query: *"What should I do before deploying?"*

```
Results (multi-layer retrieval):

📋 PROCEDURE: Pre-deployment checklist (confidence: 0.85)
   1. Run scripts/check-config.sh (prevents config-as-code-bug confusion)
   2. Ping #deploys channel (Alex needs 10min advance notice)
   3. Verify secrets are set in target environment
   4. Deploy
   5. Run smoke tests
   Owner: Sarah | Created: from deployment failure on Monday

💡 LESSON: Always run config check before deploying (importance: 0.8)
   "Config issues masquerade as code bugs — spent 45 minutes debugging
    what turned out to be a missing DATABASE_URL"
   Learned from: Episode on Monday morning
   Prevention: Run scripts/check-config.sh

👤 PERSON: Alex (DevOps) (trust: 0.7)
   Prefers: Slack notification in #deploys, 10+ min before deploy
   Known frustration: surprise deployments
   Preferred style: direct, advance notice

📝 EPISODE: Deploy failure → Process improvement (emotional arc: frustrated → determined)
   Started with a 45-min debugging session, ended with team agreement
   on a structured checklist. Resolution: positive — failure led to
   better process.

🔗 CONNECTIONS:
   - The config failure CAUSED the checklist creation
   - Alex's feedback INFLUENCED the notification step
   - The checklist ADDRESSES both the technical issue AND the team dynamic
```

## The Difference

| Aspect | Vector retrieval | After sleep |
|--------|-----------------|-------------|
| Results | 3 text chunks | Procedure + lesson + person + episode + connections |
| Understanding | "These texts are similar" | "Here's what to do, why, who cares, and how we got here" |
| Actionable? | You'd have to read and synthesize 3 chunks | Ready-to-execute checklist with context |
| Relationships | None | Causal chains, person preferences, emotional arcs |
| Learning | None | Extracted lesson prevents repeating the mistake |

**That's the difference between a filing cabinet and a brain.**
