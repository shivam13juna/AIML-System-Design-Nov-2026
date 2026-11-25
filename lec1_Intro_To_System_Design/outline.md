```
START — A business problem needs to be solved
  │
  ▼
[P1] FROM A MODEL TO A SYSTEM
  │
  ├── Research ML vs. Production ML
  │     ├── Research: maximise performance on a fixed dataset
  │     └── Production: deliver business value within real constraints
  │
  ├── Understand the two paths of an ML system
  │     ├── Online path: input → model → prediction → decision → action
  │     └── Offline loop: outcomes → labels → training → evaluation
  │                       → model registry → approved deployment
  │
  └── Three areas where system design decisions reside
        ├── Data          — inputs, labels and their availability
        ├── Serving       — latency, cost and fallback mechanisms
        └── Feedback loop — how system decisions shape future training
  │
  ▼
[P2] SHOULD THIS BE ML AT ALL?
  │
  ├── Is the answer an existing, stored fact?
  │     └── YES → Database lookup
  │
  ├── Is the pattern simple, stable and explicitly definable?
  │     └── YES → Rule-based system
  │
  ├── Is the pattern complex, recurring and learnable from data?
  │     └── YES → Train an ML model
  │
  ├── Is the problem common across businesses?
  │     └── YES → Consider buying a pretrained model or service
  │                 ├── Cost per request
  │                 ├── Data privacy
  │                 └── Dependency on external providers
  │
  └── Are incorrect decisions expensive or difficult to reverse?
        └── YES → Human-in-the-loop
  │
  │   Evaluate every approach against:
  │     ├── Complexity of implementation
  │     ├── Availability of training data
  │     ├── Cost and consequences of errors
  │     ├── Reversibility of decisions
  │     └── Expected business value
  │
  ▼
[P3] UNDERSTAND WHAT CAN GO WRONG IN PRODUCTION
  │
  ├── Conventional software failures
  │     └── Crashes · timeouts · exceptions · service unavailability
  │
  ├── Silent failures in ML systems
  │     └── Service remains healthy while prediction quality deteriorates
  │
  ├── Identify the type of change
  │     ├── Data drift    — distribution of inputs changes
  │     ├── New output    — the prediction space itself changes
  │     ├── Concept drift — relationship between input and label changes
  │     └── Feedback loop — model decisions influence future labels
  │
  ├── Can retraining alone resolve the problem?
  │     ├── Depends on the underlying cause of degradation
  │     └── Changes to the prediction space may require new labels,
  │         reformulation and model redesign
  │
  └── Establish two complementary monitoring signals
        ├── Fast signal — does not require ground-truth labels
        │     ├── Prediction confidence
        │     └── Distribution of predicted outputs
        │
        ├── Slow signal — requires ground-truth labels
        │     ├── Accuracy
        │     └── Task-specific performance metrics
        │
        └── Periodic human-reviewed samples bridge the gap
              → Confidence is a clue, not proof of correctness
  │
  ▼
[P4] DEFINE SYSTEM REQUIREMENTS
  │
  ├── Four fundamental requirements
  │     ├── Reliability
  │     │     └── Consistently produce correct and dependable decisions
  │     ├── Scalability
  │     │     └── Handle growth in traffic, model count and model size
  │     ├── Maintainability
  │     │     └── Reproduce, understand and modify the system
  │     └── Adaptability
  │           └── Respond to changing data and business requirements
  │
  ├── Convert abstract requirements into measurable targets
  │     ├── Prediction quality
  │     ├── Latency — percentile-based: p95 rather than mean
  │     ├── Cost per prediction
  │     ├── Service availability
  │     └── Human fallback capacity
  │
  └── Distinguish service availability from prediction reliability
        → A system can be available and still make incorrect decisions
  │
  ▼
[P5] FRAME THE BUSINESS PROBLEM AS AN ML PROBLEM
  │
  ├── Establish the objective hierarchy
  │     │
  │     Business objective
  │          ↓
  │     Decision the system needs to make
  │          ↓
  │     ML objective
  │          ↓
  │     Prediction required from the model
  │
  ├── Establish the moment of prediction
  │     ├── What information exists before this moment?
  │     │     └── Eligible input features
  │     └── What information becomes available afterwards?
  │           ├── Potential ground-truth labels
  │           └── Including it as input causes data leakage
  │
  ├── Identify the source of labels
  │     ├── Naturally generated labels from operational logs
  │     ├── Human-annotated labels
  │     └── Challenges
  │           ├── Label noise
  │           ├── Delayed labels
  │           └── Labels influenced by previous system decisions
  │
  ├── Establish a clean, human-verified evaluation dataset
  │
  ├── Choose the appropriate ML formulation
  │     ├── Predict a category       → Classification
  │     ├── Predict multiple classes → Multi-label classification
  │     ├── Predict a quantity       → Regression
  │     └── Predict relative order   → Ranking
  │
  └── Separate model output from system decision
        ├── Model produces probabilities, estimates or scores
        └── Decision layer converts predictions into actions
              ├── Thresholds
              ├── Capacity constraints
              ├── Business rules
              └── Human intervention
  │
  ▼
[P6] DEFINE METRICS THAT TRACK THE BUSINESS
  │
  ├── Establish the metric hierarchy
  │     │
  │     Offline metrics — held-out evaluation data
  │          ↓
  │     Online metrics  — behaviour on live production traffic
  │          ↓
  │     Business metrics — actual organisational outcomes
  │
  │     Each connection is an assumption that must be validated.
  │     Improving a proxy metric may not improve the business outcome.
  │
  ├── Determine whether all prediction errors are equally costly
  │     ├── YES → Conventional performance metrics may suffice
  │     └── NO  → Cost-sensitive evaluation
  │                 ├── Cost of false positives
  │                 ├── Cost of false negatives
  │                 └── Cost-weighted prediction errors
  │
  ├── Compare evaluation against different label sources
  │     ├── Accuracy against operational labels
  │     └── Accuracy against human-verified ground truth
  │
  ├── Optimise the decision layer
  │     ├── Adjust classification thresholds
  │     ├── Evaluate precision-recall trade-offs
  │     ├── Evaluate total expected error cost
  │     └── Check downstream operational capacity
  │
  └── Establish main metrics and guardrail metrics
        ├── Main metric — what the system should improve
        ├── Guardrail   — what the system must not sacrifice
        └── Validate actual improvements through online A/B testing
  │
  ▼
[P7] HANDLE COMPETING BUSINESS OBJECTIVES
  │
  ├── Identify objectives that conflict with each other
  │     └── Improving one objective may deteriorate another
  │
  ├── APPROACH 1 — Single model with a blended label
  │     ├── Combine objectives into one training target
  │     ├── Train a model to predict the combined target
  │     ├── Business priorities are embedded in model parameters
  │     └── Changing priorities requires:
  │           Relabelling → Retraining → Evaluation → Deployment
  │
  ├── APPROACH 2 — Decoupled objectives
  │     ├── Predict each objective independently
  │     │     ├── Separate models, or
  │     │     └── One model with multiple outputs
  │     ├── Transform predictions into comparable score scales
  │     ├── Combine scores at serving time
  │     │     └── Final score = w × Score₁ + (1 − w) × Score₂
  │     └── Business owns the configurable weight
  │           → Priorities can change without retraining
  │
  ├── Evaluate the trade-offs across different weights
  │     ├── Measure improvement in each objective
  │     ├── Measure deterioration in competing objectives
  │     └── Identify diminishing returns
  │
  └── Operational considerations
        ├── Score calibration
        ├── Additional model and monitoring pipelines
        ├── Stability of score interpretation after retraining
        └── Hard constraints for non-negotiable requirements
  │
  ▼
[P8] BACK-OF-THE-ENVELOPE ESTIMATION
  │
  ├── Estimate traffic
  │     ├── Requests/day = Users × Actions per user per day
  │     ├── Average QPS = Requests/day ÷ 86,400
  │     └── Peak QPS = Average QPS × Peak traffic factor
  │
  ├── Estimate computational workload
  │     └── Peak scores/second = Peak QPS × Scores per request
  │
  ├── Estimate data storage
  │     ├── Storage/day = Requests/day × Bytes logged per request
  │     └── Long-term storage and retention requirements
  │
  ├── Estimate operational cost
  │     ├── Cost per prediction
  │     └── Total daily processing cost
  │
  └── Let the estimates constrain the architecture
        ├── Is computation a significant bottleneck?
        ├── Can a single machine handle the workload?
        ├── Is distributed serving necessary?
        ├── Is batch processing sufficient?
        ├── Is real-time inference necessary?
        └── Does the workload require a multi-stage architecture?
  │
  ▼
══════════════════════ DESIGN FRAMEWORK ══════════════════════════
  │
  ▼
[P9] CONSOLIDATE EVERYTHING INTO SIX DESIGN STEPS
  │
  ├── STEP 1: CLARIFY                                      [P2, P4, P5]
  │     ├── Identify the business objective
  │     ├── Identify the user and the decision
  │     ├── Establish the existing baseline
  │     ├── Define measurable requirements
  │     └── Document assumptions and scope
  │
  ▼
  ├── STEP 2: ESTIMATE                                            [P8]
  │     ├── Traffic and throughput
  │     ├── Computational workload
  │     ├── Storage requirements
  │     └── Cost
  │
  ▼
  ├── STEP 3: OUTLINE                                       [P1, P2]
  │     ├── Define the high-level architecture
  │     ├── Design the online serving path
  │     ├── Design the offline training loop
  │     └── Start with the simplest viable solution
  │
  ▼
  ├── STEP 4: DEEP DIVE                                     [P5, P6]
  │     ├── Identify the hardest components of the system
  │     ├── Investigate data and label quality
  │     ├── Investigate model and decision requirements
  │     └── Resolve the constraints exposed by the estimates
  │
  ▼
  ├── STEP 5: TRADE-OFFS                                    [P6, P7]
  │     ├── Compare alternative design choices
  │     ├── Accuracy vs. error cost
  │     ├── Automation vs. human intervention
  │     ├── Simplicity vs. adaptability
  │     └── Competing business objectives
  │
  ▼
  └── STEP 6: FAILURE MODES                                       [P3]
        ├── Identify potential silent failures
        ├── Establish fast and slow monitoring signals
        ├── Assign ownership of monitoring
        ├── Establish fallback mechanisms
        └── Define adaptation and retraining mechanisms
  │
  │   Does a later design decision invalidate an earlier assumption?
  │     ├── YES → Revisit the relevant design step
  │     └── NO  → Proceed to design synthesis
  │
  ▼
```