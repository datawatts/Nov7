# ClinicalTrials product scope

**Status.** Scope only, written 2026-11-07 about 2:45 PM ET. Devpost closes 5:00 PM ET. [datawatts/Nov7](https://github.com/datawatts/Nov7) is initial commit `d556a64` (2:40 PM ET) and a README titled Nov7. No product is built. This web MVP fits the hours before 5:00 PM.

## Product and user

A patient or caregiver supplies a plain-language summary. The web app finds public ClinicalTrials.gov studies with an Indiana site and explains criteria against those facts, including gaps. The study team determines eligibility. Labels are potentially relevant, criterion met, criterion not met, criterion unknown, and needs study-team review. Output is information: no diagnosis, treatment recommendation, enrollment, or medical advice.

Hackathon profiles are synthetic. Real patient details stay out of the repo, logs, analytics, and model traces, and are dropped after the session.

## Journey and typed flow

Web UI first. Order: intake, retrieval, extraction, validation, evaluation, presentation. Jev is optional and unused.

1. **Intake.** Facts: `summary`; `ageYears` (number or unknown); `sex` (`FEMALE`, `MALE`, or unknown); `conditionTerms` copied from the summary; `place` (Indiana, city optional). Unstated age, sex, or diagnosis stays unknown.
2. **Retrieval.** `GET https://clinicaltrials.gov/api/v2/studies` with `query.cond`, `query.locn=Indiana`, `filter.overallStatus=RECRUITING`, small page. Save query and time, or use the labeled cache.
3. **Sites.** Keep a study only if a location row has state Indiana. `query.locn` still returns other states. Show `overallStatus` apart from each Indiana `status` and `statusVerifiedDate`.
4. **Extract and validate.** Copy structured age and sex. Free-text criteria need an exact source substring. Interpretation is a separate field. Registry text and the summary are untrusted data, not instructions.
5. **Evaluate and present.** Rules compare facts with criteria and store a reason. Every card says needs study-team review. Evidence is `{nctId, sourceQuote, patientFact | missingFact, reason}`: quote as source, reason as interpretation.

Fields: `nctId`, `briefTitle`, `overallStatus`, `statusVerifiedDate`, `sex`, `minimumAge`, `maximumAge`, `eligibilityCriteria`, location facility, city, state, country, status. A criterion has kind (`age_min`, `age_max`, `sex`, `free_text`), polarity, quote, source field, and extraction (`structured`, `unambiguous`, `ambiguous`, `unsupported`).

Age inside a parsed bound is met; outside, not met; missing or unparsable is unknown. Sex `ALL` is met for a stated female or male fact; a conflict is not met; unknown sex is unknown. A met exclusion, or an inclusion not met, removes potentially relevant. Ambiguous, unsupported, or unmatched free text stays unknown and visible. Potentially relevant also needs an Indiana site at `RECRUITING` with `statusVerifiedDate` in the demo month or the month before (a demo rule). Other Indiana rows stay visible as not recruiting, unknown, or possibly stale.

## Priority

**Must.** Web page and three synthetic cases; preserved unknowns; live v2 or labeled cache; Indiana-site filter; separate study and site status; age and sex rules; evidence and labels above; study-team disclaimer; one test that returns an ambiguous model "met" to unknown.

**Should.** Study-page link; stale-date label; a line when a rule overrides model text; Indianapolis before other Indiana cities.

**Leave out.** EHR, SMS, enrollment or contact automation, diagnosis or treatment recommendations, nationwide matching, accounts, saved summaries, market or accuracy claims, and any prescribed stack. Software Factory rewards developer-productivity tools; this assistant does not qualify merely by using a harness. An Indiana community note is fair. No prize is guaranteed.

Devpost needs a name, description, and `https://github.com/datawatts/Nov7` by 5:00 PM ET. Commit in this repo through the afternoon. Do not open another repo or copy in an unrelated research hub.

## Acceptance and synthetic cases

Name live or cached, with the retrieval time. Do not say eligible, qualified, or enroll. Drop a hit with no Indiana row. Do not copy study-wide recruiting onto a site.

1. **Missing fact.** "I live in Indianapolis and have type 2 diabetes." Age and sex stay unknown. Potentially relevant only with those gaps named.
2. **Explicit exclusion.** A 70-year-old man in Fort Wayne; fixture maximum age 65 years. Age is not met. Evidence: fixture id, age field, and 70 years.
3. **Stale or unknown site.** Study-wide `RECRUITING`; Indiana site `ACTIVE_NOT_RECRUITING` or missing; `statusVerifiedDate` `2026-05`. Show both statuses and the date. Listed, not potentially relevant.

## Public data and cache

OpenAPI 3.0.3 fetched 2026-11-07 is API 2.0.5 at `https://clinicaltrials.gov/api/v2`, with no security scheme. Docs: [about the API](https://clinicaltrials.gov/data-api/about-api), [study data structure](https://clinicaltrials.gov/data-api/about-api/study-data-structure).

Live check, 19:44:50 UTC: HTTP 200, no key, `query.locn=Indiana`, `filter.overallStatus=RECRUITING`, `pageSize=2`, `nextPageToken` set, no total. Publish no trial count. `NCT04710576` was study-wide `RECRUITING`, verified `2026-05`; Indianapolis site Indiana University Health Melvin and Bren Simon Cancer Center was `ACTIVE_NOT_RECRUITING` (121 locations). `NCT07110246` was study-wide `RECRUITING`, verified `2026-10`, with Riley Hospital for Children at Indiana University Health recruiting. Public observations at that time, not recommendations.

`sex` is `FEMALE`, `MALE`, or `ALL`. Age bounds and location fields are structured. `eligibilityCriteria` and `genderDescription` are free text. Study `overallStatus` uses `Status`; location `status` uses `RecruitmentStatus`.

Cache: JSON labeled `cached snapshot`, time `2026-11-07T19:44:50Z`, plus query, ids, titles, statuses, dates, age and sex bounds, criteria text, and Indiana facility, city, state, and status. Omit contact names, phones, and emails. The UI marks it as a cache.

## Risks, milestones, demo

Most free-text criteria will be unknown; that is intended. Location search is study-shaped, dates can be a month and old, and the API may fail. The cache covers a failed call. Open: Devpost name; listing `NOT_YET_RECRUITING` Indiana sites (default yes); page cap (default 10). The one-month window is a demo choice.

| By (ET) | Outcome |
| --- | --- |
| 3:10 | Fixtures, expected decisions, and labeled cache saved |
| 3:45 | Rule checks green, including the overridden model guess |
| 4:25 | Web UI completes case 1 live or from cache |
| 4:45 | Cases 2 and 3 clickable; both statuses visible |
| 4:55 | Devpost name, description, and repo link submitted |

**Five-minute demo.** The repo started as a README; the screen prepares a study-team conversation. Case 1: unknown age, NCT IDs, an Indiana facility, and a site status that differs from the study. Open one evidence row, then the rule check holding an ambiguous quote at unknown. Case 2: the age exclusion and its evidence. Case 3: both statuses and `2026-05`, left out of potentially relevant. Close on needs study-team review, the live-or-cache label, and not medical advice.
