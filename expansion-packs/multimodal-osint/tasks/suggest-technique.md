<!-- Powered by BMAD™ Multimodal OSINT Expansion Pack -->

# suggest-technique

Suggest next DISARM technique to hunt based on data characteristics and prior results.

## Purpose

Implement adaptive technique selection using explore-exploit strategy. Suggests promising DISARM techniques based on dataset characteristics, multimodal capabilities, and hunting history.

## Inputs

```yaml
required:
  - data_info: '{data_exploration_results}' # From init-data.md
  - prior_results: [] # Previous hunt results
  - techniques_tried: [] # Already investigated technique IDs

optional:
  - strategy: 'balanced' # 'explore', 'exploit', or 'balanced'
  - iteration: 1 # Current iteration number
  - platform: null # Platform context
```

## Process

### Step 1: Load DISARM Framework

```python
# Read DISARM techniques from reference
# Load all techniques with their characteristics

disarm_techniques = load_all_disarm_techniques()

print(f"📚 Loaded {len(disarm_techniques)} DISARM techniques")

# Filter out already tried
available_techniques = [t for t in disarm_techniques if t['id'] not in techniques_tried]

print(f"🎯 Available to try: {len(available_techniques)} techniques")
```

### Step 2: Score Techniques by Data Fit

```python
def score_technique_fit(technique, data_info):
    """Score how well technique matches dataset characteristics"""

    score = 0.0

    # Multimodal techniques require images
    multimodal_techniques = ['T0021', 'T0086', 'T0044']  # Memes, Narratives, Seed Kernel

    if technique['id'] in multimodal_techniques:
        if data_info['image_availability'] > 0.3:
            score += 5.0  # Strong fit
        elif data_info['image_availability'] > 0:
            score += 2.0  # Moderate fit
        else:
            score -= 3.0  # Poor fit (no images)

    # Text-heavy techniques
    text_techniques = ['T0001', 'T0022', 'T0023']  # 5Ds, Conspiracy, Distort Facts

    if technique['id'] in text_techniques:
        score += 3.0  # Always relevant for text

    # Platform-specific techniques
    if data_info.get('platform') == 'telegram':
        if technique['id'] in ['T0104.003', 'T0057']:  # Private networks, Events
            score += 2.0

    # Execution phase techniques more likely to be visible
    if technique['phase'] == 'Execute':
        score += 1.0

    return score

# Score all available techniques
technique_scores = []
for tech in available_techniques:
    score = score_technique_fit(tech, data_info)
    technique_scores.append({
        'technique': tech,
        'score': score
    })

technique_scores.sort(key=lambda x: x['score'], reverse=True)
```

### Step 3: Apply Explore-Exploit Strategy

```python
def select_by_strategy(technique_scores, prior_results, strategy, iteration):
    """Select technique based on explore-exploit strategy"""

    # Count prior successes
    prior_passes = [r for r in prior_results if r['status'] == 'PASS']

    if strategy == 'explore':
        # Prioritize techniques from different tactics
        # Try to diversify investigation
        print(f"🔭 Explore strategy: trying new areas")

        # Group by tactic, pick from least-explored tactic
        tried_tactics = set(r['technique']['tactic'] for r in prior_results if 'technique' in r)

        for scored in technique_scores:
            if scored['technique']['tactic'] not in tried_tactics:
                return scored['technique']

        # Fallback to highest scored
        return technique_scores[0]['technique']

    elif strategy == 'exploit':
        # Dig deeper into successful findings
        # If found evidence, try related techniques
        print(f"💡 Exploit strategy: digging deeper into findings")

        if prior_passes:
            # Get tactics from successful techniques
            successful_tactics = set(r['technique']['tactic'] for r in prior_passes if 'technique' in r)

            # Try another technique from same tactic
            for scored in technique_scores:
                if scored['technique']['tactic'] in successful_tactics:
                    return scored['technique']

        # Fallback if no prior successes
        return technique_scores[0]['technique']

    else:  # balanced
        # Mix of exploration and exploitation
        print(f"⚖️  Balanced strategy: mixing exploration and exploitation")

        # First few iterations: explore
        if iteration <= 2:
            # Explore: prioritize diverse tactics
            tried_tactics = set(r['technique']['tactic'] for r in prior_results if 'technique' in r)

            for scored in technique_scores:
                if scored['technique']['tactic'] not in tried_tactics:
                    return scored['technique']

        # Later iterations: exploit if found something
        elif prior_passes:
            # Exploit: related to successes
            successful_tactics = set(r['technique']['tactic'] for r in prior_passes if 'technique' in r)

            for scored in technique_scores:
                if scored['technique']['tactic'] in successful_tactics:
                    return scored['technique']

        # Default: highest scored
        return technique_scores[0]['technique']

selected_technique = select_by_strategy(technique_scores, prior_results, strategy, iteration)

print(f"\n✨ Selected: {selected_technique['id']} - {selected_technique['name']}")
```

### Step 4: Generate Selection Reasoning

```python
def generate_reasoning(selected_technique, data_info, strategy, prior_results):
    """Explain why this technique was selected"""

    reasoning_parts = []

    # Data fit reasoning
    if selected_technique['id'] in ['T0021', 'T0086'] and data_info['image_availability'] > 0.3:
        reasoning_parts.append(f"High image availability ({data_info['image_availability']:.1%}) suits multimodal analysis")

    # Strategy reasoning
    if strategy == 'explore':
        reasoning_parts.append("Exploring new tactic area to diversify investigation")
    elif strategy == 'exploit':
        reasoning_parts.append("Building on prior successful findings")
    else:
        prior_passes = [r for r in prior_results if r['status'] == 'PASS']
        if prior_passes:
            reasoning_parts.append("Investigating related technique based on prior successes")
        else:
            reasoning_parts.append("Prioritizing technique with strongest data fit")

    # Technique characteristics
    reasoning_parts.append(f"{selected_technique['phase']} phase technique with observable patterns")

    reasoning = ". ".join(reasoning_parts) + "."

    return reasoning

reasoning = generate_reasoning(selected_technique, data_info, strategy, prior_results)
```

### Step 5: Return Suggestion

```python
suggestion = {
    'technique_id': selected_technique['id'],
    'technique_name': selected_technique['name'],
    'technique': selected_technique,
    'reasoning': reasoning,
    'strategy_used': strategy,
    'iteration': iteration,
    'alternatives': [ts['technique']['id'] for ts in technique_scores[1:4]]  # Top 3 alternatives
}

return suggestion
```

## Helper: Load DISARM Techniques

```python
def load_all_disarm_techniques():
    """Load DISARM techniques from framework reference"""

    # Parse disarm-framework-reference.md
    # Extract all techniques with metadata

    # Simplified example:
    techniques = [
        {
            'id': 'T0001',
            'name': '5Ds (Dismiss, Distort, Distract, Dismay, Divide)',
            'phase': 'Execute',
            'tactic': 'Deliver Content',
            'description': 'Use 5D tactics...'
        },
        {
            'id': 'T0021',
            'name': 'Create Memes',
            'phase': 'Execute',
            'tactic': 'Deliver Content',
            'description': 'Create and distribute memes...'
        },
        # ... more techniques
    ]

    return techniques
```

## Output

```python
{
    'technique_id': 'T0021',
    'technique_name': 'Create Memes',
    'technique': {
        'id': 'T0021',
        'name': 'Create Memes',
        'phase': 'Execute',
        'tactic': 'Deliver Content',
        ...
    },
    'reasoning': 'High image availability (68%) suits multimodal analysis. Prioritizing technique with strongest data fit. Execute phase technique with observable patterns.',
    'strategy_used': 'balanced',
    'iteration': 1,
    'alternatives': ['T0086', 'T0044', 'T0001']
}
```

## Integration

Called by [execute-hunt.md](execute-hunt.md) for each iteration to select next technique to hunt.
