# Kubernetes v1.37 Docs Lead — complete AI handoff guide

> **Self-contained.** Paste this whole file as the first message to any AI assistant with shell access
> and it can pick up the v1.37 Docs Lead work with no other context.

---

## 0. How to use this document

**If you are an AI reading this:** you are assisting **Tushar Mittal (`@kernel-kun`)**, the SIG Release
**Docs Lead for Kubernetes v1.37**. Release day is **Wednesday 26 August 2026**.

Do this, in order, at the start of every session:

1. Read §4 (timeline) and §5 (state snapshot). **The snapshot was taken 18 Aug 2026 — treat every
   line as stale.** Re-verify with the commands in §5.1 before acting on anything.
2. Work the task queue in §7. It is chronological. Do not skip ahead — several tasks have hard
   ordering dependencies (e.g. the `release-1.36` branch must exist before the 1.36 `hugo.toml` PR
   can be retargeted).
3. Use the templates in Appendix A verbatim where provided — they are the handbook's own wording and
   the community expects them.
4. When you report status, be explicit about what you **verified** vs. what you **assumed**. Tushar
   cannot see your tool output.

**Non-negotiables — violating any of these breaks the release:**

- ❌ Never merge anything into `k/website` `main` before the release is officially cut on 26 Aug.
- ❌ Never remove `do-not-merge/hold` from the integration branch PR without explicit go-ahead.
- ❌ Never merge the four previous-release `hugo.toml` PRs before the release has occurred.
- ✅ The Final Release Notes PR must be **merged ≥24 h before the release cut**, or `k/k` ships a stale
  `CHANGELOG`. This is the hardest deadline in the whole cycle.

---

## 1. Role and scope

Tushar is Docs Lead. That means he personally owns:

- The **release notes** for the whole cycle, culminating in a Final Release Notes review PR against
  `kubernetes/sig-release`.
- The **integration branch** (`dev-1.37` → `main`), which he merges on release day. This is what
  actually "ships the docs".
- **Publishing the release blog post** on release day (Comms writes it; Docs merges it).
- **Freezing and unfreezing** `kubernetes/website` around the release.
- Site configuration (`hugo.toml`, `data/releases/schedule.yaml`) across `main`, `dev-1.37`, and the
  four previous release branches.
- Setting up the **v1.38** Docs Lead for success the day after release.

He directs 5 docs shadows (§6) but the release-day steps are his alone.

---

## 2. Canonical sources — read these, don't guess

| What | URL |
|---|---|
| **v1.37 release timeline** | https://github.com/kubernetes/sig-release/blob/master/releases/release-1.37/README.md |
| **Docs role handbook (the playbook)** | https://github.com/kubernetes/sig-release/blob/master/release-team/role-handbooks/docs/Release-Timeline.md |
| Docs role README | https://github.com/kubernetes/sig-release/blob/master/release-team/role-handbooks/docs/README.md |
| v1.37 release team roster | https://github.com/kubernetes/sig-release/blob/master/releases/release-1.37/release-team.md |
| v1.37 links (meeting docs, boards) | https://github.com/kubernetes/sig-release/blob/master/releases/release-1.37/links.md |
| **Release Tracking Board** (Docs view) | https://github.com/orgs/kubernetes/projects/264/views/3 |
| v1.37 milestone in k/k | https://github.com/kubernetes/kubernetes/milestone/70 |
| Release phases definitions | https://github.com/kubernetes/sig-release/blob/master/releases/release_phases.md |
| Exceptions process | https://github.com/kubernetes/sig-release/blob/master/releases/EXCEPTIONS.md |
| Docs style guide | https://kubernetes.io/docs/contribute/style/style-guide/ |
| `krel release-notes` docs | https://github.com/kubernetes/release/blob/master/docs/krel/release-notes.md |
| Branch sync script | https://github.com/kubernetes/sig-release/tree/master/release-team/role-handbooks/docs/branch-sync-script |
| Prior leads' activity log (Tushar's own repo) | https://github.com/kernel-kun/release-team-utils/blob/main/docs-team/docs-leads-activity-log.md |

Prior-cycle timelines follow the same URL pattern — swap the version:
`https://github.com/kubernetes/sig-release/blob/master/releases/release-1.36/README.md`

**Slack channels:** `#sig-release`, `#sig-docs`, `#sig-docs-maintainers`, `#release-docs`,
`#release-notes`, `#release-management`, `#release-comms`, `#chairs-and-techleads`,
`#kubernetes-contributors`, `#enhancements`, `#release-ci-signal`, `#sig-cluster-lifecycle`.

---

## 3. Tooling and verification constraints

**Read this before trying to query GitHub.**

- `gh` is installed but authenticated **only to Disney hosts** (`github.twdcgrid.net`,
  `github.prod.hulu.com`) and those tokens are currently invalid. **`gh` will not work against
  github.com.**
- Fallback: unauthenticated `curl https://api.github.com/...`. Rate limits are tight and shared by IP:
  **60 req/hour core, 10 req/min search.** Budget your calls; batch with `python3` parsing.
- `raw.githubusercontent.com` does **not** count against the API limit — prefer it for file contents.
- Check remaining budget: `curl -sS https://api.github.com/rate_limit`
- If Tushar can authenticate `gh` to github.com (`gh auth login -h github.com`), everything gets easier
  — suggest it, and note he can run it himself by typing `! gh auth login -h github.com` in Claude Code.
- Prefer the **core** API (`/repos/...`) over the **search** API (`/search/issues`) — the core limit is
  6× more generous. `GET /repos/kubernetes/website/pulls?state=open&base=dev-1.37` beats a search query.

Tushar pushes via a fork (`kernel-kun/<repo>`) and PRs from it; he does not have direct write to the
kubernetes org repos except where explicitly granted.

---

## 4. The v1.37 timeline (immutable facts)

Release cycle began Monday 18 May 2026. All deadlines are **AoE** (Anywhere on Earth — the day ends
when it ends in UTC−12, i.e. cutoff is the next day at 12:00 UTC).

| What | Who | When | Week |
|---|---|---|---|
| Production Readiness Freeze | Enhancements | Tue 9 Jun (AoE) | 4 |
| Enhancements Freeze | Enhancements | Tue 16 Jun (AoE) | 5 |
| **Docs deadline — open placeholder PRs** | **Docs Lead** | **Thu 2 Jul (AoE) / Fri 3 Jul 12:00 UTC** | 7 |
| Feature blog freeze | Comms | Thu 9 Jul (AoE) | 8 |
| **Code Freeze + Test Freeze** | Branch Manager | **Wed 22 Jul (AoE) / Thu 23 Jul 12:00 UTC** | 10 |
| Burndown begins (Mon/Wed/Fri) | ALL | Mon 27 Jul | 11 |
| Deprecations & Removals blog | Comms | Mon 27 Jul | 11 |
| **Docs deadline — PRs ready for review** | **Docs Lead** | **Tue 28 Jul** | 11 |
| Release Highlights deadline | Comms | Tue 28 Jul | 11 |
| Burndown daily (Tue & Thu over Slack) | ALL | from Mon 3 Aug | 12 |
| `release-1.37` branch + jobs, `v1.37.0-rc.0` | Branch Manager | Wed 5 Aug | 12 |
| **Start final draft of Release Notes** | **Docs Lead** | **Wed 5 Aug** | 12 |
| **DOCS FREEZE** | **Docs Lead** | **Wed 5 Aug (AoE) / Thu 6 Aug 12:00 UTC** | 12 |
| Release blog ready to review | Comms / Docs | Thu 13 Aug (AoE) / Fri 14 Aug 12:00 UTC | 13 |
| `v1.37.0-rc.1` | Branch Manager | **Wed 19 Aug** | 14 |
| **Release Notes complete** | **Docs Lead** | Wed 26 Aug | 15 |
| **v1.37.0 RELEASED** | Branch Manager | **Wed 26 Aug** | 15 |
| Release blog published | Comms Lead | Wed 26 Aug | 15 |
| Thaw | Branch Manager | Wed 26 Aug | 15 |
| Feature blog publication starts | Comms Lead | Thu 27 Aug | 15 |

**Derived working calendar for the run-out:**

```
Tue 18 Aug  W14  R-8   ← snapshot taken here
Wed 19 Aug  W14  R-7   v1.37.0-rc.1 cut
Thu 20 Aug  W14  R-6
Fri 21 Aug  W14  R-5   weekly branch sync day; SIG feedback cutoff
Mon 24 Aug  W15  R-2   FINAL RELEASE NOTES PR MUST MERGE
Tue 25 Aug  W15  R-1   freeze day (see §7.5)
Wed 26 Aug  W15  R-0   RELEASE DAY (~4 hours of work)
Thu 27 Aug  W15  R+1   set up v1.38
```

---

## 5. State snapshot — verified 17–18 August 2026

> ⚠️ **Stale by construction.** Re-verify with §5.1 before acting.

### Completed (merged PRs by @kernel-kun)

| PR | What |
|---|---|
| `kubernetes/k8s.io#9535` | v1.37 docs shadows → `release-team`, `release-team-shadows` groups |
| `kubernetes/sig-release#3031` | Docs shadows added to `release-1.37/release-team.md` |
| `kubernetes/org#6408` | Docs team → `website-milestone-maintainers`, `release-team-docs` |
| `kubernetes/website#55933` | `OWNERS_ALIASES` updated for v1.37 |
| `kubernetes/website#55930` | `hugo.toml` on `dev-1.37` (`latest = "v1.37"`, dropdown updated) |
| `kubernetes/website#56104` | Branch sync main → dev-1.37 (13 Jun) |
| `kubernetes/sig-release#3074` | `v1.37.0-rc.0` release notes draft (12 Aug) |

Also done (confirmed verbally, not visible via API): **v1.38 Docs Lead nominated** and cleared with the
Subproject Leads / Release Lead; **kubeadm docs verified** with SIG Cluster Lifecycle; **the Docs view
of the tracking board is current**.

### Open and in-flight

**`kubernetes/website#55932` — the integration branch PR** (`dev-1.37` → `main`)
- Title: `[WIP] Official v1.37 Release Docs`
- 202 commits, 161 files, +7155/−2605
- Labels: `cncf-cla: yes`, `size/XXL`, **`do-not-merge/work-in-progress`**, `sig/docs`,
  **`do-not-merge/hold`**, `language/en`, `language/zh`, `area/localization`
- `mergeable: true`, `mergeable_state: unstable` (checks failing/pending)
- ⚠️ **No `lgtm`, no `approved`.** Cannot merge on release day in this state.

**`kubernetes/website#56990` — the release blog post** (`main` ← `v1.37-release-announcement`)
- `Kubernetes v1.37 Release Announcement Blog`, author **@SwathiR03 (Comms Lead)**, opened 14 Aug
- 1 file: `content/en/blog/_posts/2026/kubernetes-v1-37-release/index.md`, +853 lines, 15 commits
- Labels: `cncf-cla: yes`, `size/XL`, `tide/merge-method-squash`, `language/en`, `area/blog`
- Heavy review traffic (graz-dev, danwinship, adrianmoisey, dipesh-rawat, lmktfy, TineoC, natalisucks,
  aibarbetta, yliaog, michaelasp, jackfrancis)
- ⚠️ **No `lgtm`, no `approved`, NO `do-not-merge/hold`, no milestone.** Open `CHANGES_REQUESTED` from
  @TineoC (×3) and @jackfrancis as of 17 Aug.

### Branch state (`kubernetes/website`)

| Branch | Exists? | Note |
|---|---|---|
| `dev-1.37` | ✅ | **30 commits behind `main`**, 202 ahead. Last sync `#56998` (@Caesarsage, 16 Aug) |
| `release-1.35` | ✅ | |
| `release-1.36` | ❌ | Correct — created 25 Aug (§7.5) |
| `dev-1.38` | ❌ | Correct — created 27 Aug (§7.7) |

### Does not exist yet — these are the gaps

| Gap | Severity | Section |
|---|---|---|
| **Final Release Notes review PR** in `k/sig-release` — not started | 🔴 critical | §7.1 |
| **Known Issues umbrella issue** in `k/k` | 🔴 | §7.2 |
| **SIG Docs release-day PoC** not confirmed | 🔴 | §7.3 |
| `1.37` entry in `data/releases/schedule.yaml` | 🟡 | §7.4a |
| `hugo.toml` PRs for 1.33 / 1.34 / 1.35 / 1.36 | 🟡 | §7.4b |
| `hugo.toml` patch bumps on `dev-1.37` | 🟡 | §7.4c |
| `website-maintainers` org PR (temp write access) | scheduled 25 Aug | §7.5 |
| `tide/merge-blocker` freeze issue | scheduled 25 Aug | §7.5 |

### 16 PRs still open against `dev-1.37`

The tracking board is current, so these are **decided** — the work is mechanical retargeting so they
aren't stranded when `dev-1.37` merges into `main`.

| PR | Author | Note |
|---|---|---|
| `#56956` | tico88612 | Metrics API in observability concepts — `lgtm`, `hold`, WIP |
| `#56916` | mateenali66 | Pod shared pool glossary term |
| `#56912` | locker95 | drop teletype style for CompositePodGroup — `hold` |
| `#56639` | **Caesarsage (docs shadow)** | Repoint DRA links — `lgtm`, `hold` |
| `#56515` | p0lyn0mial | WatchListCompression feature gate |
| `#56401` | pmengelbert | KEP-6060 blog entry — WIP |
| `#56375` | sreeram-venkitesh | KEP-4960 — WIP |
| `#56368` | yongruilin | KEP-5958 placeholder — WIP |
| `#56339` | nispriha | KEP-6035 — WIP |
| `#56328` | helayoty | KEP-5732 TAS beta — WIP |
| `#56319` | BhargaviGudi | KEP-6063 Per-Pod PID — WIP |
| `#56250` | everpeace | KEP-5491 DRA list types — WIP |
| `#56237` | matthyx | KEP-4438 placeholder — WIP |
| `#56235` | radoslawc | KEP-6122 — WIP |
| `#56218` | SchSeba | KEP-5941 DRA shared capacity — `needs-rebase` |
| `#56134` | ehearne-redhat | KEP-5681 authz/webhook — WIP |

### 5.1 Re-verification commands

```bash
# Rate budget first
curl -sS https://api.github.com/rate_limit | python3 -c "import json,sys;d=json.load(sys.stdin);print(d['resources']['core'],d['resources']['search'])"

# Integration branch PR
curl -sS https://api.github.com/repos/kubernetes/website/pulls/55932 | python3 -c "
import json,sys;d=json.load(sys.stdin)
print(d['title']);print('state',d['state'],d.get('mergeable_state'));print([l['name'] for l in d['labels']])"

# Release blog PR
curl -sS https://api.github.com/repos/kubernetes/website/pulls/56990 | python3 -c "
import json,sys;d=json.load(sys.stdin)
print(d['title']);print('state',d['state'],d.get('mergeable_state'));print([l['name'] for l in d['labels']])"

# Branch existence
for b in dev-1.37 release-1.36 dev-1.38; do
  echo "$b -> $(curl -sS -o /dev/null -w '%{http_code}' https://api.github.com/repos/kubernetes/website/branches/$b)"
done

# How far behind is dev-1.37?
curl -sS https://api.github.com/repos/kubernetes/website/compare/main...dev-1.37 | python3 -c "
import json,sys;d=json.load(sys.stdin);print('ahead',d['ahead_by'],'behind',d['behind_by'])"

# Open PRs against dev-1.37 (core API, cheap)
curl -sS "https://api.github.com/repos/kubernetes/website/pulls?state=open&base=dev-1.37&per_page=100" | python3 -c "
import json,sys
for p in json.load(sys.stdin):
    print(p['created_at'][:10],'#%s'%p['number'],p['user']['login'],p['title'][:60],[l['name'] for l in p['labels'] if 'merge' in l['name'] or l['name'] in ('lgtm','approved','needs-rebase')])"

# Tushar's PRs (search API — costs against the 10/min budget)
curl -sS "https://api.github.com/search/issues?q=author:kernel-kun+org:kubernetes+type:pr+created:%3E2026-08-01&sort=created&order=desc" | python3 -c "
import json,sys;d=json.load(sys.stdin)
[print(i['created_at'][:10],i['state'],i['html_url']) for i in d.get('items',[])]"

# Latest patch versions (needed for hugo.toml)
curl -sS "https://api.github.com/repos/kubernetes/kubernetes/releases?per_page=40" | python3 -c "
import json,sys,re
seen={}
for r in json.load(sys.stdin):
    m=re.fullmatch(r'v(1\.\d+)\.(\d+)',r['tag_name'])
    if m and m.group(1) not in seen: seen[m.group(1)]=r['tag_name']
print(seen)"

# Release notes draft files
curl -sS https://api.github.com/repos/kubernetes/sig-release/contents/releases/release-1.37/release-notes | python3 -c "
import json,sys;[print(f['name'],f['size']) for f in json.load(sys.stdin)]"
```

---

## 6. The people

| Role | Lead | Shadows |
|---|---|---|
| **Release Team Lead** | Dipesh Rawat `@dipesh-rawat` (Slack `@Dipesh`) | Agustina Barbetta `@aibarbetta`, Prajyot Parab `@Prajyot-Parab`, Rayan Das `@rayandas`, **Sayan Chowdhury `@sayanchowdhury` (Slack `@yudocaa`)** |
| Enhancements | Subhasmita Swain `@whtssub` | ChengHao Yang `@tico88612`, Dhanisha Phadate `@dhanishaphadate`, Karim Farid `@karimzakzouk`, Lauri Apple `@lasomethingsomething`, Ofir Cohen `@ofirc` |
| **Communications** | **Swathi Rao `@SwathiR03`** (author of blog PR #56990) | Arsh Sharma `@RinkiyaKeDad`, **Christopher Tineo `@TineoC`**, Kirti Goyal `@kirti763`, Sophia Ugochukwu `@SophiaUgo`, Troy Connor `@troy0820` |
| Release Signal | Muhammad Adil Ghaffar `@adilGhaffarDev` | Aman Shrivastava `@aman4433`, El Mahdi El Araby `@x0rw`, Keisuke Ishigami `@kei01234kei`, M Junaid Shaukat `@junaiddshaukat`, Peppi-Lotta Kurjenhovi `@peppi-lotta`, Tatiana Selezneva `@TatianaSelezneva` |
| **Docs (us)** | **Tushar Mittal `@kernel-kun`** | Chad M. Crowell `@chadmcrowell`, Destiny Erhabor `@Caesarsage`, Josh Michielsen `@jmickey`, Saurabh Kumar Singh `@singh1203`, Yashasvi Misra `@yashasvimisra2798` |
| **Branch Manager** | **Agustina Barbetta `@aibarbetta`** | Jenny Shu `@jenshu`, Ryota Sawada `@rytswd` |

> 💡 **@sayanchowdhury was the v1.36 Docs Lead and is a Release Lead shadow this cycle.** He has
> personally done every remaining task on this list, four months ago. He is the cheapest possible
> source of help — use him.

**Escalation path for an unresponsive PR/KEP owner** (handbook order):
1. Comment on the PR + Slack DM the owner
2. Ping SIG chairs/tech leads in `#chairs-and-techleads`
3. Join the SIG's regular meeting and raise it live
4. Last resort: Subproject Lead(s) + Release Lead — give them ≥1 week's notice

**People who must be on the hook for release-day permissions** (Tushar does not have these):
- A **SIG Docs chair** — creates `release-1.36`, applies `tide/merge-blocker`, approves the
  `website-maintainers` org PR, does the Netlify site config.
- **@aibarbetta** (Branch Manager) — owns the release cut timing.

---

## 7. Task queue

### 7.1 🔴 CRITICAL — Final Release Notes review PR

**Status: not started. Merge deadline Mon 24 Aug. This is the single biggest risk in the cycle.**

The handbook says start this at Docs Freeze (5 Aug) and finish the bulk before `rc.1` (19 Aug). It is
late. The binding constraint is **SIG chair review latency, not editing speed** — so open a rough PR
and ping the chairs *first*, then edit. Do not wait for the content to be good before asking.

Files (in `kubernetes/sig-release`, path `releases/release-1.37/release-notes/`):
- `release-notes-draft.md` — 124 KB
- `release-notes-draft.json` — 396 KB
- ⚠️ **Both files must receive identical changes and stay in sync.**

```bash
git clone https://github.com/kernel-kun/sig-release.git && cd sig-release
git remote add upstream https://github.com/kubernetes/sig-release.git
git fetch upstream master && git checkout -b kernel-kun/v1.37-final-release-notes-review upstream/master
# ... edit both files ...
git add releases/release-1.37/release-notes/
git commit -m "Final review and cleanup of v1.37 release notes"
git push origin kernel-kun/v1.37-final-release-notes-review
```

PR title: `Final review and cleanup of v1.37 release notes`
Precedents: [`sig-release#2928`](https://github.com/kubernetes/sig-release/pull/2928) (v1.35),
[`sig-release#3004`](https://github.com/kubernetes/sig-release/pull/3004) (v1.36).

**Day-by-day plan (working backwards from Mon 24 Aug):**

| Day | Action |
|---|---|
| **Tue 18 Aug** | Branch. One structural pass (ACTION_REQUIRED placement, SIG attribution, drop non-user-facing noise). Push. **Open the PR.** Post Template A-1 to `#chairs-and-techleads` with a **Fri 21 Aug (AoE)** cutoff. Split the draft across the 5 shadows by SIG and post Template A-6 in the docs Slack group. |
| **Wed 19 Aug** | `rc.1` notes PR (§7.6). Fold new notes into the final PR same-day. |
| **Thu 20–Fri 21** | Bulk copy-edit with shadows. Hand-validate the **External Dependencies** section (auto-aggregated, frequently malformed). |
| **Fri 21 (AoE)** | SIG feedback cutoff. Unanswered SIGs → proceed on best available info and tell the Release Lead. Do not silently omit a SIG's major themes. |
| **Sat 22–Sun 23** | Incorporate feedback; chase `lgtm` + `approved`. |
| **Mon 24 Aug** | **MERGE.** Tue 25 is buffer only. |

**Review checklist** (what "good" looks like):
- Past tense throughout — the changes already happened
- Technical jargon (`VAC`, "scheduling hints") briefly explained or backticked
- User-facing context added where a note is bare; check the originating `k/k` PR for intent
- `ACTION_REQUIRED` section is *sparing* — consult the SIG before putting a note there
- SIG attribution on each note is correct
- Every `k/k` PR merged since the last patch release is accounted for
- Release Highlights (from the Comms meeting) are reflected in the Highlights section
- `.md` and `.json` in sync

**Tell the Release Lead today that this is amber.** The handbook explicitly prefers an early yellow to
a late surprise. Message: Template A-7.

---

### 7.2 🔴 Known Issues umbrella issue

**Status: does not exist.**

Create in `kubernetes/kubernetes`. Template:
https://github.com/kubernetes/sig-release/blob/master/release-team/role-handbooks/docs/known-issues-bucket.md
Precedents: [1.25 #110336](https://github.com/kubernetes/kubernetes/issues/110336),
[1.24 #109027](https://github.com/kubernetes/kubernetes/issues/109027),
[1.23 #104885](https://github.com/kubernetes/kubernetes/issues/104885).

- [ ] Create it (today)
- [ ] Link it in the Template A-1 chairs post — that template already asks chairs to flag Known Issues,
      so one message serves both §7.1 and §7.2
- [ ] Ask CI Signal (`@adilGhaffarDev`) and Bug Triage at burndown whether anything belongs in it
- [ ] **It must be closed and resolved on 25 Aug** (§7.5 step 9)

---

### 7.3 🔴 Release-day support and permissions

**Status: no SIG Docs PoC confirmed.** Three of the 25/26 Aug steps need permissions Tushar doesn't
have. Without a named person these become release-day blockers.

- [ ] Post Template A-2 in `#sig-docs` **and** raise it at the SIG Docs meeting
- [ ] Get a **named SIG Docs chair** committed for 25 Aug (create `release-1.36`, apply
      `tide/merge-blocker`, approve the org PR) and 26 Aug (Netlify)
- [ ] Confirm **generated docs are ready**: Kubernetes API reference, kubectl docs, components docs
      (produced by a SIG Docs tech lead)
- [ ] Confirm the exact **release cut time** with @aibarbetta in `#release-management` (Template A-3),
      and restate the ≥24 h release-notes rule back to them

---

### 7.4 🟡 Prep-now, merge-later PRs — open today, leave on hold

> ⚠️ **DO NOT MERGE ANY OF THESE UNTIL THE RELEASE HAS OCCURRED.**

#### 7.4a Releases page entry — PR against `dev-1.37`

File: `data/releases/schedule.yaml`. Add:

```yaml
- endOfLifeDate: "2027-10-28"
  maintenanceModeStartDate: "2027-08-28"
  next:
    cherryPickDeadline: "2026-09-04"
    release: 1.37.1
    targetDate: "2026-09-09"
  release: "1.37"
  releaseDate: "2026-08-26"
```

Derivation (verified against the existing 1.36 and 1.35 entries):
- `targetDate` = 2nd Wednesday of the following month → September 2026 Wednesdays are 2/9/16/23/30 → **9 Sep**
- `cherryPickDeadline` = the Friday before `targetDate` → **4 Sep**
- `endOfLifeDate` = release month + 14 months, day 28 → Aug 2026 + 14 = **28 Oct 2027**
- `maintenanceModeStartDate` = 2 months before EOL → **28 Aug 2027**

- [ ] Ask a **Release Manager to `/lgtm` the dates**
- [ ] Merged by Docs Lead **on release day**
- Precedent: [`website#55435`](https://github.com/kubernetes/website/pull/55435), with follow-up fix
  [`#55477`](https://github.com/kubernetes/website/pull/55477) — double-check `releaseDate`

#### 7.4b `hugo.toml` for the 4 previous releases — 4 separate PRs

For v1.37 the four previous releases are **1.33, 1.34, 1.35, 1.36**.

| Release | Base branch | `baseURL` |
|---|---|---|
| 1.33 | `release-1.33` | `https://v1-33.docs.kubernetes.io/` |
| 1.34 | `release-1.34` | `https://v1-34.docs.kubernetes.io/` |
| 1.35 | `release-1.35` | `https://v1-35.docs.kubernetes.io/` |
| **1.36** | **`main` → retarget to `release-1.36` on 25 Aug** | `https://v1-36.docs.kubernetes.io/` |

In each PR:
- `baseURL` → the versioned URL above
- `latest = "v1.37"`
- `deprecated = true`
- versions list: add v1.37, **drop the oldest (v1.32)**
- `githubbranch` / `fullversion` → newest patch for each release

Latest patches **as of 18 Aug 2026** — re-verify on 25 Aug, a patch round may land first:
**v1.36.3, v1.35.7, v1.34.10, v1.33.13**

Precedents: [`#55448`](https://github.com/kubernetes/website/pull/55448),
[`#55449`](https://github.com/kubernetes/website/pull/55449),
[`#55450`](https://github.com/kubernetes/website/pull/55450),
[`#55468`](https://github.com/kubernetes/website/pull/55468)

Commands in Appendix B-3.

#### 7.4c `hugo.toml` patch bumps on `dev-1.37`

`dev-1.37`'s versions list is stale. Current → target:

| Entry | On `dev-1.37` now | Should be |
|---|---|---|
| v1.37 | `v1.37.0` | `v1.37.0` ✅ |
| v1.36 | `v1.36.0` | `v1.36.3` |
| v1.35 | `v1.35.3` | `v1.35.7` |
| v1.34 | `v1.34.6` | `v1.34.10` |
| v1.33 | `v1.33.10` | `v1.33.13` |

#### 7.4d Localization heads-up

- [ ] Comment on the release-cycle GitHub discussion in `k/sig-release` (Template A-4): freeze **25 Aug**,
      release **26 Aug**, branches in sync

---

### 7.5 Week 14 remainder and Week 15 lead-in

**Wed 19 Aug — `v1.37.0-rc.1` release notes**
- [ ] Assign generator + reviewer from the Responsibilities Signup Sheet; post Template A-6
- [ ] Run `krel release-notes` for `v1.37.0-rc.1` (Appendix B-1), PR to `k/sig-release` (pattern `#3074`)
- [ ] Fold every rc.1 note into the Final Release Notes PR the same day

**Integration PR `#55932` cleanup — can be done today**
- [ ] Rename `[WIP] Official v1.37 Release Docs` → `Official v1.37 Release Docs`
- [ ] `/remove-label do-not-merge/work-in-progress`
- [ ] **Keep `do-not-merge/hold`**
- [ ] Chase the failing/pending checks (`mergeable_state: unstable`)
- [ ] Request SIG Docs review so `approved` + `lgtm` land well before 26 Aug

**Blog PR `#56990` — Comms owns it, Docs merges it**
- [ ] Ask Comms (`@SwathiR03`) to `/hold` it — it must not merge early once it gets `lgtm`
- [ ] Ask for `/milestone 1.37`
- [ ] Confirm front matter includes `release_announcement: true` (v1.36 needed fixes
      [`#55472`](https://github.com/kubernetes/website/pull/55472),
      [`#55479`](https://github.com/kubernetes/website/pull/55479))
- [ ] Track the open `CHANGES_REQUESTED` (TineoC ×3, jackfrancis) daily; needs `approved` + `lgtm` by Mon 24
- [ ] Plan a post-merge link check — v1.36 needed [`#55494`](https://github.com/kubernetes/website/pull/55494)

**Retarget the 16 orphan `dev-1.37` PRs (before the 25 Aug final sync)**
When `dev-1.37` merges into `main` on 26 Aug, anything still based on it is stranded. Post Template A-5
on each, then:
- [ ] `#56639` (Caesarsage), `#56218` (SchSeba, needs rebase) — decide in/out; if in, drive to
      `lgtm`+`approved` before 25 Aug
- [ ] `#56956` `#56916` `#56912` `#56515` — non-KEP content; retarget base to `main`
- [ ] `#56401` `#56375` `#56368` `#56339` `#56328` `#56319` `#56250` `#56237` `#56235` `#56134` —
      WIP KEP placeholders that missed Docs Freeze; retarget to `dev-1.38` once it exists (27 Aug), or
      park on `main` for the v1.38 lead
- [ ] Flag any **approved Docs Freeze exception** to the Release Lead — those must land before 25 Aug
- [ ] Put the retarget list in the v1.38 handover notes

**Fri 21 Aug — weekly branch sync**
- [ ] `main → dev-1.37` (it was 30 commits behind on 18 Aug). Use the branch-sync-script or Appendix B-2
- [ ] Report branch health at burndown

**Mon 24 Aug**
- [ ] **Final Release Notes PR merged** (needs `lgtm` + `approved` by Sun 23)
- [ ] Post week-15 assignments
- [ ] Review the [v1.37 milestone](https://github.com/kubernetes/kubernetes/milestone/70) — bump anything not landing
- [ ] Verify all §7.4 PRs are open, reviewed, and held
- [ ] Confirm the release cut time one final time

---

### 7.6 Tue 25 Aug — day before release

**Order matters.** Coordinate every step with a SIG Docs chair.

1. **Temporary write access** — PR against `kubernetes/org` adding `kernel-kun` to
   [`website-maintainers`](https://github.com/orgs/kubernetes/teams/website-maintainers). Assign to SIG
   Docs chairs. Precedent: [`org#6308`](https://github.com/kubernetes/org/pull/6308).
   Then locally: `git remote set-url --push upstream no_push` — with elevated access it becomes possible
   to push directly to upstream by accident.
2. **Freeze `k/website`** — file an issue and apply the **`tide/merge-blocker`** label. Apply it via the
   Labels gear icon, **not** a `/tide` command. A SIG Docs chair may need to do it. Precedent:
   [`website#55451`](https://github.com/kubernetes/website/issues/55451). Then announce the freeze
   (Template A-8) in `#sig-docs`, `#sig-release`, `#kubernetes-contributors`.
   *No PR may merge to `k/website` until the release PR merges — your release PRs bypass this.*
3. **Create `release-1.36`** from `main` in `k/website` (needs write access). ⚠️ First confirm no stale
   `release-1.36` exists — it did not on 18 Aug. In v1.29 an abandoned branch left the team with a
   release branch 2,330 commits behind `main`.
4. **Retarget** the 1.36 `hugo.toml` PR from `main` → `release-1.36`.
5. **Netlify** — with a SIG Docs chair, create a site building from `release-1.36`:
   site name `k8s-v1-36`, custom domain `v1-36.docs.kubernetes.io`. Defaults are fine; compare against
   an existing site when unsure.
6. **Final syncs** — merge `main → dev-1.37` (precedent [`#55469`](https://github.com/kubernetes/website/pull/55469))
   and `main → release-1.36` (precedent [`#55470`](https://github.com/kubernetes/website/pull/55470)).
   Merge both manually with **Create a merge commit**.
7. **📌 Record the last commit hash of `release-1.36`** — needed for tagging on release day.
8. **Approvals** — integration PR `#55932` and all config PRs reviewed and approved, WIP label gone,
   `hold` still on.
9. **Close the Known Issues issue**; confirm everything in it is resolved.
10. **Review the milestone**; move out anything that won't land and tell the owners.

---

### 7.7 Wed 26 Aug — RELEASE DAY (~4 hours)

Coordinate timing with Branch Release Management, the SIG Docs chair, and the Release Lead. For v1.21,
docs merged at 11:00 PDT and the blog at 11:30 PDT.

1. **Wait for the release to be officially cut.** Do not start early.
2. **Merge integration PR `#55932`** manually via **Create a merge commit**.
   - Verify `approved` + `lgtm`; remove `hold`
   - ⚠️ **Do not delete `dev-1.37`** when GitHub offers
   - *Note: in v1.28 the team hit failing CLA checks here and chose to merge anyway rather than rewrite
     history — the commits had passed CLA when merged to `dev-1.28`. Precedent exists for that call.*
3. **📌 Record the integration-merge commit hash.**
4. **Validate** — check [Netlify build logs](https://app.netlify.com/sites/kubernetes-io-main-staging/deploys),
   then the live site: navigation, version dropdown,
   [generated API reference](https://kubernetes.io/docs/reference/),
   [supported doc versions](https://kubernetes.io/docs/home/supported-doc-versions/), random clicks.
5. **Publish the release blog** — merge `#56990` manually with a merge commit. Verify labels, remove
   hold. Confirm it renders at https://kubernetes.io/blog/. **Post the link to the release team** so the
   Release Lead can announce to `kubernetes-dev`.
6. **Merge the `schedule.yaml` / releases-page PR.**
7. **Unfreeze `k/website`** — remove `tide/merge-blocker`, close the issue, announce (Template A-9).
8. **Merge the 4 previous-release `hugo.toml` PRs** — remove holds, let Prow merge them.
9. **Close the v1.37 milestone.** ⚠️ Close, do not delete.
10. **Tag** (can be done any time post-release) — Appendix B-4.

---

### 7.8 Thu 27 Aug onward — set up the v1.38 lead

- [ ] **Create `dev-1.38`** from `main` immediately — this unblocks the next cycle
      (`git commit --allow-empty -m "Tracking commit for v1.38 docs"`)
- [ ] **`k/test-infra` PR**: milestone applier `dev-1.37: 1.37` → `dev-1.38: 1.38`. Precedent:
      [`test-infra#36913`](https://github.com/kubernetes/test-infra/pull/36913) (v1.36 lead did this at R+6)
- [ ] Create the **1.38 milestone** in `k/website`; move anything missed into it
- [ ] **Netlify**: point `kubernetes-io-vnext-staging` at `dev-1.38`; delete the oldest docs site
      (**v1-32**); verify at https://kubernetes-io-vnext-staging.netlify.com/
- [ ] Announce in `#sig-docs` that `dev-1.38` is open for feature docs
- [ ] Reassign leftover issues/PRs to the incoming Docs Lead
- [ ] **Refresh patch-release data** in `data/releases/schedule.yaml` — it was stale on 18 Aug (`1.36.3`
      still listed as `next` with a July target date). Use
      `schedule-builder -uc data/releases/schedule.yaml -e data/releases/eol.yaml`.
      Precedent: [`website#55517`](https://github.com/kubernetes/website/pull/55517)
- [ ] **Access cleanup**: PR against `k/org` removing Tushar from `website-maintainers` and the shadows
      from `website-milestone-maintainers`; PR against `k/website` removing him from `sig-docs-en-owners`.
      ⚠️ He stays in `website-milestone-maintainers` while v1.37 is supported; the incoming lead stays too.
- [ ] Hold the **docs-only retro** with SIG Docs + shadows; open handbook-improvement PRs
- [ ] **Transition call** with the v1.38 lead — focus on the last-week and release-day steps, which only
      the Docs Lead performs
- [ ] Update `kernel-kun/release-team-utils` with a v1.37 activity log so the next lead inherits it
- [ ] 🎉 Celebrate

---

## Appendix A — Message templates

### A-1 · `#chairs-and-techleads` — Final Release Notes review
```
Hello #chairs-and-techleads! 👋

The Release Docs Team has prepared the final draft of the v1.37 release notes, and we'd appreciate
your help in reviewing the notes related to your SIG.

Please review the draft here: <PR link>

Things to review:
- Search (Ctrl+F) for your SIG and suggest moving notes to an appropriate location in PR comments.
- If you are aware of anything that should appear in the "Known Issues" section, please leave a
  comment with the issue and a draft of the note text. Tracking issue: <known issues issue link>
- Please review for technical inaccuracies, grammar inconsistencies, etc.
- Please remove minor and/or non-user-facing release notes contributed by your SIG.
- For notes associated with multiple SIGs, suggest the most appropriate SIG categorization.

Feedback cutoff: Friday 21 August 2026 (AoE). The PR must merge by Monday 24 August, one day before
the release cut.

Thank you so much for supporting the release process!
```

### A-2 · `#sig-docs` — release-day support
```
Hi @sig-docs-leads 👋

The v1.37 release is scheduled for Wednesday, 26 August 2026.

Could someone from SIG Docs be the point of contact for the Docs Lead on release day? Specifically:
1. Release day support — help with Netlify configuration updates and release blog post publication
2. Day-before (25 Aug) — creating the release-1.36 branch and applying the tide/merge-blocker label
3. Generated docs status — please confirm these are ready: Kubernetes API reference, kubectl
   documentation, components documentation

Thank you!
```

### A-3 · `#release-management` — release cut timing
```
Hi @aibarbetta 👋 v1.37 Docs Lead here.

Can you confirm the exact release cut time on Wednesday 26 August?

I need it to schedule the Final Release Notes PR merge — that has to land at least 24 hours before
the cut, otherwise the CHANGELOG in k/k ships stale. My plan is to merge it Monday 24 August.

Also flagging: I'm starting the final release notes review later than ideal, so I'd call it amber
right now. I'll update daily at burndown.
```

### A-4 · Localization teams (comment on the sig-release discussion)
```
Hello localization team leads!

I don't think any action is required from you, but I wanted to let you know that we are on track for
the release on Wednesday 26 August 2026. The kubernetes/website repo will be frozen from Tuesday
25 August until the release PR merges. All branches (main, dev-1.37) are up to date.

Let me know if I can help with anything! Thanks!
```

### A-5 · Missed Docs Freeze (comment on the PR/KEP)
```
Hello {doc/KEP owners} 👋! v1.37 Docs team here.

This PR did not meet the deadline for docs freeze
(https://github.com/kubernetes/sig-release/blob/master/releases/release_phases.md#docs-freeze).
Enhancements without required documentation may be removed from the current release. If you still
wish to include this enhancement in v1.37, please file an exception request:
https://github.com/kubernetes/sig-release/blob/master/releases/EXCEPTIONS.md

Otherwise, please retarget this PR — dev-1.37 merges into main on 26 August and will no longer be a
valid base branch after that.

Thank you!
```

### A-6 · Weekly assignments (docs team Slack DM group)
```
Hey folks, welcome to Week <N>! :kubernetes-intensifies:

The tasks for this week, along with the assigned owners:
-----------------------------------------------------------------------------------
Release Team Updates (Meeting Link)
<DATE> (<DAY>)
APAC Meeting (<TIME> UTC): @<handle>
EMEA Meeting (<TIME> UTC): @<handle>
-----------------------------------------------------------------------------------
Weekly Branch Sync PR:
<DATE> (<DAY>)   Assignee: @<handle>
-----------------------------------------------------------------------------------
BURNDOWN Meeting
<DATE> (<DAY>)   APAC (<TIME> UTC): @<handle>   EMEA (<TIME> UTC): @<handle>
-----------------------------------------------------------------------------------
v1.37.0-rc.<N> release notes
<DATE> (<DAY>)   Assignee: @<handle>   Reviewer: @<handle>
-----------------------------------------------------------------------------------
Final Release Notes review — assigned SIG sections
@chadmcrowell: <SIGs>   @Caesarsage: <SIGs>   @jmickey: <SIGs>
@singh1203: <SIGs>      @yashasvimisra2798: <SIGs>
```

### A-7 · Status colour for the release team meeting
SIG Docs explicitly welcomes yellow/red. Report yellow when: branch sync has lapsed 1–2 weeks, or
<80% of docs PRs are at the expected state a week before a deadline. Report red at 3+ lapsed syncs or
<60%. Current honest status for the final release notes: **amber**, started late, mitigation is
parallel SIG review with a Fri 21 Aug cutoff.

### A-8 · Website freeze announcement
```
Hi all 👋 v1.37 Docs Lead here.

The kubernetes/website repo is now FROZEN ahead of the v1.37 release on Wednesday 26 August.
No PRs will merge until the release PR has merged. Tracking issue: <issue link>

Thanks for your patience!
```

### A-9 · Website unfreeze announcement
```
Hi all 👋 Kubernetes v1.37 is out! 🎉

The kubernetes/website repo is now UNFROZEN and merging normally again.
Release blog: https://kubernetes.io/blog/...

The dev-1.38 branch is open for v1.38 feature docs.

Thanks everyone!
```

---

## Appendix B — Commands and runbooks

### B-1 · Generate release notes for an rc
Set up `krel` first: https://github.com/kubernetes/release/tree/master/docs/krel#installation
Full usage: https://github.com/kubernetes/release/blob/master/docs/krel/release-notes.md

```bash
# generate the bundle for a tag, then open a PR against k/sig-release
krel release-notes --tag v1.37.0-rc.1
# interactive edit/fix flow used weekly during the cycle
krel release-notes --fix
```
Output lands in `releases/release-1.37/release-notes/` — `release-notes-draft.md`,
`release-notes-draft.json`, plus `maps/` and `sessions/`.

Known tooling gotcha: `krel release-notes` has historically skipped PRs caught in RC fast-forward sync
merges — see [`k/release#4381`](https://github.com/kubernetes/release/issues/4381). Cross-check the
`k/k` merge list if notes look thin.

### B-2 · Branch sync (`main` → `dev-1.37`)
Prefer the [branch-sync-script](https://github.com/kubernetes/sig-release/tree/master/release-team/role-handbooks/docs/branch-sync-script). Manual fallback:

```bash
git clone git@github.com:kernel-kun/website.git && cd website
git remote add upstream https://github.com/kubernetes/website.git
git remote set-url --push upstream no_push
git fetch upstream main
git fetch upstream dev-1.37
git checkout --track upstream/dev-1.37
git pull --ff-only
git merge upstream/main          # conflicts likely
git checkout -b merged-main-dev-1.37
git commit -m "Merge main into dev-1.37 to keep in sync"
git push origin merged-main-dev-1.37
# PR from the fork branch, base = dev-1.37
```

**Conflict guidance:** the dev-branch side is almost always an in-progress feature doc — preserve it and
layer the `main` update on top where they don't conflict. If a conflict isn't obvious, `git merge --abort`
and ask SIG Docs before retrying. Verify the branch builds before pushing.

### B-3 · `hugo.toml` for a previous release

For a release that already has a branch (e.g. 1.35):
```bash
git fetch upstream release-1.35
git checkout --track upstream/release-1.35
# edit hugo.toml per §7.4b
git checkout -b update-release-1.35-hugo.toml
git commit -am "Updates v1.35 hugo.toml for release v1.37"
git push origin update-release-1.35-hugo.toml
# PR base = release-1.35
```

For the immediately previous release (1.36 — branch doesn't exist until 25 Aug):
```bash
git fetch upstream main && git checkout --track upstream/main
# edit hugo.toml per §7.4b
git checkout -b update-release-1.36-hugo.toml
git commit -am "Updates v1.36 hugo.toml for release v1.37"
git push origin update-release-1.36-hugo.toml
# PR base = main initially; RETARGET to release-1.36 on 25 Aug after creating that branch
```

### B-4 · Tagging the snapshots (release day or later)
```bash
git clone https://github.com/kubernetes/website/ && cd website && git checkout main

# confirm the commit immediately before the integration merge
git show <integration-merge-commit>^1     # should be the tip of release-1.36

git tag -a snapshot-final-v1.36 <last commit of release-1.36> -m "Release 1.36 final snapshot"
git tag -a snapshot-initial-v1.37 <integration-merge-commit> -m "Release 1.37 initial snapshot"
git push --tags origin main
```
Then draft a GitHub release at https://github.com/kubernetes/website/releases on the
`snapshot-initial-v1.37` tag — title `snapshot-initial-v1.37: Release 1.37`, description
`Release 1.37 initial snapshot`.

### B-5 · Create `dev-1.38` (27 Aug)
```bash
git clone https://github.com/kubernetes/website.git && cd website
git checkout -b dev-1.38
git commit --allow-empty -m "Tracking commit for v1.38 docs"
git push -u origin dev-1.38
```

---

## Appendix C — Precedent map (what the last two leads actually did)

Offsets are relative to release day (R). Use these to sanity-check timing.

**v1.36** — @sayanchowdhury, released **Wed 22 Apr 2026**:

| Offset | Date | Action |
|---|---|---|
| R−2 | Mon 20 Apr | [`#55435`](https://github.com/kubernetes/website/pull/55435) add 1.36 to release schedule |
| R−1 | Tue 21 Apr | [`org#6308`](https://github.com/kubernetes/org/pull/6308) website-maintainers access |
| R−1 | Tue 21 Apr | [`sig-release#3004`](https://github.com/kubernetes/sig-release/pull/3004) **final release notes — opened AND merged same day** ⚠️ |
| R−1 | Tue 21 Apr | [`#55448`](https://github.com/kubernetes/website/pull/55448)/[`#55449`](https://github.com/kubernetes/website/pull/55449)/[`#55450`](https://github.com/kubernetes/website/pull/55450) hugo.toml v1.32/33/34 |
| R−1 | Tue 21 Apr | [`#55451`](https://github.com/kubernetes/website/issues/55451) freeze issue |
| R−0 | Wed 22 Apr | [`#55468`](https://github.com/kubernetes/website/pull/55468) hugo.toml v1.35; [`#55469`](https://github.com/kubernetes/website/pull/55469) final sync into dev-1.36; [`#55470`](https://github.com/kubernetes/website/pull/55470) main→release-1.35; [`#55477`](https://github.com/kubernetes/website/pull/55477) fix releaseDate |
| R+2 | Fri 24 Apr | [`#55517`](https://github.com/kubernetes/website/pull/55517) patch-release data refresh |
| R+6 | Tue 28 Apr | [`test-infra#36913`](https://github.com/kubernetes/test-infra/pull/36913) milestone applier dev-1.37 |

**v1.35** — @Urvashi0109, released **Wed 17 Dec 2025**: same shape, but the schedule and hugo.toml PRs
were opened ~R−6/R−5, **closed, and re-opened at R−0** — evidence that opening them too early without
a clear hold/retarget plan causes churn. Open them now, hold them, and keep them rebased.

**Read across both:** the previous-release config PRs and the final release notes have consistently been
compressed into the last 48 hours. That is the pattern to break, not copy — especially the final release
notes, which need multi-SIG sign-off that cannot be compressed.

---

## Appendix D — Failure modes and gotchas

| Gotcha | Why it matters |
|---|---|
| Final Release Notes merged <24 h before the cut | `k/k` ships a stale `CHANGELOG`. Unfixable after the fact. |
| `.md` and `.json` drift apart in the release notes | The JSON drives tooling; silent inconsistency. Always edit both. |
| A stale `release-1.36` branch already exists | v1.29 inherited a branch 2,330 commits behind `main`. Verify before creating; a repo/org admin can delete it, with SIG Docs chair agreement. |
| Blog PR merges early | It has no `hold` today. Once it gets `lgtm`, Tide can merge it. Get `/hold` on it now. |
| Merging config PRs before the release | Breaks the live site's version dropdown and banners. Hold all four. |
| Deleting `dev-1.37` when GitHub offers after the integration merge | Loses history and breaks orphaned PRs. Decline. |
| Using `/tide` to apply `tide/merge-blocker` | Doesn't work — apply the label via the gear icon in the Labels section. |
| Pushing to upstream after getting write access | Run `git remote set-url --push upstream no_push` immediately. |
| Closing vs. deleting the milestone | Close it. Deleting loses the association for every issue in it. |
| `krel` skipping PRs in RC fast-forward syncs | [`k/release#4381`](https://github.com/kubernetes/release/issues/4381) — cross-check against `k/k` merges. |
| CLA checks failing on the integration merge | v1.28 merged anyway rather than rewrite history; commits had passed CLA on `dev-1.28`. Precedent exists. |
| Blocking a `/lgtm` on style nits | Most PR authors aren't writers and many are non-native English speakers. Correctness and completeness first; open a follow-up PR for style. |

---

## Appendix E — Definition of done

The release is complete when all of these are true:

- [ ] Final Release Notes PR merged into `k/sig-release` (≥24 h before the cut)
- [ ] Known Issues umbrella issue closed and resolved
- [ ] Integration PR `#55932` merged into `main`; site builds; docs validated
- [ ] Release blog `#56990` merged and live at kubernetes.io/blog; link posted to the release team
- [ ] `schedule.yaml` PR merged — v1.37 appears on kubernetes.io/releases
- [ ] All 4 previous-release `hugo.toml` PRs merged
- [ ] `k/website` unfrozen and announced
- [ ] v1.37 milestone closed (not deleted)
- [ ] `snapshot-final-v1.36` and `snapshot-initial-v1.37` tags pushed; GitHub release drafted
- [ ] `dev-1.38` created; test-infra milestone applier updated; 1.38 milestone created
- [ ] Netlify: `v1-36.docs.kubernetes.io` live, vnext pointed at `dev-1.38`, oldest site (v1-32) deleted
- [ ] Access cleanup PRs opened
- [ ] Retro held; transition call with the v1.38 lead done
- [ ] `release-team-utils` activity log updated for v1.37
