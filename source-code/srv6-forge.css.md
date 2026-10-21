:root {  
  \--bg-primary: \#030712;  
  \--bg-secondary: \#0b1120;  
  \--bg-card: \#0d1527;  
  \--border-color: \#1e293b;  
  \--border-focus: \#06b6d4;  
  \--text-primary: \#f8fafc;  
  \--text-secondary: \#94a3b8;  
  \--text-muted: \#64748b;  
  \--accent-cyan: \#06b6d4;  
  \--accent-teal: \#14b8a6;  
  \--accent-blue: \#3b82f6;  
  \--accent-purple: \#8b5cf6;  
  \--status-pass: \#10b981;  
  \--status-warn: \#f59e0b;  
  \--status-fail: \#ef4444;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-primary);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding: 20px;  
  min-height: 100vh;  
}

/\* Header \*/  
.app-header {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 16px 24px;  
  margin-bottom: 20px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  flex-wrap: wrap;  
  gap: 16px;  
}

.brand-section {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
}

.brand-badge {  
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));  
  color: \#ffffff;  
  font-size: 11px;  
  font-weight: 800;  
  padding: 4px 10px;  
  border-radius: 6px;  
  letter-spacing: 0.5px;  
}

.brand-title {  
  font-size: 20px;  
  font-weight: 800;  
  color: \#ffffff;  
  letter-spacing: \-0.5px;  
}

.brand-version {  
  font-size: 12px;  
  color: var(--accent-cyan);  
  font-weight: 600;  
}

.brand-subtitle {  
  font-size: 13px;  
  color: var(--text-secondary);  
  display: block;  
}

.header-controls {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
}

.preset-dropdown {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  color: var(--text-primary);  
  padding: 8px 14px;  
  border-radius: 8px;  
  font-size: 13px;  
  cursor: pointer;  
  outline: none;  
}

.preset-dropdown:focus {  
  border-color: var(--border-focus);  
}

.btn {  
  display: inline-flex;  
  align-items: center;  
  gap: 6px;  
  padding: 8px 16px;  
  border-radius: 8px;  
  font-size: 13px;  
  font-weight: 600;  
  cursor: pointer;  
  transition: all 0.15s ease;  
  border: none;  
}

.btn-primary {  
  background: var(--accent-cyan);  
  color: \#030712;  
}

.btn-primary:hover {  
  background: \#22d3ee;  
}

.btn-outline {  
  background: transparent;  
  border: 1px solid var(--border-color);  
  color: var(--text-secondary);  
}

.btn-outline:hover {  
  background: var(--bg-card);  
  color: var(--text-primary);  
  border-color: var(--accent-cyan);  
}

/\* Studio Layout \*/  
.studio-grid {  
  display: grid;  
  grid-template-columns: 460px 1fr;  
  gap: 20px;  
  align-items: start;  
}

@media (max-width: 1080px) {  
  .studio-grid {  
    grid-template-columns: 1fr;  
  }  
}

.card {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 20px;  
  margin-bottom: 20px;  
}

.card-title {  
  font-size: 15px;  
  font-weight: 700;  
  color: \#ffffff;  
  margin-bottom: 16px;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

/\* Form Controls \*/  
.form-group {  
  margin-bottom: 14px;  
}

.form-label {  
  display: block;  
  font-size: 11px;  
  font-weight: 700;  
  color: var(--text-secondary);  
  text-transform: uppercase;  
  letter-spacing: 0.5px;  
  margin-bottom: 6px;  
}

.form-input, .form-select, .form-textarea {  
  width: 100%;  
  background: \#02040a;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 8px 12px;  
  color: var(--text-primary);  
  font-size: 13px;  
  font-family: var(--font-mono);  
  outline: none;  
}

.form-input:focus, .form-select:focus, .form-textarea:focus {  
  border-color: var(--border-focus);  
}

.form-row {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 12px;  
}

.checkbox-label {  
  display: flex;  
  align-items: center;  
  gap: 8px;  
  font-size: 12px;  
  color: var(--text-primary);  
  cursor: pointer;  
  margin-bottom: 8px;  
}

/\* Audit Score Banner \*/  
.score-banner {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 16px;  
  display: flex;  
  align-items: center;  
  gap: 20px;  
  margin-bottom: 20px;  
}

.score-badge {  
  width: 60px;  
  height: 60px;  
  border-radius: 12px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-size: 26px;  
  font-weight: 900;  
  font-family: var(--font-mono);  
}

.grade-a { background: rgba(16, 185, 129, 0.15); color: \#34d399; border: 1.5px solid \#10b981; }  
.grade-b { background: rgba(59, 130, 246, 0.15); color: \#60a5fa; border: 1.5px solid \#3b82f6; }  
.grade-c { background: rgba(245, 158, 11, 0.15); color: \#fbbf24; border: 1.5px solid \#f59e0b; }  
.grade-f { background: rgba(239, 68, 68, 0.15); color: \#f87171; border: 1.5px solid \#ef4444; }

.score-details h4 {  
  font-size: 16px;  
  font-weight: 700;  
  margin-bottom: 4px;  
}

.score-details p {  
  font-size: 12px;  
  color: var(--text-secondary);  
}

/\* Rubric List \*/  
.rubric-list {  
  display: flex;  
  flex-direction: column;  
  gap: 8px;  
  margin-bottom: 20px;  
}

.rubric-item {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 10px 14px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  font-size: 12px;  
}

.rubric-item.pass { border-left: 3px solid var(--status-pass); }  
.rubric-item.warn { border-left: 3px solid var(--status-warn); }  
.rubric-item.fail { border-left: 3px solid var(--status-fail); }

.rubric-status {  
  font-size: 11px;  
  font-weight: 800;  
  padding: 2px 8px;  
  border-radius: 4px;  
  text-transform: uppercase;  
  font-family: var(--font-mono);  
}

.status-pass-badge { background: rgba(16, 185, 129, 0.2); color: \#34d399; }  
.status-warn-badge { background: rgba(245, 158, 11, 0.2); color: \#fbbf24; }  
.status-fail-badge { background: rgba(239, 68, 68, 0.2); color: \#f87171; }

/\* MTU Headroom Visual Meter \*/  
.mtu-meter-card {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 14px;  
  margin-bottom: 20px;  
}

.mtu-bar-container {  
  height: 12px;  
  background: \#02040a;  
  border-radius: 6px;  
  overflow: hidden;  
  margin: 10px 0;  
  display: flex;  
}

.mtu-bar-payload { background: var(--accent-cyan); height: 100%; }  
.mtu-bar-overhead { background: var(--accent-purple); height: 100%; }  
.mtu-bar-headroom { background: var(--status-pass); height: 100%; }

.mtu-legend {  
  display: flex;  
  justify-content: space-between;  
  font-size: 11px;  
  color: var(--text-secondary);  
}

/\* Tabs & Output View \*/  
.tab-header {  
  display: flex;  
  gap: 8px;  
  border-bottom: 1px solid var(--border-color);  
  margin-bottom: 14px;  
  overflow-x: auto;  
}

.tab-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-secondary);  
  padding: 8px 14px;  
  font-size: 12px;  
  font-weight: 700;  
  cursor: pointer;  
  border-bottom: 2px solid transparent;  
  transition: all 0.15s ease;  
  white-space: nowrap;  
}

.tab-btn.active {  
  color: var(--accent-cyan);  
  border-bottom-color: var(--accent-cyan);  
}

.code-output {  
  background: \#02040a;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 14px;  
  font-family: var(--font-mono);  
  font-size: 11px;  
  color: \#38bdf8;  
  max-height: 380px;  
  overflow-y: auto;  
  white-space: pre;  
  line-height: 1.4;  
}

.hex-dump-output {  
  background: \#02040a;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 14px;  
  font-family: var(--font-mono);  
  font-size: 11px;  
  color: \#a78bfa;  
  max-height: 250px;  
  overflow-y: auto;  
  white-space: pre;  
}  
