[Project overview](../README.md) · [Architecture](../ARCHITECTURE.md)

# Engineering guide

MemoryPath is a team prototype with synthetic care records and local-file persistence. This guide follows the code that turns a proposed update into a reviewable change.

## Read one complete path

1. **[proposal.ts](../src/lib/proposal.ts)** gathers records, the current profile, and optional GBrain context before requesting a proposal.
2. **[profile-agent.ts](../src/lib/profile-agent.ts)** uses a direct structured model call and schema parsing. An agent configuration also exists; the runtime path is not a multi-agent orchestration system.
3. **[verify.ts](../src/lib/verify.ts)** checks cited record IDs and normalized quote matches. It drops unsupported citations and changes without supporting evidence.
4. **[ProposalReview.tsx](../src/components/ProposalReview.tsx)** presents editable changes for a person to review.
5. **[The approval route](../src/app/api/proposals/[id]/approve/route.ts)** checks proposal status and base version, then applies accepted changes through **[approve.ts](../src/lib/approve.ts)** and stores a new profile version.

## Decisions and tradeoffs

| Decision | Benefit | Remaining limitation |
| :--- | :--- | :--- |
| Separate proposals from approved profiles | Generation alone does not update the approved profile. | Approval is an explicit API action, not authenticated nurse authorization. |
| Verify citations against stored records | Rejects fabricated record IDs and unmatched quotations in generated proposals. | Normalized substring matching does not establish semantic support or clinical correctness. Human-edited approval payloads are schema-checked but not passed through this verifier again. |
| Reject stale proposal versions | Stops a sequential update based on an older profile. | Local JSON reads/writes are not a database transaction; concurrent approvals need further work. |
| Keep GBrain optional | The base workflow continues when retrieval is disabled or unavailable. | Retrieval failure can reduce context; it is not evidence that proposal quality is unchanged. |
| Use synthetic records and local files | Makes the workflow straightforward to inspect. | It provides neither production access control nor a multi-user storage design. |

Clinical-language checks produce warnings. They do not guarantee that a model cannot produce clinical advice.

## Reproduce the checks

Use Node.js 22 and npm. No live API key or GBrain installation is needed for this test/build path:

```sh
git clone https://github.com/Jsmitty78/MemoryPath.git
cd MemoryPath
npm ci --ignore-scripts --no-audit --no-fund
npm test
npm run typecheck
npm run lint
npm run build
```

For interactive model generation and voice, use the separate API-key setup in the [README](../README.md#run-locally).

The [CI workflow](../.github/workflows/ci.yml) runs these checks on Node 22. On 13 November 2026, the inspected application revision `93a1a1a` passed **58 tests**, TypeScript, lint, and production build locally on Node 26.5.0. The build emitted a warning about the Next.js ESLint plugin configuration; standalone ESLint and the build completed successfully. This is not a claim of live model, voice, or clinical evaluation.

## Choose a test to inspect

| Test file | What it exercises |
| :--- | :--- |
| [proposal.test.ts](../tests/proposal.test.ts) | Citation filtering, structured field parsing, and proposal handling |
| [approve.test.ts](../tests/approve.test.ts) | Version conflicts, edited approvals, resolved proposals, and rejection |
| [safety-contract.test.ts](../tests/safety-contract.test.ts) | Proposal generation leaves approved profiles unchanged; rejection preserves them; optional retrieval can fail |
| [realtime.test.ts](../tests/realtime.test.ts) | Voice-session instructions and ephemeral-secret response handling with mocked provider calls |
| [stores.test.ts](../tests/stores.test.ts) | Record/profile storage and resident separation |

Tests use temporary storage and simulated model responses. Passing them is useful evidence for those cases, not a substitute for authorization, concurrency, security, or clinical validation.

## Collaboration and timing

Jake Smith and Yu Yoshimuta collaborated on MemoryPath. Jake's portfolio describes his interviews, product direction, workflow, and implementation contributions. The architecture and test suite are presented as the team system. This repository includes work after the original hackathon; this guide and the CI workflow were added during the November 2026 portfolio review.
