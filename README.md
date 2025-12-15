# Interaction Effects Analysis: Gut-Brain Axis Research

**Synergistic Effects in Microbiome-Diet-Cognition Relationships**

**Author:** Farid Hakimi  
**Date:** December 2025  
**Language:** R

---

## ðŸ“‹ Project Overview

This project demonstrates advanced statistical modelling using interaction terms to explore whether the effect of gut microbiome diversity on cognitive function depends on diet quality. Unlike additive models where predictors contribute independently, interaction models reveal synergistic or antagonistic relationships between variablesâ€”critical for understanding biological systems where factors work together.

### Research Question

**Does the effect of gut bacteria diversity on cognitive function depend on diet quality?**

Or stated differently: Do gut diversity and diet quality work synergistically, such that their combined effect differs from the sum of their individual effects?

---

## ðŸ”¬ Conceptual Foundation

### Additive vs. Interaction Effects

**Additive Model (Day 7 approach):**
```
Cognitive_Score = Î²â‚€ + Î²â‚(Gut_Diversity) + Î²â‚‚(Diet_Quality) + Îµ
```
- Each predictor contributes independently
- Gut diversity helps cognition by X points *regardless* of diet quality
- Predictors simply add up

**Interaction Model (Day 8 approach):**
```
Cognitive_Score = Î²â‚€ + Î²â‚(Gut_Diversity) + Î²â‚‚(Diet_Quality) + 
                  Î²â‚ƒ(Diversity Ã— Diet) + Îµ
```
- The benefit of gut diversity *changes* depending on diet quality
- Predictors work together (synergy or antagonism)
- The interaction term captures this interdependence

### Biological Hypothesis

**Why expect an interaction?**

Gut bacteria require dietary substrates (fibre, prebiotics) to produce beneficial metabolites (short-chain fatty acids, neurotransmitter precursors). Therefore:
- **High diversity + poor diet:** Limited benefit (bacteria lack substrate)
- **High diversity + good diet:** Maximum benefit (bacteria + substrate synergy)
- **Low diversity + good diet:** Modest benefit (substrate available but limited processing capacity)

**Prediction:** The effect of diversity depends on diet qualityâ€”this is an *interaction*.

---

## ðŸ“Š Dataset Characteristics

- **Sample Size:** 100 participants
- **Variables:**
  - `Gut_Diversity`: Shannon diversity index equivalent (23.5-89.2)
  - `Diet_Quality`: Composite score 0-100 scale (25.6-92.1)
  - `Cognitive_Score`: Standardised cognitive assessment (35.2-95.8)

---

## ðŸ“ˆ Analysis Pipeline

### Model Comparison Strategy

**Model 1 (Additive - No Interaction):**
- Tests whether diversity and diet each predict cognition
- Assumes independent, additive effects

**Model 2 (With Interaction):**
- Tests whether the diversity effect *depends on* diet quality
- Captures synergistic/antagonistic relationships

**Statistical Comparison:**
- ANOVA F-test for nested models
- AIC/BIC information criteria
- Residual sum of squares reduction

---

## ðŸ“Š Key Findings

### Model Performance Comparison

| Model | RÂ² | Adjusted RÂ² | AIC | BIC | RSS |
|-------|-----|------------|-----|-----|-----|
| **Model 1 (Additive)** | 0.9987 | 0.9987 | 201.3 | 209.1 | 41.281 |
| **Model 2 (Interaction)** | 0.9989 | 0.9988 | **189.9** âœ“ | **200.3** âœ“ | **36.103** âœ“ |

**Key Observation:** Model 2 has lower AIC/BIC (better fit) and reduced RSS (less unexplained variance).

### Statistical Model Comparison (ANOVA F-Test)

```
Analysis of Variance Table

Model 1: Cognitive_Score ~ Gut_Diversity + Diet_Quality
Model 2: Cognitive_Score ~ Gut_Diversity + Diet_Quality + 
         Gut_Diversity:Diet_Quality
  
  Res.Df    RSS  Df  Sum of Sq      F    Pr(>F)    
1     97  41.281                                  
2     96  36.103   1     5.1781  13.769  0.0003465 ***
```

**Interpretation:**
- **F(1,96) = 13.769, p = 0.000347** (highly significant)
- Adding the interaction term significantly improves model fit
- The improvement is not due to chance (p < 0.001)
- **Conclusion:** The interaction effect is real and meaningful

### Model 2 Coefficients

| Term | Coefficient (Î²) | SE | t-statistic | p-value |
|------|----------------|-----|-------------|---------|
| Intercept | 7.864 | 0.696 | 11.29 | <0.001 |
| Gut Diversity | **0.587** | 0.031 | 18.75 | <0.001 |
| Diet Quality | **0.475** | 0.026 | 18.37 | <0.001 |
| **Diversity Ã— Diet** | **-0.0008** | 0.0002 | **-3.71** | **0.000347** |

### Interpreting the Negative Interaction

**Coefficient:** Î²â‚ƒ = -0.0008

**What does this mean?**

The **negative** interaction indicates **diminishing returns**, not antagonism:

**At Low Diet Quality (score = 30):**
- Gut diversity effect = 0.587 + (-0.0008 Ã— 30) = 0.563 per unit
- Increasing diversity by 10 units â†’ +5.63 points cognitive score

**At High Diet Quality (score = 90):**
- Gut diversity effect = 0.587 + (-0.0008 Ã— 90) = 0.515 per unit
- Increasing diversity by 10 units â†’ +5.15 points cognitive score

**Biological Interpretation:**
As diet quality approaches optimal levels, further increases in gut diversity yield diminishing returns due to ceiling effectsâ€”cognitive function has upper limits, and optimal diet already maximises bacterial metabolite production.

---

## ðŸ“ˆ Visual Evidence: Stratified Regression Plot

Created stratified regression showing gut diversity effect across three diet quality groups:

| Diet Group | Diversity Effect (Slope) | Interpretation |
|-----------|------------------------|----------------|
| **Low diet (25-50)** | **Steepest slope** | Diversity matters most when diet is poor |
| Medium diet (50-75) | Moderate slope | Intermediate benefit |
| **High diet (75-95)** | **Flattest slope** | Approaching ceiling; diminishing returns |

**Visual Pattern:** Non-parallel slopes confirm interaction effectâ€”the diversity-cognition relationship differs across diet quality levels.

---

## ðŸ”§ Technical Considerations

### Multicollinearity in Interaction Terms

**Expected Warning:**
```
The condition number is large, 5.16e+04. This might indicate 
strong multicollinearity or other numerical problems.
```

**Why This Occurs:**
Interaction terms are **mathematically constructed** from parent variables:
- `Gut_Diversity Ã— Diet_Quality` is inherently correlated with both predictors
- High diversity Ã— high diet = very large product values
- This creates artificial correlation

**Solution: Variable Centering**

```r
# Center variables (subtract mean)
data$Diversity_centered <- data$Gut_Diversity - mean(data$Gut_Diversity)
data$Diet_centered <- data$Diet_Quality - mean(data$Diet_Quality)

# Refit model with centered variables
model_centered <- lm(Cognitive_Score ~ Diversity_centered + Diet_centered + 
                     Diversity_centered:Diet_centered, data = data)
```

**Results After Centering:**
- âœ… Interaction coefficient unchanged: Î² = -0.0008 vs -0.0007856
- âœ… Interaction p-value unchanged: p = 0.000347
- âœ… RÂ² unchanged: 0.9989
- âœ… Multicollinearity warning eliminated or greatly reduced

**Key Insight:** Centering doesn't change the **meaning** of resultsâ€”it only reorganises where we measure from (mean vs. zero), resolving numerical issues without affecting statistical inference.

---

## ðŸ§  Clinical Implications

### Personalised Intervention Design

**Scenario 1: Individual with Low Diversity + Poor Diet**
- **Priority:** Improve diversity first (probiotics/prebiotics)
- **Rationale:** Diet improvement alone has limited benefit without bacteria to process nutrients
- **Strategy:** Microbial intervention â†’ dietary optimisation

**Scenario 2: Individual with High Diversity + Poor Diet**
- **Priority:** Improve diet quality
- **Rationale:** Existing bacterial diversity can immediately utilise better nutrients
- **Strategy:** Dietary intervention â†’ maximise existing microbial capacity

**Scenario 3: Individual with Low Diversity + Good Diet**
- **Priority:** Still improve diversity
- **Rationale:** Substrate available but limited processing capacity
- **Expected Outcome:** Modest gains until diversity increases

**Key Insight:** Interaction effects inform *sequential* intervention strategiesâ€”not all individuals benefit equally from the same intervention.

---

## ðŸ’» Technologies Used

**R Programming:**
- `lm()` - Linear regression with interaction terms
- `anova()` - Nested model comparison
- `AIC()`, `BIC()` - Information criteria for model selection
- `ggplot2` - Stratified visualisation
- Variable centering and transformation

**Statistical Methods:**
- Interaction term specification (`Variable1:Variable2`)
- ANOVA F-test for nested models
- Model comparison (AIC, BIC, RSS)
- Residual diagnostics
- Effect size interpretation across variable ranges

---

## ðŸ“ Repository Structure

```
gut-brain-interaction-effects/
â”œâ”€â”€ README.md                              # This file
â”œâ”€â”€ gut_brain_interaction_day8.csv        # Dataset (100 participants)
â””â”€â”€ interaction_effects_analysis.R        # R analysis script (centered & uncentered models)
```

---

## ðŸš€ How to Run

### Requirements
- R version 4.0+
- Optional: `ggplot2` for visualisation (install via `install.packages("ggplot2")`)

### Execution

```r
# Load and run the analysis
source("interaction_effects_analysis.R")
```

**Outputs:**
- Model 1 (additive) summary
- Model 2 (interaction) summary
- ANOVA comparison table
- Centered model results
- Stratified regression plot (if ggplot2 available)

---

## ðŸ“Š Decision Framework: When to Include Interactions

### Include Interaction Terms When:

âœ… **Theoretical basis:** Biological/mechanistic reason to expect synergy  
âœ… **Significant F-test:** ANOVA shows interaction improves fit (p < 0.05)  
âœ… **Lower AIC/BIC:** Information criteria favour interaction model  
âœ… **Visual evidence:** Stratified plots show non-parallel slopes  
âœ… **Practical importance:** Interaction coefficient large enough to matter  

### Don't Include Interaction Terms When:

âŒ **No theoretical justification:** "Fishing expedition" for significant effects  
âŒ **Non-significant F-test:** Interaction doesn't improve fit (p > 0.05)  
âŒ **Higher AIC/BIC:** Added complexity not justified by fit improvement  
âŒ **Parallel slopes:** Visual inspection suggests additive effects  
âŒ **Trivial magnitude:** Interaction coefficient too small for practical relevance  

---

## ðŸŽ¯ Skills Demonstrated

This project showcases:

âœ… **Interaction Effects Modelling:** Understanding when and how predictors work synergistically  
âœ… **Nested Model Comparison:** ANOVA F-tests, AIC/BIC criteria  
âœ… **Variable Centering:** Resolving multicollinearity in interaction terms  
âœ… **Effect Size Interpretation:** Translating interaction coefficients to practical scenarios  
âœ… **Visual Analysis:** Stratified regression plots showing non-parallel slopes  
âœ… **Biological Translation:** Converting statistical interactions to mechanistic understanding  
âœ… **Clinical Application:** Personalised intervention design based on baseline characteristics  

---

## ðŸ”— Related Projects

**This analysis builds on:**
- [Multiple Regression Analysis](https://github.com/farid-bioinfo/multiple-regression-asd-anxiety) - Independent predictor effects
- [Statistical Hypothesis Testing](https://github.com/farid-bioinfo/diabetes-bp-analysis) - Foundational inference

**This analysis leads to:**
- Three-way interactions (e.g., diversity Ã— diet Ã— inflammation)
- Mediation analysis (does diet mediate diversity effects?)
- Longitudinal interaction models (how do interactions change over time?)

---

## ðŸ“§ Contact

**GitHub:** [farid-bioinfo](https://github.com/farid-bioinfo)  
**LinkedIn:** [linkedin.com/in/farid-hakimi-32525a45](https://www.linkedin.com/in/farid-hakimi-32525a45)  
**Email:** fh.faridhakimi@gmail.com

---

## ðŸŽ“ Academic Connection

This analysis applies advanced regression techniques from **CEMP MSc Biostatistics & Bioinformatics** (Grade: 9.43/10):
- Module 3: Biostatistics & R II (Grade: 9.59/10)
- Topics: Interaction effects, model comparison, ANOVA, variable transformation

Demonstrates proficiency in complex statistical modelling relevant to mechanistic questions in gut-brain axis research.

---

*Part of comprehensive portfolio demonstrating progression from basic hypothesis testing through advanced multivariable modelling with interaction effects.*

