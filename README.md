# PhishGuard AI — Phishing Detection Platform

A full-featured, AI-powered phishing detection web application built as a single-file platform.

---

## Features

| Module | Description |
|---|---|
| **Dashboard** | Live stats, detection trends, threat categories, origin map, activity feed |
| **URL Analyzer** | AI analysis of single URLs with risk scoring, indicators, and verdict |
| **Bulk Scanner** | Scan up to 100 URLs at once via paste input |
| **Email Scanner** | Paste raw email content for phishing detection (sender spoofing, DKIM, urgency analysis) |
| **Threat Log** | Full filterable log of all detected threats with block/details actions |
| **Reports** | Monthly charts and AI model performance metrics |
| **Settings** | Toggle AI features, alerts, data retention, privacy controls |

---

## Quick Start (No Backend Needed)

```bash
# Option 1: Open directly in browser
open index.html

# Option 2: Serve with Python
python3 -m http.server 8080
# Visit http://localhost:8080

# Option 3: Serve with Node.js
npx serve .
# Visit http://localhost:3000
```

---

## Production Setup (With Real AI Backend)

### 1. Install Dependencies

```bash
npm init -y
npm install express cors axios dotenv
pip install flask transformers torch scikit-learn requests phishing-detective
```

### 2. Backend API (Node.js / Express)

```js
// server.js
const express = require('express');
const cors = require('cors');
const app = express();
app.use(cors());
app.use(express.json());

// POST /api/v2/analyze
app.post('/api/v2/analyze', async (req, res) => {
  const { url, deep_scan } = req.body;
  
  // Feature extraction
  const features = extractFeatures(url);
  
  // Call your ML model
  const score = await mlModel.predict(features);
  
  res.json({
    url,
    risk_score: score,
    verdict: score > 0.8 ? 'phishing' : score > 0.5 ? 'suspicious' : 'safe',
    confidence: Math.round(score * 100),
    features,
    analyzed_at: new Date().toISOString()
  });
});

app.listen(3001, () => console.log('PhishGuard API running on :3001'));
```

### 3. Python ML Model (Scikit-learn / Transformers)

```python
# model.py
from sklearn.ensemble import RandomForestClassifier
from transformers import pipeline
import re, urllib.parse, whois, ssl, socket

def extract_features(url):
    parsed = urllib.parse.urlparse(url)
    return {
        'url_length': len(url),
        'has_ip': bool(re.match(r'\d+\.\d+\.\d+\.\d+', parsed.netloc)),
        'num_dots': url.count('.'),
        'has_at': '@' in url,
        'has_suspicious_tld': parsed.netloc.endswith(('.ru','.tk','.xyz','.ml','.ga')),
        'has_typosquat': detect_typosquat(parsed.netloc),
        'path_depth': len(parsed.path.split('/')) - 1,
        'has_token': bool(re.search(r'token|verify|confirm|login', url, re.I)),
        'domain_age_days': get_domain_age(parsed.netloc),
        'ssl_valid': check_ssl(parsed.netloc),
    }

# Train model
clf = RandomForestClassifier(n_estimators=200, max_depth=15)
clf.fit(X_train, y_train)

# Save model
import joblib
joblib.dump(clf, 'phishguard_model.pkl')
```

### 4. Connect Frontend to Backend

In `index.html`, replace the `analyzeURL()` function's mock logic:

```js
async function analyzeURL() {
  const url = document.getElementById('url-input').value.trim();
  if (!url) return;
  
  showLoading(true);
  
  try {
    const res = await fetch('http://localhost:3001/api/v2/analyze', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ url, deep_scan: true })
    });
    const data = await res.json();
    showURLResult(url, data);
  } catch (err) {
    console.error('Analysis failed:', err);
  } finally {
    showLoading(false);
  }
}
```

---

## AI Model Options

| Model | Accuracy | Speed | Use Case |
|---|---|---|---|
| RandomForest (sklearn) | ~94% | Fast | Production default |
| XGBoost | ~96% | Medium | Better precision |
| BERT fine-tuned | ~98.5% | Slow | Highest accuracy |
| Claude API (Anthropic) | ~99% | API call | Zero training needed |

### Using Claude API for Analysis

```js
// In your Express backend
const Anthropic = require('@anthropic-ai/sdk');
const client = new Anthropic();

app.post('/api/v2/analyze', async (req, res) => {
  const { url } = req.body;
  
  const message = await client.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 500,
    messages: [{
      role: 'user',
      content: `Analyze this URL for phishing indicators and return JSON:
URL: ${url}
Return: { risk_score: 0-100, verdict: "phishing|suspicious|safe", indicators: [], explanation: "" }`
    }]
  });
  
  const result = JSON.parse(message.content[0].text);
  res.json(result);
});
```

---

## Threat Intelligence Sources

Integrate these free APIs for enhanced detection:

```bash
# PhishTank (free API)
curl "https://checkurl.phishtank.com/checkurl/" \
  -d "url=BASE64_ENCODED_URL&format=json&app_key=YOUR_KEY"

# Google Safe Browsing
curl "https://safebrowsing.googleapis.com/v4/threatMatches:find?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"client":{"clientId":"phishguard"},"threatInfo":{"threatTypes":["MALWARE","SOCIAL_ENGINEERING"],"threatEntryTypes":["URL"],"threatEntries":[{"url":"TARGET_URL"}]}}'

# VirusTotal
curl "https://www.virustotal.com/api/v3/urls" \
  -H "x-apikey: YOUR_VT_KEY" \
  --data-urlencode "url=TARGET_URL"
```

---

## Environment Variables

```env
PORT=3001
PHISHTANK_API_KEY=your_key_here
GOOGLE_SAFE_BROWSING_KEY=your_key_here
VIRUSTOTAL_API_KEY=your_key_here
ANTHROPIC_API_KEY=sk-ant-...
JWT_SECRET=your_secret
DB_URL=postgresql://localhost/phishguard
```

---

## Project Structure (Full Stack)

```
phishguard/
├── index.html          ← Frontend (this file)
├── server.js           ← Express API server
├── model/
│   ├── train.py        ← ML model training
│   ├── predict.py      ← Inference endpoint
│   └── phishguard.pkl  ← Saved model
├── data/
│   ├── phishing_urls.csv
│   └── benign_urls.csv
├── .env
└── package.json
```

---

## Dataset for Training

- **PhishTank**: https://phishtank.org/developer_info.php
- **OpenPhish**: https://openphish.com/
- **Alexa Top 1M** (benign): for safe URL samples
- **ISCXURL2016**: benchmark phishing dataset

---

## License

MIT License — free for educational and commercial use.
