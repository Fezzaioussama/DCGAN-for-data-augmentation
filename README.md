# DCGAN for Data Augmentation

> A Deep Convolutional GAN in PyTorch — generator and discriminator trained adversarially to synthesise new training images.

When a dataset is too small, one option is to generate more of it. A DCGAN
learns the distribution of the real images well enough to sample new ones that
sit inside it — useful as augmentation for a downstream classifier, where plain
transforms (flip, rotate, crop) only ever recombine what you already have.

## Run it

Open [`GAN.ipynb`](GAN.ipynb) in Colab — it mounts Google Drive for the dataset
and checkpoints.

```bash
pip install torch torchvision matplotlib numpy
```

To run locally, replace the `google.colab.drive` mount with a local path.

## Configuration

| Parameter | Value |
|---|---|
| `batch_size` | 128 |
| `num_epochs` | 2000 |
| `lr` | 0.0002 |
| Optimizer | Adam |

The training loop records a frame per interval and assembles them into an
animation, so you can watch sample quality develop over training.

## How a GAN works

Two networks train against each other:

- **Generator** maps a random noise vector to an image, via transposed
  convolutions that progressively upsample.
- **Discriminator** is a convolutional classifier that judges whether an image
  is real or generated.

They optimise opposing objectives. The discriminator improves at spotting fakes;
the generator improves at fooling it. Neither can settle, and the generator's
output gets more realistic as a side effect of losing less often.

## What makes it *deep convolutional*

DCGAN is the set of architectural constraints that made GAN training stable
enough to be practical, from Radford et al., *[Unsupervised Representation
Learning with Deep Convolutional Generative Adversarial
Networks](https://arxiv.org/abs/1511.06434)* (2015):

- **No fully-connected layers.** Strided convolutions in the discriminator,
  transposed convolutions in the generator — the network learns its own
  up/downsampling instead of using pooling.
- **Batch normalization** in both networks, which keeps activations from
  drifting and stops the generator collapsing all its outputs to one point.
- **ReLU in the generator** (Tanh at the output), **LeakyReLU in the
  discriminator**, so gradients keep flowing when the discriminator is winning.
- **Adam with lr 2e-4 and β₁ = 0.5** — the momentum default of 0.9 destabilises
  adversarial training.

These aren't arbitrary. Each one targets a specific failure mode that plain GANs
hit, mode collapse most of all.

## Failure modes to watch

- **Mode collapse** — the generator finds one output that reliably fools the
  discriminator and produces only that. Visible as a batch of near-identical
  samples.
- **Discriminator dominance** — if it gets too good too fast, generator
  gradients vanish and learning stalls.
- **Oscillation** — losses that swing without converging. Unlike ordinary
  training, a *falling* generator loss isn't necessarily good news; the sample
  grid is the real metric.
