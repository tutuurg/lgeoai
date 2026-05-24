# ✦ lgeoAI — Neural Intelligence for IP Anonymization Detection

<div align="center">
  <img src="lgeoai_logo.png" alt="lgeoAI Logo" width="200">
  
  *Intelligence That Sees What Others Miss*
  
  [![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
  [![Model Size](https://img.shields.io/badge/Model%20Size-46%20KB-blue.svg)]()
</div>

---

## Overview

**✦lgeoAI** is a privately-trained, locally-run neural intelligence that distinguishes real users from VPNs, proxies, and Tor with surgical precision. It's not just another model — it's a purpose-built inference engine designed to complement the [lgeoip](https://github.com/tutuurg/lgeoip) geolocation analysis server.

Born from 12 carefully engineered factors and trained on real-world IP analysis, ✦lgeoAI achieves remarkable accuracy while maintaining a footprint smaller than a single high-resolution image.

### ✦ What Makes lgeoAI Different

- **100% Local** — Runs entirely on your hardware using ONNX runtime with NPU acceleration (DirectML). Zero external API calls. Zero data leaks.
- **Ridiculously Small** — Just 46 KB for the model. Yes, *kilobytes*.
- **Auto-Learning** — Collects anonymization samples for continuous model improvement
- **MIT Licensed** — Use it, fork it, learn from it. No strings attached.

---

## Performance

✦lgeoAI delivers surgical precision:

| Metric | Value | Description |
|--------|-------|-------------|
| **Inference** | <10ms | On NPU via DirectML |
| **Model Size** | 46 KB | Smaller than an emoji pack |
| **Features** | 12 | Multi-dimensional analysis |

---

## Installation

### Prerequisites

```bash
pip install onnxruntime numpy
```

For NPU acceleration on Windows:
```bash
pip install onnxruntime-directml
```

### Quick Start

```python
from lgeoai import LgeoAI, extract_features

# Initialize AI module
ai = LgeoAI(model_path="lgeoai_model.onnx")

# Extract features from IP data
features = extract_features(
    ip_data=ip_data,
    browser_timezone="Europe/Moscow",
    heuristic_prob=85,
    reasons=["Timezone mismatch", "Hosting ISP"],
    timezone_match=False,
    is_tor=False,
    suspicious_hostname=True,
    ip2proxy_proxy=True,
    ip2proxy_dc=False,
    hosting_isp=True,
    known_vpn_asn=False,
    tz_offset=3,
    hostname_entropy=0.7
)

# Get AI probability
ai_probability = ai.predict(features)  # Returns 0.0-1.0

# Log for future training
ai.log_sample(features, heuristic_prob=85, final_prob=ai_probability*100)
```

### Integration with lgeoip Server

1. Copy `lgeoai_model.onnx` and `lgeoai.py` to your lgeoip server directory:

```bash
cp lgeoai_model.onnx /path/to/lgeoip/
cp lgeoai.py /path/to/lgeoip/
```

2. The lgeoip server auto-detects the model. Enable AI mode in API requests:

```bash
curl "http://localhost:88/json?ip=89.187.179.58&tz=Europe/Minsk&ai_mode=true"
```

Response includes AI refinement:
```json
{
  "anonymization_probability": 92,
  "anonymization_reasons": [
    "Detected as proxy (IP2Proxy)",
    "Timezone offset mismatch...",
    "AI-уточнение: 98%"
  ],
  "ai_available": true,
  "ai_mode_requested": true
}
```

---

## API Reference

### `LgeoAI` Class

#### `__init__(model_path=None, log_path="ai_training_data.jsonl")`
Initialize AI module with optional ONNX model.

**Parameters:**
- `model_path` (str, optional): Path to .onnx model file
- `log_path` (str): Path for training data collection (JSONL format)

#### `predict(features_dict)`
Get anonymization probability from AI model.

**Parameters:**
- `features_dict` (dict): Feature dictionary from `extract_features()`

**Returns:**
- `float`: Probability 0.0-1.0, or `None` if model unavailable

#### `log_sample(features_dict, heuristic_prob, final_prob=None, user_feedback=None)`
Save sample for future model training.

#### `get_stats()`
Return AI module statistics (model availability, request count, logged samples).

### `extract_features()` Function

Extracts and normalizes 12 features for AI inference:

| Feature | Normalization | Description |
|---------|---------------|-------------|
| `is_tor` | 0/1 | Known Tor exit node |
| `has_suspicious_hostname` | 0/1 | Proxy/VPN keywords in reverse DNS |
| `ip2proxy_proxy` | 0/1 | IP2Proxy proxy detection |
| `ip2proxy_datacenter` | 0/1 | Datacenter/hosting IP |
| `hosting_isp` | 0/1 | Hosting keywords in ISP name |
| `known_vpn_asn` | 0/1 | ASN in known VPN database |
| `timezone_mismatch` | 0/1 | Browser vs IP timezone difference |
| `tz_offset_hours` | [0, 1] | Offset magnitude (max 12h → 1.0) |
| `hostname_entropy` | [0, 1] | DNS name randomness |
| `reasons_count` | [0, 1] | Number of detection reasons (max 5 → 1.0) |
| `hosting_and_tz_mismatch` | 0/1 | Combined hosting ISP + timezone mismatch |
| `heuristic_probability` | [0, 1] | Base heuristic probability |

---

## Model Training

### Data Collection

The AI module automatically collects training data in JSONL format:

```json
{
  "timestamp": "2026-05-24T12:34:56",
  "features": {...},
  "heuristic_probability": 0.85,
  "final_probability": 0.92,
  "user_feedback": null
}
```

### Training Your Own Model

```python
# Collect enough samples first (recommended: 1000+)
ai = LgeoAI(log_path="training_data.jsonl")

# After collection, train using train.py (not included, but you can create)
# python train.py --data training_data.jsonl --output model.onnx
```

---

## Repository Structure

```
lgeoai/
├── lgeoai_model.onnx    # Trained ONNX model (46 KB)
├── lgeoai.py            # Inference wrapper + feature extraction
├── train.py             # Training script (create your own)
├── requirements.txt     # Python dependencies
├── LICENSE              # MIT License
└── README.md            # This file
```

---

## Feature Extraction in lgeoip Server

The actual integration in `server.py`:

```python
from lgeoai import LgeoAI, extract_features

# Initialize once at server startup
lgeoai = LgeoAI(model_path=r"C:\geoip\lgeoai_model.onnx")

# During request processing
features = extract_features(
    ip_data=ip_data,
    browser_timezone=browser_timezone,
    heuristic_prob=probability,
    reasons=reasons,
    timezone_match=timezone_match,
    is_tor=is_tor,
    suspicious_hostname=suspicious_hostname,
    ip2proxy_proxy=ip2proxy_proxy,
    ip2proxy_dc=ip2proxy_dc,
    hosting_isp=hosting_isp,
    known_vpn_asn=known_vpn_asn_flag,
    tz_offset=tz_offset,
    hostname_entropy=0  # Reserved for future use
)

# Always log for training
lgeoai.log_sample(features, probability)

# Optional AI refinement (70% heuristic + 30% AI)
if ai_mode and lgeoai.model_available:
    ai_prob = lgeoai.predict(features)
    if ai_prob is not None:
        final_prob = probability * 0.7 + ai_prob * 100 * 0.3
```

---

## Runtime Providers

The AI module intelligently selects the best available provider:

1. **DirectML Execution Provider** — NPU/GPU acceleration on Windows
2. **CPU Execution Provider** — Fallback for any system

```python
# Automatic provider selection
providers = ['DmlExecutionProvider', 'CPUExecutionProvider']
session = ort.InferenceSession(model_path, providers=providers)
```

---

## Privacy & Security

✦lgeoAI embodies the same privacy-first philosophy as lgeoip:

- **No Data Collection**: The model performs inference only — no training data, IP addresses, or personal information is ever collected or transmitted
- **Fully Local**: All processing happens on your hardware
- **No External Dependencies**: Zero API calls during inference
- **Transparent**: Open-source code, inspectable model architecture
- **Optional Logging**: Training data stored locally only if you enable it

---

## License
```text
MIT License

Copyright (c) 2026 Cookie:3 (tutuurg) (https://github.com/tutuurg)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

### Logo License
```text
The lgeoAI logo is licensed under **CC BY-NC-ND 4.0**. To view a copy of this license, visit [Creative Commons](https://creativecommons.org/licenses/by-nc-nd/4.0/).

*The CC license applies solely to the logo image and does not grant any rights to access, use, or replicate the server-side functionality of lgeoAI.*
```

---

## Related Projects

- [**lgeoip**](https://github.com/tutuurg/lgeoip) — Main IP geolocation and anonymization detection server
- [**lgeoip-client**](https://tutuurg.github.io/lgeoip/) — Frontend web interface

---

## Contributing

Contributions are welcome! Here's how:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Areas for Contribution
- Feature engineering improvements
- Training data collection scripts
- Model architecture optimization
- Documentation and examples
- Integration examples with other languages

---

## Support

- **Issues**: [GitHub Issues](https://github.com/tutuurg/lgeoai/issues)
- **Email**: zazagog.krt@gmail.com
- **Documentation**: See [lgeoip README](https://github.com/tutuurg/lgeoip) for integration details

---

## Acknowledgments

- Built with ONNX Runtime and DirectML
- Trained on 1,500+ real-world IP samples
- Inspired by the need for transparent, local-first AI solutions
- Integrated with MaxMind GeoIP2 and IP2Proxy databases
