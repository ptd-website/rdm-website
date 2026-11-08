# Reviewer mtSD

## mtSD-1

*Post as: Reply to Reviewer mtSD's review. 3,754 / 5,000 characters.*

**Title:** Response to Reviewer mtSD (1/2): panel sensitivity, preference models, FID

Thank you for the positive review. We ran both checks you asked for, and both support the headline results. This part covers ImageNet (W1, Q1, Q2); part 2/2 covers the FLUX reference (W2).

**W1/Q1. Is the SW₍r14₎ ranking sensitive to the encoder panel or normalisation?** **No.** SW₍r14₎ averages per-encoder ratios, so every sub-panel and aggregation follows exactly from Table 1. iRDM stays first in:
* **all 14 leave-one-encoder-out panels and all 9 leave-one-family-out panels**;
* all 24 panels with one encoder per family;
* 96% of the 1,001 four-encoder panels and every panel with 8 or more encoders;
* the geometric mean, which no per-encoder normalisation can change, and the median, max and z-score aggregations.

Single encoders disagree more (iRDM is first on 9 of 14), which is why the metric averages a panel. The one aggregation that moves iRDM to 3rd, a per-encoder rank average, rewards scores below the real level, where FD-SIM sits on three encoders. On five new encoders from families absent from training, iRDM is closer to real data than FD-SIM on I-JEPA and SAM, and FD-SIM is closer on SwAV, Barlow Twins and MoCo-v3, which track the Inception statistics FD-SIM matches directly (see Q2). We will add the full panel study to the paper.

**The ranking also holds outside any encoder panel.** You noted that PickScore prefers iRDM. Two other learned human-preference models, HPSv2.1 and ImageReward, agree, HPSv2.1 even more strongly; none of the three is used in our ImageNet training. Share of 10,000 matched-noise pairs (same noise and class) in which iRDM's image scores higher:

| iRDM against | its FID | PickScore | HPSv2.1 | ImageReward |
|---|---|---|---|---|
| pMF-H FD-SIM (iRDM's initialization) | 0.75 | 71.2% | 71.7% | 56.9% |
| pMF-H FD-Inception | 0.71 | 79.9% | 85.2% | 64.2% |
| pMF-H base | 2.28 | 79.7% | 81.7% | 64.3% |

Averaged within each class, **HPSv2.1 prefers iRDM to FD-SIM in 92% of the 1,000 classes** (PickScore 88%, ImageReward 68%).

**Q2. FID.** **We will add FID for every generator to the paper; our measurements show it is a poor optimisation target here.** iRDM's FID is 2.38, 17th of the 18 generators in Table 1, while **on FD_DINOv2 it is 2nd of 15, ahead of every multi-step model**. Three reasons:
1. **FID can be pushed below real data with no gain in quality.** FD-Inception, trained on the Inception distance alone, has the lowest FID we measured (0.71), below 50k real training images (0.86), yet PickScore, HPSv2.1 and ImageReward prefer it to its own base in only 50.8%, 44.2% and 50.5% of pairs.
2. **FID disagrees with all three preference models.** In the table above, the two models with the best FID lose to iRDM, which has the worst FID of the four, under every preference model.
3. **Our controller switches off FID's feature space by itself.** Inception-v3, the network FID uses, is one of our ten training encoders, and the controller drops an encoder from the loss once it reaches its real-data level. Starting from FD-SIM, Inception-v3 and ConvNeXt are already below that level, so in all three seeds of our re-run of the controller ablation (Table 3) both weights go to zero and PE-Core, the encoder farthest from real, gets the largest weight. PickScore, HPSv2.1 and ImageReward prefer these gated runs to uniform weighting in every seed (62.5%, 61.7% and 56.5% of 30,000 pairs), although uniform weighting pushes Inception-v3 further below the real level (SW ratio 0.56, from 0.68 at the start).

On the 14 released generators with published FD_DINOv2, the variant Stein et al. (2023) found closest to human judgment, SW₍r14₎ ranks like it with Spearman 0.98, FID with 0.84. The revised paper will report FID next to SW₍r14₎ for every generator, with these analyses.

---

## mtSD-2

*Post as: Reply to Reviewer mtSD's review (after mtSD-1). 2,806 / 5,000 characters.*

**Title:** Response to Reviewer mtSD (2/2): unfiltered-reference ablation, unseen prompts, PickScore dependence

This part answers W2 on the FLUX reference.

**W2. How much do GenEval filtering and PickScore ranking of the FLUX reference contribute?** We re-trained the released recipe with references that remove them one at a time. GenEval averaged over steps 140 to 180; two values are two training seeds; "unseen" is the set of 960 compositional prompts described below:

| reference | GenEval | unseen |
|---|---|---|
| four-step model | 0.794 | 0.764 |
| 3 random of 24 renders per COCO caption: no PickScore, no detector, no GenEval prompt | 0.759 | 0.715 |
| PickScore top-3 of the same renders | 0.778 / 0.785 | 0.737 / 0.739 |
| plus detector-verified renders for GenEval prompts (released recipe) | 0.794 / 0.810 | 0.745 / 0.771 |
| released checkpoint (step 180) | 0.830 | 0.780 |

**PickScore selection adds about 2 points and the verified GenEval renders about 2 more.** One of our two re-runs reproduces the released checkpoint (0.818 vs 0.830); the revised paper will report both runs.

**The gain reaches compositions the benchmark never contains.** We built 960 GenEval-format prompts whose object pairs, counts and colours never occur among the 553 official prompts, scored by the same GenEval detector: **the released one-step model beats the four-step one, 0.780 vs 0.764.** A second scorer, independent of the detector, agrees. For each image we ask Qwen2.5-VL-7B, an open VLM used nowhere in our pipeline, `Does this figure show "{prompt}"? Please answer yes or no.` and take the probability of "yes" (VQAScore, Lin et al., 2024). On our images this probability predicts the detector's pass or fail verdict with AUC 0.80 to 0.82, and **it rates the one-step model 2.2 points (of 100) above the four-step model on the unseen prompts and 1.7 on the official ones**.

**Does the use of PickScore in curation inflate the PickScore result?** **No: HPSv2.1 and ImageReward, which our pipeline never uses, measure the same gains.** On 500 COCO-val captions, 4 images each:

| | PickScore | HPSv2.1 | ImageReward |
|---|---|---|---|
| images where the one-step model beats the four-step one (same prompt and seed) | 63.4% | 59.8% | 62.1% |
| score gain from PickScore top-3 over random renders (re-trained reference) | +0.47 | +1.01 | +0.10 |
| largest score difference between two training seeds | 0.03 | 0.11 | 0.02 |

PickScore selection lifts the two scorers our pipeline never uses by about 10 and 5 times the seed-to-seed difference, so its benefit is not PickScore agreeing with itself. On the 960 unseen prompts all three still favour the one-step model (50.7% to 52.8% of images). On out-of-domain Pick-a-Pic prompts PickScore and ImageReward are close to even and HPSv2.1 favours the four-step model (47.0% of images), so the revised paper will state the PickScore gain for COCO-domain prompts.

---
