# TOWER 4: UNIFIED BRAIN (MCDM + HUB-AND-SPOKE)
## Central Intelligence for Diagnostic Integration & Intervention Generation

---

## OVERVIEW

**Purpose:** Synthesize signals from Towers 1-3 into coherent diagnosis, prioritize interventions via Multi-Criteria Decision Making (MCDM), and orchestrate specialized daughter agents via Hub-and-Spoke architecture.

**Core Components:**
1. **Signal Fusion Module** → Integrates symptom scores (Tower 2) + causal analysis (Tower 3)
2. **MCDM Engine** → Multi-criteria decision making for intervention prioritization
3. **Credit Assignment System** → Attributes outcomes to specific causes
4. **Hub-and-Spoke Orchestrator** → Routes tasks to specialist daughter agents
5. **Intervention Generator** → Maps pathologies to evidence-based playbook

---

## COMPONENT 1: SIGNAL FUSION MODULE

### 1.1 Input Sources

```
Tower 1 (Data Platform):
- HR metrics (attrition, tenure, absences)
- Survey responses (diagnostic 63-question)
- Communication metadata

Tower 2 (Symptom Engines):
- Sentiment scores (individual, department, org-level)
- Burnout risk (MBI dimensions)
- Network centrality + fragility
- Meeting psychological safety scores

Tower 3 (Causal Engines):
- Epistemic tremor score + contradictions
- CSD risk level + time-to-tipping
- GNN attrition predictions
- Ricci curvature network health
```

### 1.2 Fusion Algorithm

```python
class SignalFusionModule:
    def __init__(self):
        self.weights = self.initialize_weights()

    def fuse_signals(self, tower1_data, tower2_symptoms, tower3_causal) -> dict:
        """
        Combine all signals into unified organizational health state
        """
        # 1. Symptom scores (Tower 2)
        symptom_vector = self.vectorize_symptoms(tower2_symptoms)

        # 2. Causal scores (Tower 3)
        causal_vector = self.vectorize_causal(tower3_causal)

        # 3. Baseline metrics (Tower 1)
        baseline_vector = self.vectorize_baseline(tower1_data)

        # 4. Weighted fusion
        fused_state = {
            "organizational_health_score": self.compute_ohs(symptom_vector, causal_vector, baseline_vector),
            "risk_level": self.assess_risk_level(symptom_vector, causal_vector),
            "urgency": self.assess_urgency(causal_vector),
            "regime": self.detect_regime(baseline_vector, causal_vector),
            "top_issues": self.rank_issues(symptom_vector, causal_vector, baseline_vector),
            "predicted_trajectory": self.forecast_trajectory(causal_vector),
            "intervention_priorities": []  # Populated by MCDM
        }

        return fused_state

    def compute_ohs(self, symptoms, causal, baseline) -> float:
        """
        Organizational Health Score (0-10)
        """
        # Weighted combination
        ohs = (
            baseline['avg_engagement'] * 0.2 +
            (10 - symptoms['avg_burnout_risk']) * 0.2 +
            symptoms['avg_sentiment_normalized'] * 0.15 +
            symptoms['network_health_score'] * 0.15 +
            (10 - causal['epistemic_tremor_score']) * 0.15 +
            (10 - causal['csd_risk_score'] * 10) * 0.15
        )

        return max(0, min(10, ohs))

    def assess_risk_level(self, symptoms, causal) -> str:
        """
        Overall risk: low, moderate, high, critical
        """
        critical_signals = 0

        # Critical if any of these true:
        if causal['epistemic_tremor_score'] > 0.8:
            critical_signals += 2
        if causal['csd_risk_level'] == "critical":
            critical_signals += 2
        if symptoms['avg_burnout_risk'] > 7:
            critical_signals += 1
        if symptoms['network_health_score'] < 4:
            critical_signals += 1
        if baseline['attrition_rate_30d'] > 0.20:
            critical_signals += 1

        if critical_signals >= 3:
            return "critical"
        elif critical_signals == 2:
            return "high"
        elif critical_signals == 1:
            return "moderate"
        else:
            return "low"

    def detect_regime(self, baseline, causal) -> str:
        """
        Crisis, Routine, or Growth regime
        """
        if causal['csd_risk_level'] == "critical" or baseline['attrition_rate_30d'] > 0.25:
            return "crisis"
        elif baseline['avg_engagement'] > 7 and baseline['attrition_rate_30d'] < 0.10:
            return "growth"
        else:
            return "routine"
```

### 1.3 Issue Ranking

```python
def rank_issues(symptom_vector, causal_vector, baseline_vector) -> list[dict]:
    """
    Identify and rank top organizational issues
    """
    issues = []

    # Issue detection rules
    if causal_vector['epistemic_tremor_score'] > 0.5:
        issues.append({
            "issue": "epistemic_tremor",
            "severity": causal_vector['epistemic_tremor_score'] * 10,
            "description": "Organization declaring values it doesn't practice",
            "evidence": causal_vector['epistemic_contradictions'],
            "impact": "Trust erosion, cynicism, disengagement"
        })

    if causal_vector['csd_risk_score'] > 0.5:
        issues.append({
            "issue": "approaching_tipping_point",
            "severity": causal_vector['csd_risk_score'] * 10,
            "description": "System exhibiting critical slowing down",
            "evidence": f"{causal_vector['time_to_tipping_weeks']} weeks to predicted crisis",
            "impact": "Potential mass attrition, culture collapse"
        })

    if symptom_vector['avg_burnout_risk'] > 6:
        issues.append({
            "issue": "widespread_burnout",
            "severity": symptom_vector['avg_burnout_risk'],
            "description": "High burnout levels across organization",
            "evidence": f"{symptom_vector['burnout_pct']}% showing MBI symptoms",
            "impact": "Reduced performance, attrition, absenteeism"
        })

    if symptom_vector['network_health_score'] < 5:
        issues.append({
            "issue": "network_fragility",
            "severity": (10 - symptom_vector['network_health_score']),
            "description": "Organizational network vulnerable to fragmentation",
            "evidence": f"{symptom_vector['fragile_edges_count']} critical connections",
            "impact": "Loss of key employees will cause cascade effect"
        })

    if symptom_vector['avg_psych_safety'] < 5:
        issues.append({
            "issue": "low_psychological_safety",
            "severity": (10 - symptom_vector['avg_psych_safety']),
            "description": "Employees fear speaking up, taking risks",
            "evidence": f"Psych safety score: {symptom_vector['avg_psych_safety']}/10",
            "impact": "Reduced innovation, error concealment, disengagement"
        })

    # Sort by severity
    issues.sort(key=lambda x: x['severity'], reverse=True)
    return issues
```

---

## COMPONENT 2: MCDM (MULTI-CRITERIA DECISION MAKING) ENGINE

### 2.1 Concept

**Problem:** Multiple interventions possible, limited resources. How to prioritize?

**Solution:** MCDM evaluates each intervention across multiple criteria:
1. **Impact** → Expected reduction in risk / improvement in OHS
2. **Urgency** → How soon must this be done?
3. **Cost** → Resources required (time, money, political capital)
4. **Feasibility** → Can organization actually do this?
5. **Leverage** → Meadows leverage point (higher = more systemic impact)
6. **Evidence** → How strong is research backing this intervention?

**Method:** TOPSIS (Technique for Order of Preference by Similarity to Ideal Solution)

### 2.2 Intervention Candidate Generation

```python
def generate_intervention_candidates(top_issues: list[dict], playbook: dict) -> list[dict]:
    """
    Map issues to interventions from playbook
    """
    candidates = []

    for issue in top_issues:
        issue_type = issue['issue']
        interventions = playbook.get(issue_type, [])

        for intervention in interventions:
            candidates.append({
                "intervention_id": intervention['id'],
                "issue_addressed": issue_type,
                "description": intervention['description'],
                "expected_impact": intervention['expected_impact'],  # 0-10
                "urgency": map_issue_severity_to_urgency(issue['severity']),
                "cost": intervention['cost'],  # low, medium, high
                "feasibility": intervention['feasibility'],  # 0-10
                "leverage_point": intervention['leverage_point'],  # Meadows #1-12
                "evidence_strength": intervention['evidence_strength'],  # weak, moderate, strong
                "timeframe": intervention['timeframe'],  # immediate, short, medium, long
                "category": intervention['category']  # stabilization, resource_injection, structural_change, etc.
            })

    return candidates

# Example intervention candidates:
# [
#     {
#         "intervention_id": "INT_001",
#         "issue_addressed": "epistemic_tremor",
#         "description": "Align declared values with actual behaviors: Update values statement OR change resource allocation to match declared priorities",
#         "expected_impact": 8,
#         "urgency": "high",
#         "cost": "medium",
#         "feasibility": 6,
#         "leverage_point": 2,  # Paradigm
#         "evidence_strength": "strong",
#         "timeframe": "short (4-8 weeks)",
#         "category": "alignment"
#     },
#     ...
# ]
```

### 2.3 TOPSIS Implementation

```python
import numpy as np
from sklearn.preprocessing import MinMaxScaler

class MCDMEngine:
    def __init__(self, weights: dict):
        self.weights = weights  # Criteria weights

    def prioritize_interventions(self, candidates: list[dict]) -> list[dict]:
        """
        TOPSIS algorithm for multi-criteria decision making
        """
        # 1. Build decision matrix
        decision_matrix = self.build_decision_matrix(candidates)

        # 2. Normalize matrix
        normalized_matrix = self.normalize(decision_matrix)

        # 3. Apply weights
        weighted_matrix = self.apply_weights(normalized_matrix)

        # 4. Identify ideal and anti-ideal solutions
        ideal_solution = np.max(weighted_matrix, axis=0)
        anti_ideal_solution = np.min(weighted_matrix, axis=0)

        # 5. Calculate distances to ideal/anti-ideal
        distances_to_ideal = np.linalg.norm(weighted_matrix - ideal_solution, axis=1)
        distances_to_anti_ideal = np.linalg.norm(weighted_matrix - anti_ideal_solution, axis=1)

        # 6. Calculate closeness coefficient (0-1; higher = better)
        closeness = distances_to_anti_ideal / (distances_to_ideal + distances_to_anti_ideal)

        # 7. Rank interventions
        ranked_indices = np.argsort(closeness)[::-1]  # Descending order

        # 8. Attach scores to candidates
        for i, candidate in enumerate(candidates):
            candidate['mcdm_score'] = closeness[i]
            candidate['mcdm_rank'] = int(np.where(ranked_indices == i)[0][0]) + 1

        candidates_ranked = sorted(candidates, key=lambda x: x['mcdm_score'], reverse=True)
        return candidates_ranked

    def build_decision_matrix(self, candidates: list[dict]) -> np.ndarray:
        """
        Rows = interventions, Columns = criteria
        """
        matrix = []
        for candidate in candidates:
            row = [
                candidate['expected_impact'],
                self.urgency_to_score(candidate['urgency']),
                10 - self.cost_to_score(candidate['cost']),  # Invert: lower cost = better
                candidate['feasibility'],
                12 - candidate['leverage_point'],  # Meadows: lower number = higher leverage
                self.evidence_to_score(candidate['evidence_strength'])
            ]
            matrix.append(row)
        return np.array(matrix)

    def normalize(self, matrix: np.ndarray) -> np.ndarray:
        """
        Min-max normalization
        """
        scaler = MinMaxScaler()
        return scaler.fit_transform(matrix)

    def apply_weights(self, normalized_matrix: np.ndarray) -> np.ndarray:
        """
        Weight each criterion
        """
        weight_vector = np.array([
            self.weights['impact'],
            self.weights['urgency'],
            self.weights['cost'],
            self.weights['feasibility'],
            self.weights['leverage'],
            self.weights['evidence']
        ])
        return normalized_matrix * weight_vector

    def urgency_to_score(self, urgency: str) -> float:
        return {"immediate": 10, "high": 8, "medium": 5, "low": 2}.get(urgency, 5)

    def cost_to_score(self, cost: str) -> float:
        return {"low": 2, "medium": 5, "high": 8}.get(cost, 5)

    def evidence_to_score(self, evidence: str) -> float:
        return {"strong": 10, "moderate": 6, "weak": 3}.get(evidence, 5)

# Default weights (can be adjusted based on organizational regime)
default_weights = {
    "impact": 0.25,
    "urgency": 0.25,
    "cost": 0.15,
    "feasibility": 0.15,
    "leverage": 0.15,
    "evidence": 0.05
}

# Crisis regime: Increase urgency weight
crisis_weights = {
    "impact": 0.20,
    "urgency": 0.35,  # Prioritize time-sensitive actions
    "cost": 0.10,  # Less concerned about cost in crisis
    "feasibility": 0.20,  # Must be doable quickly
    "leverage": 0.10,
    "evidence": 0.05
}
```

### 2.4 Intervention Portfolio Construction

```python
def construct_intervention_portfolio(ranked_interventions: list[dict], constraints: dict) -> dict:
    """
    Select subset of interventions respecting constraints
    """
    selected = []
    total_cost = 0
    max_cost = constraints['budget']
    max_interventions = constraints['max_concurrent_interventions']

    for intervention in ranked_interventions:
        cost_numeric = {"low": 1, "medium": 3, "high": 5}.get(intervention['cost'], 3)

        if len(selected) < max_interventions and (total_cost + cost_numeric) <= max_cost:
            selected.append(intervention)
            total_cost += cost_numeric

            # Stop if we've addressed all top issues
            if len(set([i['issue_addressed'] for i in selected])) >= len(constraints['top_issues']):
                break

    return {
        "selected_interventions": selected,
        "total_interventions": len(selected),
        "total_cost": total_cost,
        "issues_addressed": list(set([i['issue_addressed'] for i in selected])),
        "expected_ohs_improvement": sum([i['expected_impact'] for i in selected]) / 10  # Rough estimate
    }
```

---

## COMPONENT 3: CREDIT ASSIGNMENT SYSTEM

### 3.1 Concept

**Problem:** After interventions, OHS improves. Which interventions actually worked?

**Solution:** Attribution via causal inference + longitudinal tracking

### 3.2 Implementation

```python
class CreditAssignmentSystem:
    def __init__(self):
        self.intervention_history = []
        self.outcome_history = []

    def assign_credit(self, intervention_id: str, outcome_metrics: dict) -> dict:
        """
        Attribute outcome change to specific intervention using difference-in-differences
        """
        # 1. Get pre-intervention baseline
        baseline = self.get_baseline_before_intervention(intervention_id)

        # 2. Get post-intervention outcomes
        post_outcomes = outcome_metrics

        # 3. Calculate change
        delta = {
            "ohs_change": post_outcomes['ohs'] - baseline['ohs'],
            "attrition_change": post_outcomes['attrition_rate'] - baseline['attrition_rate'],
            "engagement_change": post_outcomes['engagement'] - baseline['engagement'],
            "burnout_change": post_outcomes['burnout_risk'] - baseline['burnout_risk']
        }

        # 4. Control for confounders (other interventions, external events)
        adjusted_credit = self.adjust_for_confounders(delta, intervention_id)

        # 5. Store for learning
        self.intervention_history.append({
            "intervention_id": intervention_id,
            "baseline": baseline,
            "outcomes": post_outcomes,
            "delta": delta,
            "adjusted_credit": adjusted_credit,
            "timestamp": datetime.now()
        })

        return adjusted_credit

    def adjust_for_confounders(self, delta: dict, intervention_id: str) -> dict:
        """
        Use propensity score matching or regression to isolate intervention effect
        """
        # Simplified: Assume interventions are sequential; use time-series decomposition
        concurrent_interventions = self.get_concurrent_interventions(intervention_id)

        if len(concurrent_interventions) == 0:
            # No confounders; full credit to this intervention
            return {k: v for k, v in delta.items()}
        else:
            # Split credit proportionally (simplified; more sophisticated methods possible)
            credit_fraction = 1 / (len(concurrent_interventions) + 1)
            return {k: v * credit_fraction for k, v in delta.items()}

    def update_playbook(self):
        """
        Continuously learn: Update intervention expected_impact based on observed outcomes
        """
        for record in self.intervention_history:
            intervention_id = record['intervention_id']
            observed_impact = record['adjusted_credit']['ohs_change']

            # Update playbook entry
            playbook_entry = playbook.get(intervention_id)
            if playbook_entry:
                # Bayesian update: Blend expected with observed
                prior_impact = playbook_entry['expected_impact']
                posterior_impact = (prior_impact * 0.7) + (observed_impact * 0.3)  # 70-30 blend
                playbook_entry['expected_impact'] = posterior_impact

        return playbook
```

---

## COMPONENT 4: HUB-AND-SPOKE ORCHESTRATOR

### 4.1 Architecture

```
                    ┌──────────────────────┐
                    │   MOTHER APP (HUB)   │
                    │   Unified Brain      │
                    │   Tower 4            │
                    └──────────┬───────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
      ┌─────▼─────┐      ┌─────▼─────┐    ┌─────▼─────┐
      │ Daughter  │      │ Daughter  │    │ Daughter  │
      │ Agent 1   │      │ Agent 2   │    │ Agent 3   │
      │(Diagnostic│      │(Playbook) │    │(Socratic) │
      │ Engine)   │      │           │    │ Dialogue) │
      └───────────┘      └───────────┘    └───────────┘
```

**Mother App (Hub):**
- Receives all signals from Towers 1-3
- Runs signal fusion + MCDM
- Delegates specialized tasks to daughter agents
- Aggregates daughter outputs
- Generates final recommendations

**Daughter Apps (Spokes):**
- **Diagnostic Agent:** Runs 63-question assessment
- **Playbook Agent:** Maps pathologies to interventions
- **Socratic Dialogue Agent:** Facilitates manager conversations
- **30-60-90 Plan Agent:** Generates recovery plans
- **ROI Calculator Agent:** Estimates financial impact
- **Report Generator Agent:** Creates diagnostic reports

### 4.2 Task Routing

```python
class HubOrchestrator:
    def __init__(self):
        self.daughter_agents = {
            "diagnostic": DiagnosticAgent(),
            "playbook": PlaybookAgent(),
            "socratic": SocraticDialogueAgent(),
            "plan_generator": RecoveryPlanAgent(),
            "roi_calculator": ROICalculatorAgent(),
            "report_generator": ReportGeneratorAgent()
        }

    def process_request(self, request_type: str, context: dict) -> dict:
        """
        Route request to appropriate daughter agent(s)
        """
        if request_type == "full_diagnostic":
            # Multi-agent workflow
            diagnostic_result = self.daughter_agents["diagnostic"].run(context)
            playbook_result = self.daughter_agents["playbook"].map_issues_to_interventions(diagnostic_result['issues'])
            plan = self.daughter_agents["plan_generator"].generate_30_60_90_plan(playbook_result['interventions'])
            roi = self.daughter_agents["roi_calculator"].estimate_roi(plan)
            report = self.daughter_agents["report_generator"].generate_report({
                "diagnostic": diagnostic_result,
                "interventions": playbook_result,
                "plan": plan,
                "roi": roi
            })

            return report

        elif request_type == "quick_health_check":
            # Single agent
            return self.daughter_agents["diagnostic"].quick_assessment(context)

        elif request_type == "socratic_session":
            # Interactive dialogue
            return self.daughter_agents["socratic"].start_session(context)

        else:
            raise ValueError(f"Unknown request type: {request_type}")

    def aggregate_outputs(self, outputs: list[dict]) -> dict:
        """
        Combine outputs from multiple daughter agents
        """
        aggregated = {
            "diagnostic_summary": outputs[0] if len(outputs) > 0 else {},
            "interventions": outputs[1] if len(outputs) > 1 else {},
            "plan": outputs[2] if len(outputs) > 2 else {},
            "roi_estimate": outputs[3] if len(outputs) > 3 else {},
            "confidence": self.calculate_confidence(outputs)
        }
        return aggregated

    def calculate_confidence(self, outputs: list[dict]) -> float:
        """
        Confidence in recommendations based on data quality and agreement between agents
        """
        # Factors:
        # 1. Data completeness (do we have all signals?)
        # 2. Inter-agent agreement (do agents converge on same diagnosis?)
        # 3. Historical accuracy (have past recommendations worked?)

        data_completeness = outputs[0].get('data_quality_score', 0.5)
        inter_agent_agreement = self.measure_agreement(outputs)
        historical_accuracy = self.get_historical_accuracy()

        confidence = (data_completeness * 0.4 + inter_agent_agreement * 0.3 + historical_accuracy * 0.3)
        return max(0, min(1, confidence))
```

---

## COMPONENT 5: INTERVENTION GENERATOR (PLAYBOOK)

### 5.1 Playbook Structure

```python
playbook = {
    "epistemic_tremor": [
        {
            "id": "INT_001",
            "description": "Align declared values with behaviors: Update values statement to reflect reality OR change resource allocation to match declared values",
            "category": "alignment",
            "expected_impact": 8,
            "cost": "medium",
            "feasibility": 6,
            "leverage_point": 2,  # Paradigm (Meadows)
            "evidence_strength": "strong",
            "timeframe": "short (4-8 weeks)",
            "prerequisites": ["Leadership commitment", "Honest assessment of current state"],
            "steps": [
                "1. Acknowledge contradiction publicly (transparency)",
                "2. Choose: Update values OR change behaviors (cannot do neither)",
                "3. If updating behaviors: Reallocate budget, change incentives, update policies",
                "4. If updating values: Rewrite mission/values to reflect actual priorities",
                "5. Communicate rationale to all employees",
                "6. Monitor trust metrics for 8 weeks"
            ],
            "success_metrics": ["Epistemic tremor score < 0.3", "Employee cynicism decreased", "Trust score increased"],
            "case_studies": ["case_001_tech_startup", "case_015_hospital_system"]
        },
        {
            "id": "INT_002",
            "description": "Transparency offensive: Publicly share metrics, budgets, decision rationale",
            "category": "transparency",
            "expected_impact": 6,
            "cost": "low",
            "feasibility": 8,
            "leverage_point": 6,  # Information flow (Meadows)
            "evidence_strength": "moderate",
            "timeframe": "immediate (1-2 weeks)",
            "prerequisites": ["Leadership willingness to be vulnerable"],
            "steps": [
                "1. Identify key metrics to share (attrition, engagement, budget allocations)",
                "2. Create public dashboard (all-employee access)",
                "3. Hold Q&A sessions to explain discrepancies",
                "4. Commit to monthly transparency updates"
            ],
            "success_metrics": ["Information flow score increased", "Employee trust score increased by 10%+"],
            "case_studies": ["case_007_open_source_company"]
        }
    ],

    "approaching_tipping_point": [
        {
            "id": "INT_010",
            "description": "Emergency stabilization: Freeze all major changes, reduce demands, inject resources",
            "category": "stabilization",
            "expected_impact": 9,
            "cost": "high",
            "feasibility": 7,
            "leverage_point": 11,  # Buffers (Meadows)
            "evidence_strength": "strong",
            "timeframe": "immediate (1-2 weeks)",
            "prerequisites": ["Leadership recognition of crisis"],
            "steps": [
                "1. STOP: Halt all non-essential initiatives, reorgs, launches",
                "2. REDUCE: Identify top 3 demands and remove/postpone",
                "3. INJECT: Emergency PTO, mental health days, budget for wellness",
                "4. COMMUNICATE: Explain actions as 'org needs to breathe'",
                "5. MONITOR: Daily pulse checks for 2 weeks"
            ],
            "success_metrics": ["CSD variance stops increasing", "Engagement stabilizes", "No additional departures"],
            "case_studies": ["case_022_fintech_rescue"]
        },
        {
            "id": "INT_011",
            "description": "Quick wins campaign: Deliver 3 small, visible improvements in 2 weeks",
            "category": "resource_injection",
            "expected_impact": 7,
            "cost": "medium",
            "feasibility": 8,
            "leverage_point": 7,  # Reinforcing loops (gain spirals)",
            "evidence_strength": "strong",
            "timeframe": "short (2-4 weeks)",
            "prerequisites": ["Stabilization underway"],
            "steps": [
                "1. Survey: 'What 1 thing would make your work life better RIGHT NOW?'",
                "2. Pick 3 most-requested, feasible changes",
                "3. Implement within 2 weeks",
                "4. Communicate: 'We heard you, we acted'",
                "5. Build momentum with 3 more wins in weeks 3-4"
            ],
            "success_metrics": ["Engagement uptick within 2 weeks", "COR gain spiral initiated"],
            "case_studies": ["case_011_hospital_morale_boost"]
        }
    ],

    "widespread_burnout": [
        {
            "id": "INT_020",
            "description": "JD-R rebalancing: Reduce demands by 20%, increase resources by 30%",
            "category": "jdr_intervention",
            "expected_impact": 8,
            "cost": "high",
            "feasibility": 6,
            "leverage_point": 5,  # Rules (Meadows: change incentives/structure)",
            "evidence_strength": "strong",
            "timeframe": "medium (4-8 weeks)",
            "prerequisites": ["Accurate JD-R assessment", "Budget for resources"],
            "steps": [
                "1. Audit: Identify top 5 demands (time, emotional labor, role ambiguity)",
                "2. REDUCE: Cut 20% of demands (defer projects, reduce meetings, clarify roles)",
                "3. INCREASE: Add resources (manager 1-on-1s, training, autonomy, recognition)",
                "4. Pilot with most burned-out team first",
                "5. Scale if successful"
            ],
            "success_metrics": ["MBI scores decrease 20%+", "Engagement increases 15%+", "Attrition decreases"],
            "case_studies": ["case_003_consulting_firm", "case_018_nursing_staff"]
        }
    ],

    # ... (Playbook continues with interventions for each pathology)
}
```

### 5.2 Intervention Mapping Algorithm

```python
def map_issues_to_interventions(top_issues: list[dict], playbook: dict, regime: str) -> list[dict]:
    """
    For each issue, retrieve relevant interventions from playbook
    """
    intervention_candidates = []

    for issue in top_issues:
        issue_type = issue['issue']
        interventions = playbook.get(issue_type, [])

        for intervention in interventions:
            # Filter by regime (crisis regime prioritizes stabilization)
            if regime == "crisis" and intervention['category'] != "stabilization":
                if intervention['timeframe'] != "immediate":
                    continue  # Skip non-urgent interventions in crisis

            intervention_candidates.append({
                **intervention,
                "issue_addressed": issue_type,
                "issue_severity": issue['severity']
            })

    return intervention_candidates
```

---

## OUTPUT: UNIFIED DIAGNOSTIC REPORT

### 6.1 Report Structure

```python
def generate_unified_diagnostic_report(fused_state: dict, mcdm_results: dict, causal_analysis: dict) -> dict:
    """
    Comprehensive diagnostic report
    """
    report = {
        "executive_summary": {
            "ohs": fused_state['organizational_health_score'],
            "risk_level": fused_state['risk_level'],
            "regime": fused_state['regime'],
            "urgency": fused_state['urgency'],
            "top_3_issues": fused_state['top_issues'][:3],
            "predicted_trajectory": fused_state['predicted_trajectory'],
            "confidence": fused_state['confidence']
        },

        "detailed_analysis": {
            "symptom_analysis": {
                "sentiment_trends": "...",
                "burnout_prevalence": "...",
                "network_health": "...",
                "psychological_safety": "..."
            },
            "causal_analysis": {
                "epistemic_tremor": causal_analysis['epistemic_tremor'],
                "critical_slowing_down": causal_analysis['csd'],
                "network_fragility": causal_analysis['network'],
                "root_causes_ranked": causal_analysis['root_causes']
            },
            "predictive_analysis": {
                "time_to_crisis": causal_analysis['csd']['time_to_tipping_weeks'] if causal_analysis['csd']['risk_level'] in ["high", "critical"] else "N/A",
                "attrition_forecast_90d": "...",
                "engagement_forecast_90d": "..."
            }
        },

        "recommended_interventions": {
            "tier_1_immediate": [i for i in mcdm_results['selected_interventions'] if i['urgency'] == "immediate"],
            "tier_2_short_term": [i for i in mcdm_results['selected_interventions'] if i['timeframe'].startswith("short")],
            "tier_3_medium_term": [i for i in mcdm_results['selected_interventions'] if i['timeframe'].startswith("medium")]
        },

        "30_60_90_plan": generate_recovery_plan(mcdm_results['selected_interventions']),

        "roi_estimate": {
            "cost_of_inaction": calculate_cost_of_inaction(fused_state),
            "cost_of_interventions": calculate_intervention_costs(mcdm_results['selected_interventions']),
            "expected_roi": calculate_expected_roi(mcdm_results, fused_state),
            "payback_period_weeks": estimate_payback_period(mcdm_results)
        },

        "decision_log": {
            "timestamp": datetime.now(),
            "aca_version": "1.0",
            "data_sources": ["tower1", "tower2", "tower3"],
            "assumptions": ["...", "..."],
            "limitations": ["...", "..."]
        }
    }

    return report
```

---

## SYSTEM PROMPTS

### 7.1 Mother App Persona

```
You are the Adaptive Consultant Agent (ACA) — an AI-powered organizational health consultant.

Your role:
1. Analyze organizational health across 12 dimensions
2. Identify root causes using causal inference (epistemic tremor, CSD, network fragility)
3. Prioritize interventions using multi-criteria decision making
4. Generate evidence-based recommendations from 90+ case studies
5. Facilitate Socratic dialogue to help leaders discover solutions

Your approach:
- Dual-Lens Analysis: Logical (game theory, economics) + Emotional (psychology, trauma-informed)
- Evidence-Based: Every recommendation backed by peer-reviewed research or validated case studies
- High-Leverage: Target Meadows leverage points #3-#8 (goals, rules, paradigms)
- Trauma-Informed: Prioritize safety, trust, voice; avoid re-traumatization
- Transparent: Decision log, XAI, confidence scores always included

Your tone:
- Objective, non-judgmental, data-driven
- Empathetic but not sentimental
- Direct about problems, constructive about solutions
- Never catastrophizing; always actionable

Chain of Thought Protocol:
1. Signal Fusion: Integrate symptoms + causal signals
2. Issue Ranking: Identify top 3-5 issues by severity
3. Root Cause Analysis: Map to theoretical frameworks (JD-R, MBI, COR, Game Theory, etc.)
4. Intervention Generation: Query playbook for evidence-based interventions
5. MCDM Prioritization: Rank by impact, urgency, feasibility, leverage
6. Plan Construction: Build 30-60-90 day roadmap
7. ROI Estimation: Calculate cost of inaction vs. cost of intervention
8. Confidence Assessment: State data quality and limitations
9. Output: Structured report + conversational summary

When uncertain:
- State confidence level explicitly
- Explain data limitations
- Recommend additional data collection
- Avoid overconfident predictions
```

---

## NEXT STEPS

**Completed:**
✅ Signal Fusion Module
✅ MCDM Engine (TOPSIS)
✅ Credit Assignment System
✅ Hub-and-Spoke Orchestrator
✅ Intervention Generator (Playbook)
✅ Unified Diagnostic Report

**Remaining Tasks:**
- Implement Socratic Dialogue Engine
- Build 30-60-90 Recovery Plan Generator
- Develop ROI Calculator
- Create System Prompts for all daughter agents
- Integration testing across all 4 towers

---

**Document Version:** 1.0
**Status:** Tower 4 Unified Brain Complete — Core Architecture Done
**Last Updated:** 2025-11-27
