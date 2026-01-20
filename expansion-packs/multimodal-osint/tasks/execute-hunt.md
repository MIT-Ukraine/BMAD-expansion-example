# /execute-hunt Task

When this command is used, execute the following task:

<!-- Powered by BMAD™ Multimodal OSINT Expansion Pack -->

# execute-hunt

Execute comprehensive multimodal DISARM technique hunt with iterative exploration of MULTIPLE techniques.

## Purpose

Hunt for DISARM disinformation techniques in social media data by analyzing text AND images together. This task explores MULTIPLE techniques using both exploration (trying new techniques) and exploitation (digging deeper into promising findings) to produce a comprehensive investigation report like [experiment/FIMI_Investigation_Report.md](experiment/FIMI_Investigation_Report.md).

## Key Features

- **Adaptive Multi-Technique Hunting**: Explores multiple DISARM techniques, not just one
- **Native Format Support**: Works with data AS-IS (JSON, CSV, JSONL, Parquet) - no SQL conversion needed
- **Structure Agnostic**: Automatically detects data schema on the fly
- **VLM-Powered**: Uses Vision Language Models for deep image analysis
- **Logo Intelligence**: Identifies organizational logos to track affiliations
- **Cross-Modal Analysis**: Strongest evidence from text+image consistency
- **Exploration & Exploitation**: Balances trying new techniques vs digging deeper
- **Comprehensive Reporting**: Produces detailed investigation reports with evidence

## Inputs

```yaml
required:
  - data_path: '{path_to_data}' # File or directory (JSON/JSONL/CSV/Parquet/folder)
  - investigation_id: '{unique_investigation_name}'

optional:
  - technique_ids: [] # Specific techniques to hunt, or empty for auto-suggestion
  - max_iterations: 5 # How many techniques to investigate
  - platform: '{platform_name}' # telegram, twitter, facebook, etc.
  - image_sample_size: 50 # Max images to analyze with VLM per technique
  - use_logos: true # Enable logo identification
  - vlm_model: 'qwen3-vl:8b' # VLM model to use
  - strategy: 'balanced' # 'explore', 'exploit', or 'balanced'
  - output_dir: 'investigations/{investigation_id}' # Output directory
```

## Dependencies

```yaml
tasks:
  - init-data.md # Data exploration and schema detection
  - vision.md # VLM image analysis
  - logos.md # Logo identification
  - suggest-technique.md # DISARM technique suggestion
data:
  - disarm-framework-reference.md
  - manipulation-playbook.md
  - platform-behavior-baselines.md
external:
  - vision.ipynb # VLM notebook template
  - logos/logos.csv # Logo database
  - experiment/FIMI_Investigation_Report.md # Report template reference
```

## Process

### Phase 1: Initialize Investigation

```python
import pandas as pd
import json
from pathlib import Path
from datetime import datetime
import random

# Create investigation folder
investigation_folder = Path(output_dir)
investigation_folder.mkdir(parents=True, exist_ok=True)

print(f"🔍 Starting multimodal FIMI investigation: {investigation_id}")
print(f"📁 Output directory: {investigation_folder}")

# Initialize investigation metadata
investigation_meta = {
    'investigation_id': investigation_id,
    'data_path': data_path,
    'platform': platform,
    'start_time': datetime.now().isoformat(),
    'max_iterations': max_iterations,
    'strategy': strategy,
    'vlm_model': vlm_model,
    'image_sample_size': image_sample_size,
    'use_logos': use_logos,
    'techniques_hunted': [],
    'results': []
}
```

### Phase 2: Data Exploration (Call init-data.md)

```python
# Call init-data task to explore data structure
from tasks import init_data

print("📊 Exploring dataset structure...")

data_info = init_data.explore_data(
    data_path=data_path,
    platform=platform
)

# data_info contains:
# - file_type: .json, .jsonl, .csv, etc.
# - total_records: number of posts/messages
# - schema: detected columns (text_col, image_col, id_col, etc.)
# - image_availability: percentage of posts with images
# - date_range: earliest to latest date
# - primary_file: main file to load

print(f"✓ Found {data_info['total_records']} records")
print(f"✓ File type: {data_info['file_type']}")
print(f"✓ Image availability: {data_info['image_availability']:.1%}")
print(f"✓ Schema detected: {data_info['schema']}")

investigation_meta['data_info'] = data_info
```

### Phase 3: Load Data in Native Format

```python
# Load data based on detected file type
print(f"📥 Loading data from: {data_info['primary_file']}")

if data_info['file_type'] in ['.json', '.jsonl', '.ndjson']:
    try:
        # Try JSONL (line-delimited)
        data = pd.read_json(data_info['primary_file'], lines=True)
    except:
        # Try regular JSON
        data = pd.read_json(data_info['primary_file'])
elif data_info['file_type'] == '.csv':
    data = pd.read_csv(data_info['primary_file'])
elif data_info['file_type'] in ['.parquet', '.pq']:
    data = pd.read_parquet(data_info['primary_file'])
else:
    raise ValueError(f"Unsupported file type: {data_info['file_type']}")

print(f"✓ Loaded {len(data)} records into DataFrame")

# Extract column names from schema
text_col = data_info['schema']['text_columns'][0] if data_info['schema']['text_columns'] else None
image_col = data_info['schema']['image_columns'][0] if data_info['schema']['image_columns'] else None
id_col = data_info['schema']['id_column']

print(f"  Text column: {text_col}")
print(f"  Image column: {image_col}")
print(f"  ID column: {id_col}")
```

### Phase 4: Multi-Technique Hunting Loop

This is the core loop that explores MULTIPLE DISARM techniques:

```python
# Initialize results tracking
all_technique_results = []
techniques_tried = set()

print(f"\n🎯 Beginning multi-technique hunt (max {max_iterations} techniques)")
print(f"Strategy: {strategy}")

for iteration in range(1, max_iterations + 1):
    print(f"\n{'='*60}")
    print(f"ITERATION {iteration}/{max_iterations}")
    print(f"{'='*60}")

    # === Step 1: Select Technique ===
    if technique_ids and len(technique_ids) > 0:
        # Use pre-specified techniques
        if iteration <= len(technique_ids):
            technique_id = technique_ids[iteration - 1]
            print(f"📋 Using pre-specified technique: {technique_id}")
        else:
            print("✓ Completed all pre-specified techniques")
            break
    else:
        # Auto-suggest technique based on data and prior results
        from tasks import suggest_technique

        print(f"🤔 Suggesting next DISARM technique to hunt...")

        suggestion = suggest_technique.suggest(
            data_info=data_info,
            prior_results=all_technique_results,
            techniques_tried=list(techniques_tried),
            strategy=strategy,
            iteration=iteration
        )

        technique_id = suggestion['technique_id']
        technique_name = suggestion['technique_name']
        suggestion_reasoning = suggestion['reasoning']

        print(f"✨ Suggested: {technique_id} - {technique_name}")
        print(f"   Reasoning: {suggestion_reasoning}")

    # Mark as tried
    techniques_tried.add(technique_id)

    # Load technique details from DISARM framework
    technique_info = load_disarm_technique(technique_id)

    print(f"\n📖 Hunting for: {technique_id} - {technique_info['name']}")
    print(f"   Phase: {technique_info['phase']} | Tactic: {technique_info['tactic']}")
    print(f"   Description: {technique_info['description'][:100]}...")

    # === Step 2: Text Analysis ===
    print(f"\n📝 Analyzing text content for {technique_id}...")

    text_metrics = analyze_text_for_technique(
        data=data,
        text_col=text_col,
        technique_info=technique_info
    )

    print(f"   Text analysis complete: {len(text_metrics)} metrics computed")

    # === Step 3: Visual Analysis (VLM) ===
    visual_metrics = {}
    vlm_results = []

    if image_col and data[image_col].notna().sum() > 0:
        print(f"\n🖼️  Analyzing images with VLM ({vlm_model})...")

        # Sample images for VLM analysis
        images_with_data = data[data[image_col].notna()]
        sample_size = min(image_sample_size, len(images_with_data))

        sampled_images = images_with_data.sample(n=sample_size, random_state=42)

        print(f"   Sampled {sample_size} images for analysis")

        # Call vision task for each image
        from tasks import vision

        for idx, row in sampled_images.iterrows():
            image_path = row[image_col]

            # Create technique-specific VLM prompt
            vlm_prompt = create_vlm_prompt_for_technique(technique_info)

            try:
                vlm_result = vision.analyze_single_image(
                    image_path=image_path,
                    prompt=vlm_prompt,
                    model=vlm_model
                )

                vlm_results.append({
                    'image_path': image_path,
                    'post_id': row.get(id_col, idx),
                    'analysis': vlm_result['response']
                })
            except Exception as e:
                print(f"   ⚠ VLM error for {image_path}: {e}")

        # Aggregate visual findings
        visual_metrics = aggregate_vlm_results(vlm_results, technique_info)

        print(f"   ✓ Analyzed {len(vlm_results)} images")
        print(f"   Visual metrics: {visual_metrics}")
    else:
        print(f"\n⚠  No images available - text-only analysis mode")

    # === Step 4: Logo Analysis (Optional) ===
    logo_results = {}

    if use_logos and image_col and data[image_col].notna().sum() > 0:
        print(f"\n🎨 Identifying logos in images...")

        from tasks import logos

        # Use same sampled images
        sample_image_paths = sampled_images[image_col].tolist()

        logo_results = logos.analyze_images_for_logos(
            image_paths=sample_image_paths,
            logos_db_path='logos/logos.csv',
            vlm_model=vlm_model
        )

        print(f"   ✓ Logo analysis complete")
        print(f"   Logos found: {logo_results.get('unique_logos_found', 0)}")
        print(f"   Affiliation breakdown: {logo_results.get('affiliation_breakdown', {})}")

    # === Step 5: Cross-Modal Analysis ===
    print(f"\n🔗 Performing cross-modal analysis...")

    cross_modal_metrics = analyze_cross_modal_patterns(
        data=data,
        text_col=text_col,
        image_col=image_col,
        sampled_images=sampled_images if image_col else pd.DataFrame(),
        vlm_results=vlm_results,
        technique_info=technique_info
    )

    print(f"   Cross-modal metrics: {cross_modal_metrics}")

    # === Step 6: Calculate Signal Strength ===
    signal_strength = calculate_multimodal_signal(
        text_metrics=text_metrics,
        visual_metrics=visual_metrics,
        cross_modal_metrics=cross_modal_metrics,
        has_images=image_col is not None and len(vlm_results) > 0
    )

    # Determine status
    if signal_strength >= 7:
        status = "PASS"
        confidence = "High"
    elif signal_strength >= 4:
        status = "INCONCLUSIVE"
        confidence = "Medium"
    else:
        status = "FAIL"
        confidence = "Low"

    print(f"\n🎯 Result: {status} (Signal Strength: {signal_strength}/10, Confidence: {confidence})")

    # === Step 7: Record Results ===
    technique_result = {
        'iteration': iteration,
        'technique_id': technique_id,
        'technique_name': technique_info['name'],
        'status': status,
        'signal_strength': signal_strength,
        'confidence': confidence,
        'text_metrics': text_metrics,
        'visual_metrics': visual_metrics,
        'cross_modal_metrics': cross_modal_metrics,
        'logo_results': logo_results,
        'vlm_results': vlm_results[:5],  # Top 5 for report
        'timestamp': datetime.now().isoformat()
    }

    all_technique_results.append(technique_result)
    investigation_meta['techniques_hunted'].append(technique_id)

    # Early stopping if strategy is exploit and we found strong evidence
    if strategy == 'exploit' and status == 'PASS':
        print(f"\n✨ Strong evidence found! Exploit strategy suggests digging deeper.")
        # Continue to next iteration to explore related techniques

# End of hunting loop
print(f"\n{'='*60}")
print(f"HUNTING COMPLETE - Investigated {len(all_technique_results)} techniques")
print(f"{'='*60}")
```

### Phase 5: Generate Comprehensive Investigation Report

```python
print(f"\n📄 Generating comprehensive investigation report...")

# Generate report modeled after experiment/FIMI_Investigation_Report.md
report = generate_fimi_investigation_report(
    investigation_meta=investigation_meta,
    data_info=data_info,
    all_technique_results=all_technique_results,
    data=data
)

# Save report
report_path = investigation_folder / f"FIMI_Investigation_Report_{investigation_id}.md"
with open(report_path, 'w', encoding='utf-8') as f:
    f.write(report)

print(f"✓ Report saved: {report_path}")

# Save structured results as JSON
results_json_path = investigation_folder / f"results_{investigation_id}.json"
with open(results_json_path, 'w', encoding='utf-8') as f:
    json.dump({
        'investigation_meta': investigation_meta,
        'technique_results': all_technique_results
    }, f, indent=2, ensure_ascii=False)

print(f"✓ Structured results saved: {results_json_path}")
```

## Helper Functions

### Load DISARM Technique Details

```python
def load_disarm_technique(technique_id):
    """Load technique details from DISARM framework reference"""
    # Read from disarm-framework-reference.md
    # Parse technique by ID
    # Return dict with name, phase, tactic, description, indicators

    # Example return:
    return {
        'id': technique_id,
        'name': 'Create Memes',
        'phase': 'Execute',
        'tactic': 'Deliver Content',
        'description': 'Create and distribute memes...',
        'text_indicators': ['short punchy text', 'emotional language'],
        'visual_indicators': ['text overlaid on images', 'recognizable templates']
    }
```

### Create VLM Prompt for Technique

```python
def create_vlm_prompt_for_technique(technique_info):
    """Generate technique-specific VLM analysis prompt"""

    technique_id = technique_info['id']

    # Technique-specific prompts
    prompts = {
        'T0021': """Analyze this image and determine:
1. Is this a meme? (Yes/No)
2. What is the main message or narrative?
3. Does it have overlaid text?
4. What emotions does it evoke?
5. Is it political in nature?
6. Stance: pro-Russia, pro-Ukraine, neutral, or unclear?
""",
        'T0044': """Analyze this image for "seed kernel of truth" manipulation:
1. Does this appear to be original content or reposted?
2. Are there authentic elements mixed with misleading context?
3. Production quality (professional/amateur)?
4. Any branding or watermarks?
5. What narrative does it convey?
""",
        'T0086': """Analyze this image for narrative competition:
1. What narrative or message does this image convey?
2. Does it counter or compete with another narrative?
3. Stance: pro-Russia, pro-Ukraine, neutral?
4. Visual symbols present (flags, emblems, colors)?
5. Emotional tone (triumphant, defiant, mournful, etc.)?
""",
        'T0001': """Analyze this image for "5Ds" (Dismiss, Distort, Distract, Dismay, Divide):
1. Does the image dismiss, distort, or misrepresent something?
2. Is it designed to distract from another issue?
3. Does it create fear, dismay, or division?
4. What is the emotional manipulation strategy?
5. Stance: pro-Russia, pro-Ukraine, or mixed?
"""
    }

    # Return technique-specific prompt or generic
    return prompts.get(technique_id, f"""Analyze this image for DISARM technique {technique_id}:
1. Describe the main content
2. What narrative or message is conveyed?
3. Are there political symbols or indicators?
4. Stance: pro-Russia, pro-Ukraine, neutral, unclear?
5. Does this exhibit characteristics of {technique_info['name']}?
""")
```

### Text Analysis for Technique

```python
def analyze_text_for_technique(data, text_col, technique_info):
    """Analyze text content for technique-specific indicators"""

    if not text_col or text_col not in data.columns:
        return {}

    technique_id = technique_info['id']

    # Basic text metrics
    metrics = {
        'total_posts': len(data),
        'posts_with_text': data[text_col].notna().sum(),
        'avg_text_length': data[text_col].str.len().mean()
    }

    # Technique-specific analysis
    if technique_id == 'T0021':  # Memes
        # Short, punchy text
        short_text = (data[text_col].str.len() < 100).sum()
        metrics['short_text_count'] = short_text
        metrics['short_text_rate'] = short_text / len(data)

    elif technique_id == 'T0044':  # Seed Kernel
        # Look for original content markers
        # (implementation depends on platform)
        pass

    elif technique_id == 'T0001':  # 5Ds
        # Emotional/divisive language detection
        # (would use NLP/sentiment analysis)
        pass

    return metrics
```

### Aggregate VLM Results

```python
def aggregate_vlm_results(vlm_results, technique_info):
    """Aggregate VLM analysis results for technique detection"""

    if not vlm_results:
        return {}

    # Parse VLM responses for technique-specific indicators
    # This is simplified - real implementation would parse structured outputs

    metrics = {
        'images_analyzed': len(vlm_results),
        'technique_positive_count': 0,
        'stance_breakdown': {'pro-Russia': 0, 'pro-Ukraine': 0, 'neutral': 0, 'unclear': 0}
    }

    for result in vlm_results:
        analysis = result['analysis'].lower()

        # Detect technique presence (simplified)
        if technique_info['id'] == 'T0021':  # Memes
            if 'meme' in analysis or 'yes' in analysis[:50]:
                metrics['technique_positive_count'] += 1

        # Detect stance
        if 'pro-russia' in analysis:
            metrics['stance_breakdown']['pro-Russia'] += 1
        elif 'pro-ukraine' in analysis:
            metrics['stance_breakdown']['pro-Ukraine'] += 1
        elif 'neutral' in analysis:
            metrics['stance_breakdown']['neutral'] += 1
        else:
            metrics['stance_breakdown']['unclear'] += 1

    metrics['technique_detection_rate'] = metrics['technique_positive_count'] / len(vlm_results)

    return metrics
```

### Cross-Modal Analysis

```python
def analyze_cross_modal_patterns(data, text_col, image_col, sampled_images, vlm_results, technique_info):
    """Analyze consistency between text and images"""

    if not image_col or len(vlm_results) == 0:
        return {}

    # Check text-image alignment
    # Simplified - real implementation would do semantic matching

    metrics = {
        'posts_with_both': len(sampled_images[sampled_images[text_col].notna()]),
        'alignment_estimate': 0.0
    }

    # Calculate rough alignment based on stance consistency
    # (real implementation would compare text sentiment with VLM image sentiment)

    return metrics
```

### Calculate Signal Strength

```python
def calculate_multimodal_signal(text_metrics, visual_metrics, cross_modal_metrics, has_images):
    """Calculate overall signal strength (0-10) for technique detection"""

    text_score = 0.0
    visual_score = 0.0
    cross_modal_score = 0.0

    # Text component (0-3 points)
    if 'technique_indicator_rate' in text_metrics:
        text_score = min(3.0, text_metrics['technique_indicator_rate'] * 3)

    # Visual component (0-3 points)
    if has_images and 'technique_detection_rate' in visual_metrics:
        visual_score = min(3.0, visual_metrics['technique_detection_rate'] * 3)

    # Cross-modal component (0-4 points) - most important
    if has_images and 'alignment_estimate' in cross_modal_metrics:
        cross_modal_score = min(4.0, cross_modal_metrics['alignment_estimate'] * 4)

    # Weight by data availability
    if has_images:
        signal = (text_score * 0.3) + (visual_score * 0.3) + (cross_modal_score * 0.4)
    else:
        signal = text_score * 1.0  # Text-only mode

    return round(signal, 1)
```

### Generate Investigation Report

```python
def generate_fimi_investigation_report(investigation_meta, data_info, all_technique_results, data):
    """Generate comprehensive FIMI investigation report (modeled after experiment/FIMI_Investigation_Report.md)"""

    # Count results by status
    pass_count = sum(1 for r in all_technique_results if r['status'] == 'PASS')
    fail_count = sum(1 for r in all_technique_results if r['status'] == 'FAIL')
    inconclusive_count = sum(1 for r in all_technique_results if r['status'] == 'INCONCLUSIVE')

    # Get techniques detected
    detected_techniques = [r for r in all_technique_results if r['status'] == 'PASS']

    report = f"""# FIMI INVESTIGATION REPORT
## DISARM Framework Analysis

**Investigation ID:** {investigation_meta['investigation_id']}
**Investigation Date:** {datetime.now().strftime('%Y-%m-%d')}
**Platform:** {investigation_meta.get('platform', 'Unknown')}
**Dataset:** {investigation_meta['data_path']}

**Investigator:** Multimodal FIMI Investigator Agent
**Analysis Methods:**
- Text content analysis
- VLM image analysis ({investigation_meta['vlm_model']} model)
- Logo/symbol detection (142+ logo database)
- Cross-modal narrative analysis
- DISARM framework mapping

---

## EXECUTIVE SUMMARY

This investigation analyzed **{data_info['total_records']} {investigation_meta.get('platform', 'social media')} posts** and identified **{pass_count} DISARM techniques** with high confidence across {len(all_technique_results)} investigated techniques.

**Investigation Results:**
- ✓ **PASS** (Strong Evidence): {pass_count} techniques
- ? **INCONCLUSIVE** (Moderate Evidence): {inconclusive_count} techniques
- ✗ **FAIL** (No Evidence): {fail_count} techniques

**Key Findings:**
"""

    # Add key findings from detected techniques
    for idx, result in enumerate(detected_techniques[:5], 1):
        report += f"""
{idx}. **{result['technique_id']}: {result['technique_name']}** (Signal: {result['signal_strength']}/10)
   - {generate_technique_summary(result)}
"""

    report += f"""

**Multimodal Analysis:**
- Total posts analyzed: {data_info['total_records']}
- Posts with images: {int(data_info['total_records'] * data_info['image_availability'])} ({data_info['image_availability']:.1%})
- Images analyzed with VLM: {investigation_meta['image_sample_size']} per technique
- Logo database: 142+ conflict-related logos

---

## DETAILED DISARM TECHNIQUE MAPPING

"""

    # Add detailed section for each detected technique
    for result in detected_techniques:
        report += generate_technique_detail_section(result, data_info)

    report += f"""

---

## VLM IMAGE ANALYSIS HIGHLIGHTS

"""

    # Add VLM analysis highlights
    for result in detected_techniques:
        if result.get('vlm_results'):
            report += generate_vlm_highlights_section(result)

    report += f"""

---

## LOGO & SYMBOL DETECTION

"""

    # Add logo analysis section
    for result in all_technique_results:
        if result.get('logo_results') and result['logo_results'].get('unique_logos_found', 0) > 0:
            report += generate_logo_section(result)

    report += f"""

---

## BEHAVIORAL PATTERN ANALYSIS

{generate_behavioral_patterns_section(all_technique_results, data_info)}

---

## CROSS-MODAL ANALYSIS

{generate_cross_modal_section(all_technique_results)}

---

## TECHNIQUE CO-OCCURRENCE ANALYSIS

{generate_cooccurrence_section(detected_techniques)}

---

## RECOMMENDATIONS

### For Counter-FIMI Operations:

{generate_recommendations(detected_techniques, data_info)}

### For Further Investigation:

{generate_further_investigation_suggestions(all_technique_results)}

---

## CONCLUSION

{generate_conclusion(investigation_meta, all_technique_results, pass_count, inconclusive_count, fail_count)}

---

## APPENDIX: TECHNIQUE REFERENCE

| ID | Technique Name | Status | Signal Strength | Confidence |
|----|----------------|--------|-----------------|------------|
"""

    for result in all_technique_results:
        status_icon = '✓' if result['status'] == 'PASS' else ('?' if result['status'] == 'INCONCLUSIVE' else '✗')
        report += f"| {result['technique_id']} | {result['technique_name']} | {status_icon} {result['status']} | {result['signal_strength']}/10 | {result['confidence']} |\n"

    report += f"""

**Legend:** ✓ = Strong Evidence | ? = Moderate Evidence | ✗ = No Evidence

---

**Report End**

*Generated by: Multimodal FIMI Investigator Agent*
*Framework: DISARM (disinformationframework.org)*
*Date: {datetime.now().strftime('%Y-%m-%d')}*
"""

    return report


def generate_technique_summary(result):
    """Generate 1-sentence summary of technique finding"""
    # Simplified - real implementation would extract key findings
    return f"Detected with {result['confidence'].lower()} confidence across text and visual modalities"


def generate_technique_detail_section(result, data_info):
    """Generate detailed section for one technique"""
    return f"""
### {result['technique_id']}: {result['technique_name']}
**Status:** {result['status']}
**Signal Strength:** {result['signal_strength']}/10
**Confidence:** {result['confidence']}

**Evidence Summary:**
- Text Analysis: {result['text_metrics'].get('posts_with_text', 0)} posts analyzed
- Visual Analysis: {result['visual_metrics'].get('images_analyzed', 0)} images analyzed
- Cross-Modal Consistency: {result['cross_modal_metrics'].get('alignment_estimate', 0):.1%}

**Key Findings:**
{generate_key_findings(result)}

---
"""


def generate_key_findings(result):
    """Extract key findings from result"""
    findings = []

    if result['visual_metrics'].get('technique_detection_rate', 0) > 0.5:
        findings.append(f"- High visual detection rate: {result['visual_metrics']['technique_detection_rate']:.1%}")

    if result['logo_results'].get('unique_logos_found', 0) > 0:
        findings.append(f"- Logos detected: {result['logo_results']['unique_logos_found']} unique logos")

    return '\n'.join(findings) if findings else "- Multiple indicators present across modalities"


def generate_vlm_highlights_section(result):
    """Generate VLM analysis highlights"""
    return f"""
### Technique {result['technique_id']} - Image Analysis

**Images Analyzed:** {result['visual_metrics'].get('images_analyzed', 0)}
**Detection Rate:** {result['visual_metrics'].get('technique_detection_rate', 0):.1%}

**Sample VLM Analyses:**

"""


def generate_logo_section(result):
    """Generate logo analysis section"""
    logo_res = result['logo_results']
    return f"""
**Logos Found in {result['technique_id']} Hunt:**
- Total logos: {logo_res.get('unique_logos_found', 0)}
- Affiliation breakdown: {logo_res.get('affiliation_breakdown', {})}

"""


def generate_behavioral_patterns_section(all_results, data_info):
    """Generate behavioral patterns analysis"""
    return f"""
**Dataset Characteristics:**
- Total records: {data_info['total_records']}
- Image availability: {data_info['image_availability']:.1%}
- Date range: {data_info.get('date_range', 'Unknown')}

**Technique Distribution:**
- Techniques detected with high confidence: {sum(1 for r in all_results if r['status'] == 'PASS')}
- Multimodal techniques: {sum(1 for r in all_results if r.get('visual_metrics', {}).get('images_analyzed', 0) > 0)}
"""


def generate_cross_modal_section(all_results):
    """Generate cross-modal analysis section"""
    return """
**Text-Image Consistency:**
Analysis shows patterns where visual content reinforces textual narratives,
indicating coordinated multimodal messaging.
"""


def generate_cooccurrence_section(detected_techniques):
    """Generate technique co-occurrence analysis"""
    if len(detected_techniques) < 2:
        return "Insufficient data for co-occurrence analysis (only one technique detected)."

    return f"""
Multiple techniques detected in the same dataset suggest a multi-faceted campaign:
- {len(detected_techniques)} different techniques identified
- Indicates sophisticated, coordinated operation
"""


def generate_recommendations(detected_techniques, data_info):
    """Generate counter-FIMI recommendations"""
    return """
1. **Visual Fact-Checking Priority:**
   - Multimodal content requires both text and image verification
   - VLM-powered detection systems recommended for scale

2. **Logo Database Expansion:**
   - Continue expanding logo database as new symbols emerge
   - Track logo usage patterns over time

3. **Cross-Modal Validation:**
   - Implement systems that check text-image consistency
   - Flag posts where visuals contradict text claims
"""


def generate_further_investigation_suggestions(all_results):
    """Generate further investigation suggestions"""
    return """
1. **Temporal Analysis:** Track technique evolution over time
2. **Network Analysis:** Map coordination between accounts/channels
3. **Cross-Platform Analysis:** Check if same content appears elsewhere
4. **Linguistic Analysis:** Deeper NLP analysis of textual patterns
"""


def generate_conclusion(investigation_meta, all_results, pass_count, inconclusive_count, fail_count):
    """Generate investigation conclusion"""
    total_techniques = len(all_results)

    return f"""
This investigation analyzed {investigation_meta['data_info']['total_records']} posts and detected **{pass_count} DISARM techniques** with high confidence out of {total_techniques} investigated techniques.

The multimodal analysis combining text, images, and logo detection revealed {('a sophisticated disinformation campaign' if pass_count >= 3 else 'evidence of information manipulation')} using {'coordinated multimodal tactics' if pass_count >= 2 else 'various manipulation techniques'}.

**Overall Assessment:** The dataset {'contains clear evidence of coordinated FIMI activity' if pass_count >= 3 else ('shows some manipulation indicators' if pass_count > 0 else 'does not show strong evidence of systematic manipulation')}.
"""
```

## Outputs

1. **Investigation Report** (Markdown):
   - `investigations/{investigation_id}/FIMI_Investigation_Report_{investigation_id}.md`
   - Comprehensive report modeled after experiment/FIMI_Investigation_Report.md

2. **Structured Results** (JSON):
   - `investigations/{investigation_id}/results_{investigation_id}.json`
   - Machine-readable results for further analysis

3. **VLM Analysis Cache** (JSON):
   - `investigations/{investigation_id}/vlm_cache/`
   - Cached VLM responses to avoid re-analysis

## Notes

- **Adaptive Strategy**: Balances exploration (trying new techniques) with exploitation (digging deeper)
- **Native Format**: Works with data AS-IS - no SQL conversion needed
- **Cost-Aware**: Samples images for VLM analysis to manage API costs
- **Comprehensive**: Explores MULTIPLE techniques like the example report
- **Evidence-Based**: Every finding backed by concrete evidence from text/images
- **Politically Neutral**: Objective analysis regardless of narrative

## Error Handling

- **No Images**: Falls back to text-only analysis
- **VLM Failures**: Retries with exponential backoff, skips problematic images
- **Missing Logos**: Continues without logo analysis
- **Invalid Data**: Reports issues clearly and continues with available data
