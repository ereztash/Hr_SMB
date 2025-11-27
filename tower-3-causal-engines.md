# TOWER 3: CAUSAL ENGINES
## Root Cause Detection & Predictive Early Warning Systems

---

## OVERVIEW

**Purpose:** Go beyond symptoms to identify **causal mechanisms** driving organizational dysfunction. Predict crises 4-8 weeks before they occur.

**Core Engines:**
1. **Loss Contradiction Engine** → Detects "epistemic tremor" (declared vs. behavioral misalignment)
2. **CSD/EWS Engine** → Critical Slowing Down detection for tipping point prediction
3. **GNN + Ricci Curvature Engine** → Network fragility and trust erosion analysis

**Theoretical Foundation:**
- Signaling Theory (costly vs. cheap signals)
- Complex Systems Theory (tipping points, phase transitions)
- Graph Theory + Differential Geometry (Ricci curvature)
- Information Theory (divergence, entropy)

---

## ENGINE 1: LOSS CONTRADICTION DETECTOR

### 1.1 Concept

**Definition:**
> "Epistemic tremor" = Sustained contradiction between what an organization **declares** (cheap signals) and what it **does** (costly signals)

**Mathematical Formulation:**
```
L_declared = Loss function on declared channel (surveys, values statements, leadership communications)
L_behavioral = Loss function on behavioral channel (actual behaviors, resource allocation, consequences)

Correlation: ρ(L_declared, L_behavioral)

Epistemic Tremor Score = |ρ| × Magnitude_of_Gap

Where:
- ρ > +0.5: Aligned (healthy)
- ρ ≈ 0: Disconnected (warning)
- ρ < -0.3: Contradictory (crisis)
```

### 1.2 Data Collection

**Declared Channel (Cheap Signals):**
```python
class DeclaredChannelCollector:
    def collect(self) -> dict:
        return {
            "leadership_communications": self.analyze_leadership_messages(),
            "company_values": self.extract_values_statements(),
            "policies": self.review_hr_policies(),
            "survey_stated_priorities": self.aggregate_stated_priorities(),
            "mission_statement": self.parse_mission_statement()
        }

    def analyze_leadership_messages(self) -> list[dict]:
        """
        Extract themes from CEO/leadership emails, all-hands, blog posts
        """
        messages = fetch_leadership_comms(last_90_days=True)
        themes = []
        for msg in messages:
            themes.extend(extract_themes(msg['text'], categories=[
                "work_life_balance",
                "people_first",
                "innovation",
                "transparency",
                "diversity_equity_inclusion",
                "psychological_safety",
                "employee_development"
            ]))
        return Counter(themes).most_common()

    # Example output:
    # [("people_first", 42), ("work_life_balance", 38), ("transparency", 25), ...]
```

**Behavioral Channel (Costly Signals):**
```python
class BehavioralChannelCollector:
    def collect(self) -> dict:
        return {
            "resource_allocation": self.analyze_budget(),
            "actual_behaviors": self.detect_behavioral_patterns(),
            "consequences": self.analyze_reward_punishment(),
            "network_data": self.assess_trust_network(),
            "time_allocation": self.measure_leadership_time()
        }

    def analyze_budget(self) -> dict:
        """
        Where does money actually go?
        """
        budget = fetch_budget_allocations()
        return {
            "employee_development_pct": budget['learning_development'] / budget['total'],
            "wellness_programs_pct": budget['wellness'] / budget['total'],
            "dei_initiatives_pct": budget['dei'] / budget['total'],
            "perks_pct": budget['perks'] / budget['total'],
            "executive_compensation_vs_median": budget['exec_comp'] / budget['median_salary']
        }

    def analyze_reward_punishment(self) -> dict:
        """
        Who gets promoted? Who gets punished? For what?
        """
        return {
            "promotions": self.analyze_promotions(),
            "terminations": self.analyze_terminations(),
            "recognition": self.analyze_recognition_patterns(),
            "performance_reviews": self.analyze_review_criteria()
        }

    def analyze_promotions(self) -> dict:
        promotions = fetch_promotions(last_12_months=True)
        return {
            "promoted_for_results_vs_culture": ratio_results_to_culture(promotions),
            "promoted_innovators_who_failed": count_promoted_after_failure(promotions),
            "promoted_despite_toxic_behavior": count_toxic_promoted(promotions),
            "avg_tenure_before_promotion": mean([p['tenure_months'] for p in promotions])
        }

    # If "innovation encouraged" but promoted_innovators_who_failed = 0 → CONTRADICTION
```

### 1.3 Loss Functions

**Declared Loss (L_declared):**
```python
def compute_declared_loss(declared_data: dict, employee_survey: dict) -> float:
    """
    Loss = Divergence between what's declared and what employees expect/perceive
    """
    losses = []

    for value_category in ["people_first", "work_life_balance", "transparency", "innovation"]:
        declared_emphasis = declared_data['themes'].get(value_category, 0)  # Frequency in leadership comms
        employee_perception = employee_survey[f"perceive_{value_category}"]  # Survey: "Org prioritizes X" (1-10)

        # Loss = How much organization CLAIMS to value X
        loss = declared_emphasis / 100  # Normalize to 0-1
        losses.append((value_category, loss))

    return dict(losses)

# Example output:
# {"people_first": 0.42, "work_life_balance": 0.38, "transparency": 0.25, ...}
```

**Behavioral Loss (L_behavioral):**
```python
def compute_behavioral_loss(behavioral_data: dict, employee_experience: dict) -> float:
    """
    Loss = Divergence between actual behaviors and employee experience
    """
    losses = []

    # 1. People First: Check resource allocation
    people_first_behavioral = (
        behavioral_data['budget']['employee_development_pct'] +
        behavioral_data['budget']['wellness_programs_pct']
    ) / 2
    losses.append(("people_first", people_first_behavioral))

    # 2. Work-Life Balance: Check actual time demands
    work_life_balance_behavioral = 1 - (
        behavioral_data['time']['avg_hours_per_week'] - 40) / 40  # Penalty for >40 hrs/week
    )
    work_life_balance_behavioral *= (1 - behavioral_data['time']['after_hours_email_pct'])
    losses.append(("work_life_balance", max(0, work_life_balance_behavioral)))

    # 3. Transparency: Check information sharing
    transparency_behavioral = behavioral_data['network']['information_flow_openness']  # 0-1 from ONA
    losses.append(("transparency", transparency_behavioral))

    # 4. Innovation: Check tolerance for failure
    innovation_behavioral = behavioral_data['consequences']['promoted_after_failure_rate']
    losses.append(("innovation", innovation_behavioral))

    return dict(losses)

# Example output:
# {"people_first": 0.15, "work_life_balance": 0.22, "transparency": 0.30, ...}
```

### 1.4 Correlation Calculation

```python
def calculate_loss_contradiction(declared_losses: dict, behavioral_losses: dict) -> dict:
    """
    Compute Pearson correlation between declared and behavioral losses
    """
    categories = list(declared_losses.keys())

    declared_vector = [declared_losses[cat] for cat in categories]
    behavioral_vector = [behavioral_losses[cat] for cat in categories]

    # Pearson correlation
    rho = np.corrcoef(declared_vector, behavioral_vector)[0, 1]

    # Magnitude of gap (Euclidean distance)
    gap_magnitude = np.linalg.norm(np.array(declared_vector) - np.array(behavioral_vector))

    # Epistemic Tremor Score
    tremor_score = abs(rho) * gap_magnitude if rho < 0 else 0

    # Per-category contradictions
    contradictions = []
    for cat in categories:
        if declared_losses[cat] > 0.3 and behavioral_losses[cat] < 0.2:  # High declared, low behavioral
            contradictions.append({
                "category": cat,
                "declared": declared_losses[cat],
                "behavioral": behavioral_losses[cat],
                "gap": declared_losses[cat] - behavioral_losses[cat],
                "severity": "high"
            })

    return {
        "rho": rho,
        "gap_magnitude": gap_magnitude,
        "epistemic_tremor_score": tremor_score,
        "interpretation": interpret_rho(rho),
        "contradictions": contradictions
    }

def interpret_rho(rho: float) -> str:
    if rho > 0.5:
        return "Aligned: Organization walks the talk"
    elif rho > 0:
        return "Somewhat aligned: Moderate consistency"
    elif rho > -0.3:
        return "Disconnected: Actions don't match words"
    else:
        return "CONTRADICTION: Epistemic tremor detected (organizational hypocrisy)"

# Example output:
# {
#     "rho": -0.62,
#     "gap_magnitude": 1.8,
#     "epistemic_tremor_score": 1.12,
#     "interpretation": "CONTRADICTION: Epistemic tremor detected",
#     "contradictions": [
#         {"category": "people_first", "declared": 0.42, "behavioral": 0.15, "gap": 0.27, "severity": "high"},
#         {"category": "work_life_balance", "declared": 0.38, "behavioral": 0.22, "gap": 0.16, "severity": "high"}
#     ]
# }
```

### 1.5 Early Warning Triggers

```python
def check_epistemic_tremor_alert(tremor_score: float, rho: float) -> dict:
    """
    Generate alerts based on tremor score and correlation
    """
    if tremor_score > 0.8 or rho < -0.5:
        return {
            "alert": True,
            "severity": "critical",
            "message": "CRITICAL: Severe epistemic tremor detected. Org declaring values it doesn't practice.",
            "predicted_outcome": "Trust collapse, cynicism surge, attrition spike within 4-8 weeks",
            "recommended_action": "URGENT: Either align behaviors with declarations OR update declared values to match reality"
        }
    elif tremor_score > 0.5 or rho < -0.3:
        return {
            "alert": True,
            "severity": "high",
            "message": "HIGH: Moderate epistemic tremor. Disconnect between stated and actual priorities.",
            "predicted_outcome": "Growing cynicism, disengagement within 8-12 weeks",
            "recommended_action": "Address contradictions in next 30 days"
        }
    else:
        return {"alert": False, "severity": "low", "message": "No significant epistemic tremor"}
```

---

## ENGINE 2: CRITICAL SLOWING DOWN (CSD) / EARLY WARNING SYSTEM

### 2.1 Concept

**Theory:** Before a complex system reaches a tipping point (phase transition), it exhibits "critical slowing down":
1. **Increased Variance** → Fluctuations grow larger
2. **Increased Autocorrelation** → System "remembers" past states longer (loses resilience)
3. **Slowed Recovery** → Takes longer to return to equilibrium after perturbations

**Application to Organizations:**
- **Variance Increase:** Engagement scores swing wildly week-to-week
- **Autocorrelation Increase:** This week's mood strongly predicts next week's (system stuck)
- **Slow Recovery:** After a crisis (product launch failure, reorg), morale takes months to recover (used to be days)

**Prediction Window:** CSD signals appear 4-8 weeks before tipping point (mass attrition, burnout crisis, culture collapse)

### 2.2 Data Requirements

**Time Series Needed:**
- Weekly engagement scores (per employee)
- Weekly sentiment scores (per employee)
- Weekly network density (ONA)
- Weekly response rates to surveys
- Weekly attrition rate

**Minimum History:** 12 weeks (to detect trends)

### 2.3 CSD Detection Algorithms

#### 2.3.1 Variance Calculation (Rolling Window)

```python
def calculate_variance_trend(timeseries: pd.Series, window_size: int = 4) -> pd.Series:
    """
    Calculate rolling variance over time
    Increasing variance = CSD signal
    """
    rolling_variance = timeseries.rolling(window=window_size).var()
    variance_trend = calculate_linear_trend(rolling_variance)  # Slope of variance over time

    return {
        "rolling_variance": rolling_variance.tolist(),
        "variance_trend": variance_trend,  # Positive = increasing variance (CSD warning)
        "csd_signal_variance": variance_trend > 0.1  # Threshold
    }

# Example:
# Week 1-4: Engagement variance = 0.3
# Week 5-8: Engagement variance = 0.6
# Week 9-12: Engagement variance = 1.2
# Trend: INCREASING → CSD WARNING
```

#### 2.3.2 Autocorrelation Calculation (Lag-1)

```python
def calculate_autocorrelation_trend(timeseries: pd.Series, max_lag: int = 1) -> dict:
    """
    Calculate lag-1 autocorrelation over time
    Increasing autocorrelation = CSD signal (system losing resilience)
    """
    autocorr_values = []
    for i in range(len(timeseries) - max_lag):
        window = timeseries[i:i+8]  # 8-week sliding window
        if len(window) >= 4:
            autocorr = window.autocorr(lag=max_lag)
            autocorr_values.append(autocorr)

    autocorr_trend = calculate_linear_trend(autocorr_values)

    return {
        "autocorr_values": autocorr_values,
        "autocorr_trend": autocorr_trend,  # Positive = increasing autocorr (CSD warning)
        "csd_signal_autocorr": autocorr_trend > 0.05  # Threshold
    }

# Example:
# Week 1-8: Autocorr = 0.2 (low memory; resilient)
# Week 5-12: Autocorr = 0.5 (medium memory)
# Week 9-16: Autocorr = 0.8 (high memory; system stuck)
# Trend: INCREASING → CSD WARNING
```

#### 2.3.3 Recovery Rate Calculation

```python
def calculate_recovery_rate(timeseries: pd.Series, perturbation_events: list[dict]) -> dict:
    """
    Measure how long it takes to return to baseline after a perturbation
    """
    recovery_times = []

    for event in perturbation_events:
        event_date = event['date']
        baseline = timeseries[:event_date].mean()
        post_event = timeseries[event_date:]

        # Find when system returns to within 10% of baseline
        recovery_date = None
        for i, value in enumerate(post_event):
            if abs(value - baseline) < 0.1 * baseline:
                recovery_date = i
                break

        if recovery_date:
            recovery_times.append(recovery_date)  # Days to recover
        else:
            recovery_times.append(999)  # Never recovered

    avg_recovery_time = mean(recovery_times) if recovery_times else 0
    recovery_trend = calculate_linear_trend(recovery_times) if len(recovery_times) > 1 else 0

    return {
        "avg_recovery_days": avg_recovery_time,
        "recovery_trend": recovery_trend,  # Positive = slowing recovery (CSD warning)
        "csd_signal_recovery": avg_recovery_time > 30  # Threshold: >30 days to recover
    }

# Example:
# Event 1 (product launch failure): Recovery in 5 days
# Event 2 (reorg announcement): Recovery in 14 days
# Event 3 (layoff rumor): Recovery in 45 days
# Trend: INCREASING → CSD WARNING (system losing resilience)
```

### 2.4 Composite CSD Risk Score

```python
def calculate_csd_risk_score(variance_csd: bool, autocorr_csd: bool, recovery_csd: bool, magnitude: dict) -> dict:
    """
    Combine CSD signals into risk score
    """
    signal_count = sum([variance_csd, autocorr_csd, recovery_csd])

    # Magnitude adjustment
    variance_magnitude = magnitude['variance_trend']
    autocorr_magnitude = magnitude['autocorr_trend']
    recovery_magnitude = magnitude['avg_recovery_days'] / 30  # Normalize

    csd_risk_score = (
        (variance_csd * variance_magnitude * 3) +
        (autocorr_csd * autocorr_magnitude * 3) +
        (recovery_csd * recovery_magnitude * 2)
    ) / 8  # Normalize to 0-1

    risk_level = "critical" if csd_risk_score > 0.7 else "high" if csd_risk_score > 0.5 else "moderate" if csd_risk_score > 0.3 else "low"

    return {
        "csd_risk_score": csd_risk_score,
        "signals_detected": signal_count,
        "risk_level": risk_level,
        "interpretation": interpret_csd(risk_level),
        "time_to_tipping_point_weeks": estimate_time_to_tipping(csd_risk_score)
    }

def interpret_csd(risk_level: str) -> str:
    interpretations = {
        "critical": "IMMINENT: System approaching tipping point. Expect crisis in 4-6 weeks if no intervention.",
        "high": "HIGH RISK: Early warning signals strong. Crisis likely in 6-10 weeks.",
        "moderate": "MODERATE: Some early warning signals. Monitor closely; intervene proactively.",
        "low": "LOW: System resilient. No immediate tipping point risk."
    }
    return interpretations.get(risk_level, "Unknown")

def estimate_time_to_tipping(csd_score: float) -> int:
    """
    Rough estimate: Higher CSD score = closer to tipping point
    """
    if csd_score > 0.8:
        return 4  # 4 weeks
    elif csd_score > 0.6:
        return 6
    elif csd_score > 0.4:
        return 10
    else:
        return 16  # 16+ weeks (safe)
```

### 2.5 CSD-Based Interventions

```python
def generate_csd_interventions(csd_analysis: dict) -> list[dict]:
    """
    Time-sensitive interventions based on CSD signals
    """
    interventions = []

    if csd_analysis['risk_level'] in ["critical", "high"]:
        interventions.append({
            "priority": "urgent",
            "timeframe": "immediate (week 1-2)",
            "action": "Stabilize system: Stop all major changes, reduce demands, inject resources",
            "rationale": "System at tipping point; needs immediate stabilization before any improvements"
        })

        interventions.append({
            "priority": "high",
            "timeframe": "short-term (week 3-4)",
            "action": "Quick wins: Small, visible improvements to rebuild confidence",
            "rationale": "COR Theory: Gain spirals require initial wins; quick recovery builds resilience"
        })

    if csd_analysis['signals_detected'] >= 2:
        interventions.append({
            "priority": "high",
            "timeframe": "week 1-4",
            "action": "Identify and remove top 3 stressors (demands) immediately",
            "rationale": "JD-R Model: Reduce demands when resources are depleted"
        })

    return interventions
```

---

## ENGINE 3: GNN + RICCI CURVATURE (ADVANCED NETWORK ANALYSIS)

### 3.1 Graph Neural Network (GNN) Architecture

**Purpose:** Learn representations of employees in network context; predict who is at risk based on neighborhood properties

**Model:**
```python
import torch
import torch.nn.functional as F
from torch_geometric.nn import GCNConv, global_mean_pool

class OrganizationalGNN(torch.nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim):
        super(OrganizationalGNN, self).__init__()
        self.conv1 = GCNConv(input_dim, hidden_dim)
        self.conv2 = GCNConv(hidden_dim, hidden_dim)
        self.conv3 = GCNConv(hidden_dim, output_dim)

    def forward(self, x, edge_index, edge_weight=None):
        # x: Node features [num_nodes, input_dim]
        # edge_index: Graph connectivity [2, num_edges]
        # edge_weight: Edge strengths [num_edges]

        # Layer 1
        x = self.conv1(x, edge_index, edge_weight)
        x = F.relu(x)
        x = F.dropout(x, p=0.5, training=self.training)

        # Layer 2
        x = self.conv2(x, edge_index, edge_weight)
        x = F.relu(x)
        x = F.dropout(x, p=0.5, training=self.training)

        # Layer 3 (output)
        x = self.conv3(x, edge_index, edge_weight)

        return x  # [num_nodes, output_dim] = risk scores per node

# Input Features (per employee):
# - tenure_months
# - engagement_score
# - sentiment_score
# - burnout_risk_mbi
# - degree_centrality
# - betweenness_centrality
# - closeness_centrality
# - avg_collaboration_sentiment
# - meeting_psych_safety

# Output:
# - attrition_risk_score (0-1)
```

**Training:**
```python
def train_gnn(model, data, labels, epochs=200):
    """
    Train GNN to predict attrition risk from network + individual features
    """
    optimizer = torch.optim.Adam(model.parameters(), lr=0.01, weight_decay=5e-4)
    criterion = torch.nn.BCEWithLogitsLoss()

    for epoch in range(epochs):
        model.train()
        optimizer.zero_grad()

        # Forward pass
        out = model(data.x, data.edge_index, data.edge_attr)

        # Loss (binary classification: will this employee leave in next 90 days?)
        loss = criterion(out.squeeze(), labels)

        # Backward pass
        loss.backward()
        optimizer.step()

        if epoch % 20 == 0:
            print(f"Epoch {epoch}, Loss: {loss.item():.4f}")

    return model

# Advantage of GNN: Captures network effects
# Example: Employee with low engagement BUT strong network ties = lower attrition risk
#          Employee with moderate engagement BUT isolated in network = higher attrition risk
```

### 3.2 Ricci Curvature for Network Health

**Implementation:**
```python
from GraphRicciCurvature.OllivierRicci import OllivierRicci
import networkx as nx

def compute_ricci_curvature_network_health(graph: nx.Graph) -> dict:
    """
    Calculate Ollivier-Ricci curvature for entire network
    Identifies fragile edges (negative curvature)
    """
    # Compute curvature
    orc = OllivierRicci(graph, alpha=0.5, verbose="ERROR")
    orc.compute_ricci_curvature()

    # Extract edge curvatures
    edge_curvatures = []
    for (u, v, data) in orc.G.edges(data=True):
        edge_curvatures.append({
            "employee_1_hash": u,
            "employee_2_hash": v,
            "curvature": data['ricciCurvature'],
            "fragility": "high" if data['ricciCurvature'] < -0.3 else "medium" if data['ricciCurvature'] < 0 else "low"
        })

    # Aggregate metrics
    avg_curvature = np.mean([ec['curvature'] for ec in edge_curvatures])
    fragile_edges = [ec for ec in edge_curvatures if ec['fragility'] == "high"]
    negative_curvature_pct = len([ec for ec in edge_curvatures if ec['curvature'] < 0]) / len(edge_curvatures)

    # Network health score (0-10)
    # Positive avg curvature = healthy (10)
    # Negative avg curvature = decaying (0)
    network_health_score = min(10, max(0, (avg_curvature + 0.5) * 10))

    return {
        "avg_curvature": avg_curvature,
        "network_health_score": network_health_score,
        "fragile_edges_count": len(fragile_edges),
        "fragile_edges": fragile_edges[:10],  # Top 10 most fragile
        "negative_curvature_pct": negative_curvature_pct,
        "interpretation": interpret_ricci_health(network_health_score)
    }

def interpret_ricci_health(score: float) -> str:
    if score > 7:
        return "Healthy: Network has redundant paths, resilient to departures"
    elif score > 5:
        return "Moderate: Some fragile connections, monitor key bridges"
    elif score > 3:
        return "At Risk: Many fragile edges, network vulnerable to fragmentation"
    else:
        return "Critical: Network decaying, high risk of collapse if key nodes leave"
```

### 3.3 Integration: GNN + Ricci Curvature

```python
def integrated_network_risk_assessment(graph, employee_features, ricci_data) -> list[dict]:
    """
    Combine GNN predictions with Ricci curvature analysis
    """
    # 1. GNN: Predict attrition risk per employee
    gnn_predictions = gnn_model(employee_features, graph.edge_index)

    # 2. Ricci: Identify fragile edges
    fragile_edges = ricci_data['fragile_edges']

    # 3. Cross-reference: Employees on fragile edges + high GNN risk = CRITICAL
    critical_risks = []
    for employee_id_hash in graph.nodes:
        gnn_risk = gnn_predictions[employee_id_hash]
        on_fragile_edge = any(fe for fe in fragile_edges if employee_id_hash in [fe['employee_1_hash'], fe['employee_2_hash']])

        if gnn_risk > 0.7 and on_fragile_edge:
            critical_risks.append({
                "employee_id_hash": employee_id_hash,
                "gnn_attrition_risk": gnn_risk,
                "on_fragile_edge": True,
                "impact_if_leaves": "CRITICAL: Network will fragment",
                "recommended_action": "URGENT: Immediate intervention, strengthen surrounding connections"
            })

    return critical_risks
```

---

## CAUSAL ENGINE INTEGRATION & OUTPUT

### 4.1 Unified Causal Analysis

```python
def unified_causal_analysis(org_data: dict) -> dict:
    """
    Run all 3 causal engines and synthesize findings
    """
    # Engine 1: Loss Contradiction
    loss_contradiction = calculate_loss_contradiction(
        declared_losses=org_data['declared_channel'],
        behavioral_losses=org_data['behavioral_channel']
    )

    # Engine 2: CSD/EWS
    csd_analysis = calculate_csd_risk_score(
        variance_csd=org_data['variance_signal'],
        autocorr_csd=org_data['autocorr_signal'],
        recovery_csd=org_data['recovery_signal'],
        magnitude=org_data['magnitude_data']
    )

    # Engine 3: GNN + Ricci
    network_analysis = integrated_network_risk_assessment(
        graph=org_data['network_graph'],
        employee_features=org_data['employee_features'],
        ricci_data=org_data['ricci_analysis']
    )

    # Synthesis
    return {
        "epistemic_tremor": loss_contradiction,
        "critical_slowing_down": csd_analysis,
        "network_fragility": network_analysis,
        "overall_causal_diagnosis": synthesize_diagnosis(loss_contradiction, csd_analysis, network_analysis),
        "predicted_crisis_timeline": estimate_crisis_timeline(loss_contradiction, csd_analysis),
        "root_causes_ranked": rank_root_causes(loss_contradiction, csd_analysis, network_analysis),
        "high_leverage_interventions": generate_interventions(loss_contradiction, csd_analysis, network_analysis)
    }

def synthesize_diagnosis(loss_contra, csd, network) -> str:
    """
    Natural language summary of causal findings
    """
    diagnosis = []

    if loss_contra['epistemic_tremor_score'] > 0.8:
        diagnosis.append("CRITICAL: Severe epistemic tremor (organizational hypocrisy) driving cynicism and distrust.")

    if csd['risk_level'] in ["critical", "high"]:
        diagnosis.append(f"URGENT: System exhibiting critical slowing down. Tipping point in {csd['time_to_tipping_point_weeks']} weeks.")

    if network['fragile_edges_count'] > 10:
        diagnosis.append(f"HIGH RISK: Network fragile with {network['fragile_edges_count']} vulnerable connections. Key departures will cause cascade.")

    return " ".join(diagnosis) if diagnosis else "No critical causal issues detected."
```

### 4.2 Root Cause Ranking

```python
def rank_root_causes(loss_contra, csd, network) -> list[dict]:
    """
    Identify and rank root causes by severity and leverage
    """
    root_causes = []

    # From Loss Contradiction
    for contradiction in loss_contra.get('contradictions', []):
        root_causes.append({
            "type": "epistemic_tremor",
            "category": contradiction['category'],
            "severity": 9 if contradiction['severity'] == "high" else 6,
            "description": f"Contradiction: Org claims '{contradiction['category']}' but behavior shows gap of {contradiction['gap']:.2f}",
            "leverage_point": "Paradigm (Meadows #2)",
            "intervention_urgency": "immediate"
        })

    # From CSD
    if csd['risk_level'] in ["critical", "high"]:
        root_causes.append({
            "type": "systemic_instability",
            "category": "tipping_point_approach",
            "severity": 10 if csd['risk_level'] == "critical" else 8,
            "description": f"System losing resilience. CSD signals: variance↑, autocorr↑, recovery↓. Crisis in {csd['time_to_tipping_point_weeks']} weeks.",
            "leverage_point": "Buffers (Meadows #11) + Feedback Loops (Meadows #7)",
            "intervention_urgency": "urgent (1-2 weeks)"
        })

    # From Network
    if network['network_health_score'] < 5:
        root_causes.append({
            "type": "network_fragility",
            "category": "structural_vulnerability",
            "severity": 8,
            "description": f"Network health {network['network_health_score']:.1f}/10. {network['fragile_edges_count']} fragile edges. Risk of cascade if key nodes depart.",
            "leverage_point": "Structure (Meadows #10)",
            "intervention_urgency": "high (2-4 weeks)"
        })

    # Sort by severity
    root_causes.sort(key=lambda x: x['severity'], reverse=True)
    return root_causes
```

---

## NEXT STEPS

**Completed:**
✅ Loss Contradiction Engine
✅ CSD/EWS Engine
✅ GNN + Ricci Curvature Engine
✅ Causal integration framework

**Next (Tower 4):**
- Build Unified Brain (MCDM)
- Implement Credit Assignment
- Design Hub-and-Spoke architecture
- Create intervention generation system

---

**Document Version:** 1.0
**Status:** Tower 3 Causal Engines Complete
**Last Updated:** 2025-11-27
