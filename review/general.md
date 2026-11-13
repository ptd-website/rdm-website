# General response

## GR

*Post as: Official Comment on the forum, top level (post first). 3,782 / 5,000 characters.*

**Title:** General response: evaluation, FID, scope and text-to-image

We thank the reviewers. Four questions recur across the reviews; short answers first, then the details.

1. **Is iRDM's ImageNet lead an artefact of evaluating with its own training encoders?** **No.** PickScore, HPSv2.1, ImageReward and a blinded GPT-6-Astra comparison, none used in our ImageNet training, prefer iRDM to its initialization, and iRDM stays first on SW₍r14₎ when any encoder or encoder family is left out.
2. **Where is FID?** **The revised paper will report it for every generator. iRDM is 17th of 18 on FID and 2nd of 15 on FD_DINOv2, ahead of every multi-step model.** FID is a poor optimisation target here: it can be pushed below real data with no gain in quality.
3. **Does the batch optimum above 2048 hold outside post-training?** **No. It is a post-training result, and the revised paper will say so.** A from-scratch toy sweep shows the estimator itself works at every batch size from 16 to 1024.
4. **How much of the text-to-image gain comes from curating the reference?** **Each curation step adds 2 to 3 GenEval points; curation is part of RDM's result and of its cost.** The released model's lead holds on 960 new prompts absent from GenEval, and HPSv2.1 and ImageReward, which our pipeline never uses, confirm the PickScore gain on COCO prompts.

**Details**

**1. Evaluation (mtSD-W1, Q1; mBiS-W1, Q1, Q4)**
* iRDM is first in all 14 leave-one-encoder-out and all 9 leave-one-family-out panels.
* PickScore, HPSv2.1 and ImageReward prefer iRDM to its initialization FD-SIM in 71.2%, 71.7% and 56.9% of 10,000 matched pairs; GPT-6-Astra, in a blinded two-alternative forced choice, in 67.2% of 400 judgments.
* On five encoders from families absent from training, iRDM is closer to real data on I-JEPA and SAM, and FD-SIM on three SSL encoders that track Inception statistics. A controlled ablation traces this split to the gated controller.
* Held-out metrics were never used for tuning or checkpoint selection.

**2. FID (4DFN-W2; mtSD-Q2)**
* The two FD-trained models score below real training images, and the lowest-FID model is at chance or below against its own base under all three preference models.
* iRDM has a worse FID than all three pMF-H models, yet all three preference models prefer it to each of them.
* Our controller drops FID's Inception-v3 encoder from training once it reaches the real level.

**3. Scope (4DFN-W1; mBiS-W4)**
* Trained from scratch on the toy task of Fig. 3, iRDM's estimator stays within 0.008 of the real level at every batch size from 16 to 1024.
* A new appendix will state the scope of each design choice.

**4. Text-to-image (mtSD-W2; mBiS-W2, W3, Q2, Q3)**
* Re-trained without each step: PickScore ranking adds 2.3 GenEval points, the detector-verified GenEval renders 2.1 and detector filtering 2.8.
* On the 960 new prompts the released model beats the four-step model, 0.780 vs 0.764; a VQAScore from Qwen2.5-VL-7B agrees (+2.2 of 100).
* HPSv2.1 and ImageReward prefer the one-step model on 59.8% and 62.1% of COCO-val images.
* One of two re-runs reproduces the released checkpoint; the abstract will state the run mean (0.802) next to 0.826.
* DMD2 trained on our exact prompts peaks at 0.774 GenEval, below the four-step model and below RDM.
* Training takes 90 H200 GPU-hours; building the reference is a one-time cost of about 96 H100 GPU-hours.

**Planned revisions.** The revised paper will describe SW₍r14₎ as a different discrepancy checked on held-out and family-disjoint encoders, three preference models and GPT-6-Astra; report FID and FD_DINOv2 for every generator; scope the batch claim to post-training; rename "real floor" to "real-data reference level"; add the new text-to-image results and the full cost; and add Vision-Aided GAN, Stein et al. and Morozov et al. to related work.

---
