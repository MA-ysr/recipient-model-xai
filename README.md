# Modeling the Recipient — Reference Implementation

Executable artifact accompanying the paper *"Modeling the Recipient: A Stability-Based Taxonomy of Human Factors for Adaptive Explanation in Interactive Intelligent Systems."*

The paper specifies a taxonomy of recipient factors, a mapping from those factors to seven explanation properties, a metamodel representing them, and a procedure deriving explanation requirements from factor values. This repository makes that specification executable, so that its claims can be checked rather than taken on trust.

**What it establishes.** That the specification is complete enough to execute; that execution reproduces the explanation requirements derived by hand in the paper's running example; that the well-formedness rules are verifiable automatically; that the derivation behaves as predicted in degenerate and conflicting cases; and how far the outputs depend on the pressure magnitudes the mapping leaves unspecified.

**What it does not establish.** That recipients comprehend matched explanations better than mismatched ones. That is a question about human performance, and it is the subject of the validation studies proposed in the paper. No human participants are involved anywhere in this artifact.

---

## Repository structure

```
model/
  huFaMo.ecore            the metamodel: 17 factor classes across three stability bands, the explanation requirement, and seven OCL well-formedness invariants
  personas.model          set 1 — the paper's running example
  edgecases.model         set 2 — five degenerate or conflicting configurations
  validation.model        set 3 — one well-formed and six ill-formed requirements

src/
  pressures.eol           the factor-to-property mapping encoded as signed pressure functions, plus candidate domains, burden orders and hard-constraint exclusions
  engine.eol              the derivation procedure: hard filtering, signed aggregation, and the three-case tie policy
  derive.eol              runner — derives requirements and prints a full trace
  sensitivity.eol         runner — one-factor-at-a-time weight perturbation
  validate.evl            the OCL invariants, executable

logs/
  derive-personas.txt      output of derive.eol on personas.model
  derive-edgecases.txt     output of derive.eol on edgecases.model
  sensitivity-personas.txt output of sensitivity.eol on personas.model
  validation.txt           output of validate.evl on validation.model
```

---

## The three evaluation sets

Each set conforms to the same metamodel and targets a different question.

### Set 1 — `personas.model`: does execution reproduce the running example?

The three recipients of the paper's running example, all receiving an explanation of the same declined loan decision, instantiated with exactly the factor values the paper states and no others:

| Recipient | Factors |
|---|---|
| `youssef` | domain expertise high, AI/ML literacy high, purpose `debug`, role developer, stakes low |
| `nihal` | domain expertise moderate, AI/ML literacy low, cognitive load high, purpose `justify`, role frontline, documentation duty |
| `moncef` | domain expertise low, AI/ML literacy low, language `ar`, accessibility need excluding `visualPlot` and `tabular`, purpose `contest`, role applicant, stakes high |

Run with `derive.eol` and with `sensitivity.eol`.

### Set 2 — `edgecases.model`: how does the derivation behave at the boundaries?

Five configurations the running example never reaches:

| Instance | Configuration | Expected behaviour |
|---|---|---|
| `ec1-coldstart` | no profile elicited | derives from situational factors alone; coverage reduced, no failure |
| `ec2-tie` | every factor neutral | all properties resolve to their lowest-burden value |
| `ec3-structural` | high load against high stakes on granularity | structural conflict: layered outcome |
| `ec4-unsatisfiable` | two accessibility needs excluding every modality | reports modality unsatisfiable, defers to a human decision, continues with the other properties |
| `ec5-securitycap` | confidentiality cap at the headline level | filters granularity **and** withholds the full attribution vector from content |

Run with `derive.eol`.

### Set 3 — `validation.model`: do the rules reject what they should?

Seven explanation requirements: one well formed, and six each violating exactly one invariant — selecting an excluded modality, foregrounding content that was never included, exceeding a confidentiality cap, combining one recipient's profile with another's session, offering an on-demand layer no fuller than the primary setting, and using a technical register for a recipient with low AI/ML literacy. A request without a stated purpose is included in the same model.

Run with `validate.evl`. The expected result is seven diagnostics, one per rule, and none against the well-formed requirement.

---

## Reproducing the results

### Prerequisites

- **Eclipse Modeling Tools** (any recent release).
- **Epsilon**, including its EMF driver.

### Setup

1. Create a project (a plain *General* project is sufficient) and copy `model/` and `src/` into it.
2. Nothing needs registering. Each `.model` file carries an `xsi:schemaLocation` pointing at `huFaMo.ecore`, and the launch configurations below reference the metamodel as a file.

### Running the derivation

Run → Run Configurations → **EOL Program** → New:

- **Source:** `src/derive.eol`
- **Models** → Add → **EMF Model**:
  - Name: `M`
  - Model file: `model/personas.model`
  - Metamodel: add the **file** `model/huFaMo.ecore` (not a URI)
  - *Read on load* ticked, *Store on disposal* unticked

The console prints, for each recipient and each property: the candidate values removed by hard constraints, the aggregated score of every surviving candidate, and how the value was resolved — clear winner, conservative tie-break, or layered. Compare against `logs/derive-personas.txt`.

Duplicate the configuration with `model/edgecases.model` for set 2, and compare against `logs/derive-edgecases.txt`.

### Running the validation

Run Configurations → **EVL Validation**, source `src/validate.evl`, model `model/validation.model`, same model settings. Compare against `logs/validation.txt`.

### Running the sensitivity analysis

**EOL Program**, source `src/sensitivity.eol`, model `model/personas.model`. Each factor's weight is set in turn to 0, 0.5, 2 and 4 while the others stay at 1, and every derived value that changes is reported with the weight at which it changes. Compare against `logs/sensitivity-personas.txt`.
