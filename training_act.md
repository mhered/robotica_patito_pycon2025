# Training ACT

When finished recording we can train ACT policy using this [notebook](./notebooks/training_act.ipynb) in Collab. This is based on the [original ACT training noebook](https://huggingface.co/docs/lerobot/en/notebooks#training-act) with some modifications of my own.

The notebook has cells to install conda, lerobot, set up wandb.ai, the huggingface hub, launch training and upload the pretrained policy checkpoints.

Create an account in [Weight & Biases](http://wandb.ai) - google spam, organization Duckietown 

Note: Before running any cells, select A100 GPU runtime type (requires making a purchase, the cheapest option of 100 units pay as you go  for ~10€ will be plenty. 1-2hrs of training expected).

Note: If you stop before a checkpoint you cannot resume, lose progress, by default  does first checkpoint 20k steps!!

```bash
$ lerobot-train \
  --dataset.repo_id=mhered/recording-test \
  --policy.type=act \
  --output_dir=outputs/train/act_so100_pato_test1 \
  --job_name=act_so100_pato_test \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=mhered/my_act
```

## Check progress in wandb.ai

https://wandb.ai/spam-mhered-duckietown/lerobot/workspace?nw=nwuserspammhered

![](/home/mhered/my_SOARM100/assets/training_kpis_1.png)

![](/home/mhered/my_SOARM100/assets/training_kpis_2.png)

## What are these indicators?

### Losses

- **`train/loss`** – the **total objective** used to update the model each step.
  For ACT+VAE it’s roughly:
  `total ≈ l1_loss + kl_weight * kld_loss` (you had `kl_weight=10.0`).
- **`train/l1_loss`** – **reconstruction error** (L1) between predicted actions (or decoded targets) and the demo ground truth. Lower is better.
- **`train/kld_loss`** – **KL divergence** term from the VAE. Often logged **unscaled**; the scaled piece contributes to `train/loss`. Too low (≈0) can mean the latent isn’t used; too high hurts fit.

### Optimization/throughput

- **`train/grad_norm`** – global gradient L2 norm. You clip at `grad_clip_norm` (e.g., 10). Constant clipping → try a slightly smaller LR or check instability.
- **`train/dataloading_s`** – **seconds per step spent loading/decoding data**. If this approaches `update_s`, the **DataLoader is your bottleneck** → reduce workers, cache locally, smaller images, fewer cams, etc.
- **`update_s`** – **seconds per optimization step** (forward+backward+optimizer). Should drop on A100 vs T4 and with AMP.

### Progress counters

- **`train/epochs`** – effective **passes over the dataset** (`samples_processed / dataset_size`). In your console this is `epch`.
- **`train/episodes`** – number of **trajectory demos** traversed so far (your dataset had ~61). In the console log this shows as `ep`.
- **`train/steps`** – **optimizer steps** taken (what triggers `save_freq`/`eval_freq`).
- **`train/samples`** – cumulative **training examples** processed (think “windows/batches,” not raw frames).

### Other

- **`lr`** – current **learning rate** (constant if no scheduler; otherwise it changes over time).

## What does “good” look like?

- `train/l1_loss` and total `train/loss` trending **down**.
- `train/kld_loss` settling to a small non-zero value (scaled by `kl_weight` inside total).
- `train/grad_norm` not constantly at the clip ceiling.
- `train/dataloading_s` noticeably **smaller** than `update_s`.

Overall this looks **healthy** :

- **Losses drop cleanly.** `train/loss` and `train/l1_loss` trend down across runs; no divergence.
- **KLD behaves.** `train/kld_loss` decays to a small non-zero value. With `kl_weight=10`, the latent is heavily regularized; for a single pick-and-place that’s fine, but it does mean the L1 term dominates.
- **Gradients are stable.** `train/grad_norm` starts high (~14–15), then eases to ~5–8 → fewer/softer clips; good signal the model settled.
- **Throughput is compute-bound.** `train/update_s` ~0.55s/step on the slower run vs ~0.08s/step on the fast one (A100/AMP). `train/dataloading_s` ~0.02–0.04s, well below `update_s`, so the loader isn’t your bottleneck.
- **Progress counters are sane.** `steps`, `samples`, `epochs` rise linearly; by 50k steps you’re ~10 epochs over ~61 episodes.

