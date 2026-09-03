# TunnelQuiz - cost breakdown

**Internal. Contains cost to serve and gross margins. Not for customers.**
Temporary document - delete with the rest of this directory when the pricing review closes.

Companion to the pricing model in `product/Pricing-Model.md` (main repository), sections 2.3
and 5.2. This file shows the arithmetic behind those two tables so every figure can be
checked rather than taken on trust.

Rates used throughout: **₹84 = $1**, **£1 = $1.28**, **€1 = $1.08**. LLM cost
**$0.000807 per 1,000 tokens** on the cheap tier. Cloudflare R2 at **$0.015 per GB-month**
storage, **$4.50 per million** class-A (write) operations, **zero egress**.

Each figure is tagged: **[M]** measured on a real run, **[P]** published vendor price,
**[A]** an allowance we chose, **[E]** an estimate from an assumption stated inline.

---

## 1. One attempt, itemised

### 1.1 Fully-proctored MCQ, 60 minutes

The heaviest thing a credit buys, before any AI.

| Component | Derivation | Cost |
|---|---|---|
| Webcam + screen snapshots, writes | 2 stills/min × 60 min = 120 objects; 120 × $4.50 / 1,000,000 **[P]** | $0.00054 |
| Snapshot storage, 90-day retention | ~15 MB × 3 months = 0.045 GB-months × $0.015 **[P]** | $0.00068 |
| Face check-in photo | 1 object, kept indefinitely | $0.00001 |
| Serving that media back to a reviewer | R2 egress is free **[P]** | $0 |
| Heartbeat writes | every 20s × 60 min = 180 writes **[E]** | negligible |
| Autosave writes | 1.5s debounce, ~200 writes over an hour **[E]** | negligible |
| Proctoring event rows | ~50 per attempt **[E]** | negligible |
| **Itemised marginal cost** | | **≈ $0.0013** |
| **Budgeted in the pricing model** | 5x headroom for app compute, bandwidth and the unattributed share of fixed infrastructure **[A]** | **$0.01** |

The gap between $0.0013 and $0.01 is deliberate. Attributing web, worker, Redis and Mongo
capacity per attempt is not meaningfully possible at current volume, so the model carries a
conservative allowance instead. **Every margin figure below uses the $0.01 allowance**, not
the itemised number - the real margins are higher than stated.

### 1.2 Unproctored MCQ

No media is written at all, so only the heartbeat and autosave rows survive: **≈ $0.0005**
**[E]**. This is what makes "unproctored is free on paid plans" affordable, and why the Free
tier can charge a full credit for it without the economics mattering either way.

### 1.3 The AI additions

| Attempt type | Derivation | Cost |
|---|---|---|
| Written, AI-graded | N+1 calls for a 10-question paper = 11 calls × ~1.5k tokens = 16.5k tokens **[E]** | **$0.013** |
| Code, AI-graded | Same path - execution is disabled in production, so test cases never run and the LLM reads the source **[E]** | **$0.013-0.03** |
| AI chat interview | ~12 calls with growing context, ~2.5k tokens average = 30k tokens **[E]** | **$0.024** |
| Live Follow | rrweb replay, bandwidth only, for as long as a recruiter watches **[E]** | **~$0.02 / hour watched** |

**Grading is the only real variable cost in the product.** A proctored MCQ costs a fifth of
a written answer's grading and a twentieth of a chat interview's. This is the single most
important fact in the pricing model, and it is why AI sits on Pro rather than being spread
across the tiers.

### 1.4 AI question generation - the one fully measured line

Measured on a real 45-item run, not modelled:

| Component | Tokens / item **[M]** | $ / item **[M]** | Share |
|---|---|---|---|
| Generation | 638 | $0.000513 | 70% |
| Critic | 196 | $0.000159 | **22%** |
| Duplicate check | 71 | $0.000058 | 8% |
| **All-in** | **905** | **$0.000730** | |

One AI credit is 165,000 tokens with a floor of 150 distinct questions, so **$0.130 per AI
credit** **[M]**. Sold at **₹65 ≈ $0.77**, that is an **83% margin**.

Note this supersedes the 48% margin in `AI-Credits-Pricing-Plan.md`: that document priced
the credit at $0.25, which was the cost-plus floor. ₹65 is the selling price.

---

## 2. Cost to serve per tier, worst case

Worst case means every included credit spent on the heaviest proctoring the tier permits,
and every included AI credit consumed. Payment fees at Stripe India, 2% + ₹3 **[P]**.

| | Free | Essential | Pro |
|---|---|---|---|
| Included credits | 50 | 150 | 250 |
| Heaviest allowed config | browser integrity only | full evidence capture | capture + AI face detection |
| Attempt cost | 50 × $0.0005 = **₹2** | 150 × $0.01 = **₹126** | 250 × $0.01 = **₹210** |
| AI grading | - | - | 40% of 250 written or code × $0.02 = **₹168** **[A]** |
| Included AI credits | - | - | 10 × $0.130 = **₹109** |
| Live Follow | - | - | 20 sessions × 1 h = **₹50** **[A]** |
| SMS OTP | - | 30% of invites × ₹0.20 = **₹9** **[A]** | **₹15** **[A]** |
| **Service cost** | **₹2** | **₹135** | **₹552** |
| Payment fees | - | ₹33 | ₹83 |
| **Total cost** | **₹2** | **₹168** | **₹635** |
| Revenue (tax-exclusive) | ₹0 | ₹1,499 | ₹3,999 |
| **Gross margin** | - | **₹1,331 · 89%** | **₹3,364 · 84%** |

### Where Pro's money actually goes

```
Attempt capture   ₹210  ████████████████████            33%
AI grading        ₹168  ████████████████                26%
AI generation     ₹109  ██████████                      17%
Payment fees      ₹ 83  ████████                        13%
Live Follow       ₹ 50  █████                            8%
SMS OTP           ₹ 15  █                                2%
                  ─────
                  ₹635
```

**₹277 of ₹635 - 44% - is AI.** Capture, the thing the proctoring ladder is built around, is
a third. That asymmetry is the argument for putting AI behind Pro's price rather than
bundling it lower.

---

## 3. Margin by market

Cost to serve is identical in every market; only the price and the card fees change. Fees:
India 2% + ₹3, US 2.9% + $0.30, UK 1.5% + £0.20, EU 1.5% + €0.25 **[P]**.

| Market | Essential price | Cost | **Margin** | Pro price | Cost | **Margin** |
|---|---|---|---|---|---|---|
| India | ₹1,499 | ₹168 | **89%** | ₹3,999 | ₹635 | **84%** |
| United States | $39 | $3.04 | **92%** | $99 | $9.74 | **90%** |
| United Kingdom | £29 | £1.90 | **93%** | £69 | £6.37 | **91%** |
| Euro area | €29 | €2.18 | **92%** | €79 | €7.53 | **90%** |

**India is the lowest-margin market by design** - it is the anchor priced for a land-grab,
and every other market is derived upward from it at the geometric mean of the nominal and
PPP exchange rates. Every market clears 84%.

---

## 4. What typical usage looks like

Worst case is not the expected case. A team that leaves 20% of its credits unused, runs half
its tests without media capture, and grades a third of written answers by hand:

| | Essential | Pro |
|---|---|---|
| Credits actually used | 120 | 200 |
| Of those, fully captured | 60 | 100 |
| Service cost | ₹56 | ₹300 |
| **Gross margin** | **₹1,410 · 94%** | **₹3,616 · 90%** |

So the honest range is **89-94% on Essential and 84-90% on Pro**, and the low end only
arrives if a customer uses everything they bought - which is the outcome we want.

---

## 5. Top-up margins

Fixed per unit, no pack or tier discounts.

| | Price | Cost to serve | **Margin** |
|---|---|---|---|
| Credit | ₹15 | ₹0.84 (one captured attempt) | **94%** |
| AI credit | ₹65 | ₹10.92 (measured) | **83%** |

Top-ups are the highest-margin revenue in the model, which is the reason included allowances
are sized to real usage rather than inflated: **a tier that includes more than its buyers
use never sells a top-up.**

---

## 6. Where these numbers could be wrong

Ordered by how much they would move the answer.

1. **AI grading share is an allowance, not a measurement.** The 40% assumption for Pro is
   the largest unmeasured input in section 2 - ₹168 of ₹635. If Pro customers write mostly
   written and coding papers rather than MCQ, this doubles and Pro's margin drops to about
   77%. **Instrument grading calls per attempt before scaling Pro.**
2. **Nothing meters grading or chat tokens today.** Question generation is fully metered
   against AI credits with real provider token counts; grading and chat are not. The
   estimator in the engine is `len(text) // 4` and grading jobs record no cost. Every
   grading figure here is therefore modelled.
3. **Retry amplification is not in any figure.** A job-level grading retry re-grades *all* N
   questions when one fails, up to three times. A systematic grading failure multiplies the
   grading line by 3.
4. **Live Follow bandwidth has never been measured.** ₹50 assumes 20 hour-long sessions. It
   is priced at zero credits on the argument that a human has to watch and so it is
   self-limiting; if that assumption fails it becomes a real line.
5. **The AI-credit redundancy allowance rests on one 45-item run.** 165,000 tokens per
   credit assumes 15.6% duplicate rate; the human reviewer discarded only 6.7%. Better
   redundancy means we are giving away headroom, worse means the 150-question floor costs us
   margin.
6. **Face-capture photos are kept indefinitely.** Excluded from the 90-day lifecycle rule, so
   their storage grows without bound. Trivial per attempt, unbounded in aggregate, and a
   biometric retention exposure independent of cost.
7. **The 90-day retention rule is configured by hand in the Cloudflare dashboard**, not in
   code. If it is ever removed, storage cost grows linearly and silently.
8. **Fixed infrastructure is not attributed at all.** These are marginal costs. Two Redis
   instances, the engine, the backend, two workers and Mongo are paid for regardless, and at
   low volume they dominate the total bill. Marginal margin of 90% does not mean the business
   is profitable at ten customers.
