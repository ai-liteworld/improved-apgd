# Improved Untargeted APGD

**An improved implementation of Auto-PGD (APGD) featuring hybrid loss optimization for stronger and more efficient untargeted adversarial attacks.**

This repository contains the implementation of the methods proposed in:

**Breaking the Trade-off between Performance and Cost with an Improved Untargeted APGD**

Ammar Al-Najjar and Márk Jelasity  
ESANN 2026 — European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning.

[Read the paper](https://doi.org/10.14428/esann/2026.ES2026-164)

---

## Overview

Auto-PGD (APGD), a core component of [AutoAttack](https://github.com/fra31/auto-attack), uses surrogate loss functions such as Cross-Entropy (CE) and Difference of Logits Ratio (DLR) to generate adversarial examples.

Although untargeted APGD variants are computationally efficient, targeted APGD variants generally achieve stronger attacks at a substantially higher computational cost.

This work introduces hybrid untargeted APGD variants that combine CE and DLR sequentially during different stages of the optimization process.

The proposed methods improve the effectiveness of untargeted APGD while maintaining computational costs comparable to standard untargeted variants.

## Proposed Methods

We introduce three hybrid APGD variants:

### Variant A — CE to DLR

- Uses Cross-Entropy (CE) during the initialization phase.
- Transitions to untargeted DLR during the main optimization phase.

The motivation is to benefit from the strong initial gradients provided by CE while using DLR for the main optimization process.

### Variant A+ — CE to Target Refinement to DLR

Variant A+ extends Variant A with an additional initialization procedure.

- Starts with CE-based initialization.
- Performs preliminary optimization steps using the negated targeted DLR loss.
- Selects the target as the 10th most confident class.
- Monitors target stabilization and switches to standard untargeted DLR once the selected targets stabilize.

This additional procedure improves the initialization of the untargeted optimization.

### Variant B — DLR to CE

- Uses DLR during the initialization phase.
- Transitions to CE during the main optimization phase.

This variant investigates the effect of reversing the loss ordering.

## Configuration

**Important:** The proposed variants are currently selected through manual configuration inside the source code. They are not yet exposed as arguments in the `AutoAttack` constructor.

Open:

[`autoattack/autopgd_base.py`](https://github.com/ai-liteworld/improved-apgd/blob/master/autoattack/autopgd_base.py)

Locate the following flags and configure them as shown:

```python
self.standard = False
self.stability = True
self.targeted = False
```

These settings enable the proposed untargeted hybrid variants (A/A+) rather than the standard or targeted APGD configurations.

To use the proposed methods, make sure these flags are set before running the attack.

## Installation

Install directly from this repository:

```bash
pip install git+https://github.com/ai-liteworld/improved-apgd.git
```

## Usage

The implementation retains the original AutoAttack package structure and PyTorch interface.

```python
from autoattack import AutoAttack

# Your PyTorch model should return logits.
model.eval()

adversary = AutoAttack(
    model,
    norm='Linf',
    eps=8/255,
    version='standard'
)

# Generate adversarial examples.
x_adv = adversary.run_standard_evaluation(
    x_test,
    y_test,
    bs=250
)
```

**Note:** Configure the flags in `autoattack/autopgd_base.py` before running the attack, as described in the Configuration section.

Inputs are expected to be PyTorch tensors in NCHW format, with image values in the range [0, 1].

For additional interface details, see the original [AutoAttack documentation](https://github.com/fra31/auto-attack).

## Experimental Evaluation

The methods were evaluated using pretrained robust WideResNet architectures across multiple datasets:

- CIFAR-10
- CIFAR-100
- SVHN
- TinyImageNet

The experiments considered both \(L_\infty\) and \(L_2\) threat models.

The results demonstrate that:

- Variant A consistently improves upon standard APGD-DLR.
- Variant A+ provides additional improvements through its stabilization procedure.
- Variant B is generally less effective than the CE-to-DLR ordering.
- The proposed variants achieve attack effectiveness comparable to targeted APGD while maintaining computational costs similar to untargeted APGD.

For the complete experimental setup, quantitative results, and analysis, please refer to the [published paper](https://doi.org/10.14428/esann/2026.ES2026-164).

## Citation

If you use this implementation in your research, please cite the following paper:

```bibtex
@inproceedings{alnajjar2026improvedapgd,
    title = {Breaking the Trade-off between Performance and Cost with an Improved Untargeted APGD},
    author = {Al-Najjar, Ammar and Jelasity, M{\'a}rk},
    booktitle = {Proceedings of the European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning (ESANN)},
    year = {2026},
    doi = {10.14428/esann/2026.ES2026-164}
}
```

## Acknowledgements

This implementation builds upon the original [AutoAttack](https://github.com/fra31/auto-attack) repository by Francesco Croce and Matthias Hein.

We acknowledge their original implementation and retain the underlying APGD framework while introducing the modifications described in our work.

Original AutoAttack repository: https://github.com/fra31/auto-attack

## License

Please refer to the included `LICENSE` file for licensing information.
