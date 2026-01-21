<!-- Powered by BMAD™ Multimodal OSINT Expansion Pack -->

# vision

Use Vision Language Model (VLM) to analyze images for FIMI investigation.

## Purpose

Provide VLM-powered image analysis for detecting manipulation, understanding visual narratives, and extracting information from images. Uses vision.ipynb notebook template with Ollama API.

## Inputs

```yaml
required:
  - image_paths: '{list_of_image_paths}' # Images to analyze
  - analysis_prompt: '{what_to_analyze}' # Analysis question

optional:
  - vlm_model: 'qwen3-vl:8b' # Model: gemma3:4b, qwen3-vl:8b/32b/235b, etc.
  - batch_mode: false # Analyze multiple images together
  - investigation_id: null # For caching and organization
```

## Process

### Step 1: Setup VLM Notebook

```python
import shutil
from pathlib import Path
from ollama import Client

# Copy vision.ipynb template to investigation folder
if investigation_id:
    investigation_folder = Path(f"investigations/{investigation_id}")
    investigation_folder.mkdir(parents=True, exist_ok=True)

    target_notebook = investigation_folder / "vision_analysis.ipynb"
    if not target_notebook.exists():
        shutil.copy("vision.ipynb", target_notebook)
        print(f"📓 VLM notebook ready: {target_notebook}")

# Initialize Ollama client
client = Client(
    host="https://ollama.sct.sintef.no",
    headers={"Authorization": "Bearer BAEX4eMGSA4iIEmPL3BrMMBhSM3IOWms"}
)

print(f"✓ Connected to Ollama API")
print(f"🎨 Using model: {vlm_model}")
```

### Step 2: Analyze Images

```python
from datetime import datetime

results = []

if batch_mode and len(image_paths) > 1:
    # Batch analysis - all images in one query
    print(f"🖼️  Analyzing {len(image_paths)} images in batch mode...")

    response = client.chat(
        model=vlm_model,
        messages=[{
            "role": "user",
            "content": analysis_prompt,
            "images": image_paths
        }]
    )

    results.append({
        'images': image_paths,
        'image_count': len(image_paths),
        'prompt': analysis_prompt,
        'model': vlm_model,
        'response': response.message.content,
        'timestamp': datetime.now().isoformat()
    })

else:
    # Single image analysis
    print(f"🖼️  Analyzing {len(image_paths)} images individually...")

    for img_path in image_paths:
        print(f"  Processing: {img_path}")

        try:
            response = client.chat(
                model=vlm_model,
                messages=[{
                    "role": "user",
                    "content": analysis_prompt,
                    "images": [img_path]
                }]
            )

            results.append({
                'image': img_path,
                'prompt': analysis_prompt,
                'model': vlm_model,
                'response': response.message.content,
                'timestamp': datetime.now().isoformat()
            })

        except Exception as e:
            print(f"  ⚠ Error analyzing {img_path}: {e}")
            results.append({
                'image': img_path,
                'error': str(e),
                'status': 'failed'
            })

print(f"✓ Analyzed {len(results)} image(s)")
```

### Step 3: Return Results

```python
# Return results for use by calling task
return {
    'results': results,
    'total_analyzed': len([r for r in results if 'response' in r]),
    'total_failed': len([r for r in results if 'error' in r]),
    'model_used': vlm_model
}
```

## Common Analysis Prompts

### Stance Detection
```python
stance_prompt = """Analyze this image for political stance regarding Russia-Ukraine conflict.

Consider:
1. Visual symbols (flags, emblems, colors)
2. Text overlays or captions
3. Imagery content
4. Overall narrative

Answer:
- Stance: [pro-Ukraine | pro-Russia | neutral | unclear]
- Confidence: [high | medium | low]
- Key visual indicators: [list 3-5 elements]
"""
```

### Meme Detection
```python
meme_prompt = """Is this a meme?

A meme typically has:
- Overlaid text on image
- Humorous or satirical message
- Recognizable template
- Intended for viral sharing

Answer:
- Is meme? [Yes | No]
- Meme type/template: [description]
- Main message: [description]
- Political nature? [Yes | No]
"""
```

### Manipulation Detection
```python
manipulation_prompt = """Analyze for image manipulation:

Look for:
- Inconsistent lighting/shadows
- Unnatural edges or artifacts
- Copy-paste elements
- Color grading anomalies

Answer:
- Manipulation detected? [Yes | No | Unclear]
- Type and location: [description]
- Confidence: [high | medium | low]
"""
```

## Available VLM Models

- **qwen3-vl:8b** (Recommended default) - Balanced speed/quality
- **gemma3:4b** (Fast) - Quick screening, large batches
- **qwen3-vl:32b** (High quality) - Detailed analysis, important evidence
- **qwen3-vl:235b** (Highest quality) - Final verification, publication (use sparingly)

## Best Practices

1. **Be Specific in Prompts**: Detail exactly what to look for
2. **Use Structured Formats**: Request organized responses (numbered lists)
3. **Cost Management**: Sample images for large datasets (default: 50 per hunt)
4. **Cache Results**: Save VLM responses to avoid re-analysis
5. **Error Handling**: Retry failed analyses, skip problematic images

## Output Format

```python
{
    'image': 'path/to/image.jpg',
    'prompt': 'Analysis question',
    'model': 'qwen3-vl:8b',
    'response': 'VLM analysis response...',
    'timestamp': '2026-01-20T...'
}
```

## Integration

Called by [execute-hunt.md](execute-hunt.md) for multimodal DISARM technique detection.
