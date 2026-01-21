<!-- Powered by BMAD™ Multimodal OSINT Expansion Pack -->

# logos

Identify logos, emblems, and symbols in images using logo database and VLM.

## Purpose

Detect organizational logos in social media images to identify affiliation, political stance, and content sources. Uses 142+ logo database (logos/logos.csv) with VLM-powered visual matching.

## Inputs

```yaml
required:
  - image_paths: '{list_of_images}' # Images to search for logos

optional:
  - logos_db_path: 'logos/logos.csv' # Logo database CSV
  - logos_images_path: 'logos/images/' # Logo reference images directory
  - stance_filter: 'all' # 'Russia', 'Ukraine', 'neutral', 'all'
  - vlm_model: 'qwen3-vl:8b' # VLM model to use
  - detection_threshold: 0.6 # Confidence threshold
```

## Process

### Step 1: Load Logo Database

```python
import pandas as pd
from pathlib import Path

# Load logo database
logos_db = pd.read_csv(logos_db_path)

print(f"📚 Logo database: {len(logos_db)} logos")
print(f"   Affiliations: {logos_db['Affiliation/Stance'].value_counts().to_dict()}")

# Filter by stance if requested
if stance_filter != 'all':
    logos_db = logos_db[logos_db['Affiliation/Stance'] == stance_filter]
    print(f"   Filtered to {stance_filter}: {len(logos_db)} logos")

# Index logo images
logo_image_paths = {}
images_dir = Path(logos_images_path)

for idx, row in logos_db.iterrows():
    filename = row['File Name']
    logo_path = images_dir / filename
    if logo_path.exists():
        logo_image_paths[row['Name']] = str(logo_path)

print(f"✓ {len(logo_image_paths)} logo images indexed")
```

### Step 2: Detect Logos Using VLM

```python
from tasks import vision

print(f"🔍 Analyzing {len(image_paths)} images for logos...")

all_detections = []

for img_path in image_paths:
    print(f"  Processing: {img_path}")

    # Ask VLM to describe all logos/symbols in image
    description_prompt = """Identify ALL logos, emblems, flags, symbols, and organizational branding in this image.

For each logo/symbol found, provide:
- Type (flag/emblem/coat of arms/logo/symbol)
- Detailed description
- Colors
- Any text visible
- Size (large/medium/small)

List all identified elements."""

    vlm_result = vision.analyze_single_image(
        image_path=img_path,
        prompt=description_prompt,
        model=vlm_model
    )

    described_logos = vlm_result['response']

    # Match VLM descriptions to database
    matches = match_descriptions_to_database(described_logos, logos_db)

    all_detections.append({
        'image': img_path,
        'detected_logos': matches,
        'logo_count': len(matches)
    })

print(f"✓ Logo detection complete")
```

### Step 3: Match Descriptions to Database

```python
from difflib import SequenceMatcher

def match_descriptions_to_database(vlm_description, logos_db):
    """Match VLM logo descriptions to database entries"""

    matches = []
    vlm_desc_lower = vlm_description.lower()

    for idx, logo in logos_db.iterrows():
        logo_name = logo['Name'].lower()
        logo_desc = logo['Description'].lower() if pd.notna(logo['Description']) else ""

        # Calculate similarity
        name_sim = SequenceMatcher(None, vlm_desc_lower, logo_name).ratio()
        desc_sim = SequenceMatcher(None, vlm_desc_lower, logo_desc).ratio()

        similarity = max(name_sim, desc_sim)

        if similarity > detection_threshold:
            matches.append({
                'logo_name': logo['Name'],
                'affiliation': logo['Affiliation/Stance'],
                'type': logo['Type'],
                'confidence': round(similarity, 2),
                'description': logo['Description']
            })

    # Sort by confidence
    matches.sort(key=lambda x: x['confidence'], reverse=True)

    return matches[:5]  # Top 5 matches
```

### Step 4: Aggregate Results

```python
def aggregate_logo_findings(all_detections):
    """Aggregate logo detections across all images"""

    total_images = len(all_detections)
    images_with_logos = sum(1 for d in all_detections if d['logo_count'] > 0)

    # Count occurrences
    logo_counts = {}
    affiliation_counts = {'Russia': 0, 'Ukraine': 0, 'neutral': 0, 'other': 0}

    for detection in all_detections:
        for logo in detection['detected_logos']:
            logo_name = logo['logo_name']
            affiliation = logo['affiliation']

            logo_counts[logo_name] = logo_counts.get(logo_name, 0) + 1

            if affiliation == 'Russia':
                affiliation_counts['Russia'] += 1
            elif affiliation == 'Ukraine':
                affiliation_counts['Ukraine'] += 1
            elif affiliation == 'neutral':
                affiliation_counts['neutral'] += 1
            else:
                affiliation_counts['other'] += 1

    # Top logos
    top_logos = sorted(logo_counts.items(), key=lambda x: x[1], reverse=True)[:10]

    return {
        'total_images': total_images,
        'images_with_logos': images_with_logos,
        'logo_detection_rate': images_with_logos / total_images if total_images > 0 else 0,
        'unique_logos_found': len(logo_counts),
        'affiliation_breakdown': affiliation_counts,
        'top_logos': top_logos,
        'all_detections': all_detections
    }

results = aggregate_logo_findings(all_detections)

print(f"\n📊 Logo Analysis Summary:")
print(f"   Images with logos: {results['images_with_logos']}/{results['total_images']} ({results['logo_detection_rate']:.1%})")
print(f"   Unique logos found: {results['unique_logos_found']}")
print(f"   Affiliation breakdown: {results['affiliation_breakdown']}")
print(f"   Top logos: {results['top_logos'][:5]}")
```

### Step 5: Stance Analysis

```python
def analyze_stance_from_logos(logo_results):
    """Determine overall stance from logo affiliations"""

    affiliation = logo_results['affiliation_breakdown']
    total_political = affiliation['Russia'] + affiliation['Ukraine']

    if total_political == 0:
        stance = 'neutral'
        confidence = 'low'
    else:
        russia_ratio = affiliation['Russia'] / total_political
        ukraine_ratio = affiliation['Ukraine'] / total_political

        if russia_ratio > 0.7:
            stance = 'pro-Russia'
            confidence = 'high' if russia_ratio > 0.85 else 'medium'
        elif ukraine_ratio > 0.7:
            stance = 'pro-Ukraine'
            confidence = 'high' if ukraine_ratio > 0.85 else 'medium'
        else:
            stance = 'mixed'
            confidence = 'medium'

    return {
        'overall_stance': stance,
        'confidence': confidence,
        'russia_logos': affiliation['Russia'],
        'ukraine_logos': affiliation['Ukraine'],
        'reasoning': f"Detected {affiliation['Russia']} Russia-affiliated and {affiliation['Ukraine']} Ukraine-affiliated logos"
    }

stance = analyze_stance_from_logos(results)

print(f"\n🎯 Stance Analysis:")
print(f"   Overall stance: {stance['overall_stance']}")
print(f"   Confidence: {stance['confidence']}")
print(f"   Reasoning: {stance['reasoning']}")
```

## Output Format

```python
{
    'total_images': 50,
    'images_with_logos': 28,
    'logo_detection_rate': 0.56,
    'unique_logos_found': 12,
    'affiliation_breakdown': {
        'Russia': 35,
        'Ukraine': 8,
        'neutral': 3,
        'other': 1
    },
    'top_logos': [
        ('Z symbol', 12),
        ('DNR coat of arms', 8),
        ('Russian flag', 7),
        ...
    ],
    'stance_analysis': {
        'overall_stance': 'pro-Russia',
        'confidence': 'high',
        'reasoning': '...'
    }
}
```

## Logo Database

The `logos/logos.csv` database contains **142+ logos** including:

- **Government symbols**: DNR, LNR, Kherson, Melitopol administrations
- **Military units**: Azov Brigade, Wagner Group, various battalions
- **Flags**: Russian flag, Ukrainian flag, Z symbol, etc.
- **Media outlets**: State news agencies, propaganda channels
- **Organizations**: Various political and military organizations

## Use Cases

1. **Source Credibility**: Identify official vs unofficial sources by logos
2. **Coordination Detection**: Track coordinated use of specific logos
3. **Narrative Tracking**: Map logo presence to narrative themes
4. **Affiliation Analysis**: Determine political alignment objectively

## Integration

Called by [execute-hunt.md](execute-hunt.md) for affiliation tracking in multimodal DISARM technique detection.
