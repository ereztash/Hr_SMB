# TOWER 2: SYMPTOM ENGINES
## Real-Time Detection of Organizational Health Signals

---

## OVERVIEW

**Purpose:** Continuously monitor organizational "vital signs" through automated analysis of communication patterns, sentiment, network topology, and meeting dynamics.

**Core Engines:**
1. **NLP Sentiment Analyzer** → Detects emotional state from text
2. **ONA (Organizational Network Analysis)** → Maps collaboration patterns, identifies fragility
3. **Meeting Analyzer** → Assesses psychological safety, participation equity, dynamics

**Outputs:** Real-time symptom scores that feed into Tower 3 (Causal Engines) and Tower 4 (Unified Brain)

---

## ENGINE 1: NLP SENTIMENT ANALYZER

### 1.1 Architecture

```
Input: Email/Slack/Survey text → Preprocessing → Models → Sentiment Score → Output
```

**Data Sources:**
- Email (Gmail API, Outlook Graph API)
- Slack/Teams messages (Workspace API)
- Survey open-ended responses
- Stay/Exit interview transcripts

**Privacy:** Content NOT stored; only metadata + sentiment scores retained.

### 1.2 NLP Pipeline

```python
class SentimentAnalyzer:
    def __init__(self):
        self.tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")
        self.model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased-finetuned-sst-2")
        self.emotion_model = pipeline("text-classification", model="j-hartmann/emotion-english-distilroberta-base")

    def analyze(self, text: str, context: dict) -> dict:
        # Step 1: Preprocess
        cleaned_text = self.preprocess(text)

        # Step 2: Sentiment (positive/negative)
        sentiment_score = self.predict_sentiment(cleaned_text)  # -1 to +1

        # Step 3: Emotions (joy, sadness, anger, fear, surprise, disgust)
        emotions = self.predict_emotions(cleaned_text)

        # Step 4: Burnout-specific signals
        burnout_signals = self.detect_burnout_language(cleaned_text)

        # Step 5: Context adjustment
        adjusted_score = self.adjust_for_context(sentiment_score, context)

        return {
            "sentiment_score": adjusted_score,
            "emotions": emotions,
            "burnout_signals": burnout_signals,
            "confidence": self.confidence_score(cleaned_text)
        }

    def detect_burnout_language(self, text: str) -> dict:
        """Detect linguistic markers of burnout (MBI dimensions)"""
        exhaustion_keywords = ["exhausted", "burned out", "drained", "overwhelmed", "can't keep up"]
        cynicism_keywords = ["don't care", "pointless", "waste of time", "why bother", "doesn't matter"]
        efficacy_keywords = ["can't do anything", "failing", "incompetent", "not good enough"]

        return {
            "exhaustion_score": self.keyword_density(text, exhaustion_keywords),
            "cynicism_score": self.keyword_density(text, cynicism_keywords),
            "reduced_efficacy_score": self.keyword_density(text, efficacy_keywords)
        }

    def adjust_for_context(self, sentiment_score: float, context: dict) -> float:
        """Adjust sentiment based on organizational context"""
        # Example: After-hours emails more likely to indicate stress
        if context.get("is_after_hours"):
            sentiment_score *= 0.9  # Slight negative adjustment

        # Frequent short emails = potential stress/overwhelm
        if context.get("email_frequency_1h") > 10:
            sentiment_score *= 0.95

        return sentiment_score
```

### 1.3 Multi-Modal Sentiment

**Text + Metadata:**
```python
def comprehensive_sentiment(email_event: dict) -> dict:
    text_sentiment = analyze_text(email_event['text_snippet'])
    metadata_signals = {
        "time_of_day_stress": calculate_time_stress(email_event['timestamp']),
        "response_latency": email_event['time_to_respond_hours'],  # Delayed responses = potential avoidance
        "email_length": len(email_event['text_snippet'].split()),  # Very short = terse = potential frustration
        "exclamation_marks": email_event['text_snippet'].count('!'),
        "all_caps_words": count_all_caps(email_event['text_snippet'])
    }

    combined_score = (text_sentiment * 0.7) + (metadata_sentiment(metadata_signals) * 0.3)
    return combined_score
```

### 1.4 Aggregation & Trending

```python
# Individual Employee Sentiment Trend
SELECT
    employee_id_hash,
    date_trunc('day', timestamp) AS day,
    AVG(sentiment_score) AS avg_sentiment,
    STDDEV(sentiment_score) AS sentiment_variance,
    COUNT(*) AS message_count
FROM communication_events
WHERE employee_id_hash = 'a7f3c2...'
  AND timestamp > NOW() - INTERVAL '30 days'
GROUP BY employee_id_hash, day
ORDER BY day;

# Department-Level Sentiment
SELECT
    department_id_hash,
    AVG(sentiment_score) AS dept_sentiment,
    PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY sentiment_score) AS bottom_10_pct,  # At-risk employees
    COUNT(DISTINCT employee_id_hash) AS employee_count
FROM communication_events
WHERE timestamp > NOW() - INTERVAL '7 days'
GROUP BY department_id_hash;
```

### 1.5 Burnout Signal Detection

**MBI-Based NLP Classifier:**
```python
class BurnoutDetector:
    def __init__(self):
        # Fine-tuned model on labeled burnout text data
        self.model = load_model("models/mbi_burnout_classifier.pkl")

    def predict_mbi_dimensions(self, text_history: list[str]) -> dict:
        """
        Predict MBI dimensions from 30-day communication history
        Returns: {
            "emotional_exhaustion": 0-10,
            "cynicism": 0-10,
            "reduced_efficacy": 0-10,
            "overall_burnout_risk": 0-10
        }
        """
        features = self.extract_features(text_history)
        predictions = self.model.predict(features)
        return {
            "emotional_exhaustion": predictions['EE'],
            "cynicism": predictions['CY'],
            "reduced_efficacy": predictions['rPE'],
            "overall_burnout_risk": (predictions['EE'] + predictions['CY'] + predictions['rPE']) / 3
        }

    def extract_features(self, texts: list[str]) -> dict:
        return {
            "avg_sentiment": mean([analyze_sentiment(t) for t in texts]),
            "sentiment_variance": variance([analyze_sentiment(t) for t in texts]),
            "burnout_keyword_frequency": sum([count_burnout_keywords(t) for t in texts]) / len(texts),
            "message_length_trend": calculate_trend([len(t.split()) for t in texts]),  # Decreasing = withdrawal
            "response_time_trend": calculate_response_latency_trend(texts),
            "after_hours_percentage": sum([is_after_hours(t) for t in texts]) / len(texts)
        }
```

---

## ENGINE 2: ORGANIZATIONAL NETWORK ANALYSIS (ONA)

### 2.1 Network Construction

**Data Sources:**
- Email (who emails whom, frequency, sentiment)
- Slack/Teams (messages, reactions, mentions)
- Calendar (meeting invitations, attendance)
- HR data (reporting lines, team membership)

**Graph Schema (Neo4j):**
```cypher
// Nodes
CREATE (e:Employee {
    id_hash: "a7f3c2...",
    role_category: "engineering",
    role_level: "senior_ic",
    department_id_hash: "d4e7a1...",
    tenure_months: 24,
    engagement_score: 7.2
})

// Edges
CREATE (e1:Employee)-[:REPORTS_TO]->(e2:Employee)

CREATE (e1:Employee)-[:COLLABORATES_WITH {
    frequency: 15,  // interactions per week
    avg_sentiment: 0.6,
    last_interaction: timestamp,
    strength: 0.8  // 0-1 scale
}]->(e2:Employee)

CREATE (e1:Employee)-[:TRUSTS {
    strength: 0.7,  // Inferred from survey responses
    reciprocal: true
}]->(e2:Employee)
```

### 2.2 Network Metrics Calculated

#### 2.2.1 Centrality Measures

**Degree Centrality:**
```cypher
// Number of direct connections
MATCH (e:Employee)-[r:COLLABORATES_WITH]-(other:Employee)
RETURN e.id_hash, COUNT(r) AS degree_centrality
ORDER BY degree_centrality DESC;
```

**Betweenness Centrality (Bridges):**
```cypher
// Employees who connect otherwise disconnected groups
CALL gds.betweenness.stream({
    nodeProjection: 'Employee',
    relationshipProjection: 'COLLABORATES_WITH'
})
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).id_hash AS employee, score AS betweenness
ORDER BY score DESC
LIMIT 20;

// High betweenness = critical bridge; if they leave, network fragments
```

**Closeness Centrality:**
```cypher
// How quickly can information reach this employee from anyone else?
CALL gds.alpha.closeness.stream({
    nodeProjection: 'Employee',
    relationshipProjection: 'COLLABORATES_WITH'
})
YIELD nodeId, centrality
RETURN gds.util.asNode(nodeId).id_hash, centrality
ORDER BY centrality DESC;
```

#### 2.2.2 Community Detection

**Louvain Algorithm (Detect silos/clusters):**
```cypher
CALL gds.louvain.stream({
    nodeProjection: 'Employee',
    relationshipProjection: {
        COLLABORATES_WITH: {
            properties: 'strength'
        }
    },
    relationshipWeightProperty: 'strength'
})
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).id_hash AS employee,
       communityId,
       gds.util.asNode(nodeId).department_id_hash AS department
ORDER BY communityId;

// Healthy: Communities align with departments
// Unhealthy: Communities cut across departments (silos forming)
```

#### 2.2.3 Network Fragility (Ricci Curvature)

**Concept:** Ricci Curvature measures how "curved" edges are in a network.
- **Positive curvature:** Well-connected, redundant paths (resilient)
- **Negative curvature:** Bottlenecks, single points of failure (fragile)

**Implementation:**
```python
import networkx as nx
from GraphRicciCurvature.OllivierRicci import OllivierRicci

def calculate_ricci_curvature(graph: nx.Graph) -> dict:
    """
    Calculate Ollivier-Ricci curvature for each edge
    Negative curvature = fragile connection
    """
    orc = OllivierRicci(graph, alpha=0.5, verbose="INFO")
    orc.compute_ricci_curvature()

    fragile_edges = []
    for (u, v), curvature in orc.G.edges.items():
        if curvature['ricciCurvature'] < -0.3:  # Threshold
            fragile_edges.append({
                'employee_1': u,
                'employee_2': v,
                'curvature': curvature['ricciCurvature'],
                'risk': 'high'  # If this connection breaks, network fragments
            })

    return {
        'fragile_edge_count': len(fragile_edges),
        'fragile_edges': fragile_edges,
        'avg_curvature': mean([c['ricciCurvature'] for _, _, c in orc.G.edges(data=True)])
    }

# Alert if avg_curvature < -0.2 → Network decay in progress
```

#### 2.2.4 Trust Erosion Detection

**Method:** Compare trust network at T1 vs. T2
```cypher
// Trust network density over time
MATCH (e1:Employee)-[t:TRUSTS]->(e2:Employee)
WHERE t.timestamp > $start_date AND t.timestamp < $end_date
RETURN
    date_trunc('week', t.timestamp) AS week,
    COUNT(t) AS trust_edges,
    AVG(t.strength) AS avg_trust_strength
ORDER BY week;

// Declining trust_edges or avg_trust_strength = Epistemic tremor signal
```

### 2.3 Bridge Burnout Detection

**Problem:** High-betweenness employees (bridges) are vulnerable to overload.

**Detection Algorithm:**
```python
def detect_bridge_burnout_risk(graph, betweenness_scores, engagement_scores):
    """
    Employees with high betweenness + low engagement = HIGH RISK
    """
    at_risk = []
    for employee_id_hash in betweenness_scores:
        if betweenness_scores[employee_id_hash] > 0.1:  # Top 10% betweenness
            if engagement_scores[employee_id_hash] < 5:  # Low engagement
                at_risk.append({
                    'employee_id_hash': employee_id_hash,
                    'betweenness': betweenness_scores[employee_id_hash],
                    'engagement': engagement_scores[employee_id_hash],
                    'risk_level': 'critical',  # Bridge + burned out = org-wide impact if they leave
                    'recommended_action': 'Immediate intervention: reduce load, provide support'
                })
    return at_risk
```

### 2.4 Network Health Score

```python
def calculate_network_health(graph_metrics: dict) -> float:
    """
    Composite network health score (0-10)
    """
    # 1. Connectivity: Higher density = better
    density_score = min(10, graph_metrics['density'] * 20)  # Normalize

    # 2. Fragility: Lower Ricci curvature = worse
    ricci_score = 10 if graph_metrics['avg_ricci_curvature'] > 0 else max(0, 10 + (graph_metrics['avg_ricci_curvature'] * 10))

    # 3. Bridge overload: Fewer overloaded bridges = better
    bridge_score = 10 - (graph_metrics['overloaded_bridges_count'] * 2)

    # 4. Siloing: More cross-functional edges = better
    cross_functional_ratio = graph_metrics['cross_dept_edges'] / graph_metrics['total_edges']
    silo_score = cross_functional_ratio * 10

    network_health = (density_score * 0.2 + ricci_score * 0.3 + bridge_score * 0.3 + silo_score * 0.2)
    return max(0, min(10, network_health))
```

---

## ENGINE 3: MEETING ANALYZER

### 3.1 Data Sources

**Meeting Metadata:**
- Calendar events (duration, attendees, frequency)
- Zoom/Teams transcripts (if consent given)
- Meeting recordings (audio analysis for tone, interruptions)

**Privacy:** Transcripts processed in real-time, not stored. Only aggregated metrics retained.

### 3.2 Meeting Health Metrics

#### 3.2.1 Participation Equity (Voice Distribution)

**Gini Coefficient of Speaking Time:**
```python
def calculate_voice_equity(speaking_times: list[float]) -> float:
    """
    Gini coefficient: 0 = perfect equality, 1 = one person dominates
    """
    n = len(speaking_times)
    sorted_times = sorted(speaking_times)
    cumulative = np.cumsum(sorted_times)
    gini = (2 * sum((i + 1) * time for i, time in enumerate(sorted_times))) / (n * sum(sorted_times)) - (n + 1) / n
    return gini

# Gini < 0.3 = Good (equitable participation)
# Gini > 0.6 = Poor (one person dominates; others silent)
```

**Participation Rate:**
```python
def participation_rate(attendee_count: int, speakers_count: int) -> float:
    """
    % of attendees who spoke
    """
    return speakers_count / attendee_count

# < 50% = Low psychological safety (people afraid to speak)
# > 80% = High psychological safety
```

#### 3.2.2 Interruption Analysis

```python
def analyze_interruptions(transcript: list[dict]) -> dict:
    """
    Detect interruptions (speaker changes mid-sentence)
    """
    interruptions = 0
    for i in range(len(transcript) - 1):
        current_speaker = transcript[i]['speaker']
        next_speaker = transcript[i + 1]['speaker']
        current_text = transcript[i]['text']

        # Heuristic: If sentence incomplete (no period) and speaker changes = interruption
        if not current_text.strip().endswith(('.', '!', '?')) and current_speaker != next_speaker:
            interruptions += 1

    return {
        'total_interruptions': interruptions,
        'interruptions_per_minute': interruptions / (transcript[-1]['timestamp'] - transcript[0]['timestamp']) * 60,
        'most_interrupted': find_most_interrupted(transcript),
        'most_interrupter': find_most_interrupter(transcript)
    }

# High interruptions = Low psychological safety
```

#### 3.2.3 Sentiment During Meetings

```python
def meeting_sentiment_analysis(transcript: list[dict]) -> dict:
    """
    Track sentiment evolution during meeting
    """
    sentiments = [analyze_sentiment(turn['text']) for turn in transcript]

    return {
        'avg_sentiment': mean(sentiments),
        'sentiment_trend': calculate_trend(sentiments),  # Improving, stable, declining
        'negative_sentiment_percentage': sum(1 for s in sentiments if s < -0.2) / len(sentiments),
        'sentiment_variance': variance(sentiments)  # High variance = tension/conflict
    }
```

#### 3.2.4 Psychological Safety Proxy Score

```python
def calculate_psych_safety_score(meeting_metrics: dict) -> float:
    """
    Composite psychological safety score from meeting dynamics (0-10)
    """
    # 1. Participation equity (Gini coefficient)
    equity_score = 10 * (1 - meeting_metrics['gini_coefficient'])

    # 2. Participation rate
    participation_score = meeting_metrics['participation_rate'] * 10

    # 3. Interruptions (inverse)
    interruption_score = max(0, 10 - meeting_metrics['interruptions_per_minute'] * 2)

    # 4. Sentiment
    sentiment_score = (meeting_metrics['avg_sentiment'] + 1) * 5  # Normalize -1 to +1 → 0 to 10

    psych_safety = (equity_score * 0.3 + participation_score * 0.3 + interruption_score * 0.2 + sentiment_score * 0.2)
    return max(0, min(10, psych_safety))
```

### 3.3 Meeting Fatigue Detection

```python
def detect_meeting_fatigue(calendar_events: list[dict]) -> dict:
    """
    Excessive meetings = burnout risk
    """
    total_meeting_hours_per_week = sum(event['duration_hours'] for event in calendar_events if event['week'] == current_week())
    back_to_back_meetings = count_back_to_back(calendar_events)
    meetings_over_1_hour = sum(1 for event in calendar_events if event['duration_hours'] > 1)

    return {
        'total_meeting_hours_weekly': total_meeting_hours_per_week,
        'back_to_back_count': back_to_back_meetings,
        'long_meetings_count': meetings_over_1_hour,
        'fatigue_risk': 'high' if total_meeting_hours_per_week > 25 else 'moderate' if > 15 else 'low'
    }

# Alert if total_meeting_hours_weekly > 25 (research shows productivity declines)
```

---

## INTEGRATION & OUTPUT

### 4.1 Symptom Score Aggregation

**Per Employee:**
```json
{
  "employee_id_hash": "a7f3c2...",
  "timestamp": "2025-11-27T12:00:00Z",
  "symptoms": {
    "sentiment_score": -0.3,  // NLP Engine
    "burnout_risk_mbi": 6.8,  // NLP Burnout Detector
    "network_centrality": {
      "degree": 12,
      "betweenness": 0.15,  // High = bridge
      "closeness": 0.6
    },
    "network_fragility_at_risk": true,  // ONA Engine (if this person leaves, network fragments)
    "meeting_psych_safety_avg": 4.2,  // Meeting Analyzer (low = fear to speak)
    "meeting_hours_weekly": 28,  // Meeting Fatigue
    "overall_symptom_severity": 7.1  // Composite (0-10)
  }
}
```

**Per Department:**
```json
{
  "department_id_hash": "d4e7a1...",
  "timestamp": "2025-11-27T12:00:00Z",
  "symptoms": {
    "avg_sentiment": 0.2,
    "sentiment_variance": 0.4,  // High variance = mixed moods, potential conflict
    "network_health_score": 5.8,
    "fragile_edges_count": 8,
    "overloaded_bridges_count": 3,
    "avg_psych_safety": 6.1,
    "silo_score": 0.4  // Low = siloed
  }
}
```

### 4.2 Real-Time Alerts

**Alert Triggers:**
```python
def generate_alerts(symptoms: dict) -> list[dict]:
    alerts = []

    # 1. Individual at high risk
    if symptoms['burnout_risk_mbi'] > 7 and symptoms['sentiment_score'] < -0.4:
        alerts.append({
            'type': 'individual_burnout_risk',
            'severity': 'high',
            'employee_id_hash': symptoms['employee_id_hash'],
            'message': 'Employee showing high burnout risk (MBI > 7) + negative sentiment',
            'recommended_action': 'Schedule stay interview, reduce workload'
        })

    # 2. Critical bridge at risk
    if symptoms['network_centrality']['betweenness'] > 0.1 and symptoms['burnout_risk_mbi'] > 6:
        alerts.append({
            'type': 'bridge_burnout',
            'severity': 'critical',
            'employee_id_hash': symptoms['employee_id_hash'],
            'message': 'Critical bridge employee at burnout risk; departure would fragment network',
            'recommended_action': 'URGENT: Redistribute load, provide immediate support'
        })

    # 3. Meeting psychological safety collapse
    if symptoms['meeting_psych_safety_avg'] < 4:
        alerts.append({
            'type': 'psych_safety_low',
            'severity': 'medium',
            'department_id_hash': symptoms.get('department_id_hash'),
            'message': 'Psychological safety in meetings critically low',
            'recommended_action': 'Facilitate psychological safety workshop, address power dynamics'
        })

    return alerts
```

### 4.3 Dashboard Visualization

**Key Visualizations:**
1. **Sentiment Heatmap:** Department × Time
2. **Network Graph:** Interactive ONA with node size = betweenness, color = burnout risk
3. **Meeting Health Trends:** Psych safety score over time
4. **Bridge Risk Matrix:** Betweenness × Burnout risk (quadrant chart)

---

## VALIDATION & ACCURACY

### 5.1 NLP Model Performance

**Target Metrics:**
- Sentiment accuracy: 85%+ (vs. human-labeled ground truth)
- Burnout detection recall: 90%+ (minimize false negatives)
- Burnout detection precision: 75%+ (acceptable false positive rate)

**Validation Strategy:**
- Cross-validate on labeled dataset (MBI survey responses + text samples)
- Continuous learning: Update model based on feedback loop

### 5.2 ONA Validation

**Ground Truth:**
- Survey question: "Who do you collaborate with most frequently?" (compare to detected network)
- Exit interview: "Did you feel like a critical bridge?" (validate betweenness scores)

**Accuracy Target:** 80%+ edge detection accuracy

### 5.3 Meeting Analyzer Validation

**Ground Truth:**
- Post-meeting survey: "Did you feel safe speaking up?" (compare to psych safety score)
- Manual transcript review: Validate interruption detection

**Accuracy Target:** 85%+ correlation with self-reported psychological safety

---

## NEXT STEPS

**Completed:**
✅ Tower 2 architecture designed
✅ NLP sentiment analyzer specified
✅ ONA module with Ricci Curvature defined
✅ Meeting analyzer with psych safety proxy created

**Next (Tower 3):**
- Build Loss Contradiction engine
- Implement Critical Slowing Down (CSD) detector
- Integrate Ricci Curvature GNN analysis

---

**Document Version:** 1.0
**Status:** Tower 2 Symptom Engines Complete — Ready for Implementation
**Last Updated:** 2025-11-27
