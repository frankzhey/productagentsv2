# Validation Command

Run from repository root:

```bash
python3 - <<'PY'
from pathlib import Path
import xml.etree.ElementTree as ET
folder = Path('Project/spk2challenge-miniprogram/Solution/Engdesign/speaking-challenge-and-scoring-engdesign')
for svg in sorted(folder.glob('*.svg')):
    ET.parse(svg)
    print(f'valid: {svg.name}')
PY
```

PNG export can use `cairosvg` when available:

```bash
python3 -m cairosvg input.svg -o output.png
```
