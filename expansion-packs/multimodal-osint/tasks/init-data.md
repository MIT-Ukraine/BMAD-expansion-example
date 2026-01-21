<!-- Powered by BMAD™ Multimodal OSINT Expansion Pack -->

# init-data

Explore and understand dataset structure without conversion - works with data in native format.

## Purpose

Automatically detect data structure, identify text/image columns, assess multimodal capabilities, and create investigation manifest. Works with data AS-IS in its native format (JSON, JSONL, CSV, Parquet, SQLite).

## Inputs

```yaml
required:
  - data_path: '{path_to_data}' # File or directory

optional:
  - platform: '{platform_name}' # telegram, twitter, facebook, etc. (helps with column detection)
  - investigation_id: '{investigation_name}' # For manifest creation
```

## Process

### Step 1: Detect File Type and Structure

```python
from pathlib import Path
import pandas as pd
import json

data_path_obj = Path(data_path)

print(f"📂 Exploring: {data_path}")

# Determine if directory or file
if data_path_obj.is_dir():
    # Find primary data file in directory
    json_files = list(data_path_obj.glob('*.json')) + list(data_path_obj.glob('*.jsonl'))
    csv_files = list(data_path_obj.glob('*.csv'))
    parquet_files = list(data_path_obj.glob('*.parquet')) + list(data_path_obj.glob('*.pq'))

    if json_files:
        primary_file = json_files[0]
        file_type = '.jsonl' if 'jsonl' in str(primary_file) else '.json'
    elif csv_files:
        primary_file = csv_files[0]
        file_type = '.csv'
    elif parquet_files:
        primary_file = parquet_files[0]
        file_type = '.parquet'
    else:
        raise ValueError("No supported data files found in directory")

    print(f"📄 Primary file: {primary_file}")

else:
    # Single file
    primary_file = data_path_obj
    file_type = primary_file.suffix

print(f"✓ File type: {file_type}")
```

### Step 2: Load Sample and Detect Schema

```python
# Load small sample to detect schema
print(f"📊 Loading sample to detect schema...")

if file_type in ['.json', '.jsonl', '.ndjson']:
    try:
        sample = pd.read_json(primary_file, lines=True, nrows=100)
    except:
        with open(primary_file) as f:
            data_json = json.load(f)
            if isinstance(data_json, list):
                sample = pd.DataFrame(data_json[:100])
            else:
                sample = pd.DataFrame([data_json])
elif file_type == '.csv':
    sample = pd.read_csv(primary_file, nrows=100)
elif file_type in ['.parquet', '.pq']:
    sample = pd.read_parquet(primary_file).head(100)
else:
    raise ValueError(f"Unsupported file type: {file_type}")

print(f"✓ Loaded sample: {len(sample)} records")
print(f"  Columns: {list(sample.columns)}")

# Get full count
if file_type in ['.json', '.jsonl']:
    # Count lines for JSONL
    with open(primary_file) as f:
        total_records = sum(1 for _ in f)
elif file_type == '.csv':
    total_records = sum(1 for _ in open(primary_file)) - 1  # -1 for header
else:
    full_data = pd.read_parquet(primary_file) if file_type == '.parquet' else sample
    total_records = len(full_data)

print(f"✓ Total records: {total_records}")
```

### Step 3: Auto-Detect Column Roles

```python
def detect_schema(df, platform=None):
    """Auto-detect text, image, ID, date columns"""

    schema = {
        'text_columns': [],
        'image_columns': [],
        'id_column': None,
        'date_column': None,
        'metadata_columns': []
    }

    # Common column name patterns by platform
    text_patterns = ['text', 'message', 'content', 'caption', 'body', 'post_text']
    image_patterns = ['image', 'photo', 'img', 'media', 'picture', 'file_path', 'image_path']
    id_patterns = ['id', 'post_id', 'message_id', 'tweet_id', '_id']
    date_patterns = ['date', 'time', 'created', 'published', 'timestamp']

    for col in df.columns:
        col_lower = col.lower()

        # Text columns
        if any(pattern in col_lower for pattern in text_patterns):
            schema['text_columns'].append(col)

        # Image columns
        elif any(pattern in col_lower for pattern in image_patterns):
            schema['image_columns'].append(col)

        # ID column
        elif any(pattern in col_lower for pattern in id_patterns):
            if schema['id_column'] is None:
                schema['id_column'] = col

        # Date column
        elif any(pattern in col_lower for pattern in date_patterns):
            if schema['date_column'] is None:
                schema['date_column'] = col

        else:
            schema['metadata_columns'].append(col)

    return schema

schema = detect_schema(sample, platform)

print(f"\n📋 Schema detected:")
print(f"  Text columns: {schema['text_columns']}")
print(f"  Image columns: {schema['image_columns']}")
print(f"  ID column: {schema['id_column']}")
print(f"  Date column: {schema['date_column']}")
print(f"  Metadata columns: {schema['metadata_columns']}")
```

### Step 4: Assess Multimodal Capabilities

```python
# Check image availability
image_col = schema['image_columns'][0] if schema['image_columns'] else None

if image_col:
    images_present = sample[image_col].notna().sum()
    image_availability = images_present / len(sample)
    print(f"\n🖼️  Image availability: {image_availability:.1%} ({images_present}/{len(sample)} in sample)")
else:
    image_availability = 0.0
    print(f"\n⚠  No image column detected - text-only dataset")

# Date range
date_col = schema['date_column']
if date_col and date_col in sample.columns:
    try:
        sample[date_col] = pd.to_datetime(sample[date_col])
        date_range = {
            'earliest': sample[date_col].min().isoformat(),
            'latest': sample[date_col].max().isoformat()
        }
        print(f"📅 Date range: {date_range['earliest']} to {date_range['latest']}")
    except:
        date_range = None
        print(f"⚠  Could not parse dates in column: {date_col}")
else:
    date_range = None
```

### Step 5: Create Investigation Manifest

```python
# Create manifest
data_info = {
    'data_path': str(data_path),
    'primary_file': str(primary_file),
    'file_type': file_type,
    'total_records': total_records,
    'schema': schema,
    'image_availability': image_availability,
    'date_range': date_range,
    'platform': platform,
    'explored_at': datetime.now().isoformat()
}

# Save manifest if investigation_id provided
if investigation_id:
    manifest_path = Path(f"investigations/{investigation_id}/manifest.json")
    manifest_path.parent.mkdir(parents=True, exist_ok=True)

    with open(manifest_path, 'w') as f:
        json.dump(data_info, f, indent=2)

    print(f"\n💾 Manifest saved: {manifest_path}")

print(f"\n✓ Data exploration complete!")

return data_info
```

## Output

```python
{
    'data_path': 'data/telegram_posts/',
    'primary_file': 'data/telegram_posts/messages.jsonl',
    'file_type': '.jsonl',
    'total_records': 1523,
    'schema': {
        'text_columns': ['text', 'caption'],
        'image_columns': ['photo_path'],
        'id_column': 'message_id',
        'date_column': 'timestamp',
        'metadata_columns': ['channel', 'views', 'forwards']
    },
    'image_availability': 0.68,
    'date_range': {
        'earliest': '2022-02-24T...',
        'latest': '2024-12-31T...'
    },
    'platform': 'telegram',
    'explored_at': '2026-01-20T...'
}
```

## Integration

Called by [execute-hunt.md](execute-hunt.md) before hunting to understand data structure.
