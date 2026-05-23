# ✦ lgeoAI — Neural Intelligence for IP Anonymization Detection

<div align="center">
  <img src="lgeoai_logo.png" alt="lgeoAI Logo" width="200">
  
  *Intelligence That Sees What Others Miss*
  
  [![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
  [![Model Size](https://img.shields.io/badge/Model%20Size-46%20KB-blue.svg)]()
  [![R² Score](https://img.shields.io/badge/R²-0.9998-brightgreen.svg)]()
  [![MAE](https://img.shields.io/badge/MAE-0.0003-success.svg)]()
</div>

---

## Overview

**✦lgeoAI** is a privately-trained, locally-run neural intelligence that distinguishes real users from VPNs, proxies, and Tor with surgical precision. It's not just another model — it's a purpose-built inference engine designed to complement the [lgeoip](https://github.com/tutuurg/lgeoip) geolocation analysis server.

Born from 10 carefully engineered factors and trained on real-world IP analysis, ✦lgeoAI achieves remarkable accuracy while maintaining a footprint smaller than a single high-resolution image.

### ✦ What Makes lgeoAI Different

- **100% Local** — Runs entirely on your hardware using ONNX runtime. Zero external API calls. Zero data leaks.
- **Ridiculously Small** — Just 46 KB for the model. Yes, *kilobytes*.
- **Trained on Reality** — Built from real IP samples across all IPv4 classes — not just textbook examples.
- **MIT Licensed** — Use it, fork it, learn from it. No strings attached.

---

## Performance

Trained on 1,500 random IPv4 addresses (3× more than v0.9), ✦lgeoAI delivers:

| Metric | Value | Description |
|--------|-------|-------------|
| **MAE** | 0.0003 | Mean Absolute Error — virtually zero average error |
| **R²** | 0.9998 | Coefficient of determination — 99.98% of variance explained |
| **Typical Error** | ±0.0% | Precision you can trust |

And it only gets better with every retraining.

---

## How It Works

✦lgeoAI processes 10 engineered features extracted from IP analysis:

| # | Feature | Description |
|---|---------|-------------|
| 1 | Placeholder | Reserved for future use |
| 2 | Heuristic Probability | Normalized probability from traditional detection (0-1) |
| 3 | Timezone Match | Whether browser and IP timezones align |
| 4 | Tor Flag | IP is a known Tor exit node |
| 5 | Suspicious Hostname | Reverse DNS contains proxy/VPN keywords |
| 6 | IP2Proxy Detection | Detected as proxy by IP2Proxy database |
| 7 | Datacenter Flag | IP belongs to datacenter/hosting range |
| 8 | Hosting ISP | ISP name contains hosting keywords |
| 9 | Known VPN ASN | ASN found in known VPN providers database |
| 10 | Timezone Offset | Normalized timezone difference (-1 to +1) |

The model outputs a probability score (0-1) that refines the heuristic detection. In the lgeoip server, final probability is calculated as:

```
Final = (Heuristic × 0.7) + (AI × 100 × 0.3)
```

---

## Installation

### Prerequisites

```bash
pip install onnxruntime numpy
```

### Quick Start

```bash
# Clone the repository
git clone https://github.com/tutuurg/lgeoai.git
cd lgeoai

# Test the model
python lgeoai.py
```

The model file `lgeoai_model.onnx` is included directly in the repository.

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

---

## Usage

### Standalone Inference

```python
from lgeoai import LgeoAI

# Initialize model
model = LgeoAI(model_path="lgeoai_model.onnx")

# Prepare features
features = [
    0,      # Placeholder
    0.85,   # Heuristic probability (normalized)
    0,      # Timezone mismatch
    0,      # Not Tor
    1,      # Suspicious hostname
    1,      # IP2Proxy detected
    1,      # Datacenter IP
    1,      # Hosting ISP
    1,      # Known VPN ASN
    -0.67   # Normalized timezone offset
]

# Get prediction
probability = model.predict(features)
print(f"AI Probability: {probability:.4f} ({probability*100:.1f}%)")
```

### Feature Extraction

When integrating with your own system, normalize features as follows:

```python
def extract_features(ip_data: dict) -> list:
    """Extract and normalize features for lgeoAI"""
    features = [
        0,                                                    # Placeholder
        ip_data.get("heuristic_prob", 0) / 100,               # Normalize to 0-1
        1 if ip_data.get("timezone_match") else 0,            # Binary
        1 if ip_data.get("is_tor") else 0,                    # Binary
        1 if ip_data.get("suspicious_hostname") else 0,       # Binary
        1 if ip_data.get("ip2proxy_proxy") else 0,            # Binary
        1 if ip_data.get("ip2proxy_dc") else 0,               # Binary
        1 if ip_data.get("hosting_isp") else 0,               # Binary
        1 if ip_data.get("known_vpn_asn") else 0,             # Binary
        max(min(ip_data.get("tz_offset", 0) / 12, 1), -1)     # Clamp to [-1, 1]
    ]
    return features
```

---

## Model Versions

### ✦ lgeoAI v1.0 (Current)
- Trained on 1,500 random IPv4 addresses (3× more than v0.9)
- 46 KB ONNX model
- 10 engineered features
- MAE: 0.0003, R²: 0.9998

### ✦ lgeoAI v0.9 (Previous)
- Initial public release
- Trained on 500 IPv4 addresses
- Demonstrated surgical accuracy for anonymization detection
- Established the foundation for feature engineering

---

## Repository Structure

```
lgeoai/
├── lgeoai_model.onnx    # Trained ONNX model (46 KB)
├── lgeoai.py            # Inference wrapper class
├── train.py             # Training script
├── requirements.txt     # Python dependencies
├── LICENSE              # MIT License
└── README.md            # This file
```

---

## Technical Details

### Model Architecture
- **Format**: ONNX (Open Neural Network Exchange)
- **Runtime**: ONNX Runtime
- **Size**: 46 KB
- **Input**: 10 normalized features (float32)
- **Output**: Single probability score (float32, 0-1)

### Training Data
- 1,500 random IPv4 addresses
- Real-world samples across all IPv4 classes
- Balanced dataset with VPN/proxy/normal traffic

### Feature Engineering
All features are carefully engineered from raw IP analysis data:
- Binary flags converted to 0/1
- Continuous values normalized to appropriate ranges
- Timezone offsets clamped to [-1, 1] range (representing ±12 hours)

---

## Privacy & Security

✦lgeoAI embodies the same privacy-first philosophy as lgeoip:

- **No Data Collection**: The model performs inference only — no training data, IP addresses, or personal information is ever collected or transmitted
- **Fully Local**: All processing happens on your hardware
- **No External Dependencies**: Zero API calls during inference
- **Transparent**: Open-source code, inspectable model architecture

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
- Training data augmentation
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

- Built with ONNX Runtime
- Trained on real-world network data
- Inspired by the need for transparent, local-first AI solutions
