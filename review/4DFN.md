# Reviewer 4DFN

## 4DFN

*Post as: Reply to Reviewer 4DFN's review. 4,047 / 5,000 characters.*

**Title:** Response to Reviewer 4DFN: which choices generalise, from-scratch batch size, FID

Thank you for the encouraging review. Both points led to new results.

**W1. Which choices hold generally, and is a batch above 2048 still optimal from scratch?** **From scratch it is not**: the optimum above 2048 is a post-training result, and we will say so in the abstract and §3.2. A new appendix will state the scope of each choice:

| choice | scope |
|---|---|
| kernel MMD on frozen features, median bandwidth | general: toy from scratch, ImageNet, FLUX |
| exact repulsion with frozen Nyström attraction | general: from scratch at every batch size, and post-training |
| joint image-text kernel | general for conditional generation |
| encoder battery with the gated controller | general principle; the battery and budget are setting-specific |
| batch size, learning rate, bandwidth scale, steps | setting-specific |

The from-scratch test trains the MLP generator of Fig. 3 from scratch on the spiral at a fixed budget of 1.02M generated samples (3 seeds). Median distance to the curve, where real data scores 0.033:

| batch size | 16 | 64 | 128 | 512 | 1024 | 2048 |
|---|---|---|---|---|---|---|
| frozen Nyström reference (iRDM) | .025 | .026 | .027 | .033 | .035 | .043 |
| exact per-batch reference | .047 | .034 | .029 | .038 | .040 | .044 |

**iRDM's estimator stays within 0.008 of the real level at every batch size from 16 to 1024**, while the exact estimator needs moderate batches; both degrade at 2048, where the budget leaves 500 updates. Large batches pay off in post-training (App. D), where a pretrained generator is fine-tuned and each distribution is estimated in a 1k to 3k-dimensional encoder space.

**W2. FID.** We agree it belongs in the paper. The revised paper will report FID-50k (ADM protocol) next to SW₍r14₎ for all 18 generators of Table 1: published values for 15 and our re-measured values for the pMF-H family, which reproduce the published ones. iRDM ranks 17th of 18 on FID and **2nd of 15 on FD_DINOv2, ahead of every multi-step model** (RAE 29.9, REPA-E 34.6).

Read with other metrics, as you suggest, **FID turns out to be a poor optimisation target**. The pMF-H family, iRDM's starting point, shows why. FID and FD_DINOv2 are each model's own (50k real training images: 0.86 and 8.2); the last three columns give the share of 10,000 matched-noise pairs in which a learned preference model scores iRDM's image higher:

| iRDM (FID 2.38, FD_DINOv2 28.0) against | FID | FD_DINOv2 | PickScore | HPSv2.1 | ImageReward |
|---|---|---|---|---|---|
| pMF-H FD-SIM (iRDM's initialization) | 0.75 | 24.6 | 71.2% | 71.7% | 56.9% |
| pMF-H FD-Inception (trained on the Inception distance only) | 0.71 | 50.1 | 79.9% | 85.2% | 64.2% |
| pMF-H base | 2.28 | 76.7 | 79.7% | 81.7% | 64.3% |

* **FID can be pushed below real data with no gain in quality.** FD-SIM and FD-Inception score below real training images, the level an exact sampler would reach. FD-Inception has the lowest FID we measured, yet PickScore, HPSv2.1 and ImageReward prefer it to its own base in only 50.8%, 44.2% and 50.5% of pairs.
* **FID disagrees with all three preference models.** iRDM has the worst FID of the four, and every preference model prefers it to each of the other three, HPSv2.1 in 72% to 85% of pairs.
* **Our controller switches off FID's feature space by itself.** Inception-v3, the network FID uses, is one of our ten training encoders, and the controller drops an encoder from the loss once it reaches its real-data level. Starting from FD-SIM, Inception-v3 and ConvNeXt are already below that level, so in all three seeds of our re-run of the controller ablation (Table 3) both weights go to zero and PE-Core, the encoder farthest from real, gets the largest weight. PickScore, HPSv2.1 and ImageReward prefer these runs to uniform weighting in every seed, although uniform weighting pushes Inception-v3 further below the real level.
* On the 14 released generators with published FD_DINOv2, the variant Stein et al. (2023) found closest to human judgment, SW₍r14₎ ranks like it with Spearman 0.98, FID with 0.84.

---
