# Diagram Source Notes

Source brief: `Project/spk2challenge-miniprogram/Solution/speaking-challenge-and-scoring/LATEST.md`

Skill references loaded:

- `skills/fireworks-tech-graph/SKILL.md`
- `skills/fireworks-tech-graph/references/style-6-claude-official.md`

## Extracted Structure

Layers:

- Client: Mini Program, Result Page, Website Entry
- Gateway / BFF: Challenge BFF, Upload Gateway, Website Router, Identity Context
- Application: Scoring Job, Result Query, Handoff Builder
- AI + Content: AI Scoring, Band / CEFR Mapper, Content Rules
- Data / Ops: Object Storage, Scoring DB, Analytics, Monitoring

Key flows:

- Challenge topic load
- Audio upload and storage
- Idempotent scoring task creation
- AI scoring and result mapping
- Short polling result retrieval
- Website handoff using source and score bucket
- Analytics and monitoring writes
