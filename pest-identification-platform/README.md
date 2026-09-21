# Seeing Beyond the Leaf: Reinventing Crop Protection Diagnosis with Multimodal AI on AWS

---

## Executive Summary

Tea and specialty-crop plantations lose yield every season to pests and diseases that go unidentified — or are misidentified — until the damage is already done. The bottleneck has never been a lack of concern; it has been a lack of *expertise at the point of need*. A qualified plant pathologist or senior agronomist can look at a damaged leaf and, within seconds, narrow the cause to a short list of likely pests or diseases. Field supervisors, estate managers, and frontline plantation workers — the people who are actually standing in the field when damage is spotted — usually cannot. By the time a photo reaches an expert, days may have passed, and a treatable early-stage infestation can have become a plantation-wide outbreak.

This case study describes the architecture and strategic rationale behind a proof-of-concept **AI-powered visual diagnostic platform** built to close that gap: a system that lets anyone with a smartphone photograph a damaged leaf, stem, or fruit and receive, in seconds, a structured, expert-grade diagnosis — pest or disease identity, affected plant part, seasonal behavior, symptoms, and recommended treatment — without needing to know what they are looking at, and without needing to file-match or manually search a database.

The reason this problem has resisted simple automation is that it is deceptively hard. Photographs of crop damage are not clean, cropped, single-subject product images. They are taken in the field, at odd angles, in variable light, against backgrounds of soil, grass, fingers, and table surfaces, and — critically — with no annotation of *where* on the leaf the diagnostic damage actually is. Naïve approaches fail for three compounded reasons: whole-image embeddings are diluted by irrelevant background; visual similarity alone cannot distinguish two pests that happen to look alike from a distance; and a single reference photo per pest or disease category is not enough to represent how varied real-world damage photos actually look.

The solution combines four AI techniques into a single, purpose-built pipeline rather than relying on any one of them in isolation. Every photo — reference or query — first passes through **automatic, color-agnostic background removal**, deliberately avoiding a hue-based mask, because the damage this system needs to detect is *itself* frequently a color deviation from healthy green tissue; a color filter would risk erasing the very signal it is meant to isolate. The cleaned image is then broken into **five content-aware crop variants** using a sliding-window edge-density scoring technique, so that a diagnostic region anywhere in the frame is captured tightly rather than diluted inside one large, fixed frame. Each variant is embedded using **Amazon Bedrock's Titan Multimodal Embeddings model** and indexed into **Amazon OpenSearch's k-NN vector engine**. In parallel, a **Bedrock vision-capable large language model** extracts structured, plant-part-aware attributes — texture, lesion shape, color, and a freeform description — using language-based reasoning rather than pixel-based comparison alone. At query time, a fifth AI step — an **LLM evaluator** — weighs raw image similarity against these structured attributes and against the admin's own fact-checked, authoritative description of each candidate, and requires all three signals to agree before returning a confident match. Anything short of that consensus is returned honestly as "unidentified," logged for expert review, and enriched in the background with the model's own best-effort guess — never fabricated, never shown to the end user, and never silently added to the knowledge base without human sign-off.

The result is a system architected the way a production platform would be, even while it runs, for demonstration purposes, as a single process on a laptop: every data-bearing component — embeddings, vector search, structured metadata, images, and the human-review queue — is a real, managed AWS service (Bedrock, OpenSearch, DynamoDB, S3), so the path from proof-of-concept to production is a matter of swapping the compute and identity layers, not re-architecting the intelligence layer that took the most work to get right.

Strategically, this platform reframes crop protection from a scarce-expert bottleneck into a scalable, self-improving knowledge system: every unidentified case becomes an opportunity to expand what the system knows, every confirmed match reduces the load on human experts, and every admin correction makes the next diagnosis better. For an organization managing plantations across multiple estates and geographies, that is the difference between crop protection expertise that lives in a handful of people's heads and crop protection expertise that lives in an asset the organization owns.

---

## Customer / Business Background

### Industry Overview

Tea and other high-value specialty crops are grown across large, geographically distributed estates, often spanning varied microclimates, elevations, and soil conditions within a single operating group. Crop health directly drives yield, quality grade, and ultimately revenue per hectare — making early pest and disease detection one of the highest-leverage operational levers available to an estate management team.

### Business Environment

Plantation operations are inherently decentralized. Damage is first observed not in a lab or an agronomy office, but in the field, by pluckers, supervisors, and estate-level staff — the people with the least formal training in plant pathology, but the earliest possible visibility into a developing problem.

### Operational Context

Historically, diagnosis has depended on a small number of trained agronomists or plant pathologists who cannot be physically present on every estate, every day. Photos or verbal descriptions are relayed up a chain of communication before an expert weighs in — a workflow that was already slow before accounting for the fact that many pest/disease presentations are visually ambiguous even to trained eyes without side-by-side reference material.

### Existing Challenges

Field staff have no structured way to self-serve a first-pass diagnosis. Existing digital tools, where they exist at all, rely on exact filename or catalog lookups rather than recognizing "this looks like that" the way a human expert does — so a new photo of a known pest, taken from a different angle or in different light, simply does not match anything in a keyword- or ID-based system.

### Why Transformation Was Needed

The organization needed a way to put expert-level, first-pass diagnostic judgment directly into the hands of frontline staff — without requiring every estate to have its own resident plant pathologist, and without requiring the knowledge base itself to be rebuilt from scratch every time a new pest or disease needed to be added. Doing that credibly meant solving a harder problem than simple image search: recognizing visual similarity *and* reasoning about it the way a domain expert would, while remaining honest about the cases where it genuinely does not know.

---

## Business Challenges

### Challenge 1 — Diagnostic Expertise Is Scarce and Centralized

**Situation.** A small number of agronomists/plant pathologists must serve every estate in the group. Their time is the binding constraint on how quickly any given damage report gets a real diagnosis.

**Impact.** Diagnosis latency is measured in the time it takes to relay a photo or description up the chain and get a qualified answer back — not the time it takes to notice the damage.

**Risk.** Fast-moving pest and disease outbreaks can progress from a treatable, localized presentation to a plantation-wide problem in the window between "spotted" and "diagnosed."

**Business Consequence.** Yield loss, increased treatment cost (broad-spectrum reactive spraying instead of targeted early intervention), and quality grade degradation on affected lots.

### Challenge 2 — Visual Diagnosis Does Not Reduce to Filename or Catalog Lookup

**Situation.** Legacy digital reference tools, where present, work by exact ID or keyword lookup — useful only to someone who already knows roughly what they are looking at.

**Impact.** Field staff photographing an unfamiliar pest have no way to search "what does this look like" against a reference library; the tool is only useful after the diagnosis has already happened by other means.

**Risk.** The reference material an organization has painstakingly built (verified descriptions, treatment protocols, seasonal behavior notes) sits underused because there is no way to *find* the right entry from a photo alone.

**Business Consequence.** Institutional knowledge fails to reach the point of need, undermining the return on the effort already invested in documenting it.

### Challenge 3 — Real-World Field Photos Defeat Naïve Image Matching

**Situation.** Field photos are taken at arbitrary angles, in variable lighting, with the diagnostic region (a lesion, a pest, a damaged leaf edge) unmarked and often occupying only a small fraction of the frame — surrounded by soil, grass, hands, or table surfaces.

**Impact.** A single whole-image embedding compared against a single reference photo is unreliable precisely because most of the frame is irrelevant, and two visually similar photos of *different* plant parts or *different* damage types can still score deceptively close on raw pixel similarity.

**Risk.** A system that trusts raw similarity alone will occasionally hand out confident-sounding wrong answers — which is more damaging to trust and to crop outcomes than a system that honestly says "I don't know."

**Business Consequence.** Wrong or overconfident automated advice risks incorrect treatment application — a direct cost and, in the worst case, a crop-health risk in its own right.

**Challenge Summary Matrix**

| Challenge | Impact |
|---|---|
| Diagnostic expertise is scarce and centralized | Diagnosis latency scales with expert availability, not damage-detection speed |
| Visual diagnosis does not reduce to filename/catalog lookup | Existing reference knowledge is unreachable from a photo alone |
| Real-world field photos defeat naïve image matching | Naïve automation risks confident wrong answers, which are worse than no answer |

---

## Vision and Objectives

### Business Objectives
Put expert-level, first-pass diagnostic judgment directly into the hands of every estate, independent of where the nearest human expert happens to be — while making every case, confidently resolved or not, an input that grows the organization's own knowledge base rather than a one-off answer that disappears.

### User Objectives
Let a non-expert take a photo and get a structured, actionable answer — or an honest "unidentified" — in seconds, with no filename lookups, no keyword search, and no need to already know what they are looking at.

### Technology Objectives
Build the intelligence layer on managed, production-grade cloud services from day one, so the proof-of-concept's data plane (embeddings, vector search, metadata, images) never has to be rebuilt when the project moves beyond a demo.

### AI Objectives
Combine complementary AI techniques — segmentation, multi-crop embedding, vision-LLM attribute extraction, and LLM-based evaluation — so that no single technique's blind spot becomes the system's blind spot, and so that the system can recognize the limits of its own confidence rather than always producing an answer.

### Operational Objectives
Give administrators (agronomists) a workflow that mirrors how they already think about the problem — separate "Add Pest" / "Add Disease" intake, photo management independent of record management, and a review queue for anything the system could not confidently resolve — so the platform strengthens expert workflows rather than replacing them.

---

## Solution Overview

### Existing State

Diagnosis is a manual, expert-mediated, communication-latency-bound process. Reference knowledge, where documented, is organized for lookup by someone who already knows what they're looking for, not for recognition from an unlabeled photo. Every unfamiliar case is, by default, a dead end until it reaches a human expert — there is no structured mechanism to capture "we didn't know what this was" as a data point to act on later.

### Future State

A field worker photographs the damage. Within seconds, the system either returns a structured diagnosis — identity, affected part, seasonal incidence, symptoms, prevention, treatment, and chemical recommendation, each traceable to an admin-verified record — or honestly reports that nothing in the knowledge base is a confident match. In the second case, the case is automatically queued for expert review, pre-enriched with the AI's own best-effort (clearly labeled as unverified) guess at what it might be, so the expert's first step is validation rather than starting from a blank page. Every confirmed diagnosis and every resolved "unidentified" case makes the knowledge base measurably more complete than it was the day before.

### Business Transformation Narrative

This is not a story about replacing agronomists with AI. It is a story about compressing the distance between *noticing* damage and *acting correctly* on it — and about turning every "we don't know" moment, which used to simply be lost, into a durable, searchable input to the organization's own expanding body of crop-protection knowledge.

---

## Architecture Overview

### Presentation Layer
A lightweight, dependency-free web interface serves two distinct experiences from the same shell: an admin console (structured intake forms, records list, unidentified-case review) and a user-facing upload-and-result flow. Keeping these visually and structurally distinct — rather than one generic form — mirrors how each role actually thinks about the problem, reducing training and error.

**Business value:** Admins get a purpose-built curation tool instead of a generic CRUD form; field users get a two-tap "photo in, diagnosis out" experience with no learning curve.

### Application Layer
A single orchestration layer coordinates every step between "photo received" and "answer returned" — authentication, validation, and the sequence of AI and storage calls that make up ingestion and search.

**Business value:** All business rules (what "confident" means, what admins can and cannot edit, what happens on partial failure) live in one place, so behavior is consistent and auditable regardless of which screen triggered it.

### AI Intelligence Layer
The reasoning core: background removal, multi-crop embedding, vision-LLM attribute extraction, and LLM-based evaluation, each contributing a different, complementary signal.

**Business value:** No single point of AI failure — a weakness in one technique (e.g., raw similarity being fooled by a lookalike) is caught by another (the evaluator weighing structured attributes and verified text).

### Search Layer
A vector similarity engine finds visually close candidates across every reference photo and every crop variant ever ingested, without requiring exact matches.

**Business value:** New photos of a known pest — different angle, different light, different plant — still find their match, which is the entire premise the platform depends on.

### Data Layer
Structured pest/disease records, the admin-curated "ground truth" description, the raw image archive, and the human-review queue are each held in a purpose-fit store, all keyed by a shared identifier.

**Business value:** Each dataset scales and is secured independently, and the admin's fact-checked knowledge is always retrievable and treated as authoritative — never overwritten by an unverified AI guess.

### Cloud Infrastructure Layer
Every data-bearing component runs on managed AWS services from the first prototype, even though the orchestration process itself runs locally for demo convenience.

**Business value:** The parts of the system that are expensive to get wrong — data durability, vector search correctness, secrets handling at scale — are never "POC-quality"; only the compute/hosting layer is simplified for the demo, and that is the cheapest, fastest part to swap for production.

---

## Architecture Deep Dive

### Component: Background Removal

**Purpose.** Strip soil, grass, hands, and surfaces from the frame before embedding, so background never enters the similarity calculation.

**Design Rationale.** Color-agnostic segmentation was chosen over a hue mask because crop damage is often itself a color deviation from healthy tissue — a hue mask would risk erasing the signal it's meant to isolate.

**Key Responsibilities.** Runs locally ahead of every embedding call, for both admin uploads and queries; never modifies the stored original.

**Benefits.** Materially cleaner matches with zero manual annotation.

### Component: Content-Aware Multi-Crop Generation

**Purpose.** Capture the diagnostic region tightly, wherever it sits in an unmarked, unposed field photo, instead of diluting it inside one large frame.

**Design Rationale.** Rather than guessing one "right" crop or embedding every candidate window, a denser grid of sub-windows is scored locally and cheaply by edge density, and only the top scorers are sent to the embedding model.

**Key Responsibilities.** Produces five variants per photo — the full frame plus four top-scoring sub-windows — for both reference and query photos.

**Benefits.** Most of the accuracy benefit of manual annotation, at zero annotation cost and bounded AI spend.

### Component: Embedding Service (Amazon Bedrock Titan Multimodal Embeddings)

**Purpose.** Convert each crop variant into a fixed-length vector capturing its visual semantics.

**Design Rationale.** A managed multimodal embedding model removes the need to train or host a custom vision model, while still delivering production-representative accuracy.

**Key Responsibilities.** Embeds every crop of every reference and query photo.

**Benefits.** State-of-the-art visual representation with zero model-ops burden.

### Component: Vision/Text LLM (Bedrock, Vision-Capable Model)

**Purpose.** Extract structured, damage-relevant attributes — plant part, texture, lesion shape, color, and a freeform description — and, at search time, reason over candidates the way an expert would.

**Design Rationale.** Pixel similarity has no notion of *which* plant part is shown or what the damage looks like structurally, so two dissimilar pests can score deceptively close. Language-based extraction adds an independent signal that generalizes to any plant part, not just the crop this app started with.

**Key Responsibilities.** Runs once per reference photo (captioning) and twice per query (captioning plus evaluation); instructed to treat a mismatched plant part, texture, shape, or color as a poor match even when raw similarity is highest.

**Benefits.** Catches exactly the failure raw similarity is blind to: two different pests that merely look alike.

### Component: Vector Store (Amazon OpenSearch, k-NN)

**Purpose.** Nearest-neighbor search across every embedded crop of every reference photo.

**Design Rationale.** A managed k-NN index (HNSW/FAISS, cosine similarity) scales independently of the app process, with no custom nearest-neighbor infrastructure to build.

**Key Responsibilities.** Returns best-effort similarity annotations; the app recomputes exact cosine similarity locally rather than trusting the engine's raw score, and every existing category remains a candidate even if OpenSearch didn't surface it.

**Benefits.** Fast, scalable similarity search with an optional pest/disease filter for callers who already know what they want.

**Tradeoff.** OpenSearch bills per instance-hour regardless of traffic — the single largest cost line in the architecture, and a direct candidate for OpenSearch Serverless in production.

### Component: Metadata Store (Amazon DynamoDB)

**Purpose.** Hold the structured, admin-curated record — shared fields, pest-only fields, disease-only fields, and a photo list with each photo's own extracted attributes.

**Design Rationale.** A schema-flexible, on-demand store fits a record shape that varies by category type without a rigid relational schema or capacity planning.

**Key Responsibilities.** Single source of truth for every field returned to the user, keyed to the same identifier used in the vector store.

**Benefits.** No capacity planning, pay-for-use economics, independent scaling from the app process.

### Component: Image Store (Amazon S3)

**Purpose.** Durable storage of every original photo, served back through time-limited presigned URLs.

**Design Rationale.** Storing the unmodified original — background removal is applied only transiently, ahead of embedding — preserves a source of truth for any future re-processing under an upgraded model.

**Benefits.** Cheap, durable, secure image hosting with no public bucket exposure.

### Component: Unidentified Review Queue (DynamoDB)

**Purpose.** Capture every non-confident search so it becomes a knowledge-base growth opportunity rather than a dead end.

**Design Rationale.** Parallel to, but separate from, the main records table — a staging area, never a searchable source until an admin promotes it.

**Key Responsibilities.** Stores the query photo, similarity, and caption, plus a background-enriched AI guess for admin review; supports direct promotion, reusing the already-captured photo.

**Benefits.** Converts every "unidentified" moment into a concrete admin task instead of a lost signal.

---

## End-to-End Processing Flow

![Pest Identification Platform - End-to-End Data Flow](Pest%20Identification%20Platform%20-%20End-to-End%20Data%20Flow.png)

1. **User action.** A field user opens the app and submits a photo — no filename, no category selection required (an optional pest/disease hint exists for the rare case the user already knows).
2. **Request processing.** The backend runs the same background removal and five-crop-variant generation used at ingestion time, ensuring the query is processed exactly as consistently as every reference photo in the database.
3. **AI interaction.** Bedrock embeds all five crop variants and, separately, extracts the query photo's structured attributes (plant part, texture, shape, color, description) from the original, unmodified image.
4. **Search process.** OpenSearch returns best-effort nearest neighbors per crop vector, optionally narrowed by category if a hint was supplied; the application recomputes exact similarity locally and keeps, per candidate category, the single best score across every crop-to-crop pairing — along with exactly which reference photo and which query crop produced it. Every currently existing category is still a candidate regardless of whether OpenSearch surfaced it.
5. **Validation.** The LLM evaluator receives the query's attributes and every candidate's image similarity, structured attributes, authoritative verified description, and unverified free text, and returns a best-fitting pick (or an explicit "none fit"), a numeric confidence, and a verified-description match judgment.
6. **Confidence scoring.** A result is only returned as a confident match if the evaluator's confidence clears its threshold, its verified-description judgment is true, *and* the candidate's raw similarity clears its own minimum — three independent signals that must all agree. If the evaluator itself fails to return something usable, a numeric fallback rule (top similarity plus a minimum lead over the runner-up) takes over so the system degrades gracefully rather than failing closed.
7. **Response generation.** A confident result returns the full structured record, presigned photo URLs, and a detection box mapped back onto the user's own submitted photo, showing exactly which region drove the match. An unconfident result returns an honest "unidentified" and is logged to the review queue; in the background, after the response has already been sent, the same vision LLM is asked directly whether it recognizes the subject from its own general knowledge, and that unverified lead is attached to the queue entry purely for the admin's benefit.

---

## AI Innovation Framework

![AI Decision & Reasoning Flow](AI%20Decision%20%26%20Reasoning%20Flow.png)

### Why Conventional Computer Vision Was Insufficient
A single whole-image embedding compared against a single reference vector is fragile against exactly the conditions field photography produces: irrelevant background dominating the frame, and no marked diagnostic region. Traditional computer vision pipelines built for clean, centered, single-subject images do not transfer to this problem without modification.

### Why Vector Search Alone Was Insufficient
Raw image similarity has no notion of *which plant part* is shown or *what the damage looks like structurally* — two visually similar photos of different plant parts, or different pests that merely resemble each other, can score deceptively close. Trusting the top similarity score alone would occasionally produce confident, wrong answers.

### Why LLM-Based Validation Was Introduced
An LLM evaluator adds the one thing pure similarity search cannot provide: reasoning that weighs structural, textual, and curated-knowledge evidence together, and that can explicitly decide "none of these genuinely fit" rather than being forced to rank a best-of-a-bad-lot answer as if it were confident.

### How Multiple AI Techniques Work Together
Background removal isolates signal from noise before anything else happens. Multi-crop embedding ensures that signal, wherever it sits in the frame, gets a fair shot at being represented. Vision-LLM attribute extraction adds a structurally different, language-based signal that raw pixels cannot provide. The LLM evaluator is the final arbiter, requiring image similarity, its own reasoning-based confidence, and an explicit match against the admin's fact-checked description to all agree before anything is called confident — with a deterministic numeric fallback standing behind it in case the reasoning step itself is unavailable.

---

## Key Innovations

### Innovation 1 — Color-Agnostic Background Removal

**Problem.** Field photos are dominated by irrelevant background, and the obvious fix — a color/hue mask — would risk erasing the diagnostic signal, since crop damage is frequently a color deviation from healthy tissue.

**Solution.** Generic, color-agnostic foreground segmentation, applied automatically before embedding, without touching the stored original.

**Benefit.** Cleaner embeddings with zero manual annotation, and no risk of the "fix" hiding the damage the system exists to find.

### Innovation 2 — Content-Aware Multi-Crop Embedding

**Problem.** The diagnostic region is never marked and can sit anywhere in an unposed photo; a single fixed crop or whole-image embedding dilutes it.

**Solution.** A denser grid of candidate sub-windows, scored locally by edge density, with only the top scorers sent to the embedding model — five variants per photo.

**Benefit.** Tight capture of the diagnostic region regardless of frame position, at bounded, predictable AI cost, with no manual annotation.

### Innovation 3 — Language-Based Attribute Extraction

**Problem.** Pixel similarity cannot distinguish *which* plant part is shown or describe damage structurally — a blind spot for any pure vector-search approach.

**Solution.** A vision-LLM extracts plant part, texture, lesion shape, and color as structured attributes — reasoning that generalizes to any plant part or crop.

**Benefit.** A second, independent evidence type that catches lookalike-but-different cases raw similarity would miss, reusable across future crops with no retraining.

### Innovation 4 — Triple-Signal Confidence Gating

**Problem.** Trusting one signal alone risks confident wrong answers, which damage trust more than an honest "I don't know."

**Solution.** A confident match requires all three to agree: the evaluator's numeric confidence, an explicit match against the admin's fact-checked `verified_description` (authoritative, on par with image evidence), and the candidate's raw similarity clearing its own floor. Unverified free text is weighted as weak supporting context only.

**Benefit.** The system can say "none of these genuinely fit" as an informed decision, never overridden by raw similarity alone — the behavior that makes it trustworthy enough to act on.

### Innovation 5 — Human-in-the-Loop Queue with AI-Assisted Lookup

**Problem.** Without a structured mechanism, every unresolved case is lost, and the knowledge base never grows from its own blind spots.

**Solution.** Every unresolved search is queued for review, and in the background — after the user already has their answer — the same vision LLM is asked directly whether it recognizes the subject from its own knowledge, presented to admins as an unverified suggestion only.

**Benefit.** Converts every knowledge gap into a low-friction admin task rather than a dead end, with no new external dependency beyond the Bedrock credentials already in use.

---

## AWS Architecture

![Pest Identification Platform AWS Architecture](Pest%20Identification%20Platform%20AWS%20Architecture.png)

### Amazon Bedrock

**Purpose.** Supplies both the multimodal embedding model (Titan Multimodal Embeddings) that powers similarity search, and the vision-capable LLM that performs attribute extraction, evaluation, and unidentified-case enrichment.

**Benefits.** A single managed service covers two structurally different AI capabilities (embedding and reasoning), with no model hosting, fine-tuning, or GPU infrastructure for the team to operate.

### OpenSearch

**Purpose.** k-NN vector index over every crop embedding of every reference photo, with optional metadata filtering by category.

**Benefits.** Purpose-built, horizontally scalable similarity search that scales independently of the application process; supports cosine similarity natively.

### DynamoDB

**Purpose.** Structured metadata storage for pest/disease records, the unidentified review queue, and admin-editable seasonal-instance configuration.

**Benefits.** Schema flexibility for records that vary by category type, on-demand billing that removes capacity planning at POC scale, and effortless independent scaling.

### Amazon S3

**Purpose.** Durable, private storage of every original reference and query photo, served through time-limited presigned URLs.

**Benefits.** Low-cost, durable object storage with no public exposure of plantation image data, and a stable source of truth independent of any preprocessing applied downstream.

### Integration Approach

The application orchestrates all four services through simple, direct API calls — no message queue or event bus, since ingestion and search are synchronous, request-scoped workflows. The one non-AWS step, background removal, runs locally because it's a pure preprocessing transform with no data-bearing responsibility, keeping every stateful piece on managed infrastructure.

### Overall Cloud Architecture Benefits

Every dataset worth protecting or scaling — vectors, metadata, images, the review queue — already lives on managed AWS services from the first prototype. The riskiest, hardest-to-retrofit part of a proof-of-concept, the data layer, never has to be rebuilt; only hosting and identity need to mature for production.

---

## Security Architecture

### Authentication
End users and admins authenticate via a signed session cookie for the bundled web UI (`/api/login`, `/api/search`). Credentials for the POC are sourced from environment configuration rather than a production identity provider.

### Authorisation
Roles are enforced at the route level — admin-only routes reject non-admin sessions. Admin routes also accept an entirely separate mechanism, a static `ADMIN_API_KEY` header, so a dedicated admin frontend can integrate directly without depending on the browser session — keeping both paths independently rotatable and revocable.

### API Security
A third static key (`APP_API_KEY`) gates a separate app-developer API surface, so the user UI, admin UI, and third-party integrations each hold credentials revocable without touching the others.

### Data Security
All AWS endpoints, credentials, and demo-account passwords are held in a gitignored environment file, never hardcoded in source.

### Image Security
Images are stored in a private S3 bucket and served exclusively through short-lived, time-limited presigned URLs — never through a permanently public link.

### Cloud Security Controls
The OpenSearch domain used in the POC is protected by master username/password (basic auth); DynamoDB and S3 access is scoped through the application's own AWS credentials.

### Production Security Improvements
The architecture already anticipates its own hardening path:

| POC Approach | Production Target | Rationale |
|---|---|---|
| Hardcoded users in environment file | Amazon Cognito | Real user pools, MFA, token-based auth |
| Secrets in environment file | AWS Secrets Manager / SSM Parameter Store | Rotation, no plaintext file to leak |
| Basic-auth OpenSearch domain | IAM (SigV4) auth or OpenSearch Serverless | Removes a shared password; scales ops-free |
| Single local process | AWS App Runner / ECS Fargate / Lambda | Same application, cloud-hosted, minimal rearchitecting |
| — | CloudFront + WAF | Public-facing hardening once exposed beyond a controlled demo |

---

## Scalability and Reliability

### Current POC Architecture
DynamoDB, OpenSearch, and S3 already scale independently of the application process; only the orchestration layer runs as a single local instance, appropriate for demo-level volume.

### Scaling Strategy
Moving the app onto App Runner, ECS Fargate, or Lambda allows horizontal scaling of orchestration without touching the data layer, which was never built to POC-only standards.

### High Availability
Each managed AWS service already provides its own availability guarantees; the single-instance app process is the one component needing a multi-instance or serverless model before running unattended in production.

### Performance Considerations
Multi-crop embedding and the caption/evaluator calls meaningfully increase Bedrock usage per request — roughly six calls per reference photo attached (five crop embeddings plus one caption) and seven per search (five embeddings, one caption, one evaluation), versus one in a naïve whole-image approach. This is the accepted tradeoff for not depending on a single embedding or reference photo per category.

### Reliability Controls
Attaching photos is transactional across S3 and OpenSearch per call — a failed batch rolls back only what it added, never touching photos attached successfully earlier. AI-lookup enrichment is fire-and-forget and best-effort: any failure just leaves the queue entry without a lead, never affecting the response already sent.

### Future State Production Architecture
A containerized or serverless app layer, fronted by CloudFront and WAF, backed by Cognito and Secrets Manager, with OpenSearch migrated to IAM-authenticated or serverless mode — every change targeted at hosting and identity, none at the AI or data architecture that already works.

---

## Knowledge Management Framework

### Admin Experience
Administrators work from purpose-built tabs — "Add Pest," "Add Disease," and "Add Photos" — matching the fixed field set of a real intake form rather than a generic selector, reducing training overhead and data-entry error.

### Knowledge Creation
Creating a category and attaching its first photo are two independent calls chained into one admin action; a category with zero photos is a valid state, though not matchable until a first photo is attached.

### Knowledge Expansion
Additional reference photos can be attached to any existing category at any time — from "Add Photos," from a record's own detail view, or right after creation — all three calling the identical, transactional endpoint.

### Unidentified Queue Management
Every non-confident search becomes a queue entry carrying the query photo, similarity, caption, and — once enrichment completes — an unverified AI-suggested identity for the admin to investigate.

### Human-in-the-Loop Learning
A queue entry can be promoted directly into a real record, reusing the photo already captured from the original search rather than requiring a re-upload — collapsing "the system didn't know" and "now it does" into one action.

### Continuous Improvement Cycle
Every promoted entry, added photo, and corrected record makes the next search more capable — the knowledge base compounds, driven by the organization's own domain expertise, not a black-box retraining process.

---

## Data Architecture

### Vector Data
Multiple embedding vectors per reference photo — one per crop variant — held in OpenSearch, tagged with category identifier, photo identifier, and category type, enabling search to be grouped back to a best score and winning photo per category.

### Metadata
The authoritative, admin-curated record in DynamoDB: shared fields, pest-only fields, disease-only fields, and a photo list where each entry carries its own extracted visual attributes.

### Images
Original, unmodified photos in S3, served through presigned URLs — a source of truth any future re-processing under an upgraded model can always be re-run against.

### AI Enrichment Data
Per-photo structured attributes generated at ingestion, and per-query unverified AI lookup suggestions generated at enrichment time — kept distinct from the admin-verified fields they support but never replace.

### Search Data
The unidentified review queue is a dataset in its own right: every case preserved rather than discarded is raw material for the next round of knowledge-base growth.

### Relationships Between Datasets
A single category identifier threads through every store — DynamoDB record, OpenSearch vectors, S3 objects — so a similarity hit, a metadata lookup, and an image fetch all resolve back to the same logical entity.

---

## Key Architecture Decisions

| Decision | Why Chosen | Benefit |
|---|---|---|
| Local app process, real AWS data services | De-risk the proof-of-concept without sacrificing production-representative data behavior | Path to production changes hosting/identity only, not the data or AI architecture |
| Color-agnostic background removal, not a hue mask | Damage is often itself a color deviation from healthy tissue | Signal isolation without risking the loss of the very signal being sought |
| Five crop variants per photo, chosen by edge-density scoring | The diagnostic region is unmarked and can sit anywhere in the frame | Tight capture of the relevant region at bounded, predictable AI cost |
| Vision-LLM structured attribute extraction | Pixel similarity can't express "which plant part" or "what shape lesion" | A second, independent evidence type that generalizes beyond one crop |
| Triple-signal confidence gating (evaluator confidence + verified-description match + similarity floor) | Trusting one signal alone risks confident wrong answers | A trustworthy, honest "no match" instead of a forced best guess |
| Verified description authoritative; free text weak signal only | Admin-curated fact-checked content should outrank unverified text | Protects match quality from noisy or speculative free-text fields |
| Every existing category always a candidate | A category should never be silently excluded just because its photos scored low on raw similarity | Prevents a systematic blind spot from developing as the knowledge base grows |
| Category-type filter is opt-in, never auto-detected | An auto-detected plant part could be wrong and would then hard-exclude the true match | Keeps a helpful shortcut without introducing new failure risk |
| Numeric fallback rule when the evaluator fails | The system must degrade gracefully, not fail closed | Continuity of service even during an LLM outage or bad response |
| Direct LLM lookup instead of live web search for unidentified enrichment | Testing showed comparable usefulness with no new credentials or external dependency | Simpler operations at the cost of an unverified, non-cited answer |
| Fire-and-forget background enrichment | Enrichment is a nice-to-have for admins, not something the user should wait on | Zero latency impact on the response the user actually receives |
| Transactional photo-attach per call | A failed batch should never leave orphaned vectors behind | Data consistency without blocking unrelated, already-successful uploads |
| Three independent API credentials (session, admin key, app key) | User, admin, and third-party integration each need independently revocable access | Smaller blast radius if any single credential is compromised |
| Category creation and photo attachment as separate, chainable endpoints | A category with zero photos is a valid state, not an error | Flexibility for different clients and workflows without forcing a rigid single-call design |
| Edit form excludes photos and category type | Switching category after creation would orphan the wrong half of pest/disease-only fields | Prevents an entire class of data-integrity error by construction |

---

## Implementation Approach

### Phase 1 — Proof of Concept (Delivered)

**Scope.** Single-tenant, demo-account architecture running locally, backed by real AWS Bedrock, OpenSearch, DynamoDB, and S3; core admin ingestion, user search, and unidentified-queue workflows.

**Deliverables.** Working end-to-end diagnosis pipeline; admin console for category and photo management; unidentified review queue with AI-assisted lookup and promotion.

### Phase 2 — Production Hardening

**Scope.** Replace environment-file credentials and hardcoded users with Amazon Cognito and AWS Secrets Manager; move the application process onto App Runner or ECS Fargate; migrate OpenSearch to IAM-authenticated or serverless mode.

**Deliverables.** Production-grade identity, secrets management, and hosting, with the existing AI and data architecture unchanged.

### Phase 3 — Scale and Expand

**Scope.** Multi-tenant support across estates/business units, CloudFront + WAF for public-facing exposure, expanded crop/pest coverage driven by the knowledge base's own organic growth through the unidentified-queue workflow.

**Deliverables.** A shared platform serving multiple estates or organizations, each with isolated data but common infrastructure, and a measurably larger, field-validated knowledge base than the one it launched with.

---

## Business Benefits

### Operational Benefits
Frontline staff get a first-pass diagnosis without waiting on expert availability; experts focus on genuinely ambiguous cases (the unidentified queue) rather than routine ones.

> **Assumption:** Time from observation to actionable first-pass diagnosis drops from days (expert-relay-dependent) to seconds, for cases the system confidently resolves.

### Financial Benefits
Earlier detection enables targeted, early-stage treatment instead of reactive, broad-spectrum intervention after an outbreak spreads.

> **Assumption:** Meaningful reduction in treatment cost per affected hectare where early detection allows spot treatment over estate-wide spraying — magnitude to be validated against real cost data.

### Productivity Benefits
Agronomists are relieved of routine, easily-recognized cases and can focus on the unidentified queue — the cases that genuinely need a trained eye.

> **Assumption:** Reduction in expert time on routine diagnosis, freeing capacity for field visits, training, and knowledge-base curation.

### User Experience Benefits
Field staff use a two-step "photo in, answer out" flow requiring no domain expertise, filename lookup, or keyword search — matching how they'd describe the problem to a human expert.

### Strategic Benefits
Crop-protection expertise becomes an owned, growing digital asset — more capable every time an admin resolves an unidentified case — rather than knowledge that lives only in a few people's heads.

---

## Success Metrics

| KPI | Baseline | Target |
|---|---|---|
| Detection accuracy (confident matches later confirmed correct by an expert) | *Assumption: not yet measured* | ≥ 90% precision on confident matches |
| Time to first-pass diagnosis | Hours to days (expert-relay-dependent) | Seconds (system response time) — *Assumption* |
| Unidentified rate (share of searches returning "unidentified") | *Assumption: high at knowledge-base launch* | Decreasing month-over-month as the knowledge base grows |
| Unidentified case resolution rate (queue entries promoted or dismissed within a review cycle) | *Assumption: 0% (no queue existed previously)* | ≥ 80% of queue entries reviewed within one week — *Assumption target* |
| Knowledge base growth (categories and reference photos added per month) | Baseline photo/record count at launch | Steady month-over-month growth via queue promotion — *Assumption* |
| Field user adoption (active users submitting searches) | *Assumption: 0 (pre-launch)* | Adoption across a defined set of pilot estates within the first quarter — *Assumption target* |
| Match confidence distribution (share of matches passing all three confidence signals) | *Assumption: to be established during pilot* | Stable or improving as reference photo coverage per category increases |

---

## Risks and Mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| AI hallucination in unidentified-case lookup | Admin could mistake an unverified AI guess for a verified fact | Presented explicitly and only as an unverified suggestion, never shown to end users, never auto-added to the searchable database |
| Similarity bias (lookalike pests/diseases scoring deceptively close) | A confident wrong match could lead to incorrect treatment | Triple-signal confidence gating (evaluator confidence + verified-description match + similarity floor); evaluator explicitly instructed to treat mismatched structural attributes as a poor match |
| Cloud/Bedrock dependency | An LLM evaluator outage could stall search results | Deterministic numeric fallback rule (top similarity plus minimum margin over runner-up) keeps the system operating without the evaluator |
| Cost escalation (OpenSearch billed per instance-hour; multi-crop matching multiplies Bedrock calls) | Idle infrastructure and per-request AI cost could scale faster than expected | OpenSearch domain deleted between active demo/pilot periods at current scale; migration to OpenSearch Serverless planned for production; Bedrock call volume already bounded and predictable per photo/search |
| Data quality (poor or mislabeled reference photos) | Weak reference data degrades match quality for an entire category | Admin curation workflow (edit, remove single photo, promote from queue) keeps reference data correctable without requiring a full record rebuild |
| Model drift (embedding or LLM model updates changing behavior) | Silent shifts in match quality after an underlying model version change | Original images are always retained in S3, enabling full re-embedding/re-processing against any future model version |
| Adoption risk (field staff distrust or bypass the tool) | Low usage undermines the entire business case | Honest "unidentified" responses (never a forced guess) build trust; simple two-step user flow requires no training |

---

## Lessons Learned

### Architecture Learnings
Keeping the AI/data layer on real managed services from day one, even while orchestration stays local for demo convenience, meant the hardest design decisions were validated against production-representative infrastructure from the first prototype.

### AI Learnings
No single AI technique was sufficient alone; segmentation, multi-crop embedding, attribute extraction, and LLM evaluation each compensated for a different blind spot in the others. A direct LLM knowledge lookup, tested against a live web-search alternative, proved a comparably useful and simpler substitute — validate the simpler approach before assuming complexity is necessary.

### Operational Learnings
Treating "unidentified" as a first-class, structured outcome — not an error to minimize away — turned knowledge gaps into a concrete admin workflow instead of lost information.

### Product Learnings
Matching each admin tab to the real intake form it replaces, rather than one generic form, reduced complexity for day-to-day users.

### Business Learnings
A system that can honestly say "I don't know" is more valuable to expert users than one that always sounds confident — trust lost to a wrong "confident" answer is expensive to rebuild.

---

## Future Roadmap

### Short-Term Enhancements
Expand reference-photo coverage across existing categories via the unidentified-queue promotion workflow; begin production hardening (Cognito, Secrets Manager, containerized hosting).

### Medium-Term Enhancements
Native mobile application for field capture; multi-tenant support across additional estates or business units; expanded crop coverage beyond the initial focus area.

### Long-Term Vision
A conversational agronomy assistant layered on top of the existing structured knowledge base; predictive analytics correlating historical detection data with seasonal and weather patterns; a broader plantation-intelligence platform where pest/disease diagnosis is one module among several.

---

## Architectural Differentiators

1. Background removal that is deliberately color-agnostic, so the removal step itself never risks erasing the diagnostic signal.
2. Multi-crop embedding driven by cheap, local edge-density scoring rather than exhaustive or manually annotated cropping.
3. A vision-LLM attribute layer that reasons in language, not just pixels — generalizing to any plant part or crop without retraining.
4. A confidence model that requires three independent signals to agree, rather than trusting any single score.
5. Admin-verified descriptions treated as authoritative evidence on equal footing with image similarity, not as passive reference text.
6. An explicit, first-class "unidentified" outcome that feeds a structured human-in-the-loop growth workflow instead of disappearing.
7. AI-assisted lookup on unidentified cases that is transparently labeled unverified and never reaches the end user or the searchable database automatically.
8. A deterministic fallback rule that keeps the system operational even if the LLM evaluator is unavailable.
9. Three independently rotatable credential mechanisms separating end-user, admin, and third-party integration access.
10. A data architecture built entirely on managed, production-grade AWS services from the very first prototype, so the proof-of-concept never accrues data-layer technical debt.

---

## Executive Summary for Leadership

### Challenge
Plantation crop-protection expertise is scarce, centralized, and slow to reach the field, while existing digital tools only work for someone who already knows what they're looking for.

### Solution
A multimodal AI platform lets any field user photograph crop damage and receive an instant, structured diagnosis — or an honest "unidentified" — built on background removal, multi-crop embedding, vision-LLM attribute extraction, and LLM-based evaluation, backed by managed AWS services (Bedrock, OpenSearch, DynamoDB, S3).

### Innovation
Four complementary AI techniques combine so no single technique's blind spot becomes the system's; a triple-signal confidence gate claims confidence only when image evidence, structural reasoning, and admin-verified knowledge all agree.

### Business Impact
Frontline staff gain expert-level first-pass diagnosis without waiting on expert availability; every unresolved case becomes a low-friction opportunity to grow the knowledge base rather than a lost signal.

*(Quantified outcomes above are explicitly labeled as planning assumptions pending pilot validation, not measured results.)*

### Strategic Value
The organization converts scarce, person-dependent expertise into an owned, compounding digital asset, on an architecture whose data and AI layers are already production-grade — the path to enterprise deployment is a hosting and identity upgrade, not a rebuild.

---

## Appendix A — Technology Stack

| Layer | Technology | Purpose | Justification |
|---|---|---|---|
| Presentation | Static HTML/CSS/vanilla JavaScript | Admin console and user upload/result UI | No build step; fast to iterate during a proof-of-concept |
| Application | FastAPI (Python) | Request orchestration, auth, validation | Lightweight, async-friendly framework well suited to I/O-bound AI orchestration |
| Session Auth | Starlette `SessionMiddleware` | End-user/admin web login | Signed cookie session with no additional infrastructure |
| Admin/API Auth | Static API keys via `X-API-Key` header | Admin and third-party API access | Independent, rotatable credentials separate from the session flow |
| Background Removal | `rembg` (local) | Generic foreground segmentation ahead of embedding | Color-agnostic; avoids stripping color-deviation damage signals |
| Embeddings | AWS Bedrock — Titan Multimodal Embeddings | Vector representation of each photo crop | Managed multimodal embedding with no model hosting burden |
| Vision/Reasoning LLM | AWS Bedrock — vision-capable model | Attribute extraction, evaluation, unidentified-case lookup | Single managed service covering captioning and reasoning |
| Vector Search | Amazon OpenSearch (k-NN, HNSW/FAISS, cosine) | Nearest-neighbor similarity search | Purpose-built, independently scalable vector engine |
| Metadata Store | Amazon DynamoDB (on-demand) | Structured pest/disease records, review queue, season config | Schema-flexible, no capacity planning at POC scale |
| Image Store | Amazon S3 (private, presigned URLs) | Original photo storage and retrieval | Durable, low-cost, securely time-limited access |
| Configuration | `.env` via `python-dotenv` | Environment-specific and sensitive settings | Central, gitignored configuration for the POC stage |

---

## Appendix B — Architecture Highlights

- Every data-bearing component — embeddings, vector search, metadata, images, review queue — already runs on managed AWS services, even in proof-of-concept form.
- Matching works by visual similarity, not filename or ID lookup, so a new photo of a known pest matches an existing record regardless of angle, lighting, or crop instance.
- Four AI techniques are deliberately layered — background removal, multi-crop embedding, vision-LLM attribute extraction, LLM evaluation — because no single technique alone was sufficient.
- A confident result requires three independent signals to agree; an honest "unidentified" is a valid outcome, never a forced best guess.
- Every unresolved case is captured, enriched with an unverified AI lead, and promotable directly into the knowledge base.
- Three independently rotatable credential mechanisms cleanly isolate end-user, admin, and third-party access.

---

## Appendix C — Key Takeaways

1. **Problem solved:** Field-taken, unposed, unannotated photos of crop damage can now be matched by visual similarity to a structured, expert-curated diagnosis — without exact filename or ID lookup.
2. **Innovation delivered:** A four-technique AI pipeline (segmentation, multi-crop embedding, language-based attribute extraction, LLM evaluation) combined with a triple-signal confidence gate that can honestly decline to guess.
3. **Expected benefits:** Faster first-pass diagnosis, more targeted early treatment, and reduced routine load on scarce expert time — currently planning assumptions pending pilot-stage measurement.
4. **Strategic importance:** Converts person-dependent crop-protection expertise into an owned, continuously growing organizational asset.
5. **Future potential:** A foundation for mobile field capture, predictive analytics, and a conversational agronomy assistant, built on a data and AI architecture that is already production-grade at the proof-of-concept stage.
