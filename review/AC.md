# Summary for the AC

## AC

*Post as: Official Comment on the forum, to all reviewers and the AC (post first). 3,425 / 5,000 characters.*

**Title:** Summary for the AC: new evidence on evaluation, FID, scope and text-to-image

We thank the reviewers. The reviews ask for evaluation evidence that does not share our training encoders, for FID, for the scope of the design choices, and for cleaner text-to-image comparisons. Each reply answers its reviewer in full; this note summarises the new results.

**ImageNet evaluation (mtSD-W1, mBiS-W1, Q1, Q4).**
* The SW₍r14₎ ranking is stable: **iRDM is first in all 14 leave-one-encoder-out and all 9 leave-one-family-out panels**, in 96% of four-encoder panels, and under every aggregation except a per-encoder rank average.
* **Tests outside our pipeline favour iRDM over its initialization FD-SIM**: PickScore, HPSv2.1 and ImageReward prefer it in 71.2%, 71.7% and 56.9% of 10,000 matched pairs (HPSv2.1 in 92% of the 1,000 classes), and a blinded two-alternative forced choice by GPT-6-Astra prefers it in 67.2% of 400 judgments.
* Five encoders from families absent from training split: iRDM leads on I-JEPA and SAM, FD-SIM on three SSL encoders that track Inception statistics. A controlled ablation traces the split to the gated controller.
* Held-out metrics were never used for tuning or checkpoint selection.

**FID (4DFN-W2, mtSD-Q2).** iRDM is 17th of 18 on FID and **2nd of 15 on FD_DINOv2, ahead of every multi-step model**. **FID is a poor optimisation target here**: the two FD-trained models score below real training images; the lowest-FID model is preferred to its own base by none of the three preference models; iRDM has a worse FID than all three pMF-H models, yet all three preference models prefer it to each of them; and our controller drops FID's Inception-v3 encoder from training once it reaches the real level. The revised paper will report FID for every Table 1 generator.

**Scope (4DFN-W1, mBiS-W4).** **The batch optimum above 2048 is a post-training result**. A from-scratch sweep shows that iRDM's estimator works at every batch size from 16 to 1024. The revised paper will state the scope of each design choice.

**Text-to-image (mBiS-W2, W3, Q2, Q3; mtSD-W2).**
* On 960 new compositional prompts absent from GenEval, **the released one-step model beats the four-step model (0.780 vs 0.764)**, and a VQAScore from Qwen2.5-VL-7B agrees (+2.2 of 100).
* HPSv2.1 and ImageReward, never used in our pipeline, prefer the one-step model on 59.8% and 62.1% of COCO-val images; PickScore selection raises both by 5 to 10 times the seed-to-seed difference.
* Re-trained ablations measure each curation step with two training seeds; one re-run reproduces the released checkpoint, and the abstract will state the run mean (0.802) next to 0.826.
* DMD2 re-run on our exact prompts peaks at 0.774 GenEval, **below the four-step model and below RDM**, and declines with more compute.
* The full cost is reported: training plus a one-time reference construction of about 96 GPU-hours.

**Planned revisions.** In the revised paper we will describe SW₍r14₎ as a different discrepancy checked on held-out and family-disjoint encoders, three preference models and GPT-6-Astra; report FID and FD_DINOv2 for every generator, with the controller ablation that explains where metrics disagree; scope the batch claim to post-training; rename "real floor" to "real-data reference level"; add the unseen prompts, the curation ablations with seeds, the run mean, the full cost and the matched DMD2 runs to the T2I section; and add Vision-Aided GAN, Stein et al. and Morozov et al. to related work.

---
