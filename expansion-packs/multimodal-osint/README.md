# Multimodal FIMI/DISARM Investigator

Your friendly multimodal OSINT expansion pack for detecting Foreign Information Manipulation and Interference (FIMI) in social media campaigns that use both text and images!

## Overview

This expansion pack provides comprehensive multimodal analysis capabilities for investigating disinformation campaigns using the DISARM framework. Unlike text-only analysis, this detective examines BOTH text and images together, using Vision Language Models (VLMs) to understand visual manipulation, detect logos for affiliation tracking, and identify coordinated multimodal campaigns.

Perfect for analyzing Telegram channels, Twitter campaigns, meme warfare, and any social media content where visuals play a key role in the manipulation.

### What Makes This Pack Special

1. **Truly Multimodal** - Analyzes text AND images together for comprehensive understanding
2. **VLM-Powered** - Uses state-of-the-art Vision Language Models for deep image analysis
3. **Logo Intelligence** - Identifies 142+ conflict-related logos to track affiliations objectively
4. **Adaptive Hunting** - Explores MULTIPLE DISARM techniques using explore-exploit strategy
5. **Native Format Support** - Works with your data AS-IS (JSON, JSONL, CSV, Parquet) - no SQL needed
6. **Politically Neutral** - Objective analysis focused on manipulation techniques, not "sides"
7. **Comprehensive Reports** - Produces detailed investigation reports like [experiment/FIMI_Investigation_Report.md](experiment/FIMI_Investigation_Report.md)

## Core Features

### 1. Multi-Technique DISARM Hunting

Doesn't just look for one technique - adaptively explores MULTIPLE DISARM techniques based on your data:

- **Iteration 1**: Explores data structure, detects schema automatically
- **Iterations 2-N**: Hunts for techniques using balanced explore/exploit strategy
  - **Explore**: Tries new technique areas to diversify investigation
  - **Exploit**: Digs deeper into promising findings
  - **Balanced**: Mixes both approaches intelligently

Produces comprehensive reports identifying ALL detected manipulation techniques.

### 2. VLM-Powered Image Analysis

Uses Vision Language Models to understand:

- **Meme Detection**: Overlaid text, recognizable templates, viral formats
- **Stance Analysis**: Pro-Russia, pro-Ukraine, neutral determination
- **Manipulation Detection**: Visual editing, deepfakes, misleading context
- **Symbol Identification**: Flags, emblems, military insignia
- **Narrative Analysis**: What message is the image conveying?

Available VLM Models:
- `gemma3:4b` - Fast, for large batches
- `qwen3-vl:8b` - **Recommended default**, balanced speed/quality
- `qwen3-vl:32b` - High quality for detailed analysis
- `qwen3-vl:235b` - Maximum accuracy (use sparingly, expensive)

### 3. Logo Database (142+ Logos)

Automatically identifies logos in images to track affiliations:

**Logo Categories:**
- Government symbols (DNR, LNR, Kherson, Melitopol administrations)
- Military units (Azov Brigade, Wagner Group, various battalions)
- Flags (Russian, Ukrainian, Z symbol, etc.)
- Media outlets (state news agencies, propaganda channels)
- Organizations (political and military groups)

**Affiliation Tracking:**
- Objectively categorizes logos by affiliation (Russia, Ukraine, neutral)
- Tracks logo co-occurrence patterns
- Identifies coordinated use of specific symbols

### 4. Cross-Modal Validation

Strongest evidence comes from consistency across modalities:

- Text-image alignment detection
- Narrative reinforcement patterns
- Coordinated multimodal messaging indicators

## File Structure

```
multimodal-osint/
├── config.yaml                           # Pack configuration
├── README.md                             # This file
├── agents/
│   └── multimodal-investigator.md        # Your friendly FIMI detective
├── tasks/
│   ├── init-data.md                      # Data exploration & schema detection
│   ├── execute-hunt.md                   # Main multi-technique hunting workflow
│   ├── suggest-technique.md              # Adaptive technique selection
│   ├── vision.md                         # VLM image analysis capabilities
│   └── logos.md                          # Logo identification system
├── workflows/
│   └── disarm-hunt.yaml                  # Comprehensive hunt workflow
└── data/                                 # Knowledge base
    ├── disarm-framework-reference.md     # DISARM techniques reference
    ├── manipulation-playbook.md          # Known manipulation patterns
    └── platform-behavior-baselines.md    # Normal behavior baselines
```

## Quick Start

### Prerequisites

1. **VLM Access**: Ollama API with VLM models (configured in vision.ipynb)
2. **Logo Database**: `logos/logos.csv` and `logos/images/` in project root
3. **Social Media Dataset**: With images (JSON, JSONL, CSV, or Parquet format)

### Installation

Install via BMAD framework:

```bash
# Add to your .bmad/config or install directly
bmad install expansion-packs/multimodal-osint
```

### Usage

Activate the agent:

```bash
/multimodal-investigator
```

Run a comprehensive hunt:

```
*hunt data_path="data/telegram_campaign/" investigation_id="campaign_001"
```

The agent will:
1. ✅ Explore your data and detect structure automatically
2. ✅ Hunt for multiple DISARM techniques (default: 5 iterations)
3. ✅ Analyze both text and images with VLMs
4. ✅ Identify logos and track affiliations
5. ✅ Produce comprehensive investigation report

### Command Reference

**Main Commands:**

- `*hunt` - Comprehensive multimodal DISARM technique hunt
  - Auto-detects data structure
  - Explores multiple techniques
  - Produces full investigation report

- `*vision` - Analyze specific images with VLM
  - Single or batch image analysis
  - Custom analysis prompts
  - Stance, meme, manipulation detection

- `*logos` - Identify logos in images
  - Uses 142+ logo database
  - Tracks affiliations objectively
  - VLM-powered visual matching

- `*suggest` - Get technique suggestion for your data
  - Analyzes data characteristics
  - Recommends promising technique
  - Explains reasoning

- `*list-disarm` - Show DISARM framework reference
- `*status` - Check investigation progress
- `*help` - Show all available commands
- `*exit` - Exit agent mode

## Example Investigation

```bash
# Activate agent
/multimodal-investigator

# Run comprehensive hunt on Telegram dataset
*hunt data_path="data/telegram_posts.jsonl" investigation_id="telegram_fimi_2024" max_iterations=10 strategy="balanced"

# The agent will:
# - Detect that data is JSONL with text and image columns
# - Hunt for 10 different DISARM techniques adaptively
# - Analyze ~50 images per technique with VLM
# - Identify logos in images
# - Produce comprehensive report at:
#   investigations/telegram_fimi_2024/FIMI_Investigation_Report_telegram_fimi_2024.md
```

### Example Output

The investigation report includes:

```markdown
# FIMI INVESTIGATION REPORT
## DISARM Framework Analysis

**Investigation Results:**
- ✓ PASS (Strong Evidence): 6 techniques
- ? INCONCLUSIVE (Moderate Evidence): 2 techniques
- ✗ FAIL (No Evidence): 2 techniques

**Key Findings:**
1. **T0021: Create Memes** (Signal: 8.5/10)
   - 68% of images identified as political memes with consistent messaging

2. **T0086: Develop Competing Narratives** (Signal: 7.8/10)
   - Visual content reinforces pro-Russian narratives across 87% of images

[... detailed analysis for each technique ...]

## VLM IMAGE ANALYSIS HIGHLIGHTS
[... sample VLM analyses with evidence ...]

## LOGO & SYMBOL DETECTION
[... detected logos and affiliation breakdown ...]

## BEHAVIORAL PATTERN ANALYSIS
[... coordination indicators, temporal patterns ...]

## RECOMMENDATIONS
[... counter-FIMI recommendations ...]
```

See [experiment/FIMI_Investigation_Report.md](experiment/FIMI_Investigation_Report.md) for full example.

## Data Requirements

### Input Data Format

The agent works with data in its **native format** - no conversion needed:

**Supported Formats:**
- JSON/JSONL (line-delimited)
- CSV
- Parquet
- SQLite databases
- Directory with multiple files

**Required Fields:**
- Text content (automatically detected: `text`, `message`, `content`, `caption`, etc.)
- Image paths (automatically detected: `image`, `photo`, `file_path`, etc.)
- Post IDs (optional: `id`, `post_id`, `message_id`, etc.)
- Dates (optional: `date`, `timestamp`, `created_at`, etc.)

**Example JSONL:**
```json
{"message_id": "123", "text": "Political message...", "photo_path": "images/post_123.jpg", "timestamp": "2024-01-15"}
{"message_id": "124", "text": "Another post...", "photo_path": "images/post_124.jpg", "timestamp": "2024-01-16"}
```

The agent auto-detects the structure - just point it at your data!

### Image Requirements

- **Formats**: JPG, PNG (standard image formats)
- **Availability**: Works even with partial image coverage (20%+ recommended)
- **Paths**: Can be absolute or relative file paths
- **Location**: Images can be in same directory or separate image folder

## Multimodal DISARM Techniques

This pack specializes in techniques that benefit from visual analysis:

### Highly Multimodal (Best with Images)

- **T0021**: Create Memes - Detect meme templates, overlaid text, viral formats
- **T0044**: Seed Kernel of Truth - Real images with misleading context
- **T0086**: Develop Competing Narratives - Visual narrative competition
- **T0057**: Organize Events - Photos of staged events
- **T0097**: Create Personas - Profile images, visual identity

### Text + Visual (Enhanced with Images)

- **T0001**: 5Ds (Dismiss, Distort, Distract, Dismay, Divide) - Visual distortion
- **T0022**: Conspiracy Narratives - Visual "evidence"
- **T0019**: Information Pollution - Visual spam
- **T0007**: Inauthentic Pages - Profile images, branding

### Text-Focused (Works without Images)

- All other DISARM techniques analyzed through text

## Technical Architecture

### Workflow Flow

```
┌─────────────────────────────────────────┐
│ PHASE 1: Data Exploration               │
│ - Auto-detect file format               │
│ - Identify text/image columns           │
│ - Assess multimodal capabilities        │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ PHASE 2: Multi-Technique Hunting Loop  │
│ (Iterations 1 to max_iterations)        │
│                                         │
│  ① Suggest Technique (explore/exploit) │
│  ② Analyze Text Content                │
│  ③ Analyze Images with VLM            │
│  ④ Identify Logos (optional)          │
│  ⑤ Cross-Modal Analysis                │
│  ⑥ Calculate Signal Strength           │
│  ⑦ Record Results                      │
│                                         │
│  Loop until max iterations or criteria  │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│ PHASE 3: Report Generation              │
│ - Aggregate all technique findings      │
│ - VLM analysis highlights               │
│ - Logo & symbol detection summary       │
│ - Behavioral patterns                   │
│ - Cross-modal analysis                  │
│ - Recommendations                       │
│ - Comprehensive FIMI report             │
└─────────────────────────────────────────┘
```

### Explore-Exploit Strategy

**Exploration (Trying New Things):**
- Prioritizes techniques from unexplored tactics
- Diversifies investigation coverage
- Used in early iterations or after failures

**Exploitation (Digging Deeper):**
- Focuses on techniques related to prior successes
- Investigates related tactics when evidence found
- Used after strong positive findings

**Balanced (Default):**
- Early iterations: Explore to map landscape
- Later iterations: Exploit if found evidence
- Adapts based on findings

## Performance & Cost Considerations

### VLM Usage

VLM analysis is the most resource-intensive operation:

**Default Settings:**
- Sample size: 50 images per technique
- Model: qwen3-vl:8b
- Caching: Enabled (avoid re-analysis)

**Cost Management Tips:**
1. Reduce `image_sample_size` for quick analysis (10-20 images)
2. Use faster model (`gemma3:4b`) for screening
3. Use high-quality model (`qwen3-vl:32b`) only for critical evidence
4. Results are cached - re-running same investigation is fast

### Dataset Size Handling

- **Small (<100 posts)**: Analyzes all images
- **Medium (100-1000 posts)**: Samples intelligently
- **Large (1000+ posts)**: Adaptive sampling per technique
- **Very Large (10k+ posts)**: Consider pre-filtering or multiple investigations

## Agent Personality

Your Multimodal FIMI Investigator has a unique personality:

- **Bubbly & Enthusiastic**: Makes investigation fun and engaging
- **Detail-Oriented**: Never compromises on analytical rigor
- **Systematic**: Follows DISARM framework methodically
- **Politically Neutral**: Absolute objectivity, no "taking sides"
- **Evidence-Based**: Data-driven conclusions only
- **Friendly Communicator**: Explains findings clearly

Think of it as having a cheerful detective friend who REALLY loves finding patterns but stays completely objective about what those patterns mean!

## Limitations

- **VLM Accuracy**: Not perfect - may miss subtle manipulations
- **Logo Database**: Limited to 142+ logos (focused on Russia-Ukraine conflict)
- **Image Quality**: Poor quality images harder to analyze
- **Cultural Context**: Some cultural nuances may be missed
- **API Dependency**: Requires Ollama API access
- **Computational Cost**: VLM analysis is resource-intensive

## Future Enhancements

- Video analysis capabilities
- Audio/multimedia analysis
- Real-time stream processing
- Advanced deepfake detection
- Network analysis integration
- Temporal trend tracking
- Cross-platform correlation

## Related Resources

**In This Project:**
- [vision.ipynb](vision.ipynb) - VLM notebook template
- [logos/logos.csv](logos/logos.csv) - Logo database
- [experiment/FIMI_Investigation_Report.md](experiment/FIMI_Investigation_Report.md) - Example report

**External:**
- [DISARM Framework](https://github.com/DISARMFoundation/DISARMframeworks)
- [Ollama Documentation](https://ollama.ai/docs)
- Digital Frontlines Project

## Contributing

This pack is part of the Digital Frontlines project. Contributions welcome:

- Add more logos to database
- Improve technique detection algorithms
- Enhance VLM prompts
- Add new DISARM techniques
- Improve report templates

## License

Part of the Digital Frontlines OSINT toolkit.

---

**Happy Investigating! 🔍🎨**

*Remember: We detect manipulation techniques objectively, regardless of which "side" uses them. Evidence-based analysis only!*
