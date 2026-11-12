# Reviewer mBiS

## mBiS-1

*Post as: Reply to Reviewer mBiS's review. 4,176 / 5,000 characters.*

**Title:** Response to Reviewer mBiS (1/3): redundancy, family-level holdouts, blinded comparisons, tuning, real floor

Thank you for a careful review with concrete tests. We ran each of them; this part covers W1, Q1 and Q4.

**Q1a. How redundant are the held-out encoders with the training encoders?** **Three of the four are close to a training encoder**, which is why we added encoders from new families (Q1b). Maximum debiased CKA to any training encoder, on 10k real images: SigLIP-v1 0.85 and RADIO 0.83 (both to PE-Core), FLUX-VAE 0.51, DINOv2 0.39. Generator rankings agree accordingly (Spearman 0.97 DINOv2 vs WebSSL, 0.95 SigLIP-v1 vs SigLIP2, 0.99 RADIO vs AIMv2).

**Q1b. Family-level holdouts.** Five encoders from pre-training families absent from training (maximum CKA to a training encoder 0.22 to 0.62), with banks recomputed independently. SW ratio to real data (real = 1):

| encoder (family) | iRDM | FD-SIM (iRDM's init) | pMF-H base |
|---|---|---|---|
| I-JEPA (predictive SSL) | **1.52** | 1.81 | 2.30 |
| SAM (segmentation) | **2.20** | 2.55 | 3.00 |
| SwAV (clustering SSL) | 1.44 | **0.92** | 2.78 |
| Barlow Twins | 1.36 | **0.84** | 1.79 |
| MoCo-v3 (contrastive) | 1.31 | **0.90** | 1.93 |

**iRDM improves on its initialization on the two most disjoint encoders** (I-JEPA and SAM, maximum CKA 0.45 and 0.22). On SwAV, Barlow Twins and MoCo-v3 FD-SIM is closer, even below real data: these encoders track Inception statistics, which FD-SIM matches directly (FID 0.75 against iRDM's 2.38). Matching them buys no perceptual quality: FD-Inception, trained on the Inception distance alone, moves Barlow and MoCo by −32% and −36%, yet the three preference models rate it at chance or below against its own base. A controlled ablation (3 seeds per arm) **traces the split to our gated controller**, which stops pushing an encoder once it reaches the real level: uniform weighting pushes the Inception-like encoders further below real, while gating cuts the worst encoder, PE-Core, from 5.06 to 3.38 (uniform 4.75) and is preferred by PickScore, HPSv2.1 and ImageReward in every seed (62.5%, 61.7% and 56.5% of 30,000 pairs).

**Q1c. Blinded comparisons.** **Two tests outside our pipeline favour iRDM over FD-SIM.** First, three learned human-preference models, none used in our ImageNet training, on 10,000 matched-noise pairs (same noise and class):

| iRDM scores higher than FD-SIM in | PickScore | HPSv2.1 | ImageReward |
|---|---|---|---|
| share of the 10,000 pairs | 71.2% | 71.7% | 56.9% |
| share of the 1,000 classes (class averages) | 87.8% | 91.7% | 67.9% |

Second, a blinded two-alternative forced choice by GPT-6-Astra, a fixed frontier VLM, given the instructions written for human raters: on 200 matched pairs, each shown in both left/right orders, **it prefers iRDM in 67.2% of 400 judgments, and where both orders agree iRDM wins 116 pairs to 47**. A fixed VLM lets anyone rerun the comparison on any pair of generators, and pairwise comparison is where such models agree best with people (GPT-4V matches human decisions on 77% of non-tie pairs; Chen et al., ICML 2024). [All judgments and examples](https://ptd-website.github.io/rdm-website/#astra).

**Q1d. Were held-out metrics used for tuning or checkpoint selection?** **No.** The bandwidth (0.7× median) and the final step were selected with SW on the ten training encoders only. The four held-out encoders were never consulted, and the five new encoders were added for this response.

**Q4. "Real floor" and uncertainty.** Eq. (5) sets independent real data to 1. A sample closer to the training bank than independent real data can score below 1, as FD-SIM does on the three SSL encoders above, so the revised paper will call it the "real-data reference level". A joint bootstrap over generated samples, real banks and projections moves SW₍r14₎ by 0.03 for iRDM and 0.05 for FD-SIM (one standard deviation), small against the 0.81 gap between them, **which iRDM wins in every replicate**. Across 3 training seeds, the standard deviation of SW₍r14₎ is 0.04 or less.

**Revision.** In the abstract and §3.3 we will describe SW₍r14₎ as a different discrepancy, checked on held-out and family-disjoint encoders, three preference models and GPT-6-Astra, and §4.1 will report where the encoders split.

---

## mBiS-2

*Post as: Reply to Reviewer mBiS's review (after mBiS-1). 3,909 / 5,000 characters.*

**Title:** Response to Reviewer mBiS (2/3): unseen prompts, separate curation ablations, PickScore dependence

**Q2. How much of the GenEval gain remains without the evaluation prompts?** We tested this two ways: on new prompts, and by re-training without them.

*Unseen prompts.* We built 960 GenEval-format prompts whose compositional units (object pairs, object counts, object colours) never occur among the 553 official prompts, scored by the same detector. **The released one-step model beats the four-step model on them, 0.780 vs 0.764** (on the official prompts, 0.830 vs 0.794). A second scorer, independent of the detector, confirms the gain. For each image we ask Qwen2.5-VL-7B, an open VLM used nowhere in our pipeline, `Does this figure show "{prompt}"? Please answer yes or no.` and take the probability of "yes" (VQAScore, Lin et al., 2024). On our images this probability predicts the detector's pass or fail verdict with AUC 0.80 to 0.82. **It rates the one-step model 2.2 points (of 100) above the four-step model on the unseen prompts and 1.7 on the official ones.** [Examples](https://ptd-website.github.io/rdm-website/#unseen).

*Re-trained ablations.* We re-trained the released 180-step recipe, changing only the reference. GenEval averaged over steps 140 to 180; two values are two training seeds:

| reference | GenEval | unseen |
|---|---|---|
| four-step model | 0.794 | 0.764 |
| COCO captions only, PickScore top-3: no GenEval prompt anywhere | 0.778 / 0.785 | 0.737 / 0.739 |
| plus detector-verified renders for GenEval prompts (released recipe) | 0.794 / 0.810 | 0.745 / 0.771 |
| released recipe without detector verification | 0.774 | 0.734 |
| COCO only, 3 random of 24 renders: no PickScore | 0.759 | 0.715 |

* With no GenEval prompt in the reference or the conditioning pool, the one-step model **ties the four-step model on the official prompts** (−1.2) and trails it by 2.6 on the unseen ones; Qwen2.5-VL rates it on par (+0.6).
* Separately: **detector filtering is worth 2.8 points on the official prompts and 2.4 on the unseen ones, and PickScore selection 2.3 on both**.
* The verified GenEval renders add 2.1 official and 2.0 unseen points under the detector. Qwen2.5-VL sees +0.7 on unseen prompts and no gain on the official ones, whose verified renders are in the reference; that part of the detector's credit is detector-specific, as you suspected, and the revised paper will say so.

*Reproduction and checkpoint choice.* **One re-run reproduces the released checkpoint at step 180** (0.818 vs 0.830 official, 0.780 vs 0.780 unseen); the other lands about 3 points lower. The abstract will report the mean of the two runs (0.802) next to the released 0.826. **The released checkpoint was chosen by the training MMD**, which keeps decreasing through step 180 in both runs; GenEval peaks at step 20 or 40 instead, so selecting by GenEval would have picked an early checkpoint ([plot](https://ptd-website.github.io/rdm-website/#selection)).

**W2. Does the PickScore result depend on the scorer used in curation?** **No: HPSv2.1 and ImageReward, which our pipeline never uses, measure the same gains.** On 500 COCO-val captions, 4 images each:

| | PickScore | HPSv2.1 | ImageReward |
|---|---|---|---|
| images where the one-step model beats the four-step one (same prompt and seed) | 63.4% | 59.8% | 62.1% |
| score gain from PickScore top-3 over random renders (re-trained reference) | +0.47 | +1.01 | +0.10 |
| largest score difference between two training seeds | 0.03 | 0.11 | 0.02 |

PickScore selection lifts the two scorers our pipeline never uses by about 10 and 5 times the seed-to-seed difference, so its benefit is not PickScore agreeing with itself. On the 960 unseen prompts all three still favour the one-step model (50.7% to 52.8% of images). On out-of-domain Pick-a-Pic prompts PickScore and ImageReward are close to even and HPSv2.1 favours the four-step model (47.0% of images), so the revised paper will state the PickScore gain for COCO-domain prompts.

---

## mBiS-3

*Post as: Reply to Reviewer mBiS's review (after mBiS-2). 3,240 / 5,000 characters.*

**Title:** Response to Reviewer mBiS (3/3): full cost, DMD2 under matched prompts and compute, claims

**Q3a. Does the 90 GPU-hour budget include the reference?** **No, it covers training only.** Building the reference is a one-time cost of about 96 H100 GPU-hours, which every training run reuses:

| stage | H100 GPU-h |
|---|---|
| teacher renders: 82,783 COCO captions × 24 candidates, PickScore ranking | 72.5 |
| teacher renders for the 553 GenEval prompts | 5.7 |
| GenEval detector filtering | 12.5 |
| encoder features and Nyström reference | 2.6 |
| text features | 2.6 |

Most of it is the 24-fold oversampling for PickScore top-3; a reference of random renders costs about 14. The revised paper will include this table.

**Q3b. How does iRDM compare with DMD2 under matched prompts and compute?** We ported the official DMD2 code to FLUX.2 [klein] with the recipe of App. H.2 and trained it for 5,000 iterations on our exact prompt pool (COCO captions plus GenEval prompts) and, separately, on COCO captions only. H100 GPU-h; RDM's 90 H200 GPU-h correspond to about 179 H100 GPU-h ([curves](https://ptd-website.github.io/rdm-website/#dmd2)):

| | GPU-h | GenEval | unseen | PickScore vs four-step |
|---|---|---|---|---|
| four-step model | | 0.794 | 0.764 | 0 |
| DMD2, our prompts, best (iteration 250) | 7 | 0.774 | 0.742 | −0.54 |
| DMD2, our prompts, iteration 5,000 | 35 | 0.298 | 0.239 | −2.75 |
| RDM, step 20, two seeds | 20 + reference | 0.811 / 0.819 | 0.786 / 0.786 | −0.33 / −0.32 |
| RDM, step 180, two seeds | 179 + reference | 0.797 / 0.818 | 0.749 / 0.780 | +0.10 / +0.13 |

* On the same prompts, **RDM at step 180 leads DMD2's best checkpoint by 2.3, 4.4 and 5.7 GenEval points** (two seeds and the release), and **no DMD2 checkpoint reaches the four-step model**. The GenEval prompts do not help DMD2: with COCO captions only it peaks at the same level (0.776).
* DMD2 reaches its peak far more cheaply, but **more compute lowers it at every later checkpoint**, as the student drifts toward crowded scenes that ignore the prompt ([samples](https://ptd-website.github.io/rdm-website/#samples)). RDM holds between 0.79 and 0.82 over all 180 steps of both runs.
* On COCO captions only, DMD2's peak ties PickScore-curated RDM on GenEval and trails it on PickScore (−0.21); it beats RDM with random renders on PickScore and unseen prompts. Curation is part of RDM's cost and of its result.

We will add both DMD2 runs with their full curves to App. H.2.

**W4. Claims.** (i) *Controller.* §1 will describe the controller by what the ablation measures: it reallocates weight to the worst-matched encoder, cutting that ratio from 5.06 to 3.38 (uniform weighting 4.75), and PickScore, HPSv2.1 and ImageReward prefer it in every seed. (ii) *Scope.* All large-scale results are post-training results; the abstract and §3.2 will say so, and a new appendix will give the from-scratch evidence, a toy-scale batch sweep. (iii) *Vision-Aided GAN.* Kumari et al. already supervise with frozen pretrained feature ensembles, through trainable adversarial heads and progressive model selection. RDM matches kernel statistics against a frozen full-data reference, with no adversary or trainable head, and weights encoders by a constrained controller. We will rewrite §2 and §3.3 accordingly and add Stein et al. and Morozov et al.

---
