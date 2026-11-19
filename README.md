# Credit Card Default Risk

**[Explore the interactive project website](https://gestione-del-rischio.github.io/credit-card-default-risk/)**

The website walks through the data, modelling choices, and results, with an interactive tool for exploring how the decision threshold affects missed defaults and false alarms.

This project estimates each client's probability of credit card default in the following month. It compares five models with a baseline that gives every client the same probability.

**Authors:** [Simone Nardi](https://www.linkedin.com/in/simone-nardi-931259439) and [Thomas Petrera](https://www.linkedin.com/in/thomas-petrera-312850392)

## Results at a glance

These results come from a held-out test set of 6,000 clients. Model and threshold selection used the validation set.

| | Baseline | Final model (Random Forest) |
|---|---:|---:|
| Cross-entropy loss (lower is better) | 0.5284 | **0.4321** |
| ROC-AUC | 0.5000 | **0.7738** |
| Accuracy at the maximum-accuracy threshold (0.49) | 0.7788 | **0.8170** |
| Accuracy at the prudent threshold (0.31) | 0.7788 | **0.8003** |
| Share of defaults caught at the prudent threshold | 0.0000 | **0.5343** |
| Missed defaults at the prudent threshold | 1,327 | **618** |

The baseline gives every client the same probability, the default rate in the training set. At both reported thresholds, it predicts "no default" for everyone. It achieves 77.88% accuracy while missing every default, which is why the comparison also reports recall.

<p align="center">
  <img src="figures/roc_curves.png" width="620" alt="Test-set ROC curves for the five models, with the random-ranking diagonal for reference.">
</p>
<p align="center"><em>ROC curves on the test set. Each point shows the false-positive rate on the horizontal axis and the true-positive rate on the vertical axis for a given threshold.</em></p>

## The question

The project has three parts:

1. **Estimate** the probability that each client defaults next month.
2. **Compare** five models with a baseline that uses no client information. This comparison shows what the client features add.
3. **Decide** whether to flag each client based on their estimated probability. A false negative misses a default. A false positive flags a client who does not default. The prudent threshold prioritises catching defaults within limits on accuracy and precision.

## Data

The UCI Machine Learning Repository's "Default of Credit Card Clients" dataset covers 30,000 clients of a bank in Taiwan observed from April to September 2005. The target is 1 if the client defaults in October 2005.

- The 23 input variables include credit limit, sex, education, marital status, age, six months of repayment status, six statement amounts and six payment amounts.
- No missing values and no duplicate client IDs.
- 6,636 clients default (22.12%), so the classes are imbalanced.

## Method

1. **Variables.** The repayment codes become months of delay (`DELAY_k = max(PAY_k, 0)`). Sex, education and marital status become 0/1 indicators, with the undocumented codes grouped as "other".
2. **Split.** A 60/20/20 split stratified on the target, with a fixed seed. There are 18,000 clients for training, 6,000 for validation and 6,000 for testing, each with a default rate of approximately 22.12%. The training set is used to fit the models, the validation set guides model and threshold selection, and the test set is reserved for final evaluation.
3. **Features and scaling.** Among pairs of quantitative variables with a correlation of at least 0.70 in absolute value on the training set, the one from the older month is dropped (`BILL_AMT2` to `BILL_AMT6`, `DELAY_5`). 20 features remain. Payment amounts are log-transformed, and all quantitative features are standardised with the mean and standard deviation of the training set only.
4. **Models.** Logistic regression and a neural network (Keras, with early stopping), a Random Forest (500 trees), Gradient Boosting, and an ensemble that averages the four probabilities. Network size, leaf size and number of trees are chosen on the validation set. The final model is the one with the lowest validation loss.
5. **Threshold.** For each model, we choose the threshold with the highest validation accuracy and a second, *prudent* threshold. The prudent threshold has the fewest missed defaults among thresholds that give up at most 2 percentage points of accuracy relative to the best threshold and keep precision at 0.50 or more.
6. **Final check.** Every choice is fixed before the test set is touched.

## Results

### Choosing the model on the validation set

| Model | Validation loss | Validation accuracy (threshold 0.50) | Validation ROC-AUC |
|---|---:|---:|---:|
| Baseline | 0.5284 | 0.7788 | 0.5000 |
| Logistic regression | 0.4477 | 0.8173 | 0.7669 |
| Neural network | 0.4327 | 0.8202 | 0.7786 |
| **Random Forest (final model)** | **0.4259** | **0.8218** | **0.7861** |
| Gradient Boosting | 0.4291 | 0.8213 | 0.7802 |
| Ensemble | 0.4280 | 0.8212 | 0.7842 |

All five models beat the baseline. The Random Forest has the lowest validation loss and becomes the final model. The validation losses of the non-linear models differ by only a few thousandths.

### Choosing the threshold

<p align="center">
  <img src="figures/threshold_metrics.png" width="760" alt="Accuracy, precision and recall of the Random Forest on the validation set as the threshold moves from 0.05 to 0.95, with the maximum-accuracy threshold 0.49 and the prudent threshold 0.31 marked.">
</p>
<p align="center"><em>Random Forest on the validation set. Accuracy is almost flat over a wide band, so the threshold can be lowered a little to catch many more defaults.</em></p>

For the Random Forest the maximum-accuracy threshold is 0.49 and the prudent threshold is 0.31. Under a simple decision model with calibrated probabilities and fixed error costs, a threshold of 0.31 corresponds to a missed default costing about 2.23 times as much as a false alarm. The prudent rule uses two chosen constraints. The notebook shows how the threshold changes when the accuracy tolerance varies.

### The final check on the test set

| Model | Test loss | Test ROC-AUC | Accuracy, maximum-accuracy threshold | Accuracy, prudent threshold | Recall, prudent threshold | Missed defaults, prudent threshold |
|---|---:|---:|---:|---:|---:|---:|
| Baseline | 0.5284 | 0.5000 | 0.7788 | 0.7788 | 0.0000 | 1,327 |
| Logistic regression | 0.4470 | 0.7545 | 0.8148 | 0.7960 | 0.4996 | 664 |
| Neural network | 0.4339 | 0.7757 | 0.8167 | 0.7988 | 0.5177 | 640 |
| **Random Forest (final model)** | 0.4321 | 0.7738 | 0.8170 | 0.8003 | 0.5343 | 618 |
| Gradient Boosting | 0.4309 | 0.7780 | 0.8182 | 0.8013 | 0.5079 | 653 |
| Ensemble | 0.4314 | 0.7771 | 0.8173 | 0.8013 | 0.5049 | 657 |

<p align="center">
  <img src="figures/confusion_matrices.png" width="760" alt="Confusion matrices of the Random Forest on the test set at the maximum-accuracy threshold 0.49 and at the prudent threshold 0.31.">
</p>
<p align="center"><em>Random Forest on the test set at the two thresholds. Rows are the actual outcome, columns the prediction.</em></p>

- Moving from the maximum-accuracy threshold to the prudent one, missed defaults fall from 851 to 618 and recall rises from 0.3587 to 0.5343. In exchange, false alarms rise from 247 to 580 and accuracy goes from 0.8170 to 0.8003, still above the baseline.
- In this test sample, the observed default rate rises from 0.0643 in the lowest risk band to 0.6657 in the highest. Observed rates are close to the mean predicted probability in each band.
- In the validation-set permutation analysis, shuffling the September payment delay produced the largest increase in loss. Shuffling the demographic indicators had much smaller effects.

## Limits

- **Sample and period.** One bank in Taiwan, one period (2005), a one-month horizon. For another portfolio or another economic cycle the probabilities would have to be re-estimated.
- **Validation.** Model selection depends partly on the 6,000 clients in a single random validation split.
- **Hyperparameters.** The search covers three network architectures, four minimum leaf sizes for the Random Forest and three maximum leaf counts for Gradient Boosting.
- **Error costs.** The actual costs of the two errors are unknown. The prudent threshold follows the accuracy and precision constraints set for this project.
- **Margin over the baseline.** The features explain only part of client behaviour, so the gain over the baseline is moderate, and even at the prudent threshold some defaults go undetected.

## A note on method

All five models are evaluated on the test set for comparison. The Random Forest was selected on the validation set, so its scores are the primary test results. On this test split, Gradient Boosting has slightly lower loss and higher ROC-AUC.

## Repository contents

| File | Content |
|---|---|
| `rischio_di_credito_cinque_modelli_v2.ipynb` | The full analysis with code, outputs and comments in Italian. The walkthrough covers the same work in English. |
| `index.html` | The self-contained English walkthrough. Open it in a browser to view the site offline. |
| `figures/` | The figures shown in this README. |
| `default of credit card clients.xls` | The dataset, as distributed by the UCI Machine Learning Repository under the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) licence (see the data source below). |

## How to run

1. Install Python and the required libraries.

   ```bash
   pip install numpy pandas scikit-learn tensorflow matplotlib seaborn xlrd notebook
   ```

2. Keep the dataset file `default of credit card clients.xls` in the same folder as the notebook, which reads it from there.
3. Open the notebook and run all cells. It takes a few minutes on a laptop CPU.

Random seeds are fixed at 42. The numbers above were obtained with Python 3.13, TensorFlow 2.21 (Keras 3.15), scikit-learn 1.9, pandas 2.3 and NumPy 2.5. The neural-network results can differ slightly with other versions or hardware.

## Data source

Yeh, I-C. (2009). Default of Credit Card Clients [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C55S3H. Licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). The repository includes the original data file. The notebook creates the derived variables.

Yeh, I-C., & Lien, C-H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. *Expert Systems with Applications*, 36(2), 2473–2480.
