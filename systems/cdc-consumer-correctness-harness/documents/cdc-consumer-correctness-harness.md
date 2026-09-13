<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# CDC Consumer Correctness Harness

**Project Link:** [View Project](https://nextwork.ai/projects/54de60ad-307c-4881-8e2b-441a1952bdba)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_hfgyxjqz)

## The Mission: Proving That the Wrong Answer Can Look Right

### Committing to the problem before any tool is opened

I built this system to prove that a CDC consumer could report the expected row count while storing stale values. A count could expose obvious duplication, but it could not establish that every key held its latest state. I therefore treated row counts as an initial signal and SHA-256 checksums as the deciding evidence for content correctness.

Before opening any implementation tool, I fixed the question, failure cases, and acceptance criteria. The harness had to generate deterministic inserts, updates, deletes, and update preimages; run two intentionally wrong consumers; and compare their outputs with a known-good target. The correct consumer also had to reject unknown change types, remove every expected preimage, remain idempotent, and use PostgreSQL 18 RETURNING OLD/NEW data for write auditing. This framing kept a plausible-looking count from being accepted as proof of correctness.

## Setting Up the Stack and Delivery Pipeline

### What this step accomplishes

I established a version-controlled workspace for the generator, consumers, verification harness, evidence, tests, and documentation. I initialized the GitHub repository and organized the Linear work so each implementation stage could be traced to a defined requirement. This gave the system a consistent delivery structure before any change feed was processed.

I separated generated fixtures from consumer outputs and verification results. That prevented a consumer from overwriting source evidence or making a failed run appear clean by changing expected data. Repository history preserved the order in which requirements, tests, implementation changes, and measurements were introduced. Linear tracked the work, while Git remained the technical record. Together, they supported repeatable execution without treating task completion as evidence that the stored CDC state was correct.

### Verified package versions: psycopg, duckdb, deltalake

I verified the components before building the harness so failures could be interpreted against known dependency versions. The check printed psycopg 3.3.4, duckdb 1.5.5, and deltalake 1.6.2. I preserved those values in the build record instead of relying on an unconstrained specification.

Each dependency served a distinct role. Psycopg connected the Python consumers to PostgreSQL 18 and exposed results to the audit logic. DuckDB compared expected and persisted datasets independently of the consumer. Deltalake produced and read the real Delta Change Data Feed used for the shape comparison. Recording the versions did not prove every later environment would behave identically, but it made this measured result reproducible and provided a clear starting point if package behaviors changed.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_7le5aea3)

## Design Artifacts and the Seeded Change Feed Generator

### What this step accomplishes

I wrote the interrogation checklist, decision records, topology, and evidence plan before implementing the seeded generator. These artifacts defined the feed contract, ordering risks, supported change types, expected target state, and consumer checks. The generator then created a deterministic fixture covering inserts, updates, deletes, and update preimages.

A fixed seed mattered because every consumer needed the same events. If each run generated different records, a count or checksum difference could come from changed input rather than consumer behavior. The generator produced repeatable keys, values, metadata, and expected state. I kept the source feed, expected target, and outputs separate so the verifier compared independent artifacts. This baseline demonstrated visible row inflation and silent stale-value persistence without shifting data between runs.

### Why consequences belong beside each question in the interrogation checklist

I placed a consequence beside each interrogation question so every decision carried its risk. Asking whether update preimages were present was incomplete unless the checklist stated that processing them as current values could overwrite the correct postimage. Asking about unknown change types also needed to state that silently accepting one could corrupt the target.

This pairing turned the checklist into an interface agreement instead of yes-or-no prompts. Both Authorizing Officials could see the required evidence and the failure tied to an incomplete answer. I framed the exchange under NIST SP 800-47 Rev. 1 because the feed crossed a system boundary where responsibilities and assumptions had to be explicit. The annex did not prove compliance. It documented the questions, consequences, and ownership needed to review the connection without hiding risk behind an affirmative response.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_str3cln2)

## Running Two Wrong Consumers and Measuring the Damage

### What this step accomplishes

I built two incorrect PostgreSQL consumers and measured their outputs with an independent DuckDB verifier. The append consumer wrote every CDC event as a new row, ignoring whether it represented a preimage, postimage, or deletion. The upsert consumer kept one row per key but used an order that let stale preimages overwrite current postimages.

These failures created different detection problems. Append produced 2,120 rows where the target contained 80, so a count exposed the defect. Upsert retained exactly 80 keys, causing the same check to report a match although stored values were wrong. I calculated SHA-256 checksums over canonical target rows to distinguish structural agreement from content agreement. This evidence showed why a successful write count or expected key count could conceal corrupted state.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_1id8relr)

### Why the upsert failure is the harder failure mode

The append defect was conspicuous because its output contained 2,120 rows instead of the expected 80. Any count assertion comparing actual and expected rows would fail immediately. The upsert defect was harder because it produced the correct 80 keys and therefore satisfied the same count-based check that the correct consumer passed.

The corruption existed inside those rows. Lexicographic sorting placed update_postimage before update_preimage, so the stale before-value became the last write. The final key count did not expose that reversal. The SHA-256 checksum failed because it represented canonical row content, not only the number of records. This was the central lesson: cardinality could prove that objects were missing or duplicated, but it could not prove retained values were current. A plausible row count could therefore create false confidence around an incorrect target.

## Building the Correct Consumer and Proving Correctness by Checksum

### What this step accomplishes

I built a filtered consumer that interpreted each supported CDC type before writing to PostgreSQL. It retained inserts and update postimages as current values, applied deletes to remove keys, and excluded update preimages from the target. I then compared the persisted rows with the deterministic expected state using both count and SHA-256 checksum assertions.

The implementation used PostgreSQL 18 RETURNING OLD/NEW results to audit each write. This made the path observable without treating a statement as proof that the intended transition occurred. I replayed the same feed to test idempotency. A correct replay had to preserve the final row count and checksum instead of duplicating rows or reversing newer values. Passing both measurements showed that the consumer reached the expected state for this fixture. It did not establish correctness for every schema, order, or production feed.

### The two guard assertions and what each one prevents

The first guard raised ValueError when the feed contained a _change_type outside the supported set. This prevented a fifth event type from passing through an accidental default branch. Failing the pipeline was safer than guessing whether an unfamiliar record represented current state, historical context, or a deletion instruction.

The second guard asserted that removed == expected_preimage_count. This proved the filter removed the exact number of update_preimage rows expected. If the filter misspelled update_preimage, zero rows would be removed and stale values could reach the target. Removing too many rows would also fail. Together, the guards made the contract explicit: accept only named change types and exclude exactly the known preimages. They did not prove a new type’s meaning; they forced deliberate handling before that data could be written.

### Why count-based checks are insufficient

Row count measured size, not data. The append consumer demonstrated the case where cardinality was enough: it produced 2,120 rows against an expected 80, so the mismatch exposed uncontrolled accumulation. The upsert consumer demonstrated the dangerous case: it retained exactly 80 keys, allowing a count-only test to report MATCH even though several values came from update preimages.

The checksum closed that gap by hashing a canonical representation of every expected key and value. When stale preimages overwrote postimages, row content changed and the SHA-256 digest no longer matched. Checksums did not replace every validation. A digest could not explain which field was wrong without a row-level comparison, and canonicalization rules had to remain stable. It provided the deciding signal here because it tested content across the complete target instead of treating the row total as proof.

## Validating Against a Real Change Data Feed and Shipping the Release

### What this step accomplishes

I generated a real Delta-rs Change Data Feed and compared its structure with the synthetic fixture before shipping. This tested whether the harness used realistic metadata names and change-type values instead of a contract invented for the local demonstration. I retained the comparison as a one-time structural artifact beside the regression results.

After confirming the shape, I completed the documentation, added CI/CD checks, and tagged the final version. The release joined seeded inputs, wrong-consumer evidence, correct output, checksum verification, and scope limits under one state. I did not treat the real CDF sample as a production benchmark. It confirmed the interface shape, while the dated fixture remained the basis for measured counts and checksums. This kept schema realism distinct from claims about scale, retention, or operational performance.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_uijc16ty)

### What the real CDF comparison confirmed and what it did not

The Delta-rs comparison confirmed that the real feed exposed _change_type, _commit_version, and _commit_timestamp, matching the metadata columns used by the synthetic fixture. It also confirmed underscore-based values such as insert, update_preimage, and update_postimage. This ruled out a harness built around hyphenated event names or unrelated metadata fields.

The check was narrow. It did not prove that the generator reproduced upstream retention, schema-change notices, concurrency, load ordering, or storage behavior. It also did not establish that one CDF represented every Delta implementation or version. The evidence supported one structural claim: the fixture matched the observed event vocabulary and metadata shape of a real Delta-rs feed. Performance, durability, and production-scale conclusions remained outside the release because the comparison did not measure them.

## Readout for Two Audiences: Engineering Leadership and Stakeholders

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/54de60ad-307c-4881-8e2b-441a1952bdba_6ccv7p8q)

### Separating structural claims from regression checks

I separated claims that followed from the consumer design from measurements tied to the dated fixture. The append consumer’s behavior was structural: writing every CDC event without interpretation necessarily retained records that were not current rows. Its measured output of 2,120 rather than 80 was a regression result specific to this generated feed.

The upsert failure followed from another structural defect. Lexicographic ordering placed update_postimage before update_preimage, letting the stale preimage become the final write. The 80-key count and failed checksum belonged in the dated regression table because another fixture could differ. I presented both layers so engineering leadership could inspect the mechanism while stakeholders could see its effect. The readout identified the costly case: every expected key existed, yet values were wrong and a count-only control passed.

## Lessons Learned and Honest Limits

### Key tools and concepts from the project

I used PostgreSQL 18 and its RETURNING OLD/NEW syntax to observe database changes, Python 3.10+ for the generator and consumers, DuckDB 1.5.5 for independent verification, and Deltalake 1.6.2 for the real CDF shape check. GitHub preserved implementation and evidence history, while Linear tracked delivery.

The main lesson was that structural correctness and regression evidence answered different questions. Consumer rules defined possible failures, while fixture counts and checksums showed whether they occurred in one run. SHA-256 comparison exposed stale values that cardinality checks missed, and replay testing measured idempotency. I also placed consequences beside interface questions so stakeholders could judge incomplete answers. The harness proved its declared cases, not every CDC architecture.

### Time to complete and most challenging part

I completed the build in approximately 60 minutes. The hardest part was keeping structural claims separate from fixture-specific regression results. A statement such as “preimages can overwrite postimages under this ordering” described the consumer logic, while “2,120 rows instead of 80” described one dated execution. Combining them would have made the readout sound broader than the evidence supported.

I also translated the engineering details into an annex stakeholders could review without losing the failure mechanism. The checklist had to state what each system owner must answer and what followed from a weak response. The leadership readout also had to preserve counts, checksums, event ordering, and scope limits. Maintaining those layers required more care than implementing the incorrect consumers because the final evidence had to remain understandable and traceable.

### Reflecting on the build

I completed this build to understand how CDC consumers could preserve the expected number of keys while persisting stale data. The two wrong consumers made the contrast measurable: append corruption failed visibly at 2,120 rows versus 80, while the faulty upsert retained 80 keys and failed only when the SHA-256 checksum tested their content. The correct consumer combined explicit type handling, exact preimage removal, audited writes, and replay testing.

My next step is to apply the method to streaming systems where events arrive late, repeat, or appear out of order. That work will require sequencing, idempotency keys, checkpoint recovery, poison-event handling, and concurrency tests. I will keep the evidence discipline: define expected behavior before execution, separate structural claims from fixture results, and verify content instead of accepting processing counts as proof of state.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/54de60ad-307c-4881-8e2b-441a1952bdba)*
