# Skills Journal — Product & Technical Design

**Status:** Design candidate for implementation review  
**Date:** 2026-08-21  
**Platforms:** iOS + Android  
**Repository:** `seriafrancesco/SkillsJournal`  
**Product principle:** A personal atlas of real-world capability, not a habit tracker with a skill-tree skin.

## 1. Product thesis

Skills Journal turns real-life learning into a persistent, explorable map of capability. Users choose a discipline, complete concrete practice challenges, provide evidence when useful, increase mastery in specific nodes, and unlock deeper branches.

The product's core fantasy is: **“I can see what I have actually learned.”**

The tree is the product. Progress bars, XP, achievements, AI and analytics are supporting systems.

### Primary user outcome

A user should be able to answer three questions immediately:

1. What am I learning?
2. What should I practice next?
3. How has my capability grown over time?

### Core loop

`Discover → Choose → Practice → Verify → Progress → Unlock`

### Non-goals for MVP

- Social feed or public community.
- Competitive leaderboards.
- Marketplace for user-generated trees.
- Chat between users.
- Fully AI-generated progression rules.
- Medical, financial or professional certification claims.
- Punitive streaks.
- Complex gamified currencies.

## 2. Product scope

### MVP pillars

1. **Atlas** — browse owned disciplines and enter their trees.
2. **Interactive skill tree** — pan/zoom, branches, prerequisites and node states.
3. **Challenge engine** — concrete practice tasks with multiple verification modes.
4. **Deterministic progression** — mastery and unlock rules are computed from explicit requirements.
5. **Practice view** — a focused answer to “what can I do now?”.
6. **Progress view** — meaningful history, depth and breadth rather than dashboard vanity metrics.
7. **Account + sync** — progress available across devices.
8. **Offline-tolerant reading and completion** — recent tree content remains usable; safe mutations queue for sync.

### Post-MVP expansion

- Custom trees.
- AI Coach.
- AI-assisted tree drafting.
- Evidence analysis.
- Achievements.
- Reminders and non-punitive practice rhythm.
- Shareable completion cards.
- Premium paths.
- Unlimited trees.
- Advanced analytics.

## 3. Product language

The app uses four main progression concepts.

### Discipline

A broad field such as Cooking, Photography, Programming, Drawing or Spanish.

### Skill node

A specific capability inside a discipline, such as Knife Skills, Fresh Pasta or Portrait Lighting.

### Mastery

Depth of capability in one skill node. Mastery is not interchangeable with XP.

### XP

A lightweight account-level activity measure. XP may support milestones and delight, but never proves skill mastery by itself.

## 4. Verification model

Every challenge has an explicit verification type.

Supported MVP verification types:

- `self` — user confirms completion.
- `count` — user reaches a numeric target.
- `timer` — practice duration is measured.
- `quiz` — deterministic structured questions.
- `photo` — image evidence is attached.
- `evidence` — text/file/reference evidence is attached.
- `milestone` — completion is derived from other requirements.
- `compound` — multiple requirements are combined.

Verification strictness is separate from verification type:

- `self`
- `optional_evidence`
- `required_evidence`

AI may interpret evidence later, but AI does not directly mutate mastery or unlock state. It produces a recommendation/assessment consumed by deterministic rules.

## 5. Progression rules

Progression is server-authoritative when online and reproducible locally for known static path definitions.

Each node declares:

```ts
interface SkillNodeDefinition {
  id: string;
  disciplineId: string;
  parentIds: string[];
  title: string;
  summary: string;
  masteryLevels: MasteryLevelDefinition[];
  unlockRule: UnlockRule;
  display: NodeDisplayDefinition;
}
```

Each mastery level declares explicit requirements:

```ts
interface MasteryLevelDefinition {
  level: 1 | 2 | 3 | 4 | 5;
  requiredChallengeIds: string[];
  minimumCompletedChallenges?: number;
  prerequisiteNodeLevels?: Array<{
    nodeId: string;
    minimumLevel: number;
  }>;
}
```

### Node states

- `hidden`
- `discovered`
- `locked`
- `available`
- `started`
- `completed`
- `mastered`

Colour is never the only state signal.

## 6. Information architecture

Primary navigation has four destinations.

### Atlas

The default landing destination. Shows the user's disciplines, their visual growth and the currently relevant branches.

Navigation levels:

1. Atlas overview.
2. Discipline tree.
3. Node detail.
4. Challenge detail/completion.

### Practice

A prioritised list of currently actionable challenges. It may be filtered by context such as quick, project, indoors or outdoors, but filters remain intentionally limited.

### Progress

Shows meaningful longitudinal information:

- mastered nodes;
- mastery depth by discipline;
- breadth across disciplines;
- practice time where measured;
- recent milestones;
- historical learning activity.

### You

Profile, preferences, accessibility, notifications, account, subscription, privacy, legal and support.

## 7. Visual direction

### Theme: Living Atlas + Field Journal

The app should feel like a contemporary personal atlas that becomes richer as the user learns.

It must not default to:

- generic dark navy dashboard;
- purple/blue AI gradients;
- glassmorphism;
- excessive glow;
- oversized rounded cards;
- icons automatically enclosed in circles;
- generic neo-grotesque typography everywhere;
- decorative particles.

### Tree geometry as information

Branches communicate state.

- Fine line: possible path.
- Strong line: travelled/unlocked path.
- Interrupted line: blocked prerequisite.
- Branch split: specialisation.
- Permanent mark: milestone/mastery.

### Node rendering

Nodes are integrated into the canvas rather than represented as repeated floating cards. Touch targets may be larger than their visible geometry.

### Typography

Two intentional roles:

1. Editorial display face for discipline names, major mastery values and selected large headings.
2. Highly legible UI face/system-native typography for navigation, controls, metadata and helper copy.

No Trebuchet MS. No small Georgia/Times utility text. Typography must be validated on-device before final font licensing/selection is frozen.

### Colour

Base surfaces use warm paper/bone and graphite/ink in light mode, with charcoal/ink equivalents in dark mode.

Disciplines receive one restrained pigment family for orientation. Meaningful state must also be represented using geometry/iconography/text.

### Motion

Motion explains growth and continuity.

Challenge completion sequence:

1. immediate pressed feedback;
2. small haptic where enabled;
3. node state transition;
4. branch extends to newly unlocked content;
5. new node appears in spatially coherent position.

No mandatory celebratory wait. Reduced-motion mode substitutes fades/state changes without spatial traversal.

## 8. Accessibility

Minimum requirements from first implementation:

- Screen-reader labels for all tree nodes and icon-only controls.
- Equivalent non-canvas/list navigation for tree actions where canvas semantics are insufficient.
- Minimum appropriate touch targets without visually bloating controls.
- Text scaling support.
- High-contrast compatible state treatment.
- Reduced motion support.
- Logical focus order.
- No colour-only mastery or lock indication.
- Dynamic changes announced where necessary.

The interactive tree must never be the only way to access a node. A structured accessible outline/list must expose equivalent content and actions.

## 9. Technical architecture

### Client

- Expo SDK 57.
- React Native 0.86.
- React 19.2.x.
- TypeScript strict.
- Expo Router.
- Development Build used early, not Expo Go as a production constraint.

### Client organisation

Feature-first boundaries:

```text
app/                     route composition only
src/
  features/
    atlas/
    practice/
    progression/
    challenges/
    auth/
    profile/
    subscription/
  design-system/
  platform/
  data/
  config/
  test/
assets/
supabase/
  migrations/
  seed/
docs/
```

Routes must not own business rules. Domain functions remain testable without rendering React components.

### Backend

Supabase is the default backend because the product needs account identity, cross-device progress, structured relational data, row-level security, optional evidence storage and later Edge Functions for AI/server tasks.

Use:

- Supabase Postgres.
- Supabase Auth.
- Row Level Security.
- Supabase Storage only when evidence uploads are enabled.
- Edge Functions/server-side code for AI and secret-bearing integrations.

The client may contain the Supabase project URL and publishable key. Security relies on authentication, RLS and server-side authorization, never secrecy of the client key.

No `service_role`, AI provider secret or payment secret may exist in the client or Git history.

## 10. Data model

### Auth-owned identity

`auth.users` is the account identity source.

### profiles

- `id uuid PK references auth.users`
- `display_name text`
- `locale text`
- `timezone text`
- `created_at timestamptz`
- `updated_at timestamptz`

### disciplines

Curated path metadata.

- `id uuid PK`
- `slug text unique`
- `title text`
- `summary text`
- `version integer`
- `is_premium boolean`
- `status text`

### skill_nodes

- `id uuid PK`
- `discipline_id uuid FK`
- `slug text`
- `title text`
- `summary text`
- `position_x numeric`
- `position_y numeric`
- `max_mastery smallint`
- `definition_version integer`

### node_edges

- `discipline_id uuid FK`
- `from_node_id uuid FK`
- `to_node_id uuid FK`
- `edge_type text`

### challenges

- `id uuid PK`
- `node_id uuid FK`
- `title text`
- `instructions text`
- `verification_type text`
- `verification_policy jsonb`
- `xp integer`
- `definition_version integer`

### user_discipline_enrolments

- `user_id uuid FK`
- `discipline_id uuid FK`
- `enrolled_at timestamptz`
- `archived_at timestamptz nullable`

### challenge_attempts

Append-oriented user activity.

- `id uuid PK`
- `user_id uuid FK`
- `challenge_id uuid FK`
- `client_operation_id uuid unique per user`
- `status text`
- `started_at timestamptz`
- `completed_at timestamptz nullable`
- `duration_seconds integer nullable`
- `response jsonb`
- `created_at timestamptz`

### evidence_items

- `id uuid PK`
- `user_id uuid FK`
- `attempt_id uuid FK`
- `kind text`
- `storage_path text nullable`
- `text_value text nullable`
- `created_at timestamptz`

### user_node_progress

Server-computed/validated projection for fast reads.

- `user_id uuid FK`
- `node_id uuid FK`
- `mastery_level smallint`
- `state text`
- `progress_version integer`
- `updated_at timestamptz`

### subscriptions

Do not trust client-derived entitlement state. Persist only provider-derived status required for server decisions.

## 11. Authorisation and RLS

Default-deny posture.

Public/curated definitions may be readable by authenticated users and, if product onboarding requires it, selectively readable without auth.

User-owned records must enforce `auth.uid() = user_id`.

Evidence storage uses per-user object ownership policies.

Progress mutations are not direct arbitrary writes from the client. The server validates challenge completion and recomputes affected node progression.

Administrative/curation writes never use normal client credentials.

## 12. Offline and sync

The app is offline-tolerant, not fully peer-to-peer offline-first in MVP.

### Cached locally

- enrolled discipline definitions;
- skill node/edge definitions;
- recent progress projection;
- actionable practice items;
- pending safe challenge completion operations.

### Local database

Use `expo-sqlite` for structured cached content and queued operations.

### Mutation strategy

Each locally queued mutation receives a stable `client_operation_id` UUID. Server-side uniqueness makes completion submission idempotent.

Safe queued mutations can retry with backoff. Evidence uploads remain explicitly pending until storage succeeds.

### Conflict model

Challenge attempts are append-oriented. Mastery is a derived projection, so two-device completions merge through server recomputation rather than last-write-wins mastery counters.

Curated path definitions carry versions. A client never rewrites a current server definition using an older cached copy.

## 13. Authentication

MVP may support browsing a limited sample path before sign-in, but cloud progress requires an account.

Recommended first account methods:

- Apple where required/appropriate on iOS.
- Google.
- Email magic link or OTP as a neutral fallback.

Account creation automatically includes an in-app deletion path. A public web deletion-request resource is also required for Google Play compliance when account creation ships.

Session persistence must use the platform-appropriate secure/current Supabase integration pattern and be validated on real devices.

## 14. Subscription architecture

Premium features include later:

- unlimited active trees;
- premium curated paths;
- AI Coach quota;
- custom tree generation;
- advanced analytics.

Digital subscriptions use Apple IAP and Google Play Billing. RevenueCat is the preferred entitlement abstraction unless a later implementation review identifies a concrete reason to own billing infrastructure directly.

Entitlement checks must be server-verifiable for premium server resources such as AI calls.

No subscription implementation belongs in the first atlas vertical slice beyond interfaces/feature boundaries necessary to avoid architectural coupling.

## 15. AI boundary

AI is intentionally downstream of deterministic domain logic.

Allowed AI responsibilities:

- propose custom tree drafts;
- propose challenges;
- rewrite challenge difficulty/wording;
- explain concepts;
- coach reflection;
- assess optional evidence and return structured recommendations.

Disallowed AI responsibilities:

- directly set mastery;
- directly unlock nodes;
- decide payment entitlement;
- bypass prerequisites;
- issue professional certification claims;
- execute arbitrary tools from model output.

Architecture:

`Mobile client → authenticated Edge Function/backend → provider AI`

Provider secrets exist only server-side. Every AI path has per-user quota, timeout, model/version logging, structured output validation and cost bounds.

AI output is treated as untrusted input.

AI is not required for the MVP core loop to function.

## 16. Curated content model

Start with a small number of deep, manually reviewed disciplines rather than dozens of shallow generated trees.

Initial seed target for implementation testing:

- Cooking as the canonical complete vertical slice.
- A second small contrasting discipline such as Photography to validate that the engine is not hard-coded to cooking.

Content definitions must be versioned and separable from user progress.

## 17. Error handling

Every data surface defines:

- loading;
- success;
- empty;
- offline;
- permission denied where applicable;
- recoverable error;
- unrecoverable error.

Technical errors never leak raw SQL, stack traces, tokens or provider internals to users.

Queued offline operations show pending state and eventual reconciliation rather than pretending server success.

## 18. Observability and analytics

### Crash monitoring

Add Sentry or equivalent before production beta, not necessarily in the first scaffold commit.

### Product analytics

Only one analytics provider if/when required. Initial product instrumentation schema should prioritise:

- onboarding completion;
- discipline enrolment;
- first challenge completion;
- first mastery increase;
- 7/30-day return;
- practice selection to completion;
- sync failures.

Do not collect evidence payloads, private reflection text or sensitive content into analytics.

## 19. Performance budgets

Initial budgets to validate and refine on realistic Android hardware:

- Atlas overview interactive quickly after app shell loads.
- Tree interactions target smooth 60 fps under normal node counts.
- No whole-screen re-render for a single node completion where avoidable.
- Discipline definitions are paged/versioned and cached.
- Large trees must not render hundreds of complex React subtrees unnecessarily.

Tree rendering implementation is chosen only after a small benchmark comparing standard RN views/SVG/Skia for the actual interaction requirements. Do not install Skia solely because the product contains a graph.

## 20. Testing strategy

### Domain tests

Highest priority:

- prerequisite evaluation;
- mastery calculation;
- challenge completion validation;
- idempotent mutation behaviour;
- path-version handling;
- practice eligibility selection.

### Integration tests

- Supabase RLS policies.
- Auth session lifecycle.
- queued completion sync.
- evidence ownership.
- migration correctness.

### UI/component tests

- node state semantics;
- accessible labels;
- loading/error/empty states;
- reduced-motion behaviour.

### E2E

At minimum before beta:

1. onboarding → choose Cooking → enter tree;
2. open available node → complete challenge → mastery updates → branch unlocks;
3. complete offline → reconnect → idempotent sync;
4. sign out/in → progress restored;
5. account deletion flow when accounts ship;
6. subscription purchase/restore once monetisation ships.

## 21. Security baseline

- `.env`, private keys, service accounts and credentials ignored before normal development begins.
- `.env.example` contains names only.
- Secret scanning in CI/pre-push workflow.
- No `service_role` client-side.
- RLS enabled before user data is relied upon.
- Input validation on server boundaries.
- Evidence uploads constrained by MIME/size/ownership.
- Minimal permissions.
- HTTPS only.
- Dependency review before release.
- No sensitive values in logs or analytics.

## 22. Git and repository workflow

Use short-lived trunk-based feature branches.

`main` stays releasable.

Examples:

- `chore/bootstrap-app`
- `feat/progression-core`
- `feat/atlas-core`
- `feat/challenge-completion`
- `feat/supabase-sync`
- `fix/...`

Workflow:

1. branch from current `main`;
2. implement one independently reviewable slice;
3. typecheck/lint/tests/secret scan;
4. draft PR;
5. review diff and CI;
6. squash/merge once green;
7. delete branch.

Do not maintain a permanent `develop` branch.

## 23. CI quality gate

GitHub Actions should eventually block merge on:

- dependency install with locked versions;
- TypeScript typecheck;
- ESLint;
- unit tests;
- secret scan;
- migration/static validation where available.

Store builds remain separate from every-PR CI until needed.

## 24. Implementation sequence

This specification is intentionally decomposed into independently testable slices.

### Slice 1 — Foundation + progression core

- Expo/Router/TypeScript scaffold.
- design tokens and app shell.
- typed curated content fixture.
- deterministic progression engine with tests.
- basic accessible Atlas/Tree/Node vertical navigation using local data.

### Slice 2 — Challenge completion

- challenge detail.
- self/count/timer MVP verification.
- completion transaction.
- tree state transition.
- local persistence.

### Slice 3 — Supabase account + sync

- migrations/RLS.
- auth.
- enrolments/attempts/progress projection.
- idempotent sync queue.

### Slice 4 — Evidence

- photo/text evidence.
- storage policy.
- upload pending/retry states.

### Slice 5 — Practice + Progress

- eligibility selector.
- practice presentation.
- longitudinal progress views.

### Slice 6 — Release hardening

- crash reporting.
- accessibility audit.
- E2E.
- store privacy/support/account deletion infrastructure.
- performance benchmark.

### Later slices

- subscription.
- AI Coach.
- custom trees.

## 25. Definition of Done for MVP

MVP is not “done” until:

- user can understand Atlas without external explanation;
- deterministic rules explain every unlock/mastery change;
- Cooking vertical slice works end-to-end;
- a second discipline validates generic engine behaviour;
- core flow works on iOS and Android Development Builds;
- offline completion survives restart and reconciles safely;
- cross-device progress restores after authentication;
- RLS isolation tests pass;
- accessible alternative tree navigation exists;
- reduced motion works;
- critical states have loading/empty/error/offline handling;
- typecheck/lint/tests/secret scan pass;
- no secret exists in client or Git history;
- real-device smoke tests pass;
- release/store requirements are rechecked at release date.

## 26. Open implementation choices intentionally deferred

These are not product ambiguities; they are benchmark/configuration decisions to make during their owning slice:

- exact display and UI font families after real-device typography comparison and licence review;
- RN/SVG/Skia tree renderer after a representative interaction benchmark;
- exact analytics provider before product analytics is needed;
- final RevenueCat adoption at monetisation slice review;
- final AI provider/model at AI slice review.

Deferring these prevents premature dependencies while preserving explicit interfaces and quality requirements.
