# Frontend files

Combined snapshot of the active top-level React frontend. Generated dependencies (`node_modules/`, `package-lock.json`) and build output (`build/`) are intentionally excluded.

## `frontend\package.json`

````json
{
  "name": "phishing-detection-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.21.0",
    "recharts": "^2.10.1",
    "axios": "^1.6.2",
    "react-scripts": "5.0.1",
    "lucide-react": "^0.303.0",
    "jsqr": "^1.4.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": ["react-app"]
  },
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version", "last 1 firefox version"]
  },
  "proxy": "http://localhost:5000"
}
````

## `frontend\Dockerfile`

````dockerfile
# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Serve with Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
````

## `frontend\nginx.conf`

````text
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API to backend
    location /api/ {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
````

## `frontend\public\index.html`

````html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#0f1117" />
    <meta name="description" content="AI-Powered Phishing Detection System" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
    <title>PhishGuard AI</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
````

## `frontend\src\App.js`

````jsx
import React, { useState } from 'react';
import Dashboard from './pages/Dashboard';
import Scanner from './pages/Scanner';
import BulkScan from './pages/BulkScan';
import History from './pages/History';
import './App.css';

function App() {
  const [activePage, setActivePage] = useState('dashboard');

  const pages = {
    dashboard: <Dashboard />,
    scanner: <Scanner />,
    bulk: <BulkScan />,
    history: <History />,
  };

  return (
    <div className="app">
      <nav className="sidebar">
        <div className="sidebar-logo">
          <span className="logo-icon">🛡️</span>
          <span className="logo-text">PhishGuard AI</span>
        </div>
        <ul className="nav-links">
          {[
            { id: 'dashboard', icon: '📊', label: 'Dashboard' },
            { id: 'scanner', icon: '🔍', label: 'Scanner' },
            { id: 'bulk', icon: '📦', label: 'Bulk Scan' },
            { id: 'history', icon: '📋', label: 'History' },
          ].map(({ id, icon, label }) => (
            <li
              key={id}
              className={`nav-item ${activePage === id ? 'active' : ''}`}
              onClick={() => setActivePage(id)}
            >
              <span className="nav-icon">{icon}</span>
              <span>{label}</span>
            </li>
          ))}
        </ul>
      </nav>
      <main className="main-content">
        {pages[activePage]}
      </main>
    </div>
  );
}

export default App;
````

## `frontend\src\App.css`

````css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-primary: #0f1117;
  --bg-secondary: #1a1d27;
  --bg-card: #1e2130;
  --accent: #6366f1;
  --accent-glow: rgba(99, 102, 241, 0.3);
  --danger: #ef4444;
  --warning: #f59e0b;
  --success: #10b981;
  --text-primary: #f1f5f9;
  --text-secondary: #94a3b8;
  --border: rgba(255,255,255,0.08);
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  min-height: 100vh;
}

.app {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ─────────────────────────────────────────────── */
.sidebar {
  width: 240px;
  background: var(--bg-secondary);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  padding: 24px 0;
  position: fixed;
  height: 100vh;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 24px 32px;
  border-bottom: 1px solid var(--border);
  margin-bottom: 16px;
}

.logo-icon { font-size: 24px; }
.logo-text {
  font-size: 17px;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.3px;
}

.nav-links {
  list-style: none;
  padding: 0 12px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 11px 14px;
  border-radius: 8px;
  cursor: pointer;
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 500;
  transition: all 0.15s ease;
  margin-bottom: 4px;
}

.nav-item:hover {
  background: var(--bg-card);
  color: var(--text-primary);
}

.nav-item.active {
  background: var(--accent-glow);
  color: #818cf8;
  border: 1px solid rgba(99,102,241,0.25);
}

.nav-icon { font-size: 16px; }

/* ── Main Content ─────────────────────────────────────────── */
.main-content {
  flex: 1;
  margin-left: 240px;
  padding: 32px;
  min-height: 100vh;
}

/* ── Cards ───────────────────────────────────────────────── */
.card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 24px;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 16px;
}

/* ── Stat Cards ──────────────────────────────────────────── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.stat-card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
}

.stat-label {
  font-size: 12px;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 32px;
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1;
}

.stat-sub {
  font-size: 12px;
  color: var(--text-secondary);
  margin-top: 4px;
}

/* ── Badges ──────────────────────────────────────────────── */
.badge {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.badge-critical { background: rgba(239,68,68,0.15); color: #f87171; }
.badge-high     { background: rgba(245,158,11,0.15); color: #fbbf24; }
.badge-medium   { background: rgba(234,179,8,0.15);  color: #facc15; }
.badge-low      { background: rgba(16,185,129,0.15); color: #34d399; }

/* ── Scanner Form ────────────────────────────────────────── */
.scanner-form {
  display: flex;
  gap: 12px;
  margin-bottom: 24px;
}

.scanner-input {
  flex: 1;
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 12px 16px;
  color: var(--text-primary);
  font-size: 14px;
  outline: none;
  transition: border-color 0.15s;
}

.scanner-input:focus {
  border-color: var(--accent);
}

.btn-scan {
  background: var(--accent);
  color: white;
  border: none;
  border-radius: 8px;
  padding: 12px 24px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.15s;
}

.btn-scan:hover { opacity: 0.9; }
.btn-scan:disabled { opacity: 0.5; cursor: not-allowed; }

.bulk-card {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.bulk-textarea {
  min-height: 200px;
  resize: vertical;
}

.bulk-actions {
  display: flex;
  justify-content: flex-end;
}

.bulk-summary {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  color: var(--text-secondary);
  font-size: 13px;
  margin-bottom: 16px;
}

.bulk-results {
  display: grid;
  gap: 14px;
}

.bulk-result-item {
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 10px;
  padding: 14px;
}

.bulk-result-header,
.bulk-result-meta {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}

.bulk-result-type {
  font-size: 12px;
  padding: 4px 8px;
  border-radius: 999px;
  background: rgba(99,102,241,0.15);
  color: #c7d2fe;
  font-weight: 700;
}

.bulk-result-risk {
  font-weight: 700;
}

.bulk-result-input {
  color: var(--text-primary);
  margin: 10px 0;
  font-size: 13px;
  word-break: break-all;
}

.bulk-indicators {
  margin-top: 10px;
  color: var(--text-secondary);
  font-size: 12px;
  padding-left: 18px;
}

.qr-upload-box {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.qr-upload-label {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 14px 18px;
  border: 1px dashed rgba(99, 102, 241, 0.7);
  border-radius: 10px;
  background: rgba(99, 102, 241, 0.08);
  color: #c7d2fe;
  cursor: pointer;
  font-weight: 600;
}

.qr-upload-label input {
  display: none;
}

.qr-file-name,
.qr-help-text,
.qr-decoded-banner {
  color: var(--text-secondary);
  font-size: 13px;
}

.qr-decoded-banner {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 16px;
  padding: 12px 14px;
  border-radius: 8px;
  background: rgba(16, 185, 129, 0.08);
  border: 1px solid rgba(16, 185, 129, 0.2);
  color: #d1fae5;
}

/* ── Result Card ─────────────────────────────────────────── */
.result-card {
  background: var(--bg-card);
  border-radius: 12px;
  padding: 24px;
  margin-top: 16px;
}

.result-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}

.result-icon {
  font-size: 40px;
}

.result-title {
  font-size: 22px;
  font-weight: 700;
}

.risk-bar-container {
  margin: 16px 0;
}

.risk-bar-label {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: var(--text-secondary);
  margin-bottom: 6px;
}

.risk-bar {
  height: 8px;
  background: var(--bg-secondary);
  border-radius: 4px;
  overflow: hidden;
}

.risk-bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.8s ease;
}

.indicators-list {
  margin-top: 16px;
}

.indicator-item {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 8px 0;
  font-size: 14px;
  color: var(--text-secondary);
  border-bottom: 1px solid var(--border);
}

.indicator-item:last-child { border-bottom: none; }

.page-controls {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.search-input,
.filter-select {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 8px;
  color: var(--text-primary);
  padding: 12px 14px;
  font-size: 14px;
  outline: none;
}

.search-input {
  flex: 1;
  min-width: 220px;
}

.filter-select {
  min-width: 150px;
}

.recommendation-box {
  background: rgba(15, 17, 23, 0.85);
  border: 1px solid rgba(99, 102, 241, 0.18);
  border-radius: 12px;
  padding: 18px;
  display: grid;
  gap: 12px;
}

.recommendation-item {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 10px;
  padding: 12px 14px;
  color: var(--text-primary);
  font-size: 14px;
}

.result-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 18px;
}

.btn-report {
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 12px 20px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.15s ease;
}

.btn-report:hover:not(:disabled) {
  opacity: 0.92;
}

.btn-report:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.enrichment-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(210px, 1fr));
  gap: 12px;
  margin-top: 18px;
}

.enrichment-card {
  border: 1px solid rgba(148, 163, 184, 0.2);
  background: rgba(248, 250, 252, 0.06);
  border-radius: 12px;
  padding: 14px;
}

.enrichment-title {
  color: var(--text-secondary);
  font-size: 12px;
  font-weight: 600;
  margin-bottom: 10px;
}

.enrichment-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  color: var(--text-primary);
  font-size: 13px;
}

.analysis-details {
  margin-top: 18px;
  padding: 14px;
  background: rgba(248,250,252,0.95);
  border-radius: 12px;
  border: 1px solid rgba(148,163,184,0.2);
}

.analysis-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 12px;
}

.analysis-row {
  color: #475569;
}

.analysis-key {
  font-weight: 700;
  margin-bottom: 4px;
  text-transform: capitalize;
}

.analysis-value {
  font-size: 13px;
}
````

## `frontend\src\index.js`

````jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
````

## `frontend\src\pages\About.js`

````jsx
import React from 'react';

export default function About() {
  return (
    <div>
      <div style={{marginBottom:28}}>
        <h1 style={{fontSize:26,fontWeight:800,letterSpacing:'-0.5px'}}>ℹ️ About PhishGuard AI</h1>
        <p style={{color:'var(--text-secondary)',fontSize:14,marginTop:4}}>Project గురించి complete information</p>
      </div>

      <div style={{background:'var(--bg-card)',border:'1px solid var(--border)',borderLeft:'3px solid #6366f1',borderRadius:12,padding:22,marginBottom:16}}>
        <div style={{fontWeight:700,fontSize:15,marginBottom:12}}>🛡️ ఈ Project ఏమి చేస్తుంది?</div>
        <p style={{color:'var(--text-secondary)',fontSize:14,lineHeight:1.8}}>
          PhishGuard AI ఒక <b style={{color:'var(--text-primary)'}}>AI-powered phishing detection system</b>.
          URLs మరియు Emails analyze చేసి phishing indicators detect చేస్తుంది.
          Browser లోనే 100% పని చేస్తుంది — backend, internet అవసరం లేదు.
        </p>
      </div>

      <div style={{display:'grid',gridTemplateColumns:'1fr 1fr',gap:16,marginBottom:16}}>
        {[
          {title:'URL Analysis Features',icon:'🔗',pts:['HTTPS verification','IP address detection','Suspicious TLD check (.xyz .tk etc)','Brand impersonation detection','Subdomain depth analysis','URL length & entropy','Encoded character detection','Phishing path detection']},
          {title:'Email Analysis Features',icon:'📧',pts:['Sender domain verification','Brand spoofing detection','Subject urgency analysis','CAPS LOCK detection','Credential request detection','Phishing CTA detection','Link analysis in body','Free email provider check']},
        ].map(s=>(
          <div key={s.title} style={{background:'var(--bg-card)',border:'1px solid var(--border)',borderRadius:12,padding:22}}>
            <div style={{fontWeight:700,fontSize:15,marginBottom:14}}>{s.icon} {s.title}</div>
            {s.pts.map((p,i)=>(
              <div key={i} style={{fontSize:13,color:'var(--text-secondary)',padding:'5px 0',display:'flex',gap:8,borderBottom:'1px solid rgba(255,255,255,0.04)'}}>
                <span style={{color:'#6366f1',flexShrink:0}}>→</span>{p}
              </div>
            ))}
          </div>
        ))}
      </div>

      <div style={{background:'var(--bg-card)',border:'1px solid var(--border)',borderRadius:12,padding:22,marginBottom:16}}>
        <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>⚙️ Tech Stack</div>
        <div style={{display:'grid',gridTemplateColumns:'repeat(3,1fr)',gap:12}}>
          {[
            {layer:'Frontend',tech:'React.js + Recharts',icon:'⚛️',color:'#61dafb'},
            {layer:'AI Engine',tech:'Custom JS Algorithm',icon:'🧠',color:'#f59e0b'},
            {layer:'Storage',tech:'Browser localStorage',icon:'💾',color:'#10b981'},
            {layer:'Backend',tech:'Node.js + Express',icon:'🟢',color:'#6366f1'},
            {layer:'ML Service',tech:'Python + FastAPI',icon:'🐍',color:'#3b82f6'},
            {layer:'Database',tech:'PostgreSQL + Redis',icon:'🗄️',color:'#8b5cf6'},
          ].map(t=>(
            <div key={t.layer} style={{background:'var(--bg-secondary)',borderRadius:8,padding:14,borderLeft:`3px solid ${t.color}`}}>
              <div style={{fontSize:18,marginBottom:6}}>{t.icon}</div>
              <div style={{fontWeight:700,fontSize:13}}>{t.layer}</div>
              <div style={{fontSize:12,color:'var(--text-muted)',marginTop:2}}>{t.tech}</div>
            </div>
          ))}
        </div>
      </div>

      <div style={{background:'var(--bg-card)',border:'1px solid var(--border)',borderRadius:12,padding:22}}>
        <div style={{fontWeight:700,fontSize:15,marginBottom:14}}>📊 Risk Score Guide</div>
        {[
          {level:'CRITICAL',range:'70–100',color:'#ef4444',desc:'Definitely phishing. DO NOT visit or provide any information.'},
          {level:'HIGH',    range:'50–69', color:'#f59e0b',desc:'Very suspicious. Avoid and verify via official channels only.'},
          {level:'MEDIUM',  range:'30–49', color:'#eab308',desc:'Some suspicious signs. Proceed with extreme caution.'},
          {level:'LOW',     range:'0–29',  color:'#10b981',desc:'Likely safe. No major red flags detected.'},
        ].map(r=>(
          <div key={r.level} style={{display:'flex',alignItems:'center',gap:14,padding:14,background:'var(--bg-secondary)',borderRadius:8,marginBottom:8}}>
            <span style={{background:`${r.color}20`,color:r.color,padding:'4px 14px',borderRadius:20,fontSize:12,fontWeight:700,minWidth:88,textAlign:'center'}}>{r.level}</span>
            <span style={{fontSize:14,fontWeight:700,color:r.color,minWidth:55}}>{r.range}</span>
            <span style={{fontSize:13,color:'var(--text-secondary)'}}>{r.desc}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
````

## `frontend\src\pages\BulkScan.js`

````jsx
import React, { useMemo, useState } from 'react';
import axios from 'axios';

const RISK_COLORS = {
  CRITICAL: '#ef4444',
  HIGH: '#f59e0b',
  MEDIUM: '#eab308',
  LOW: '#10b981',
};

export default function BulkScan() {
  const [mode, setMode] = useState('urls');
  const [inputText, setInputText] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);

  const placeholder = useMemo(() => {
    if (mode === 'urls') {
      return 'https://example.com/login\nhttps://secure-bank-verify.xyz\nhttps://google.com';
    }

    return 'alice@example.com, suspicious@example.com\nsubject: Verify your account\nbody: Click here to update your password';
  }, [mode]);

  const parseInput = () => {
    const lines = inputText
      .split(/\r?\n/)
      .map((value) => value.trim())
      .filter(Boolean);

    if (mode === 'urls') {
      return { urls: lines.filter((value) => value.includes('http')) };
    }

    const emails = [];
    const current = { sender: '', subject: '', body: '' };

    lines.forEach((line) => {
      if (line.toLowerCase().startsWith('sender:')) {
        current.sender = line.split(':').slice(1).join(':').trim();
      } else if (line.toLowerCase().startsWith('subject:')) {
        current.subject = line.split(':').slice(1).join(':').trim();
      } else if (line.toLowerCase().startsWith('body:')) {
        current.body = line.split(':').slice(1).join(':').trim();
      } else if (line.includes('@')) {
        current.sender = line.trim();
      } else if (line) {
        current.body = (current.body ? current.body + '\n' : '') + line;
      }

      if (current.sender && current.body) {
        emails.push({ sender: current.sender, subject: current.subject || 'Bulk email check', body: current.body });
        current.sender = '';
        current.subject = '';
        current.body = '';
      }
    });

    if (current.sender || current.body || current.subject) {
      emails.push({ sender: current.sender || 'unknown@example.com', subject: current.subject || 'Bulk email check', body: current.body || 'No body provided' });
    }

    return { emails };
  };

  const handleScan = async () => {
    setLoading(true);
    setError(null);
    setResult(null);

    try {
      const payload = parseInput();
      const hasEntries = (payload.urls && payload.urls.length > 0) || (payload.emails && payload.emails.length > 0);

      if (!hasEntries) {
        throw new Error('Add at least one valid URL or email entry before scanning.');
      }

      const response = await axios.post('/api/scan/batch', payload);
      setResult(response.data);
    } catch (err) {
      const message = err.response?.data?.error || err.message || 'Bulk scan failed';
      setError(message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h1 style={{ fontSize: 24, fontWeight: 700, marginBottom: 8 }}>Bulk Scan</h1>
      <p style={{ color: '#94a3b8', marginBottom: 24, fontSize: 14 }}>
        Run a batch scan across multiple URLs or suspicious email entries.
      </p>

      <div style={{ display: 'flex', gap: 8, marginBottom: 20, flexWrap: 'wrap' }}>
        {[
          { id: 'urls', label: '🔗 URL Batch' },
          { id: 'emails', label: '📧 Email Batch' },
        ].map((tab) => (
          <button
            key={tab.id}
            onClick={() => {
              setMode(tab.id);
              setResult(null);
              setError(null);
            }}
            style={{
              padding: '8px 20px',
              borderRadius: 8,
              border: '1px solid',
              cursor: 'pointer',
              fontSize: 14,
              fontWeight: 600,
              background: mode === tab.id ? 'rgba(99,102,241,0.2)' : 'transparent',
              borderColor: mode === tab.id ? '#6366f1' : 'rgba(255,255,255,0.1)',
              color: mode === tab.id ? '#818cf8' : '#94a3b8',
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>

      <div className="card bulk-card">
        <textarea
          className="scanner-input bulk-textarea"
          value={inputText}
          onChange={(e) => setInputText(e.target.value)}
          placeholder={placeholder}
        />

        <div className="bulk-actions">
          <button className="btn-scan" onClick={handleScan} disabled={loading || !inputText.trim()}>
            {loading ? '⏳ Scanning...' : mode === 'urls' ? '🔎 Scan URLs' : '📨 Scan Emails'}
          </button>
        </div>
      </div>

      {error && (
        <div style={{ marginTop: 16, background: 'rgba(239,68,68,0.08)', border: '1px solid rgba(239,68,68,0.3)', borderRadius: 8, padding: 16, color: '#fca5a5' }}>
          ❌ {error}
        </div>
      )}

      {result && (
        <div className="card" style={{ marginTop: 20 }}>
          <div className="card-title">Batch Summary</div>
          <div className="bulk-summary">
            <div>Total scanned: {result.total}</div>
            <div>Batch ID: {result.batch_id}</div>
            <div>Checked at: {new Date(result.scanned_at).toLocaleString()}</div>
          </div>

          <div className="bulk-results">
            {result.results.map((entry, index) => (
              <div key={`${entry.type}-${index}`} className="bulk-result-item">
                <div className="bulk-result-header">
                  <span className="bulk-result-type">{entry.type === 'url' ? 'URL' : 'Email'}</span>
                  <span className="bulk-result-risk" style={{ color: RISK_COLORS[entry.risk_level] || '#94a3b8' }}>
                    {entry.risk_level || 'LOW'}
                  </span>
                </div>
                <div className="bulk-result-input">{entry.input || entry.url || entry.sender}</div>
                <div className="bulk-result-meta">
                  <span>Score: {entry.risk_score ?? 0}</span>
                  <span>Phishing: {entry.is_phishing ? 'Yes' : 'No'}</span>
                </div>
                {entry.indicators?.length > 0 && (
                  <ul className="bulk-indicators">
                    {entry.indicators.slice(0, 3).map((indicator, i) => (
                      <li key={i}>{indicator}</li>
                    ))}
                  </ul>
                )}
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
````

## `frontend\src\pages\Dashboard.js`

````jsx
import React, { useState, useEffect } from 'react';
import { 
    BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer, 
    PieChart, Pie, Cell, Legend 
} from 'recharts';

const weeklyData = [
  { day: 'Mon', scans: 42, phishing: 8 },
  { day: 'Tue', scans: 57, phishing: 14 },
  { day: 'Wed', scans: 38, phishing: 6 },
  { day: 'Thu', scans: 71, phishing: 22 },
  { day: 'Fri', scans: 65, phishing: 18 },
  { day: 'Sat', scans: 29, phishing: 4 },
  { day: 'Sun', scans: 33, phishing: 7 },
];

const threatTypes = [
  { name: 'URL Phishing', value: 45, color: '#ef4444' },
  { name: 'Email Scam', value: 30, color: '#f59e0b' },
  { name: 'Brand Spoof', value: 15, color: '#6366f1' },
  { name: 'Credential Harvest', value: 10, color: '#ec4899' },
];

const recentThreats = [
  { url: 'paypal-secure-verify.xyz/login', risk: 'CRITICAL', time: '2 min ago' },
  { url: 'apple-id-locked.tk/verify', risk: 'CRITICAL', time: '8 min ago' },
  { url: 'amazon-prize-winner.ml', risk: 'HIGH', time: '15 min ago' },
  { url: 'support.google.net-update.com', risk: 'HIGH', time: '23 min ago' },
  { url: 'bankofamerica.account.xyz', risk: 'CRITICAL', time: '41 min ago' },
];

const BADGE = {
  CRITICAL: { bg: 'rgba(239,68,68,0.15)', color: '#f87171' },
  HIGH: { bg: 'rgba(245,158,11,0.15)', color: '#fbbf24' },
  MEDIUM: { bg: 'rgba(234,179,8,0.15)', color: '#facc15' },
  LOW: { bg: 'rgba(16,185,129,0.15)', color: '#34d399' },
};

export default function Dashboard() {
  const [scanHistory, setScanHistory] = useState([]);
  const [scanStats, setScanStats] = useState({
    total_scans: 0,
    safe_scans: 0,
    phishing_scans: 0,
    threat_rate: 0,
  });

  useEffect(() => {
    fetch('/api/history')
      .then(res => res.json())
      .then(data => setScanHistory(data))
      .catch(err => console.error('Error:', err));

    fetch('/api/stats')
      .then(res => res.json())
      .then(stats => {
        if (!stats.error) setScanStats(stats);
      })
      .catch(err => console.error('Stats fetch error:', err));
  }, []);

  return (
    <div>
      <h1 style={{ fontSize: 24, fontWeight: 700, marginBottom: 4 }}>Threat Dashboard</h1>
      <p style={{ color: '#94a3b8', marginBottom: 28, fontSize: 14 }}>
        Real-time phishing detection overview
      </p>

      {/* Stats */}
      <div className="stats-grid">
        {[
          { label: 'Total Scans', value: scanStats.total_scans, sub: 'Total scans executed' },
          { label: 'Safe Scans', value: scanStats.safe_scans, sub: 'Verified safe results' },
          { label: 'Phishing Scans', value: scanStats.phishing_scans, sub: 'Potential threats flagged' },
          { label: 'Threat Rate', value: `${scanStats.threat_rate}%`, sub: 'Phishing detection rate' },
        ].map((s) => (
          <div className="stat-card" key={s.label}>
            <div className="stat-label">{s.label}</div>
            <div className="stat-value">{s.value}</div>
            <div className="stat-sub">{s.sub}</div>
          </div>
        ))}
      </div>

      {/* Charts Row */}
      <div style={{ display: 'grid', gridTemplateColumns: '2fr 1fr', gap: 16, marginBottom: 24 }}>
        {/* Bar Chart */}
        <div className="card">
          <div className="card-title">Weekly Scan Activity</div>
          <ResponsiveContainer width="100%" height={200}>
            <BarChart data={weeklyData} barGap={4}>
              <XAxis dataKey="day" tick={{ fill: '#94a3b8', fontSize: 12 }} axisLine={false} tickLine={false} />
              <YAxis tick={{ fill: '#94a3b8', fontSize: 12 }} axisLine={false} tickLine={false} />
              <Tooltip
                contentStyle={{ background: '#1e2130', border: '1px solid rgba(255,255,255,0.08)', borderRadius: 8 }}
                labelStyle={{ color: '#f1f5f9' }}
              />
              <Bar dataKey="scans" fill="#6366f1" radius={[4,4,0,0]} name="Total Scans" />
              <Bar dataKey="phishing" fill="#ef4444" radius={[4,4,0,0]} name="Phishing" />
            </BarChart>
          </ResponsiveContainer>
        </div>
        

        {/* Pie Chart */}
        <div className="card">
          <div className="card-title">Threat Types</div>
          <ResponsiveContainer width="100%" height={200}>
            <PieChart>
              <Pie
                data={threatTypes}
                cx="50%"
                cy="50%"
                innerRadius={55}
                outerRadius={80}
                paddingAngle={3}
                dataKey="value"
              >
                {threatTypes.map((entry, i) => (
                  <Cell key={i} fill={entry.color} />
                ))}
              </Pie>
              <Tooltip contentStyle={{ background: '#1e2130', border: '1px solid rgba(255,255,255,0.08)', borderRadius: 8 }} />
              <Legend
                iconType="circle"
                iconSize={8}
                formatter={(val) => <span style={{ color: '#94a3b8', fontSize: 12 }}>{val}</span>}
              />
            </PieChart>
          </ResponsiveContainer>
        </div>
      </div>

      <div className="card">
        <div className="card-title">AI Recommendation Box</div>
        <div className="recommendation-box">
          <div className="recommendation-item">Use HTTPS-only links and avoid unsafe login prompts.</div>
          <div className="recommendation-item">Review sender domains carefully and do not trust urgent requests.</div>
          <div className="recommendation-item">Cross-check suspicious URLs against Safe Browsing and VirusTotal.</div>
          <div className="recommendation-item">Keep browser and security tools updated for the latest protections.</div>
        </div>
      </div>

      {/* Recent Threats */}
      <div className="card">
        <div className="card-title">Recent Threats Detected</div>
        <table style={{ width: '100%', borderCollapse: 'collapse' }}>
          <thead>
            <tr>
              {['URL / Sender', 'Risk Level', 'Time'].map((h) => (
                <th key={h} style={{ textAlign: 'left', padding: '8px 0', fontSize: 12, color: '#64748b', fontWeight: 600, borderBottom: '1px solid rgba(255,255,255,0.06)' }}>
                  {h}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {recentThreats.map((t, i) => (
              <tr key={i}>
                <td style={{ padding: '12px 0', fontSize: 13, color: '#cbd5e1', borderBottom: '1px solid rgba(255,255,255,0.04)' }}>
                  🔗 {t.url}
                </td>
                <td style={{ padding: '12px 0', borderBottom: '1px solid rgba(255,255,255,0.04)' }}>
                  <span style={{
                    background: BADGE[t.risk]?.bg,
                    color: BADGE[t.risk]?.color,
                    padding: '3px 10px',
                    borderRadius: 20,
                    fontSize: 11,
                    fontWeight: 700,
                  }}>
                    {t.risk}
                  </span>
                </td>
                <td style={{ padding: '12px 0', fontSize: 12, color: '#64748b', borderBottom: '1px solid rgba(255,255,255,0.04)' }}>
                  {t.time}
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
````

## `frontend\src\pages\History.js`

````jsx
import React, { useState, useEffect } from 'react';

const COLORS = {
  CRITICAL: { bg: 'rgba(239,68,68,0.15)', color: '#f87171' },
  HIGH: { bg: 'rgba(245,158,11,0.15)', color: '#fbbf24' },
  MEDIUM: { bg: 'rgba(234,179,8,0.15)', color: '#facc15' },
  LOW: { bg: 'rgba(16,185,129,0.15)', color: '#34d399' },
};

export default function History() {
  const [filter, setFilter] = useState('ALL');
  const [riskFilter, setRiskFilter] = useState('ALL');
  const [search, setSearch] = useState('');
  const [history, setHistory] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchHistory = async () => {
      try {
        setLoading(true);
        const params = new URLSearchParams();
        if (search) params.set('q', search);
        if (filter) params.set('type', filter);
        if (riskFilter) params.set('risk', riskFilter);

        const response = await fetch(`http://localhost:5000/api/history?${params.toString()}`);
        if (!response.ok) throw new Error('Failed to fetch history');
        const data = await response.json();
        setHistory(data);
        setError(null);
      } catch (err) {
        console.error('Error fetching history:', err);
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchHistory();
  }, [filter, riskFilter, search]);

  const filtered = history;

  return (
    <div>
      <h1 style={{ fontSize: 24, fontWeight: 700, marginBottom: 4 }}>Scan History</h1>
      <p style={{ color: '#94a3b8', marginBottom: 24, fontSize: 14 }}>
        Past scans and threat detections
      </p>

      <div className="page-controls">
        <input
          className="search-input"
          placeholder="Search URL, type, risk level..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
        />
        <select
          className="filter-select"
          value={riskFilter}
          onChange={(e) => setRiskFilter(e.target.value)}
        >
          {['ALL', 'THREATS', 'CRITICAL', 'HIGH', 'MEDIUM', 'LOW'].map((option) => (
            <option key={option} value={option}>{option}</option>
          ))}
        </select>
      </div>

      {loading && <p style={{ color: '#94a3b8' }}>Loading history...</p>}
      {error && <p style={{ color: '#f87171' }}>Error: {error}</p>}
      {!loading && !error && history.length === 0 && <p style={{ color: '#94a3b8' }}>No scan history yet.</p>}

      {/* Filter Tabs */}
      <div style={{ display: 'flex', gap: 8, marginBottom: 20, flexWrap: 'wrap' }}>
        {['ALL', 'URL', 'Email'].map((f) => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            style={{
              padding: '7px 16px',
              borderRadius: 8,
              border: '1px solid',
              cursor: 'pointer',
              fontSize: 13,
              fontWeight: 600,
              background: filter === f ? 'rgba(99,102,241,0.2)' : 'transparent',
              borderColor: filter === f ? '#6366f1' : 'rgba(255,255,255,0.1)',
              color: filter === f ? '#818cf8' : '#94a3b8',
            }}
          >
            {f}
          </button>
        ))}
      </div>

      <div className="card" style={{ padding: 0, overflow: 'hidden' }}>
        <table style={{ width: '100%', borderCollapse: 'collapse' }}>
          <thead>
            <tr style={{ background: 'rgba(255,255,255,0.03)' }}>
              {['#', 'Type', 'URL / Sender', 'Risk', 'Score', 'Time'].map((h) => (
                <th key={h} style={{ textAlign: 'left', padding: '14px 20px', fontSize: 12, color: '#64748b', fontWeight: 600 }}>
                  {h}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {filtered.map((row, i) => (
              <tr key={row.id} style={{ borderTop: '1px solid rgba(255,255,255,0.05)' }}>
                <td style={{ padding: '14px 20px', fontSize: 12, color: '#475569' }}>{row.id}</td>
                <td style={{ padding: '14px 20px' }}>
                  <span style={{ fontSize: 12, color: '#94a3b8', background: 'rgba(255,255,255,0.05)', padding: '3px 8px', borderRadius: 4 }}>
                    {row.type === 'URL' ? '🔗' : '📧'} {row.type}
                  </span>
                </td>
                <td style={{ padding: '14px 20px', fontSize: 13, color: '#cbd5e1', maxWidth: 280, overflow: 'hidden', textOverflow: 'ellipsis', whiteSpace: 'nowrap' }}>
                  {row.url || row.input}
                </td>
                <td style={{ padding: '14px 20px' }}>
                  <span style={{
                    background: COLORS[row.risk]?.bg,
                    color: COLORS[row.risk]?.color,
                    padding: '3px 10px',
                    borderRadius: 20,
                    fontSize: 11,
                    fontWeight: 700,
                  }}>
                    {row.risk}
                  </span>
                </td>
                <td style={{ padding: '14px 20px', fontSize: 13, fontWeight: 700, color: COLORS[row.risk]?.color }}>
                  {row.score}
                </td>
                <td style={{ padding: '14px 20px', fontSize: 12, color: '#64748b' }}>
                  {row.time}
                </td>
              </tr>
            ))}
          </tbody>
        </table>

        {filtered.length === 0 && (
          <div style={{ padding: 40, textAlign: 'center', color: '#475569' }}>
            No records found
          </div>
        )}
      </div>
    </div>
  );
}
````

## `frontend\src\pages\Scanner.js`

````jsx
import React, { useState } from 'react';
import axios from 'axios';
import jsQR from 'jsqr';

const RISK_COLORS = {
  CRITICAL: '#ef4444',
  HIGH: '#f59e0b',
  MEDIUM: '#eab308',
  LOW: '#10b981',
};

function RiskBar({ score, level }) {
  return (
    <div className="risk-bar-container">
      <div className="risk-bar-label">
        <span>Risk Score</span>
        <span style={{ color: RISK_COLORS[level] || '#94a3b8' }}>
          {score}/100 — {level}
        </span>
      </div>
      <div className="risk-bar">
        <div
          className="risk-bar-fill"
          style={{
            width: `${score}%`,
            background: RISK_COLORS[level] || '#94a3b8',
          }}
        />
      </div>
    </div>
  );
}

function ResultCard({ result, type, onDownload, downloading }) {
  if (!result) return null;
  const isPhishing = result.is_phishing;
  const actionMessages = {
    CRITICAL: '🚫 Do NOT visit this URL. It is highly suspicious and may be dangerous.',
    HIGH: '⚠️ Very suspicious. Verify the source through official channels before proceeding.',
    MEDIUM: '🟡 Risky. Exercise extreme caution and confirm details before trusting this URL.',
    LOW: '✅ Likely safe. Continue carefully and avoid entering sensitive information unless verified.',
  };

  const renderSummaryCard = (title, rows) => (
    <div className="enrichment-card">
      <div className="enrichment-title">{title}</div>
      {rows.map((row) => (
        <div key={row.label} className="enrichment-row">
          <span>{row.label}</span>
          <span>{row.value}</span>
        </div>
      ))}
    </div>
  );

  return (
    <div
      className="result-card"
      style={{
        border: `1px solid ${isPhishing ? '#ef444440' : '#10b98140'}`,
      }}
    >
      <div className="result-header">
        <div className="result-icon">{isPhishing ? '🚨' : '✅'}</div>
        <div>
          <div className="result-title" style={{ color: isPhishing ? '#f87171' : '#34d399' }}>
            {isPhishing ? 'Phishing Detected!' : 'Looks Safe'}
          </div>
          <div style={{ fontSize: 13, color: '#94a3b8', marginTop: 2 }}>
            Confidence: {(result.confidence * 100).toFixed(0)}%
          </div>
        </div>
      </div>

      <RiskBar score={result.risk_score} level={result.risk_level} />

      <div style={{ marginTop: 12, padding: 12, borderRadius: 10, background: isPhishing ? 'rgba(248,113,113,0.08)' : 'rgba(52,211,153,0.08)', color: isPhishing ? '#b91c1c' : '#047857', fontSize: 13, fontWeight: 600 }}>
        {actionMessages[result.risk_level] || 'Review the indicators and proceed with caution.'}
      </div>

      {result.indicators?.length > 0 && (
        <div className="indicators-list">
          <div style={{ fontSize: 13, fontWeight: 600, marginBottom: 8, color: '#94a3b8' }}>
            Threat Indicators ({result.indicators.length})
          </div>
          {result.indicators.map((ind, i) => (
            <div key={i} className="indicator-item">
              <span>⚠️</span>
              <span>{ind}</span>
            </div>
          ))}
        </div>
      )}

      {result.analysis && Object.keys(result.analysis).length > 0 && (
        <div className="analysis-details">
          <div style={{ fontSize: 13, fontWeight: 600, marginBottom: 10, color: '#64748b' }}>Analysis Details</div>
          <div className="analysis-grid">
            {Object.entries(result.analysis).map(([key, value]) => (
              <div key={key} className="analysis-row">
                <div className="analysis-key">{key.replace(/_/g, ' ')}</div>
                <div className="analysis-value">{typeof value === 'boolean' ? (value ? 'Yes' : 'No') : value || '—'}</div>
              </div>
            ))}
          </div>
        </div>
      )}

      {(result.whois || result.ssl || result.virus_total || result.safe_browsing) && (
        <div className="enrichment-grid">
          {result.whois && renderSummaryCard('WHOIS Lookup', [
            { label: 'Registrar', value: result.whois.registrar || 'N/A' },
            { label: 'Created', value: result.whois.created_date || 'N/A' },
            { label: 'Expiry', value: result.whois.expiry_date || 'N/A' },
            { label: 'Age (days)', value: result.whois.domain_age_days ?? 'N/A' },
          ])}
          {result.ssl && renderSummaryCard('SSL Certificate', [
            { label: 'Valid', value: result.ssl.valid ? 'Yes' : 'No' },
            { label: 'Issuer', value: result.ssl.issuer || 'N/A' },
            { label: 'Valid To', value: result.ssl.valid_to || 'N/A' },
            { label: 'Days Left', value: result.ssl.days_until_expiry ?? 'N/A' },
          ])}
          {result.virus_total && renderSummaryCard('VirusTotal', [
            { label: 'Available', value: result.virus_total.available ? 'Yes' : 'No' },
            { label: 'Malicious', value: result.virus_total.stats?.malicious ?? 'N/A' },
            { label: 'Suspicious', value: result.virus_total.stats?.suspicious ?? 'N/A' },
            { label: 'Undetected', value: result.virus_total.stats?.undetected ?? 'N/A' },
          ])}
          {result.safe_browsing && renderSummaryCard('Safe Browsing', [
            { label: 'Safe', value: result.safe_browsing.safe ? 'Yes' : 'No' },
            { label: 'Available', value: result.safe_browsing.available ? 'Yes' : 'No' },
            { label: 'Matches', value: result.safe_browsing.details?.length ?? 0 },
          ])}
        </div>
      )}

      {result.recommendations?.length > 0 && (
        <div className="recommendation-box" style={{ marginTop: 18 }}>
          <div className="recommendation-title">AI Recommendations</div>
          {result.recommendations.map((rec, i) => (
            <div key={i} className="recommendation-item">{rec}</div>
          ))}
        </div>
      )}

      {onDownload && (
        <div className="result-footer">
          <button className="btn-report" onClick={onDownload} disabled={downloading}>
            {downloading ? 'Generating report...' : 'Download PDF Report'}
          </button>
        </div>
      )}
    </div>
  );
}

export default function Scanner() {
  const [mode, setMode] = useState('url');
  const [urlInput, setUrlInput] = useState('');
  const [emailData, setEmailData] = useState({ sender: '', subject: '', body: '' });
  const [qrFileName, setQrFileName] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);
  const [downloading, setDownloading] = useState(false);
  const [error, setError] = useState(null);

  const decodeQrCodeFromFile = (file) => {
    if (!file) return;

    const reader = new FileReader();
    reader.onload = () => {
      try {
        const image = new Image();
        image.onload = () => {
          const canvas = document.createElement('canvas');
          const context = canvas.getContext('2d');
          canvas.width = image.width;
          canvas.height = image.height;
          context.drawImage(image, 0, 0, canvas.width, canvas.height);

          const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
          const decoded = jsQR(imageData.data, imageData.width, imageData.height);

          if (!decoded) {
            setError('No QR code found in the selected image. Please upload a clearer image.');
            return;
          }

          const decodedUrl = decoded.data.trim();
          if (!decodedUrl.startsWith('http://') && !decodedUrl.startsWith('https://')) {
            setError('The QR code does not appear to contain a valid URL.');
            return;
          }

          setUrlInput(decodedUrl);
          setMode('url');
          setError(null);
          setResult(null);
        };
        image.src = reader.result;
      } catch (scanErr) {
        setError('Unable to read this QR code image. Please try another file.');
      }
    };

    reader.readAsDataURL(file);
  };

  const handleScan = async () => {
    setLoading(true);
    setError(null);
    setResult(null);

    try {
      let response;
      if (mode === 'url' || mode === 'qr') {
        response = await axios.post('/api/scan/url', { url: urlInput });
      } else {
        response = await axios.post('/api/scan/email', emailData);
      }
      setResult(response.data);
    } catch (err) {
      const errorMsg = err.response?.data?.error || err.message || 'Scan failed';
      setError(errorMsg);
    } finally {
      setLoading(false);
    }
  };

  const downloadReport = async () => {
    if (!result) return;
    setDownloading(true);
    setError(null);

    try {
      const response = await fetch('/api/report', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(result),
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.error || 'Unable to generate report');
      }

      const blob = await response.blob();
      const url = window.URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `scan-report-${result.scan_id || 'report'}.pdf`;
      document.body.appendChild(link);
      link.click();
      link.remove();
      window.URL.revokeObjectURL(url);
    } catch (err) {
      setError(err.message || 'PDF export failed');
    } finally {
      setDownloading(false);
    }
  };

  return (
    <div>
      <h1 style={{ fontSize: 24, fontWeight: 700, marginBottom: 8 }}>URL & Email Scanner</h1>
      <p style={{ color: '#94a3b8', marginBottom: 24, fontSize: 14 }}>
        Scan URLs or emails for phishing indicators using AI analysis
      </p>

      {/* Mode Toggle */}
      <div style={{ display: 'flex', gap: 8, marginBottom: 20, flexWrap: 'wrap' }}>
        {['url', 'qr', 'email'].map((m) => (
          <button
            key={m}
            onClick={() => { setMode(m); setResult(null); setError(null); }}
            style={{
              padding: '8px 20px',
              borderRadius: 8,
              border: '1px solid',
              cursor: 'pointer',
              fontSize: 14,
              fontWeight: 600,
              background: mode === m ? 'rgba(99,102,241,0.2)' : 'transparent',
              borderColor: mode === m ? '#6366f1' : 'rgba(255,255,255,0.1)',
              color: mode === m ? '#818cf8' : '#94a3b8',
            }}
          >
            {m === 'url' ? '🔗 URL Scan' : m === 'qr' ? '📷 QR URL Scan' : '📧 Email Scan'}
          </button>
        ))}
      </div>

      {/* QR Mode */}
      {mode === 'qr' && (
        <div className="card" style={{ marginBottom: 16 }}>
          <div className="qr-upload-box">
            <label className="qr-upload-label">
              <input
                type="file"
                accept="image/*"
                onChange={(e) => {
                  const file = e.target.files?.[0];
                  setQrFileName(file ? file.name : '');
                  decodeQrCodeFromFile(file);
                }}
              />
              <span>Choose QR code image</span>
            </label>
            {qrFileName && <div className="qr-file-name">Selected: {qrFileName}</div>}
            <div className="qr-help-text">Upload a QR code that contains a URL, then the decoded link will be scanned automatically.</div>
          </div>
        </div>
      )}

      {/* URL Mode */}
      {mode === 'url' && (
        <div className="scanner-form">
          <input
            className="scanner-input"
            placeholder="https://example.com/login"
            value={urlInput}
            onChange={(e) => setUrlInput(e.target.value)}
            onKeyDown={(e) => e.key === 'Enter' && handleScan()}
          />
          <button className="btn-scan" onClick={handleScan} disabled={loading || !urlInput}>
            {loading ? '⏳ Scanning...' : '🔍 Scan URL'}
          </button>
        </div>
      )}

      {mode === 'qr' && urlInput && (
        <div className="qr-decoded-banner">
          <strong>Decoded URL:</strong> {urlInput}
          <button className="btn-scan" onClick={handleScan} disabled={loading} style={{ marginLeft: 12 }}>
            {loading ? '⏳ Scanning...' : '🔍 Scan Decoded Link'}
          </button>
        </div>
      )}

      {/* Email Mode */}
      {mode === 'email' && (
        <div className="card" style={{ marginBottom: 16 }}>
          <div style={{ display: 'flex', flexDirection: 'column', gap: 12 }}>
            <input
              className="scanner-input"
              placeholder="Sender email address"
              value={emailData.sender}
              onChange={(e) => setEmailData({ ...emailData, sender: e.target.value })}
            />
            <input
              className="scanner-input"
              placeholder="Email subject"
              value={emailData.subject}
              onChange={(e) => setEmailData({ ...emailData, subject: e.target.value })}
            />
            <textarea
              className="scanner-input"
              placeholder="Paste email body here..."
              rows={6}
              value={emailData.body}
              onChange={(e) => setEmailData({ ...emailData, body: e.target.value })}
              style={{ resize: 'vertical' }}
            />
            <button
              className="btn-scan"
              onClick={handleScan}
              disabled={loading || !emailData.sender || !emailData.body}
              style={{ alignSelf: 'flex-end' }}
            >
              {loading ? '⏳ Analyzing...' : '📧 Analyze Email'}
            </button>
          </div>
        </div>
      )}

      {error && (
        <div style={{
          background: 'rgba(239,68,68,0.1)',
          border: '1px solid rgba(239,68,68,0.3)',
          borderRadius: 8,
          padding: 16,
          color: '#f87171',
          fontSize: 14,
        }}>
          ❌ {error}
        </div>
      )}

      <ResultCard result={result} type={mode} onDownload={downloadReport} downloading={downloading} />
    </div>
  );
}
````

## `frontend\src\pages\Tips.js`

````jsx
import React, { useState } from 'react';

const TIPS = [
  {icon:'🔒',title:'HTTPS తప్పకుండా Check చేయి',cat:'URL',level:'Basic',
   desc:'Login page తెరిచే ముందు address bar లో 🔒 lock icon చూడు. "http://" మాత్రమే ఉంటే credential ఇవ్వకు.',
   eg:'✅ https://paypal.com  ❌ http://paypal-secure.xyz'},
  {icon:'🔗',title:'Suspicious Domains గుర్తించు',cat:'URL',level:'Basic',
   desc:'.xyz .tk .ml .ga .cf extensions తో real companies ఉండవు. Scammers cheap domains వాడతారు.',
   eg:'❌ paypal-verify.xyz  ✅ paypal.com'},
  {icon:'📧',title:'Urgent Emails Ignore చేయి',cat:'Email',level:'Basic',
   desc:'"Account suspended in 24 hours!" లాంటి urgent messages phishing. Real companies ఇలా threaten చేయవు.',
   eg:'❌ "URGENT: Verify NOW or lose access!"'},
  {icon:'👤',title:'Generic Greeting = Red Flag',cat:'Email',level:'Basic',
   desc:'"Dear Customer" వస్తే suspicious! Real companies మీ పేరు తో greet చేస్తాయి.',
   eg:'❌ "Dear Valued Customer"  ✅ "Dear Ravi Kumar"'},
  {icon:'🏦',title:'Brand Impersonation గుర్తించు',cat:'URL',level:'Intermediate',
   desc:'"paypal-secure.xyz" brand name వాడి mislead చేస్తుంది. Real URL slash కి ముందు domain మాత్రమే.',
   eg:'❌ paypal-secure.xyz/login  ✅ paypal.com/login'},
  {icon:'🔑',title:'Password Email లో ఇవ్వకు',cat:'Email',level:'Basic',
   desc:'Real banks, PayPal — ఏ company కూడా email లో password, CVV అడగవు. అడిగితే 100% scam.',
   eg:'❌ "Enter your password to verify"'},
  {icon:'📱',title:'2-Factor Authentication వాడు',cat:'Protection',level:'Intermediate',
   desc:'2FA enable చేస్తే password తెలిసినా account safe. Google Authenticator use చేయి.',
   eg:'✅ Google, PayPal, Bank అన్నింటికీ 2FA'},
  {icon:'🕵️',title:'Hover to See Real URL',cat:'URL',level:'Intermediate',
   desc:'Click ముందు link hover చేయి — status bar లో real URL కనిపిస్తుంది. Match కాకపోతే trap!',
   eg:'Display: "Click here"  Real: http://phish.xyz'},
  {icon:'📎',title:'Unknown Attachments తెరవకు',cat:'Email',level:'Intermediate',
   desc:'Unknown sender నుండి .exe .zip .docm files తెరవకు — ransomware contain అవుతుంది.',
   eg:'❌ Invoice.exe  ❌ Statement.zip from unknown'},
  {icon:'🌐',title:'URL Direct Type చేయి',cat:'URL',level:'Basic',
   desc:'Email link click చేయకుండా browser లో directly official URL type చేయి.',
   eg:'✅ Browser: paypal.com  ❌ Email link click'},
  {icon:'📞',title:'Phone Call తో Verify చేయి',cat:'Protection',level:'Advanced',
   desc:'Suspicious email వస్తే — official website నుండి number తీసుకుని call చేసి verify చేయి.',
   eg:'✅ paypal.com listed number కి call'},
  {icon:'🔄',title:'Software Update చేయి',cat:'Protection',level:'Basic',
   desc:'Browser, OS, Antivirus అన్నీ update గా ఉంచు — phishing sites automatically block అవుతాయి.',
   eg:'✅ Chrome/Windows auto-update enable చేయి'},
];

const LC = {Basic:'#10b981',Intermediate:'#f59e0b',Advanced:'#ef4444'};

export default function Tips() {
  const [cat, setCat] = useState('All');
  const [lvl, setLvl] = useState('All');
  const [exp, setExp] = useState(null);

  const data = TIPS.filter(t=>(cat==='All'||t.cat===cat)&&(lvl==='All'||t.level===lvl));

  return (
    <div>
      <div style={{marginBottom:28}}>
        <h1 style={{fontSize:26,fontWeight:800,letterSpacing:'-0.5px'}}>💡 Safety Tips</h1>
        <p style={{color:'var(--text-secondary)',fontSize:14,marginTop:4}}>Phishing నుండి protect అవ్వడానికి complete guide</p>
      </div>

      {/* Summary */}
      <div style={{display:'grid',gridTemplateColumns:'repeat(3,1fr)',gap:12,marginBottom:24}}>
        {[{icon:'🔗',label:'URL Safety',cat:'URL',color:'#6366f1'},{icon:'📧',label:'Email Safety',cat:'Email',color:'#8b5cf6'},{icon:'🛡️',label:'Protection',cat:'Protection',color:'#10b981'}].map(x=>(
          <div key={x.label} onClick={()=>setCat(x.cat)}
            style={{background:'var(--bg-card)',border:`1px solid ${cat===x.cat?x.color:'var(--border)'}`,borderLeft:`3px solid ${x.color}`,borderRadius:12,padding:'16px 20px',cursor:'pointer',transition:'all .15s'}}>
            <div style={{display:'flex',alignItems:'center',gap:10}}>
              <span style={{fontSize:22}}>{x.icon}</span>
              <div>
                <div style={{fontWeight:700,fontSize:14}}>{x.label}</div>
                <div style={{fontSize:12,color:'var(--text-muted)'}}>{TIPS.filter(t=>t.cat===x.cat).length} tips</div>
              </div>
            </div>
          </div>
        ))}
      </div>

      {/* Filters */}
      <div style={{display:'flex',gap:16,marginBottom:20,flexWrap:'wrap'}}>
        <div>
          <div style={{fontSize:11,color:'var(--text-muted)',marginBottom:6,fontWeight:600,textTransform:'uppercase'}}>Category</div>
          <div style={{display:'flex',gap:6}}>
            {['All','URL','Email','Protection'].map(c=>(
              <button key={c} onClick={()=>setCat(c)}
                style={{padding:'6px 14px',borderRadius:20,border:'1px solid',cursor:'pointer',fontSize:12,fontWeight:600,
                  background:cat===c?'rgba(99,102,241,0.2)':'transparent',
                  borderColor:cat===c?'rgba(99,102,241,0.4)':'var(--border)',
                  color:cat===c?'#a5b4fc':'var(--text-muted)'}}>
                {c}
              </button>
            ))}
          </div>
        </div>
        <div>
          <div style={{fontSize:11,color:'var(--text-muted)',marginBottom:6,fontWeight:600,textTransform:'uppercase'}}>Level</div>
          <div style={{display:'flex',gap:6}}>
            {['All','Basic','Intermediate','Advanced'].map(l=>(
              <button key={l} onClick={()=>setLvl(l)}
                style={{padding:'6px 14px',borderRadius:20,border:'1px solid',cursor:'pointer',fontSize:12,fontWeight:600,
                  background:lvl===l?'rgba(99,102,241,0.2)':'transparent',
                  borderColor:lvl===l?'rgba(99,102,241,0.4)':'var(--border)',
                  color:lvl===l?'#a5b4fc':'var(--text-muted)'}}>
                {l}
              </button>
            ))}
          </div>
        </div>
      </div>

      {/* Grid */}
      <div style={{display:'grid',gridTemplateColumns:'repeat(auto-fill,minmax(300px,1fr))',gap:14}}>
        {data.map((t,i)=>(
          <div key={i} onClick={()=>setExp(exp===i?null:i)}
            style={{background:'var(--bg-card)',border:'1px solid var(--border)',borderLeft:`3px solid #6366f1`,borderRadius:12,padding:18,cursor:'pointer',transition:'transform .15s',transform:exp===i?'none':'translateY(0)'}}>
            <div style={{display:'flex',alignItems:'flex-start',gap:10,marginBottom:10}}>
              <span style={{fontSize:24,flexShrink:0}}>{t.icon}</span>
              <div style={{flex:1}}>
                <div style={{fontWeight:700,fontSize:14,marginBottom:5}}>{t.title}</div>
                <div style={{display:'flex',gap:6}}>
                  <span style={{fontSize:10,fontWeight:700,padding:'2px 8px',borderRadius:4,background:'rgba(99,102,241,0.12)',color:'#a5b4fc'}}>{t.cat}</span>
                  <span style={{fontSize:10,fontWeight:700,padding:'2px 8px',borderRadius:4,background:`${LC[t.level]}18`,color:LC[t.level]}}>{t.level}</span>
                </div>
              </div>
              <span style={{color:'var(--text-muted)',fontSize:11}}>{exp===i?'▲':'▼'}</span>
            </div>
            <p style={{color:'var(--text-secondary)',fontSize:13,lineHeight:1.6,margin:0}}>{t.desc}</p>
            {exp===i && (
              <div style={{marginTop:12,padding:10,background:'var(--bg-secondary)',borderRadius:6,fontSize:12,fontFamily:'monospace',color:'var(--text-muted)'}}>
                {t.eg}
              </div>
            )}
          </div>
        ))}
      </div>
    </div>
  );
}
````

## `frontend\src\utils\engine.js`

````jsx
const SUSP_TLDS = ['.xyz','.tk','.ml','.ga','.cf','.gq','.top','.click','.link','.win','.loan','.download','.stream','.zip','.review','.country','.kim','.cricket','.party','.science','.trade','.work'];
const BRANDS = ['paypal','apple','google','microsoft','amazon','facebook','netflix','instagram','twitter','bank','secure','verify','account','login','ebay','chase','wells','citibank','hsbc','binance','coinbase','dropbox','whatsapp'];
const URGENCY = ['urgent','immediately','action required','account suspended','verify now','limited time','expires','warning','alert','failure to','act now','final notice','last chance','24 hours','48 hours','will be deleted','will be terminated'];
const CREDS = ['password','username','ssn','social security','credit card','bank account','routing number','pin number','cvv','confirm your','enter your','provide your','account number','date of birth','mother maiden'];

export function analyzeURL(raw) {
  if (!raw || !raw.trim()) return null;
  let url = raw.trim();
  if (!url.startsWith('http://') && !url.startsWith('https://')) url = 'http://' + url;

  let parsed;
  try { parsed = new URL(url); } catch { return { error: 'Invalid URL format' }; }

  const domain = parsed.hostname.toLowerCase();
  const path   = parsed.pathname.toLowerCase();
  const full   = url.toLowerCase();

  let score = 0;
  const inds = [];
  const add = (pts, msg) => { score += pts; inds.push(msg); };

  if (!url.startsWith('https'))                      add(20, '❌ No HTTPS — unencrypted connection');
  if (/^\d{1,3}(\.\d{1,3}){3}$/.test(domain))       add(30, '❌ IP address used as domain');
  const tld = '.' + domain.split('.').pop();
  if (SUSP_TLDS.includes(tld))                       add(22, `❌ Suspicious domain extension: ${tld}`);
  const bd = BRANDS.find(b => domain.includes(b));
  if (bd) {
    const real = domain === `${bd}.com` || domain === `www.${bd}.com` || domain.endsWith(`.${bd}.com`);
    if (!real)                                       add(22, `❌ Brand "${bd}" impersonated in domain`);
  }
  const bp = BRANDS.find(b => path.includes(b) && !domain.includes(b));
  if (bp)                                            add(12, `⚠️ Brand "${bp}" in URL path only`);
  if (url.includes('@'))                             add(25, '❌ @ symbol in URL — hides real host');
  const hyph = (domain.match(/-/g)||[]).length;
  if (hyph > 3)                                      add(8,  `⚠️ Excessive hyphens in domain (${hyph})`);
  const subs = domain.split('.').length - 2;
  if (subs > 2)                                      add(12, `⚠️ Deep subdomain nesting (${subs} levels)`);
  if (url.length > 120)                              add(8,  `⚠️ Very long URL (${url.length} chars)`);
  if (full.replace('://','').includes('//'))         add(8,  '⚠️ Double slash — possible redirect');
  const digs = (domain.replace(/\./g,'').match(/\d/g)||[]).length;
  if (digs > 5)                                      add(7,  `⚠️ Many digits in domain (${digs})`);
  if (/\/(login|signin|verify|update|secure|confirm|password|account)/.test(path) && !url.startsWith('https'))
                                                     add(15, '❌ Sensitive path without HTTPS');
  const enc = (url.match(/%[0-9a-f]{2}/gi)||[]).length;
  if (enc > 5)                                       add(6,  `⚠️ Many encoded chars (${enc}) — hiding content`);

  score = Math.min(score, 100);
  const risk = score>=70?'CRITICAL': score>=50?'HIGH': score>=30?'MEDIUM':'LOW';
  return { is_phishing:score>=40, confidence:parseFloat((score/100).toFixed(2)), risk_score:score, risk_level:risk, indicators:inds, domain, tld, has_https:url.startsWith('https') };
}

export function analyzeEmail(sender='', subject='', body='') {
  let score = 0;
  const inds = [];
  const bL = body.toLowerCase();
  const sL = subject.toLowerCase();
  const domain = sender.includes('@') ? sender.split('@')[1]?.toLowerCase() : '';
  const add = (pts, msg) => { score += pts; inds.push(msg); };

  if (domain) {
    const tld = '.' + domain.split('.').pop();
    if (SUSP_TLDS.includes(tld))                     add(22, `❌ Suspicious sender extension: ${tld}`);
    const bd = BRANDS.find(b => domain.includes(b));
    if (bd && !['paypal.com','apple.com','google.com','microsoft.com','amazon.com','facebook.com'].includes(domain))
                                                     add(25, `❌ Brand "${bd}" spoofed in sender`);
    if (/\d{4,}/.test(domain))                       add(8,  '⚠️ Suspicious numbers in sender domain');
    if (domain.split('.').length > 4)                add(10, '⚠️ Overly complex sender domain');
    const free = ['gmail.com','yahoo.com','hotmail.com','outlook.com'];
    if (free.includes(domain) && BRANDS.find(b => sL.includes(b)))
                                                     add(18, '❌ Brand in subject but free email provider');
  }

  const urgS = URGENCY.filter(k => sL.includes(k)).length;
  if (urgS > 0)                                      add(14, `⚠️ Urgent language in subject (${urgS} triggers)`);
  if (/[A-Z]{6,}/.test(subject))                    add(6,  '⚠️ Excessive CAPS in subject');
  if ((subject.match(/!/g)||[]).length > 2)          add(5,  '⚠️ Multiple !!! in subject');
  if (/(free|prize|winner|lottery|congratulations|selected|lucky|reward)/i.test(subject))
                                                     add(16, '❌ Prize/reward bait in subject');
  if (/(suspended|locked|terminated|hacked|compromised|expired)/i.test(subject))
                                                     add(12, '⚠️ Account threat keyword in subject');

  const urgB = URGENCY.filter(k => bL.includes(k)).length;
  if (urgB >= 3)                                     add(16, `❌ High urgency in body (${urgB} phrases)`);
  else if (urgB > 0)                                 add(8,  `⚠️ Urgency language in body (${urgB})`);
  const credC = CREDS.filter(k => bL.includes(k)).length;
  if (credC >= 2)                                    add(24, `❌ Requesting sensitive info (${credC} keywords)`);
  else if (credC === 1)                              add(10, '⚠️ Credential keyword in body');
  if (/dear (valued )?customer/i.test(body))         add(10, '⚠️ Generic greeting — not personalized');
  if (/click (?:here|below|this link) to (?:verify|confirm|update|login)/i.test(body))
                                                     add(18, '❌ Classic phishing call-to-action');
  if (/your account.*(suspended|locked|terminated)/i.test(body))
                                                     add(18, '❌ Account threat phrase');
  if (/we.*(detected|noticed).*(unusual|suspicious)/i.test(body))
                                                     add(12, '⚠️ Fake security alert pattern');
  const links = (body.match(/https?:\/\/[^\s]+/g)||[]);
  const badL = links.filter(l => !l.startsWith('https'));
  if (badL.length > 0)                              add(10, `❌ ${badL.length} non-HTTPS link(s) in body`);
  if (links.length > 7)                             add(8,  `⚠️ Too many links (${links.length})`);

  score = Math.min(score, 100);
  const risk = score>=70?'CRITICAL': score>=50?'HIGH': score>=25?'MEDIUM':'LOW';
  return { is_phishing:score>=36, confidence:parseFloat((score/100).toFixed(2)), risk_score:score, risk_level:risk, indicators:inds, links_count:links.length };
}

export const DEMO_URLS = [
  { url:'http://paypal-secure-verify.xyz/login',       label:'🚨 Phishing' },
  { url:'http://192.168.1.1/bank/login',               label:'🚨 IP URL' },
  { url:'http://apple-id-locked.tk/verify',            label:'🚨 Fake Apple' },
  { url:'https://amazon-gift-winner.ml/claim',         label:'🚨 Prize Scam' },
  { url:'https://www.google.com',                      label:'✅ Safe' },
  { url:'https://github.com/microsoft/vscode',         label:'✅ Safe' },
];

export const DEMO_EMAIL = {
  sender: 'security-noreply@paypal-support.xyz',
  subject: 'URGENT!! Your PayPal Account Has Been SUSPENDED!!!',
  body: `Dear valued customer,

We detected unusual and suspicious activity on your PayPal account. Your account will be permanently suspended within 24 hours.

Click here to verify your account now: http://paypal-verify.xyz/confirm

Please provide your username, password, credit card number and CVV to restore access. Failure to respond will result in permanent termination.

PayPal Security Team`,
};
````

## `frontend\src\utils\storage.js`

````jsx
const KEY = 'phishguard_history';

export function saveToHistory(item) {
  const list = getHistory();
  const entry = {
    id: Date.now() + '_' + Math.random().toString(36).slice(2,7),
    type: item.type,
    input: item.input,
    risk: item.risk,
    score: item.score,
    confidence: item.confidence || 0,
    indicators: item.indicators || [],
    time: new Date().toLocaleString(),
    ts: Date.now(),
  };
  const updated = [entry, ...list].slice(0, 300);
  try { localStorage.setItem(KEY, JSON.stringify(updated)); } catch(e) {}
  return entry;
}

export function getHistory() {
  try { return JSON.parse(localStorage.getItem(KEY) || '[]'); } catch { return []; }
}

export function clearHistory() {
  localStorage.removeItem(KEY);
}

export function deleteItem(id) {
  const updated = getHistory().filter(h => h.id !== id);
  localStorage.setItem(KEY, JSON.stringify(updated));
}

export function getStats() {
  const h = getHistory();
  const total    = h.length;
  const threats  = h.filter(x => x.risk !== 'LOW').length;
  const safe     = h.filter(x => x.risk === 'LOW').length;
  const critical = h.filter(x => x.risk === 'CRITICAL').length;
  const high     = h.filter(x => x.risk === 'HIGH').length;
  const medium   = h.filter(x => x.risk === 'MEDIUM').length;
  const urls     = h.filter(x => x.type === 'URL').length;
  const emails   = h.filter(x => x.type === 'Email').length;
  const rate     = total ? Math.round(threats / total * 100) : 0;
  const avgScore = total ? Math.round(h.reduce((s,x) => s + x.score, 0) / total) : 0;
  return { total, threats, safe, critical, high, medium, urls, emails, rate, avgScore, history: h };
}
````
