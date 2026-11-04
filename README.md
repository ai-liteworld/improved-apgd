# Improved Untargeted APGD

An improved implementation of **Auto-PGD (APGD)** for adversarial robustness evaluation, based on [AutoAttack](https://github.com/fra31/auto-attack).

This repository provides the implementation of the methods proposed in:

**Breaking the Trade-off between Performance and Cost with an Improved Untargeted APGD**

Ammar Al-Najjar and Márk Jelasity  
European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning (ESANN), 2026.

[Paper DOI: 10.14428/esann/2026.ES2026-164](https://doi.org/10.14428/esann/2026.ES2026-164)

## Overview

Auto-PGD (APGD) is an adaptive gradient-based attack used in adversarial robustness evaluation. Its standard implementation uses different objective functions, including Cross-Entropy (CE) and Difference of Logits Ratio (DLR).

This work introduces hybrid APGD variants that combine objective functions in different optimization phases to improve attack performance without introducing the computational overhead of a separate, full targeted attack.

The proposed approach preserves the underlying APGD optimization procedure, including its adaptive step-size mechanism and projection operations, while modifying the objective-function strategy and initialization.

Three variants are investigated:

- **Variant A:** Uses CE during the initialization phase, followed by the untargeted DLR objective during the main optimization phase.
- **Variant A+:** Extends Variant A with preliminary optimization steps using the negated targeted DLR objective. The target is selected as the 10th most confident class (\(r=10\)). The optimization switches to untargeted DLR once the target stabilizes.
- **Variant B:** Uses DLR during initialization, followed by CE during the main optimization phase.

The experiments show that Variant A consistently improves upon standard APGD-DLR, while Variant A+ provides additional, more modest improvements. The proposed approach achieves competitive attack performance at a computational cost comparable to untargeted APGD.

## Installation

Install the package directly from GitHub:

```bash
pip install git+https://github.com/ai-liteworld/improved-apgd.git
```

Alternatively, clone the repository:

```bash
git clone https://github.com/ai-liteworld/improved-apgd.git
cd improved-apgd
pip install -e .
```

## Configuration

**Important:** The proposed configuration must currently be enabled manually in:

`autoattack/autopgd_base.py`

Set the following flags:

```python
self.standard = False
self.stability = True
self.targeted = False
```

These settings activate the proposed untargeted hybrid APGD configuration (Variants A/A+).

The flags are currently configured directly in the source code rather than exposed as arguments in the `AutoAttack` constructor.

## Usage

### Running the improved APGD variants

The proposed method replaces the standard APGD variants in the AutoAttack evaluation pipeline.

To benefit from the computational cost reduction, run the APGD attacks directly using AutoAttack's custom version. Avoid using `version='standard'` for this purpose, as it also executes FAB-T and Square Attack.

Example:

```python
import torch
from autoattack import AutoAttack

model.eval()

adversary = AutoAttack(
    model,
    norm='Linf',
    eps=8/255,
    version='custom'
)

# Run only the APGD variants
adversary.attacks_to_run = ['apgd-ce', 'apgd-t']

x_adv = adversary.run_standard_evaluation(
    x_test,
    y_test,
    bs=250
)
```

Make sure the configuration flags described above are set before running the evaluation.

### Important distinction

The custom configuration runs only the selected APGD attacks. It does not run FAB or Square Attack.

This allows the proposed APGD improvements to be evaluated without the additional computational cost of the other attacks included in the full AutoAttack evaluation.

## Experimental Evaluation

The proposed methods were evaluated on:

- CIFAR-10
- CIFAR-100
- SVHN
- TinyImageNet

Experiments covered both \(L_\infty\) and \(L_2\) threat models and included robust WideResNet architectures.

The evaluation used 100 gradient steps and one restart, with additional comparisons against the five-restart setting used in RobustBench.

For the detailed experimental setup, results, and analysis, please refer to the published paper.

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

This implementation is based on [AutoAttack](https://github.com/fra31/auto-attack), developed by Francesco Croce and Matthias Hein.

We acknowledge the original authors for making their implementation publicly available. The original repository's license is preserved in this project.

For details about the original AutoAttack implementation, please refer to its [GitHub repository](https://github.com/fra31/auto-attack).
