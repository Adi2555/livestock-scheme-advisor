# Frontend — Pashu Yojana Sahayak

Drop these files into `livestock-scheme-advisor/frontend/`, replacing the old ones.

```
frontend/
  index.html     structure only, no hardcoded text        (unchanged)
  style.css      green and white theme                     (unchanged)
  app.js         API calls, language switching, results    (unchanged)
  i18n.json      every visible string, English + Marathi    UPDATED — 5 new scheme-kind labels
  schemes.json   scheme names, descriptions, links          UPDATED — 66 schemes, was 16
```

`index.html`, `style.css` and `app.js` are byte-for-byte what you uploaded — nothing in
this update touches them, because the card-rendering code already reads `kind` and
scheme codes generically from `schemes.json` and the backend's `GET /` response.

## Running it

Still: `cd frontend && python -m http.server 5500`, backend on port 8000. Browsers
block `fetch()` on `file://`, so double-clicking `index.html` won't load the scheme
data.

## What changed in this update

Your `rules_manifest.json` grew from 16 labels to 66 (its own `supersedes` field says
so: *"rules_manifest v1 (16 labels, livestock-only)"*). I read all 66, wrote bilingual
names, descriptions and a link for every one, and added the 5 new `kind` values
(`income_support`, `pension`, `award`, `survey`, `price_support`) to `i18n.json`. All
66 render cleanly in both languages — tested against a mocked backend before delivery.

**Before you treat this as done, there's something in the data itself you should
know about, not just the frontend files.**

### Only 32 of the 66 are actually livestock schemes

The other 34 are general-agriculture schemes — PM-KISAN, crop insurance (PMFBY), solar
irrigation pumps, soil health cards, organic farming clusters, e-NAM, a bamboo
mission, and so on. None of them are gated on breed, animal count, or anything to do
with owning livestock. They got pulled in because whoever expanded the manifest
(a research pass, going by the excellent per-scheme sourcing) mirrored the *entire*
DAHD/DA&FW scheme list rather than staying livestock-scoped:

```
C_AGROFORESTRY, C_AIF, C_CRM, C_DIGITAL_AGRI, C_FPO_10K, C_ISAM_ENAM, C_MIDH, C_MISS,
C_MIS_PSS, C_MOVCDNER, C_NAMO_DRONE_DIDI, C_NBM, C_NFSM, C_NMEO_OP, C_PDMC, C_PKVY,
C_PMDDKY, C_PMFBY, C_PM_KISAN, C_PM_KMY, C_RAD, C_RKVY, C_SHC, C_SMAE_ATMA, C_SMAM,
MH_AMBEDKAR_KRISHI_SWAVALAMBAN, MH_BIRSA_MUNDA, MH_GOPINATH_MUNDE, MH_KARJMUKTI,
MH_MAGEL_TYALA_SAUR_PUMP, MH_NAMO_SHETKARI, MH_NANAJI_DESHMUKH_POCRA,
MH_SHASHWAT_SINCHAN, MH_STATE_AGRI_MECHANIZATION
```

### The new CSV needs 8 fields your form doesn't ask for

`stage2_scheme_eligibility_slim.csv` now has `age`, `is_nt_c_dhangar`, `is_institution`,
`is_fpo_member`, `has_irrigation_source`, `grows_crops`, `is_incometax_payer`, and
`in_pmddky_district` — none of which the two-step form on this site collects. Your
backend's `/predict/eligibility` only sends the original six fields, so once a model
is retrained on this CSV, every scheme gated on one of those 8 fields will be
evaluated on a *guess*, not on what the farmer actually told you. Concretely:

- `MH_SHEEP_GRAZING_SUBSIDY` needs `is_nt_c_dhangar` — the one livestock scheme in
  the new 16 this form can't gate correctly yet.
- `MH_GOVARDHAN_GOVANSH`, `MH_GOAT_SEMEN_LAB`, `C_NPDD`, `C_SDCFPO` are institutional
  (goshalas / cooperatives / FPOs), same as the original `MH_GOSHALA` and `C_DIDF` —
  I wrote their card copy to say plainly "this is for the cooperative, not you
  directly," same pattern as before. No form change needed for these.
- Almost every one of the 34 general-agriculture schemes needs `grows_crops`,
  `has_irrigation_source`, `age`, or `is_incometax_payer` to mean anything.

The manifest itself is candid about a few entries that can **never** show as eligible
for anyone right now, no matter what a farmer answers:

- `C_CRM` — Punjab/Haryana/UP/Delhi only, hard-coded 0 for Maharashtra.
- `C_MOVCDNER` — north-eastern states only, hard-coded 0.
- `C_NMEO_OP`, `C_NBHM` — no supporting feature exists in the dataset, left at 0.
- `C_PMDDKY` — real scheme, but "*the 100-district list was not retrieved, so
  `in_pmddky_district` defaults to False for all rows... this label is 0 everywhere
  until you load the official list*" (their words). I wrote the card to say this
  outright rather than let it look like a real "not eligible" answer.

And two aren't really "schemes you're eligible for" at all: `C_LCISS` is a livestock
census (a survey, not a benefit — the manifest itself says to exclude it from ranking),
and `C_NGRA` is a competitive award you'd nominate for, not an entitlement. I wrote
both cards to say that plainly rather than showing a green "Eligible" badge that
implies a payment.

### What I'd do next — your call

1. **Ship livestock-only.** Filter `rules_manifest.json` back down to the 32 codes
   above (the original 16 plus the 16 genuine new ones), drop the 8 extra features,
   retrain, and keep this a livestock advisor. No form changes needed at all — I've
   already written correct copy for all 32.
2. **Go broad on purpose.** Keep all 66, but add the missing fields to Step 2 of the
   form (age, "are you an NT-C/Dhangar shepherd household?", "do you grow crops?",
   "irrigation source?", "income-tax payer?", FPO/institution checkbox) and retrain
   the backend against the full CSV. That's real scope — a bigger interview, a
   renamed product, and a retrain — not something I've done here.
3. **Ship as-is, eyes open.** Every one of the 66 has honest card copy now, including
   the ones that will always show "not eligible" or that flag themselves as
   institutional/award/survey. Nothing lies to the farmer. But roughly half the
   schemes still can't be gated correctly on what this form asks, so I wouldn't
   demo this as "we now cover 66 schemes" without the caveat.

I'd lean toward (1) for a college project unless the assignment specifically wants
broad farmer-scheme coverage — the breed-photo hook and the six-question form are
what make this a coherent product, and general-agriculture schemes dilute that.

## Scheme links

Same as before — `schemes.json` is keyed by scheme code, each entry has an English and
Marathi name, a one-line description, and the `url` the arrow button opens.
**Check every link before you demo.** Most central-scheme URLs I used are long-standing
official portals I'm confident about (pmkisan.gov.in, pmfby.gov.in, enam.gov.in,
rkvy.nic.in, midh.gov.in, nfsm.gov.in, agrimachinery.nic.in). A handful I could not
verify to a specific page and fell back to `agriwelfare.gov.in` (the Ministry
homepage) rather than guess a plausible-looking but unconfirmed deep link:
`C_MISS`, `C_MIS_PSS`, `C_NAMO_DRONE_DIDI`, `C_DIGITAL_AGRI`, `C_MOVCDNER`. Same
carry-over caution as last time on `C_DEDS` (NABARD, likely closed).

## Backend

Still unchanged — no edits to `app.py` in this pass. The frontend reads dropdown
values from `GET /`, so once you decide between the three options above and retrain,
the form will pick up whatever `feature_metadata.json` says without further frontend
changes — *unless* you go with option 2, which needs new form fields in `index.html`.
