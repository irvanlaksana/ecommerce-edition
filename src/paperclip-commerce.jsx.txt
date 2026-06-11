import { useState, useEffect, useRef } from "react";

// ─── Design Tokens ────────────────────────────────────────────────────────────
const T = {
  bg: "#0D0F14",
  surface: "#13161E",
  card: "#191D28",
  border: "#252A38",
  borderHover: "#3A4058",
  accent: "#6366F1",        // indigo
  accentGlow: "#6366F120",
  accentHover: "#818CF8",
  success: "#22C55E",
  warn: "#F59E0B",
  danger: "#EF4444",
  muted: "#4B5563",
  text: "#E2E8F0",
  textSub: "#94A3B8",
  textDim: "#64748B",
  commerce: "#F97316",      // orange signature for Commerce
  marketplace: "#8B5CF6",   // violet for marketplace
  social: "#EC4899",        // pink for social
  sheets: "#10B981",        // emerald for Google Sheets
};

// ─── Icons ─────────────────────────────────────────────────────────────────────
const Icon = {
  home: "⌂", agents: "◈", goals: "◎", issues: "◉", org: "⬡", 
  approval: "✦", marketplace: "◆", content: "✦", social: "❋",
  sheets: "⊞", settings: "⚙", provider: "◈", runtime: "⬡",
  storage: "⊟", database: "◫", plus: "+", check: "✓", x: "✕",
  arrow: "→", back: "←", dot: "•", star: "★", bolt: "⚡",
  search: "⌕", filter: "⊟", tag: "⊞", link: "⛓", upload: "⇑",
  ai: "◈", user: "◉", company: "⬡", gear: "⚙", menu: "≡",
  chevron: "›", lock: "⊡", globe: "◉", chart: "◈", edit: "✎",
  trash: "⊟", copy: "⊞", eye: "◎", send: "⇒", refresh: "↺",
};

// ─── Nav Structure ─────────────────────────────────────────────────────────────
const NAV = [
  { id: "dashboard", label: "Dashboard", icon: "⌂", group: "core" },
  { id: "org-chart", label: "Org Chart", icon: "⬡", group: "core" },
  { id: "goals", label: "Goals", icon: "◎", group: "core" },
  { id: "issues", label: "Issues", icon: "◉", group: "core" },
  { id: "agents", label: "Agents", icon: "◈", group: "core" },
  { id: "approvals", label: "Approvals", icon: "✦", group: "core" },
  { id: "marketplace-hub", label: "Marketplace Hub", icon: "◆", group: "commerce" },
  { id: "products", label: "Products", icon: "⊞", group: "commerce" },
  { id: "content-studio", label: "Content Studio", icon: "✦", group: "commerce" },
  { id: "social-media", label: "Social Media", icon: "❋", group: "commerce" },
  { id: "google-sheets", label: "Google Sheets", icon: "⊞", group: "integrations" },
  { id: "ai-providers", label: "AI Providers", icon: "◈", group: "settings" },
  { id: "runtime", label: "Runtime", icon: "⬡", group: "settings" },
  { id: "storage", label: "Storage", icon: "⊟", group: "settings" },
  { id: "database", label: "Database", icon: "◫", group: "settings" },
];

const GROUP_LABELS = {
  core: "Workforce Core",
  commerce: "Commerce Edition",
  integrations: "Integrations",
  settings: "Settings",
};

// ─── Data ──────────────────────────────────────────────────────────────────────
const AGENTS = [
  { id: 1, name: "Marketplace Manager", role: "commerce", provider: "Claude Sonnet", status: "active", tasks: 12, model: "claude-sonnet-4-20250514" },
  { id: 2, name: "Product Research Agent", role: "research", provider: "Gemini Flash", status: "active", tasks: 8, model: "gemini-2.0-flash" },
  { id: 3, name: "Product Content Agent", role: "content", provider: "GPT-5", status: "idle", tasks: 5, model: "gpt-5" },
  { id: 4, name: "FAQ Agent", role: "content", provider: "Gemini Pro", status: "active", tasks: 23, model: "gemini-pro" },
  { id: 5, name: "SEO Agent", role: "seo", provider: "Claude Sonnet", status: "active", tasks: 15, model: "claude-sonnet-4-20250514" },
  { id: 6, name: "Creative Agent", role: "creative", provider: "GPT-5", status: "idle", tasks: 3, model: "gpt-5" },
  { id: 7, name: "Social Media Agent", role: "social", provider: "Claude Opus", status: "active", tasks: 19, model: "claude-opus-4-20250514" },
  { id: 8, name: "Ads Agent", role: "ads", provider: "Gemini 2.5 Pro", status: "idle", tasks: 7, model: "gemini-2.5-pro" },
  { id: 9, name: "Customer Service Agent", role: "cs", provider: "DeepSeek", status: "active", tasks: 44, model: "deepseek-chat" },
  { id: 10, name: "Google Sheet Agent", role: "sheets", provider: "Gemini Flash", status: "active", tasks: 9, model: "gemini-2.0-flash" },
  { id: 11, name: "Reporting Agent", role: "reporting", provider: "Claude Sonnet", status: "idle", tasks: 2, model: "claude-sonnet-4-20250514" },
  { id: 12, name: "Prompt Agent", role: "prompt", provider: "OpenRouter", status: "active", tasks: 6, model: "openrouter/auto" },
];

const AI_PROVIDERS_DATA = [
  { id: "gemini-flash", name: "Gemini Flash", model: "gemini-2.0-flash-exp", endpoint: "https://generativelanguage.googleapis.com/v1beta", enabled: true, apiKey: "••••••••••••" },
  { id: "gemini-pro", name: "Gemini Pro", model: "gemini-pro", endpoint: "https://generativelanguage.googleapis.com/v1beta", enabled: true, apiKey: "••••••••••••" },
  { id: "gemini-25-pro", name: "Gemini 2.5 Pro", model: "gemini-2.5-pro-preview-05-06", endpoint: "https://generativelanguage.googleapis.com/v1beta", enabled: false, apiKey: "" },
  { id: "gemini-cli", name: "Gemini CLI", model: "gemini-cli", endpoint: "http://localhost:9191", enabled: false, apiKey: "" },
  { id: "gpt-5", name: "OpenAI GPT-5", model: "gpt-5", endpoint: "https://api.openai.com/v1", enabled: true, apiKey: "••••••••••••" },
  { id: "gpt-5-mini", name: "OpenAI GPT-5 Mini", model: "gpt-5-mini", endpoint: "https://api.openai.com/v1", enabled: false, apiKey: "" },
  { id: "claude-sonnet", name: "Claude Sonnet", model: "claude-sonnet-4-20250514", endpoint: "https://api.anthropic.com/v1", enabled: true, apiKey: "••••••••••••" },
  { id: "claude-opus", name: "Claude Opus", model: "claude-opus-4-20250514", endpoint: "https://api.anthropic.com/v1", enabled: true, apiKey: "••••••••••••" },
  { id: "deepseek", name: "DeepSeek", model: "deepseek-chat", endpoint: "https://api.deepseek.com/v1", enabled: true, apiKey: "••••••••••••" },
  { id: "grok", name: "Grok", model: "grok-3", endpoint: "https://api.x.ai/v1", enabled: false, apiKey: "" },
  { id: "openrouter", name: "OpenRouter", model: "openrouter/auto", endpoint: "https://openrouter.ai/api/v1", enabled: true, apiKey: "••••••••••••" },
  { id: "ollama", name: "Ollama", model: "llama3.2", endpoint: "http://localhost:11434/v1", enabled: false, apiKey: "" },
  { id: "lmstudio", name: "LM Studio", model: "local-model", endpoint: "http://localhost:1234/v1", enabled: false, apiKey: "" },
  { id: "openclaw", name: "OpenClaw", model: "openclaw-v1", endpoint: "http://localhost:8080/v1", enabled: false, apiKey: "" },
  { id: "custom", name: "Custom API", model: "custom", endpoint: "", enabled: false, apiKey: "" },
];

const GOALS = [
  { id: 1, title: "Q3 Marketplace Expansion", status: "in-progress", priority: "high", progress: 65, owner: "Marketplace Manager", issues: 8 },
  { id: 2, title: "SEO Content Strategy 2026", status: "in-progress", priority: "high", progress: 42, owner: "SEO Manager", issues: 12 },
  { id: 3, title: "Social Media Growth +50%", status: "planned", priority: "medium", progress: 15, owner: "Social Media Agent", issues: 5 },
  { id: 4, title: "Product Catalog Automation", status: "in-progress", priority: "critical", progress: 78, owner: "Product Content Agent", issues: 3 },
  { id: 5, title: "TikTok Shop Integration", status: "planned", priority: "medium", progress: 0, owner: "Marketplace Manager", issues: 6 },
];

const ISSUES_DATA = [
  { id: "ISS-001", title: "Shopee product scraper timeout", status: "in-progress", priority: "high", agent: "Product Research Agent", goal: "Product Catalog Automation", created: "2026-06-08" },
  { id: "ISS-002", title: "Instagram caption generator returns empty", status: "open", priority: "critical", agent: "Social Media Agent", goal: "Social Media Growth", created: "2026-06-09" },
  { id: "ISS-003", title: "Google Sheets sync fails on 1000+ rows", status: "review", priority: "medium", agent: "Google Sheet Agent", goal: "Product Catalog Automation", created: "2026-06-07" },
  { id: "ISS-004", title: "FAQ generation too generic for electronics", status: "open", priority: "low", agent: "FAQ Agent", goal: "SEO Content Strategy", created: "2026-06-09" },
  { id: "ISS-005", title: "TikTok hashtag API rate limiting", status: "closed", priority: "medium", agent: "Social Media Agent", goal: "Social Media Growth", created: "2026-06-06" },
  { id: "ISS-006", title: "SEO meta tags exceeding 160 chars", status: "in-progress", priority: "medium", agent: "SEO Agent", goal: "SEO Content Strategy", created: "2026-06-08" },
];

const ORG_DATA = {
  name: "Paperclip Commerce Co.",
  nodes: [
    { id: "ceo", title: "CEO", name: "Executive Director", level: 0, parent: null, type: "core" },
    { id: "coo", title: "COO", name: "Operations Lead", level: 1, parent: "ceo", type: "core" },
    { id: "cto", title: "CTO", name: "Technology Lead", level: 1, parent: "ceo", type: "core" },
    { id: "cmo", title: "CMO", name: "Marketing Lead", level: 1, parent: "ceo", type: "core" },
    { id: "mkm", title: "Marketplace Manager", name: "Marketplace Manager", level: 2, parent: "coo", type: "commerce" },
    { id: "prm", title: "Product Research Mgr", name: "Product Research Manager", level: 2, parent: "coo", type: "commerce" },
    { id: "ctm", title: "Content Manager", name: "Content Manager", level: 2, parent: "cmo", type: "commerce" },
    { id: "seom", title: "SEO Manager", name: "SEO Manager", level: 2, parent: "cmo", type: "commerce" },
    { id: "adsm", title: "Ads Manager", name: "Ads Manager", level: 2, parent: "cmo", type: "commerce" },
    { id: "repm", title: "Reporting Manager", name: "Reporting Manager", level: 2, parent: "coo", type: "commerce" },
  ]
};

const PRODUCTS_DATA = [
  { id: 1, name: "Wireless Earbuds Pro X", sku: "WEP-001", price: "Rp 299.000", platform: "Shopee", status: "synced", image: "🎧", category: "Electronics" },
  { id: 2, name: "Kaos Polos Premium Cotton", sku: "KPP-002", price: "Rp 89.000", platform: "Tokopedia", status: "pending", image: "👕", category: "Fashion" },
  { id: 3, name: "Tumbler Stainless 500ml", sku: "TS-003", price: "Rp 149.000", platform: "TikTok Shop", status: "synced", image: "🥤", category: "Kitchen" },
  { id: 4, name: "Skincare Set Vitamin C", sku: "SSVC-004", price: "Rp 459.000", platform: "Lazada", status: "error", image: "💄", category: "Beauty" },
  { id: 5, name: "Tas Ransel Anti-Air 30L", sku: "TRAA-005", price: "Rp 359.000", platform: "Shopee", status: "synced", image: "🎒", category: "Bags" },
];

const WORKFLOW_STEPS = [
  { step: 1, label: "Company", icon: "⬡", color: T.accent },
  { step: 2, label: "Goal", icon: "◎", color: "#8B5CF6" },
  { step: 3, label: "Project", icon: "⊞", color: "#06B6D4" },
  { step: 4, label: "Issue", icon: "◉", color: T.commerce },
  { step: 5, label: "Agent Assignment", icon: "◈", color: T.success },
  { step: 6, label: "Approval", icon: "✦", color: T.warn },
  { step: 7, label: "Execution", icon: "⚡", color: T.social },
  { step: 8, label: "Work Product", icon: "◆", color: T.marketplace },
  { step: 9, label: "Review", icon: "◎", color: "#F59E0B" },
  { step: 10, label: "Close", icon: "✓", color: T.success },
];

// ─── Reusable Components ────────────────────────────────────────────────────────
function Badge({ children, color = T.accent, size = "sm" }) {
  return (
    <span style={{
      background: color + "20",
      color: color,
      border: `1px solid ${color}40`,
      padding: size === "sm" ? "1px 7px" : "3px 10px",
      borderRadius: 4,
      fontSize: size === "sm" ? 10 : 11,
      fontWeight: 600,
      letterSpacing: "0.05em",
      textTransform: "uppercase",
      whiteSpace: "nowrap",
    }}>
      {children}
    </span>
  );
}

function Card({ children, style = {}, onClick }) {
  return (
    <div onClick={onClick} style={{
      background: T.card,
      border: `1px solid ${T.border}`,
      borderRadius: 8,
      padding: 16,
      cursor: onClick ? "pointer" : "default",
      transition: "border-color 0.15s",
      ...style,
    }}
    onMouseEnter={e => onClick && (e.currentTarget.style.borderColor = T.borderHover)}
    onMouseLeave={e => onClick && (e.currentTarget.style.borderColor = T.border)}
    >
      {children}
    </div>
  );
}

function StatusDot({ status }) {
  const colors = { active: T.success, idle: T.textDim, error: T.danger, pending: T.warn, open: T.warn, "in-progress": T.accent, review: T.marketplace, closed: T.muted, synced: T.success };
  return (
    <span style={{
      display: "inline-block",
      width: 7, height: 7,
      borderRadius: "50%",
      background: colors[status] || T.muted,
      boxShadow: `0 0 5px ${colors[status] || T.muted}80`,
      flexShrink: 0,
    }} />
  );
}

function ProgressBar({ value, color = T.accent }) {
  return (
    <div style={{ background: T.border, borderRadius: 4, height: 5, overflow: "hidden" }}>
      <div style={{
        width: `${value}%`, height: "100%",
        background: `linear-gradient(90deg, ${color}, ${color}cc)`,
        borderRadius: 4,
        transition: "width 0.5s ease",
      }} />
    </div>
  );
}

function Toggle({ value, onChange }) {
  return (
    <div onClick={() => onChange(!value)} style={{
      width: 38, height: 20,
      background: value ? T.accent : T.border,
      borderRadius: 10, cursor: "pointer", position: "relative",
      transition: "background 0.2s", flexShrink: 0,
    }}>
      <div style={{
        position: "absolute", top: 3,
        left: value ? 20 : 3,
        width: 14, height: 14,
        background: "#fff",
        borderRadius: "50%",
        transition: "left 0.2s",
      }} />
    </div>
  );
}

function SectionHeader({ title, sub, action }) {
  return (
    <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 20 }}>
      <div>
        <h2 style={{ margin: 0, fontSize: 18, fontWeight: 700, color: T.text }}>{title}</h2>
        {sub && <p style={{ margin: "3px 0 0", fontSize: 13, color: T.textDim }}>{sub}</p>}
      </div>
      {action}
    </div>
  );
}

function Btn({ children, onClick, variant = "primary", size = "md", style = {} }) {
  const variants = {
    primary: { background: T.accent, color: "#fff", border: "none" },
    ghost: { background: "transparent", color: T.textSub, border: `1px solid ${T.border}` },
    danger: { background: T.danger + "20", color: T.danger, border: `1px solid ${T.danger}40` },
    success: { background: T.success + "20", color: T.success, border: `1px solid ${T.success}40` },
    commerce: { background: T.commerce + "20", color: T.commerce, border: `1px solid ${T.commerce}40` },
  };
  const sizes = { sm: { padding: "4px 10px", fontSize: 11 }, md: { padding: "7px 14px", fontSize: 12 } };
  return (
    <button onClick={onClick} style={{
      ...variants[variant], ...sizes[size],
      borderRadius: 6, cursor: "pointer", fontWeight: 600,
      letterSpacing: "0.02em", transition: "opacity 0.15s",
      ...style,
    }}
    onMouseEnter={e => e.currentTarget.style.opacity = "0.8"}
    onMouseLeave={e => e.currentTarget.style.opacity = "1"}
    >
      {children}
    </button>
  );
}

// ─── Page: Dashboard ───────────────────────────────────────────────────────────
function DashboardPage() {
  const stats = [
    { label: "Active Agents", value: "8", sub: "of 12 deployed", color: T.accent, icon: "◈" },
    { label: "Open Issues", value: "5", sub: "+2 this week", color: T.warn, icon: "◉" },
    { label: "Goals In Progress", value: "3", sub: "65% avg progress", color: T.success, icon: "◎" },
    { label: "Pending Approvals", value: "4", sub: "need review", color: T.commerce, icon: "✦" },
  ];
  return (
    <div>
      <SectionHeader title="Command Center" sub="Paperclip Commerce Edition — AI Workforce Platform" />
      {/* Workflow Steps */}
      <Card style={{ marginBottom: 20, padding: 20 }}>
        <p style={{ margin: "0 0 14px", fontSize: 12, color: T.textDim, textTransform: "uppercase", letterSpacing: "0.08em", fontWeight: 600 }}>Core Workflow</p>
        <div style={{ display: "flex", alignItems: "center", flexWrap: "wrap", gap: 4 }}>
          {WORKFLOW_STEPS.map((s, i) => (
            <div key={s.step} style={{ display: "flex", alignItems: "center", gap: 4 }}>
              <div style={{
                display: "flex", alignItems: "center", gap: 5,
                background: s.color + "15", border: `1px solid ${s.color}40`,
                borderRadius: 20, padding: "4px 10px",
              }}>
                <span style={{ color: s.color, fontSize: 11 }}>{s.icon}</span>
                <span style={{ fontSize: 11, color: T.text, fontWeight: 500 }}>{s.label}</span>
              </div>
              {i < WORKFLOW_STEPS.length - 1 && <span style={{ color: T.textDim, fontSize: 10 }}>→</span>}
            </div>
          ))}
        </div>
      </Card>
      {/* Stats */}
      <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 12, marginBottom: 20 }}>
        {stats.map(s => (
          <Card key={s.label}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
              <span style={{ fontSize: 20, color: s.color }}>{s.icon}</span>
              <span style={{ fontSize: 24, fontWeight: 800, color: T.text }}>{s.value}</span>
            </div>
            <p style={{ margin: "8px 0 2px", fontSize: 12, color: T.textSub, fontWeight: 600 }}>{s.label}</p>
            <p style={{ margin: 0, fontSize: 11, color: T.textDim }}>{s.sub}</p>
          </Card>
        ))}
      </div>
      {/* Two-col */}
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Card>
          <p style={{ margin: "0 0 12px", fontSize: 12, color: T.textDim, fontWeight: 600, textTransform: "uppercase", letterSpacing: "0.07em" }}>Active Goals</p>
          {GOALS.filter(g => g.status === "in-progress").map(g => (
            <div key={g.id} style={{ marginBottom: 12 }}>
              <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5 }}>
                <span style={{ fontSize: 12, color: T.text, fontWeight: 500 }}>{g.title}</span>
                <span style={{ fontSize: 11, color: T.textDim }}>{g.progress}%</span>
              </div>
              <ProgressBar value={g.progress} color={g.priority === "critical" ? T.danger : T.accent} />
            </div>
          ))}
        </Card>
        <Card>
          <p style={{ margin: "0 0 12px", fontSize: 12, color: T.textDim, fontWeight: 600, textTransform: "uppercase", letterSpacing: "0.07em" }}>Recent Issues</p>
          {ISSUES_DATA.slice(0, 4).map(i => (
            <div key={i.id} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 9 }}>
              <StatusDot status={i.status} />
              <span style={{ fontSize: 11, color: T.text, flex: 1, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>{i.title}</span>
              <Badge color={i.priority === "critical" ? T.danger : i.priority === "high" ? T.warn : T.textDim}>{i.priority}</Badge>
            </div>
          ))}
        </Card>
      </div>
      {/* Commerce Edition Banner */}
      <Card style={{ marginTop: 12, padding: 16, background: `linear-gradient(135deg, ${T.commerce}15, ${T.marketplace}15)`, border: `1px solid ${T.commerce}30` }}>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <span style={{ fontSize: 28 }}>◆</span>
          <div>
            <p style={{ margin: 0, fontSize: 14, fontWeight: 700, color: T.text }}>Commerce Edition Active</p>
            <p style={{ margin: "2px 0 0", fontSize: 12, color: T.textSub }}>Marketplace Hub • Content Studio • Social Media • Google Sheets — all online</p>
          </div>
          <div style={{ marginLeft: "auto", display: "flex", gap: 8 }}>
            <Badge color={T.success}>Shopee ✓</Badge>
            <Badge color={T.success}>Tokopedia ✓</Badge>
            <Badge color={T.success}>TikTok Shop ✓</Badge>
            <Badge color={T.warn}>Lazada !</Badge>
          </div>
        </div>
      </Card>
    </div>
  );
}

// ─── Page: Org Chart ───────────────────────────────────────────────────────────
function OrgChartPage() {
  const typeColor = { core: T.accent, commerce: T.commerce };
  const levels = {};
  ORG_DATA.nodes.forEach(n => {
    if (!levels[n.level]) levels[n.level] = [];
    levels[n.level].push(n);
  });
  return (
    <div>
      <SectionHeader title="Org Chart" sub="AI Workforce hierarchy — Commerce Edition expanded roles" />
      <div style={{ overflowX: "auto" }}>
        {Object.entries(levels).map(([level, nodes]) => (
          <div key={level} style={{ display: "flex", justifyContent: "center", gap: 12, marginBottom: 32, position: "relative" }}>
            {nodes.map(node => (
              <div key={node.id} style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
                {node.parent && (
                  <div style={{ width: 2, height: 20, background: `${typeColor[node.type]}40`, margin: "-20px 0 0" }} />
                )}
                <Card style={{
                  width: 150, textAlign: "center", padding: "12px 14px",
                  border: `1px solid ${typeColor[node.type]}50`,
                  background: `${typeColor[node.type]}08`,
                }}>
                  <div style={{ fontSize: 22, color: typeColor[node.type], marginBottom: 4 }}>
                    {node.type === "core" ? "⬡" : "◆"}
                  </div>
                  <p style={{ margin: 0, fontSize: 12, fontWeight: 700, color: T.text }}>{node.title}</p>
                  <p style={{ margin: "3px 0 0", fontSize: 10, color: T.textDim }}>{node.name}</p>
                  {node.type === "commerce" && <Badge color={T.commerce} size="sm" style={{ marginTop: 4 }}>Commerce</Badge>}
                </Card>
              </div>
            ))}
          </div>
        ))}
      </div>
      <Card style={{ marginTop: 8 }}>
        <div style={{ display: "flex", gap: 16 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 6 }}>
            <div style={{ width: 10, height: 10, background: T.accent, borderRadius: 2 }} />
            <span style={{ fontSize: 11, color: T.textSub }}>Core Paperclip Roles</span>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: 6 }}>
            <div style={{ width: 10, height: 10, background: T.commerce, borderRadius: 2 }} />
            <span style={{ fontSize: 11, color: T.textSub }}>Commerce Edition Additions</span>
          </div>
        </div>
      </Card>
    </div>
  );
}

// ─── Page: Agents ──────────────────────────────────────────────────────────────
function AgentsPage() {
  const [selected, setSelected] = useState(null);
  const roleColor = {
    commerce: T.commerce, research: T.accent, content: T.marketplace,
    seo: T.success, creative: T.social, social: T.social,
    ads: T.warn, cs: T.accentHover, sheets: T.sheets, reporting: T.textSub, prompt: T.textDim,
  };
  return (
    <div>
      <SectionHeader
        title="Agents"
        sub="12 agents configured across all Commerce workflows"
        action={<Btn variant="primary">+ New Agent</Btn>}
      />
      <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 12 }}>
        {AGENTS.map(a => (
          <Card key={a.id} onClick={() => setSelected(selected?.id === a.id ? null : a)} style={{ cursor: "pointer" }}>
            <div style={{ display: "flex", alignItems: "flex-start", gap: 10, marginBottom: 10 }}>
              <div style={{
                width: 36, height: 36, borderRadius: 8,
                background: (roleColor[a.role] || T.accent) + "20",
                border: `1px solid ${(roleColor[a.role] || T.accent)}40`,
                display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 16, color: roleColor[a.role] || T.accent, flexShrink: 0,
              }}>◈</div>
              <div style={{ flex: 1, minWidth: 0 }}>
                <p style={{ margin: 0, fontSize: 12, fontWeight: 700, color: T.text, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>{a.name}</p>
                <p style={{ margin: "2px 0 0", fontSize: 10, color: T.textDim }}>{a.provider}</p>
              </div>
              <StatusDot status={a.status} />
            </div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
              <Badge color={roleColor[a.role] || T.accent}>{a.role}</Badge>
              <span style={{ fontSize: 10, color: T.textDim }}>{a.tasks} tasks</span>
            </div>
            {selected?.id === a.id && (
              <div style={{ marginTop: 12, paddingTop: 12, borderTop: `1px solid ${T.border}` }}>
                <p style={{ margin: "0 0 4px", fontSize: 10, color: T.textDim }}>MODEL</p>
                <code style={{ fontSize: 10, color: T.accent, background: T.accentGlow, padding: "2px 6px", borderRadius: 4 }}>{a.model}</code>
                <div style={{ marginTop: 10, display: "flex", gap: 6 }}>
                  <Btn size="sm" variant="ghost">Edit Prompt</Btn>
                  <Btn size="sm" variant="ghost">Change Model</Btn>
                </div>
              </div>
            )}
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Goals ──────────────────────────────────────────────────────────────
function GoalsPage() {
  const statusColor = { "in-progress": T.accent, planned: T.textDim, completed: T.success };
  const priorityColor = { critical: T.danger, high: T.warn, medium: T.accent, low: T.textDim };
  return (
    <div>
      <SectionHeader title="Goals" sub="Company-wide objectives aligned to Org hierarchy" action={<Btn>+ New Goal</Btn>} />
      <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
        {GOALS.map(g => (
          <Card key={g.id}>
            <div style={{ display: "flex", gap: 14, alignItems: "flex-start" }}>
              <div style={{
                width: 44, height: 44, borderRadius: 8,
                background: (statusColor[g.status] || T.accent) + "15",
                display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 20, color: statusColor[g.status] || T.accent, flexShrink: 0,
                border: `1px solid ${(statusColor[g.status] || T.accent)}30`,
              }}>◎</div>
              <div style={{ flex: 1 }}>
                <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 5 }}>
                  <span style={{ fontSize: 13, fontWeight: 700, color: T.text }}>{g.title}</span>
                  <Badge color={priorityColor[g.priority]}>{g.priority}</Badge>
                  <Badge color={statusColor[g.status]}>{g.status}</Badge>
                </div>
                <div style={{ display: "flex", gap: 16, marginBottom: 8 }}>
                  <span style={{ fontSize: 11, color: T.textDim }}>Owner: {g.owner}</span>
                  <span style={{ fontSize: 11, color: T.textDim }}>{g.issues} issues</span>
                </div>
                <ProgressBar value={g.progress} color={priorityColor[g.priority]} />
                <div style={{ display: "flex", justifyContent: "flex-end", marginTop: 4 }}>
                  <span style={{ fontSize: 10, color: T.textDim }}>{g.progress}% complete</span>
                </div>
              </div>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Issues ──────────────────────────────────────────────────────────────
function IssuesPage() {
  const [filter, setFilter] = useState("all");
  const priorityColor = { critical: T.danger, high: T.warn, medium: T.accent, low: T.textDim };
  const statusColor = { open: T.warn, "in-progress": T.accent, review: T.marketplace, closed: T.muted };
  const filtered = filter === "all" ? ISSUES_DATA : ISSUES_DATA.filter(i => i.status === filter);
  return (
    <div>
      <SectionHeader title="Issues" sub="Track and resolve workflow blockers per Goal" action={<Btn>+ New Issue</Btn>} />
      <div style={{ display: "flex", gap: 6, marginBottom: 16 }}>
        {["all", "open", "in-progress", "review", "closed"].map(f => (
          <Btn key={f} size="sm" variant={filter === f ? "primary" : "ghost"} onClick={() => setFilter(f)}>
            {f === "all" ? "All" : f}
          </Btn>
        ))}
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {filtered.map(i => (
          <Card key={i.id} style={{ padding: "12px 16px" }}>
            <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
              <StatusDot status={i.status} />
              <code style={{ fontSize: 10, color: T.textDim, fontFamily: "monospace", background: T.border, padding: "2px 6px", borderRadius: 4 }}>{i.id}</code>
              <span style={{ fontSize: 12, fontWeight: 600, color: T.text, flex: 1 }}>{i.title}</span>
              <Badge color={priorityColor[i.priority]}>{i.priority}</Badge>
              <Badge color={statusColor[i.status]}>{i.status}</Badge>
              <span style={{ fontSize: 10, color: T.textDim, whiteSpace: "nowrap" }}>{i.agent}</span>
              <span style={{ fontSize: 10, color: T.textDim }}>{i.created}</span>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Approvals ──────────────────────────────────────────────────────────
function ApprovalsPage() {
  const [approvals] = useState([
    { id: 1, title: "Product description batch — 247 items", agent: "Product Content Agent", goal: "Product Catalog Automation", risk: "low", created: "2026-06-09", type: "content" },
    { id: 2, title: "Instagram captions — Q3 campaign", agent: "Social Media Agent", goal: "Social Media Growth", risk: "medium", created: "2026-06-09", type: "social" },
    { id: 3, title: "Google Ads copy — Electronics category", agent: "Ads Agent", goal: "Q3 Marketplace Expansion", risk: "high", created: "2026-06-08", type: "ads" },
    { id: 4, title: "SEO meta tags update — 89 pages", agent: "SEO Agent", goal: "SEO Content Strategy 2026", risk: "low", created: "2026-06-08", type: "seo" },
  ]);
  const riskColor = { low: T.success, medium: T.warn, high: T.danger };
  const typeColor = { content: T.marketplace, social: T.social, ads: T.warn, seo: T.success };
  return (
    <div>
      <SectionHeader title="Approval Workflow" sub="Review and approve agent work before execution" />
      <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
        {approvals.map(a => (
          <Card key={a.id} style={{ padding: "16px" }}>
            <div style={{ display: "flex", gap: 12, alignItems: "flex-start" }}>
              <div style={{
                width: 40, height: 40,
                background: (typeColor[a.type] || T.accent) + "15",
                borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 18, color: typeColor[a.type] || T.accent,
                border: `1px solid ${(typeColor[a.type] || T.accent)}30`,
                flexShrink: 0,
              }}>✦</div>
              <div style={{ flex: 1 }}>
                <div style={{ display: "flex", gap: 8, alignItems: "center", marginBottom: 4 }}>
                  <span style={{ fontSize: 13, fontWeight: 700, color: T.text }}>{a.title}</span>
                  <Badge color={riskColor[a.risk]}>Risk: {a.risk}</Badge>
                </div>
                <p style={{ margin: "0 0 8px", fontSize: 11, color: T.textDim }}>By {a.agent} • {a.goal} • {a.created}</p>
                <div style={{ display: "flex", gap: 8 }}>
                  <Btn size="sm" variant="success">✓ Approve</Btn>
                  <Btn size="sm" variant="danger">✕ Reject</Btn>
                  <Btn size="sm" variant="ghost">◎ Review</Btn>
                </div>
              </div>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Marketplace Hub ─────────────────────────────────────────────────────
function MarketplaceHubPage() {
  const [url, setUrl] = useState("");
  const [scraping, setScraping] = useState(false);
  const [scraped, setScraped] = useState(null);
  const connectors = [
    { name: "Shopee", icon: "🛒", status: "connected", color: T.commerce },
    { name: "Tokopedia", icon: "🟢", status: "connected", color: T.success },
    { name: "TikTok Shop", icon: "🎵", status: "connected", color: T.social },
    { name: "Lazada", icon: "🔵", status: "warning", color: T.warn },
    { name: "Bukalapak", icon: "🔴", status: "disconnected", color: T.danger },
  ];
  const scrapeProduct = () => {
    if (!url.trim()) return;
    setScraping(true);
    setTimeout(() => {
      setScraped({
        name: "Wireless Earbuds Pro Max TWS",
        price: "Rp 289.000",
        images: ["🎧", "📦", "🔌"],
        description: "Earbuds TWS dengan teknologi Active Noise Cancelling terdepan. Konektivitas Bluetooth 5.3, baterai 40 jam, IPX5 water resistant.",
        tags: ["wireless", "earbuds", "TWS", "ANC", "bluetooth"],
        rating: "4.8",
        sold: "12.4k",
      });
      setScraping(false);
    }, 1800);
  };
  return (
    <div>
      <SectionHeader title="Marketplace Hub" sub="Scrape, enrich, and push products across all platforms" />
      {/* Connectors */}
      <Card style={{ marginBottom: 16, padding: 16 }}>
        <p style={{ margin: "0 0 12px", fontSize: 11, color: T.textDim, textTransform: "uppercase", letterSpacing: "0.07em", fontWeight: 600 }}>Platform Connectors</p>
        <div style={{ display: "flex", gap: 10 }}>
          {connectors.map(c => (
            <div key={c.name} style={{
              display: "flex", flexDirection: "column", alignItems: "center", gap: 5,
              padding: "10px 16px", borderRadius: 8, cursor: "pointer",
              background: (c.color) + "10", border: `1px solid ${c.color}30`,
            }}>
              <span style={{ fontSize: 20 }}>{c.icon}</span>
              <span style={{ fontSize: 11, color: T.text, fontWeight: 600 }}>{c.name}</span>
              <StatusDot status={c.status === "connected" ? "active" : c.status === "warning" ? "pending" : "error"} />
            </div>
          ))}
        </div>
      </Card>
      {/* Scraper */}
      <Card style={{ marginBottom: 16 }}>
        <p style={{ margin: "0 0 12px", fontSize: 11, color: T.textDim, textTransform: "uppercase", letterSpacing: "0.07em", fontWeight: 600 }}>Product Scraper</p>
        <div style={{ display: "flex", gap: 10, marginBottom: 12 }}>
          <input
            value={url}
            onChange={e => setUrl(e.target.value)}
            placeholder="Paste Shopee / Tokopedia / TikTok Shop product URL..."
            style={{
              flex: 1, background: T.bg, border: `1px solid ${T.border}`, borderRadius: 6,
              padding: "8px 12px", color: T.text, fontSize: 12, outline: "none",
            }}
          />
          <Btn onClick={scrapeProduct} variant="primary">
            {scraping ? "Scraping..." : "⇑ Scrape"}
          </Btn>
        </div>
        {scraping && (
          <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
            {["Extract Image", "Extract Description", "Extract Price", "Parse Tags"].map(t => (
              <div key={t} style={{ display: "flex", alignItems: "center", gap: 5, fontSize: 11, color: T.textDim }}>
                <div style={{ width: 6, height: 6, borderRadius: "50%", background: T.accent, animation: "pulse 1s infinite" }} />
                {t}
              </div>
            ))}
          </div>
        )}
        {scraped && !scraping && (
          <div style={{ borderTop: `1px solid ${T.border}`, paddingTop: 14 }}>
            <div style={{ display: "flex", gap: 14 }}>
              <div style={{ display: "flex", gap: 8 }}>
                {scraped.images.map((img, i) => (
                  <div key={i} style={{
                    width: 60, height: 60, borderRadius: 6, background: T.border,
                    display: "flex", alignItems: "center", justifyContent: "center", fontSize: 28,
                  }}>{img}</div>
                ))}
              </div>
              <div style={{ flex: 1 }}>
                <p style={{ margin: "0 0 4px", fontSize: 13, fontWeight: 700, color: T.text }}>{scraped.name}</p>
                <p style={{ margin: "0 0 6px", fontSize: 13, color: T.commerce, fontWeight: 600 }}>{scraped.price}</p>
                <p style={{ margin: "0 0 8px", fontSize: 11, color: T.textSub, lineHeight: 1.5 }}>{scraped.description}</p>
                <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
                  {scraped.tags.map(t => <Badge key={t} color={T.accent}>{t}</Badge>)}
                </div>
              </div>
            </div>
            <div style={{ display: "flex", gap: 8, marginTop: 14 }}>
              <Btn size="sm" variant="primary">✦ Generate Description</Btn>
              <Btn size="sm" variant="ghost">Generate FAQ</Btn>
              <Btn size="sm" variant="ghost">SEO Tags</Btn>
              <Btn size="sm" variant="success">Push to Sheets</Btn>
            </div>
          </div>
        )}
      </Card>
      {/* Products table */}
      <Card>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
          <p style={{ margin: 0, fontSize: 11, color: T.textDim, textTransform: "uppercase", letterSpacing: "0.07em", fontWeight: 600 }}>Product Queue</p>
          <Btn size="sm" variant="ghost">Import CSV</Btn>
        </div>
        {PRODUCTS_DATA.map(p => (
          <div key={p.id} style={{
            display: "flex", alignItems: "center", gap: 12, padding: "10px 0",
            borderBottom: `1px solid ${T.border}`,
          }}>
            <div style={{ width: 36, height: 36, background: T.border, borderRadius: 6, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 20 }}>{p.image}</div>
            <div style={{ flex: 1 }}>
              <p style={{ margin: 0, fontSize: 12, fontWeight: 600, color: T.text }}>{p.name}</p>
              <p style={{ margin: "2px 0 0", fontSize: 10, color: T.textDim }}>{p.sku} • {p.category}</p>
            </div>
            <span style={{ fontSize: 12, color: T.commerce, fontWeight: 600 }}>{p.price}</span>
            <Badge color={T.marketplace}>{p.platform}</Badge>
            <StatusDot status={p.status} />
            <Btn size="sm" variant="ghost">⇒</Btn>
          </div>
        ))}
      </Card>
    </div>
  );
}

// ─── Page: Content Studio ──────────────────────────────────────────────────────
function ContentStudioPage() {
  const [mode, setMode] = useState("description");
  const [input, setInput] = useState("");
  const [generating, setGenerating] = useState(false);
  const [output, setOutput] = useState("");
  const [platform, setPlatform] = useState("instagram");
  const modes = [
    { id: "description", label: "Product Description", icon: "📝" },
    { id: "caption", label: "Caption Generator", icon: "💬" },
    { id: "hashtag", label: "Hashtag Generator", icon: "#" },
    { id: "faq", label: "FAQ Generator", icon: "❓" },
    { id: "post", label: "Social Post", icon: "📣" },
  ];
  const platforms = ["Instagram", "Facebook", "TikTok", "Shopee", "Tokopedia"];
  const sampleOutputs = {
    description: "✨ Tingkatkan pengalaman audio Anda dengan Wireless Earbuds Pro Max TWS!\n\n🎵 Nikmati suara crystal-clear dengan teknologi driver 10mm premium\n🔇 Active Noise Cancelling (ANC) hingga -35dB — fokus tanpa gangguan\n⚡ 8 jam playtime + 32 jam dengan charging case\n💧 IPX5 water resistant — aman untuk olahraga\n\nDengan konektivitas Bluetooth 5.3, pairing instant dan koneksi stabil dalam radius 15 meter.\n\n#TWS #Earbuds #Bluetooth #ANC #Wireless",
    caption: "🎧 Musik terbaik butuh earbuds terbaik! Pro Max TWS hadir dengan ANC terdepan di kelasnya.\n\nDapatkan 40 jam total playtime dengan harga yang nggak bikin kantong jebol 💸\n\n✅ Bluetooth 5.3\n✅ IPX5 Water Resistant\n✅ Touch Control\n\nShop now → link di bio!\n\n#WirelessEarbuds #TWS #AudioGear",
    hashtag: "#WirelessEarbuds #TWS #Bluetooth #ANC #Earbuds #AudioTech #TrueWireless #IPX5 #NoiseCancel #MusicLovers #Headphones #Indonesia #ShopNow #TokopediaBest #ShopeeFashion",
    faq: "**Q: Berapa lama baterai bertahan?**\nA: 8 jam playtime per charge, dengan charging case memberikan total 40 jam.\n\n**Q: Apakah bisa digunakan saat hujan?**\nA: Ya! Rating IPX5 membuatnya tahan terhadap percikan air dan keringat.\n\n**Q: Bagaimana cara menghubungkan ke HP?**\nA: Buka case, ambil earbuds, aktifkan Bluetooth di HP Anda — koneksi otomatis!",
    post: "🚀 FLASH SALE HARI INI!\n\nWireless Earbuds Pro Max TWS\n💰 Harga Normal: Rp 459.000\n🔥 Harga Flash: Rp 289.000 (HEMAT 37%!)\n\nBerlaku hari ini 10:00 - 22:00\n\n📱 Order via:\n🛒 Shopee: bit.ly/shopee-tws\n🟢 Tokopedia: bit.ly/toped-tws\n🎵 TikTok Shop: @tokosaya",
  };
  const generate = () => {
    if (!input.trim() && mode !== "hashtag") return;
    setGenerating(true);
    setOutput("");
    setTimeout(() => {
      setOutput(sampleOutputs[mode] || "Generated content will appear here.");
      setGenerating(false);
    }, 1600);
  };
  return (
    <div>
      <SectionHeader title="Content Studio" sub="AI-powered content generation for all platforms" />
      <div style={{ display: "flex", gap: 6, marginBottom: 16 }}>
        {modes.map(m => (
          <Btn key={m.id} size="sm" variant={mode === m.id ? "primary" : "ghost"} onClick={() => setMode(m.id)}>
            {m.icon} {m.label}
          </Btn>
        ))}
      </div>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14 }}>
        <Card>
          <p style={{ margin: "0 0 10px", fontSize: 11, color: T.textDim, fontWeight: 600, textTransform: "uppercase" }}>Input</p>
          <div style={{ display: "flex", gap: 6, marginBottom: 10, flexWrap: "wrap" }}>
            <span style={{ fontSize: 11, color: T.textDim }}>Platform:</span>
            {platforms.map(p => (
              <Btn key={p} size="sm" variant={platform === p.toLowerCase() ? "primary" : "ghost"} onClick={() => setPlatform(p.toLowerCase())}>{p}</Btn>
            ))}
          </div>
          <textarea
            value={input}
            onChange={e => setInput(e.target.value)}
            placeholder={mode === "caption" ? "Describe your product..." : mode === "hashtag" ? "Enter keywords or product name..." : "Enter product name, category, key features..."}
            rows={6}
            style={{
              width: "100%", background: T.bg, border: `1px solid ${T.border}`,
              borderRadius: 6, padding: "10px 12px", color: T.text, fontSize: 12,
              resize: "vertical", outline: "none", boxSizing: "border-box",
              fontFamily: "inherit", lineHeight: 1.6,
            }}
          />
          <div style={{ marginTop: 10, display: "flex", gap: 8, alignItems: "center" }}>
            <Btn onClick={generate} variant="primary">
              {generating ? "◈ Generating..." : "⚡ Generate"}
            </Btn>
            <span style={{ fontSize: 11, color: T.textDim }}>via {AGENTS.find(a => a.role === "content")?.provider || "Claude Sonnet"}</span>
          </div>
        </Card>
        <Card>
          <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 10 }}>
            <p style={{ margin: 0, fontSize: 11, color: T.textDim, fontWeight: 600, textTransform: "uppercase" }}>Output</p>
            {output && (
              <div style={{ display: "flex", gap: 6 }}>
                <Btn size="sm" variant="ghost">Copy</Btn>
                <Btn size="sm" variant="success">Push to Sheets</Btn>
              </div>
            )}
          </div>
          {generating && (
            <div style={{ display: "flex", flexDirection: "column", gap: 8, padding: "20px 0" }}>
              {[90, 70, 80, 50].map((w, i) => (
                <div key={i} style={{ height: 10, background: T.border, borderRadius: 4, width: `${w}%`, opacity: 0.5 }} />
              ))}
            </div>
          )}
          {output && !generating && (
            <pre style={{
              margin: 0, fontSize: 12, color: T.text, whiteSpace: "pre-wrap",
              lineHeight: 1.6, fontFamily: "inherit", background: T.bg,
              border: `1px solid ${T.border}`, borderRadius: 6, padding: "12px",
              minHeight: 180,
            }}>{output}</pre>
          )}
          {!output && !generating && (
            <div style={{ textAlign: "center", padding: "40px 0", color: T.textDim, fontSize: 12 }}>
              <p style={{ margin: 0, fontSize: 24, marginBottom: 8 }}>✦</p>
              <p style={{ margin: 0 }}>Content will appear here after generation</p>
            </div>
          )}
        </Card>
      </div>
    </div>
  );
}

// ─── Page: Social Media ────────────────────────────────────────────────────────
function SocialMediaPage() {
  const [posts] = useState([
    { id: 1, platform: "Instagram", caption: "🎧 Pro Max TWS — experience next-level sound...", status: "published", engagement: "4.2k", date: "2026-06-09" },
    { id: 2, platform: "TikTok", caption: "Unboxing Wireless Earbuds terbaru! Review jujur...", status: "scheduled", engagement: "-", date: "2026-06-10" },
    { id: 3, platform: "Facebook", caption: "Flash Sale hari ini! Wireless Earbuds 37% off...", status: "draft", engagement: "-", date: "2026-06-11" },
    { id: 4, platform: "Instagram", caption: "✨ Cara rawat earbuds biar tahan lama — tips...", status: "published", engagement: "1.8k", date: "2026-06-08" },
  ]);
  const platformColor = { Instagram: T.social, TikTok: "#000", Facebook: "#1877F2", Twitter: T.accent };
  const statusColor = { published: T.success, scheduled: T.warn, draft: T.textDim };
  const stats = [
    { label: "Total Reach", value: "48.2k", delta: "+12%" },
    { label: "Engagement", value: "3.8k", delta: "+7%" },
    { label: "Posts This Week", value: "9", delta: "+3" },
    { label: "Scheduled", value: "4", delta: "" },
  ];
  return (
    <div>
      <SectionHeader title="Social Media" sub="Manage and automate posts across all platforms" action={<Btn>+ New Post</Btn>} />
      <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 10, marginBottom: 16 }}>
        {stats.map(s => (
          <Card key={s.label} style={{ padding: "12px 14px" }}>
            <p style={{ margin: "0 0 4px", fontSize: 20, fontWeight: 800, color: T.text }}>{s.value}</p>
            <p style={{ margin: 0, fontSize: 11, color: T.textDim }}>{s.label}</p>
            {s.delta && <span style={{ fontSize: 10, color: T.success }}>{s.delta}</span>}
          </Card>
        ))}
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {posts.map(p => (
          <Card key={p.id} style={{ padding: "12px 16px" }}>
            <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
              <div style={{
                padding: "5px 10px", borderRadius: 20,
                background: (platformColor[p.platform] || T.accent) + "20",
                border: `1px solid ${(platformColor[p.platform] || T.accent)}40`,
                fontSize: 11, fontWeight: 700, color: platformColor[p.platform] || T.accent,
                flexShrink: 0,
              }}>{p.platform}</div>
              <p style={{ margin: 0, fontSize: 12, color: T.textSub, flex: 1, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>{p.caption}</p>
              <Badge color={statusColor[p.status]}>{p.status}</Badge>
              <span style={{ fontSize: 11, color: T.textDim }}>{p.engagement} engagements</span>
              <span style={{ fontSize: 11, color: T.textDim }}>{p.date}</span>
              <Btn size="sm" variant="ghost">Edit</Btn>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Google Sheets ───────────────────────────────────────────────────────
function GoogleSheetsPage() {
  const [connected, setConnected] = useState(false);
  const [syncing, setSyncing] = useState(false);
  const sheets = [
    { name: "Product Catalog Master", rows: 1247, lastSync: "2026-06-09 14:32", status: "synced" },
    { name: "SEO Content Queue", rows: 384, lastSync: "2026-06-09 09:15", status: "synced" },
    { name: "Social Media Calendar", rows: 96, lastSync: "2026-06-08 20:00", status: "pending" },
    { name: "Ads Performance", rows: 512, lastSync: "2026-06-07 11:30", status: "error" },
  ];
  return (
    <div>
      <SectionHeader title="Google Sheets Integration" sub="Sync product data, content queues, and reports" />
      {!connected ? (
        <Card style={{ textAlign: "center", padding: "48px 24px", background: `${T.sheets}08`, border: `1px solid ${T.sheets}30` }}>
          <p style={{ fontSize: 36, marginBottom: 12 }}>⊞</p>
          <h3 style={{ color: T.text, margin: "0 0 8px", fontSize: 18 }}>Connect Google Workspace</h3>
          <p style={{ color: T.textDim, margin: "0 0 20px", fontSize: 13 }}>Link your Google account to sync product catalogs, content queues, and reports automatically.</p>
          <Btn onClick={() => setConnected(true)} variant="success">Connect Google Account</Btn>
        </Card>
      ) : (
        <div>
          <Card style={{ marginBottom: 14, padding: "12px 16px", background: `${T.sheets}08`, border: `1px solid ${T.sheets}30` }}>
            <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ fontSize: 18, color: T.sheets }}>⊞</span>
              <div style={{ flex: 1 }}>
                <p style={{ margin: 0, fontSize: 12, fontWeight: 700, color: T.text }}>Google Workspace Connected</p>
                <p style={{ margin: "2px 0 0", fontSize: 11, color: T.textDim }}>commerce@company.com • 4 sheets active</p>
              </div>
              <Badge color={T.success}>Connected</Badge>
              <Btn size="sm" variant="ghost" onClick={() => setConnected(false)}>Disconnect</Btn>
            </div>
          </Card>
          <div style={{ display: "flex", gap: 10, marginBottom: 14 }}>
            <Btn variant="success" onClick={() => { setSyncing(true); setTimeout(() => setSyncing(false), 2000); }}>
              {syncing ? "↺ Syncing..." : "↺ Sync All"}
            </Btn>
            <Btn variant="ghost">+ Create Sheet</Btn>
            <Btn variant="ghost">⊞ Import Mapping</Btn>
          </div>
          <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
            {sheets.map(s => (
              <Card key={s.name} style={{ padding: "12px 16px" }}>
                <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                  <span style={{ fontSize: 18, color: T.sheets }}>⊞</span>
                  <div style={{ flex: 1 }}>
                    <p style={{ margin: 0, fontSize: 12, fontWeight: 600, color: T.text }}>{s.name}</p>
                    <p style={{ margin: "2px 0 0", fontSize: 10, color: T.textDim }}>{s.rows.toLocaleString()} rows • Last sync: {s.lastSync}</p>
                  </div>
                  <StatusDot status={s.status === "synced" ? "active" : s.status === "pending" ? "pending" : "error"} />
                  <Badge color={s.status === "synced" ? T.success : s.status === "pending" ? T.warn : T.danger}>{s.status}</Badge>
                  <Btn size="sm" variant="ghost">Open</Btn>
                  <Btn size="sm" variant="ghost">Sync</Btn>
                </div>
              </Card>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}

// ─── Page: AI Providers ────────────────────────────────────────────────────────
function AIProvidersPage() {
  const [providers, setProviders] = useState(AI_PROVIDERS_DATA);
  const [editId, setEditId] = useState(null);
  const toggle = (id) => setProviders(prev => prev.map(p => p.id === id ? { ...p, enabled: !p.enabled } : p));
  return (
    <div>
      <SectionHeader title="AI Providers" sub="Configure and enable models per agent" />
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {providers.map(p => (
          <Card key={p.id} style={{ padding: "12px 16px" }}>
            <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
              <div style={{
                width: 34, height: 34, borderRadius: 7,
                background: p.enabled ? T.accent + "20" : T.border,
                border: `1px solid ${p.enabled ? T.accent + "50" : T.border}`,
                display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 16, color: p.enabled ? T.accent : T.textDim, flexShrink: 0,
              }}>◈</div>
              <div style={{ flex: 1, minWidth: 0 }}>
                <p style={{ margin: 0, fontSize: 12, fontWeight: 700, color: T.text }}>{p.name}</p>
                <p style={{ margin: "2px 0 0", fontSize: 10, color: T.textDim, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>
                  {p.model} • {p.endpoint || "endpoint not set"}
                </p>
              </div>
              {p.apiKey && (
                <code style={{ fontSize: 10, color: T.textDim, background: T.bg, padding: "2px 7px", borderRadius: 4 }}>
                  {p.apiKey}
                </code>
              )}
              <Toggle value={p.enabled} onChange={() => toggle(p.id)} />
              <Btn size="sm" variant="ghost" onClick={() => setEditId(editId === p.id ? null : p.id)}>Configure</Btn>
            </div>
            {editId === p.id && (
              <div style={{ marginTop: 12, paddingTop: 12, borderTop: `1px solid ${T.border}`, display: "flex", flexDirection: "column", gap: 8 }}>
                {["Model ID", "Endpoint URL", "API Key"].map(field => (
                  <div key={field} style={{ display: "flex", gap: 10, alignItems: "center" }}>
                    <label style={{ fontSize: 10, color: T.textDim, width: 80, flexShrink: 0 }}>{field}</label>
                    <input
                      defaultValue={field === "Model ID" ? p.model : field === "Endpoint URL" ? p.endpoint : p.apiKey}
                      placeholder={`Enter ${field.toLowerCase()}...`}
                      type={field === "API Key" ? "password" : "text"}
                      style={{
                        flex: 1, background: T.bg, border: `1px solid ${T.border}`, borderRadius: 5,
                        padding: "5px 9px", color: T.text, fontSize: 11, outline: "none",
                      }}
                    />
                  </div>
                ))}
                <div style={{ display: "flex", gap: 8 }}>
                  <Btn size="sm" variant="primary">Save</Btn>
                  <Btn size="sm" variant="ghost">Test Connection</Btn>
                </div>
              </div>
            )}
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Page: Settings Generic ────────────────────────────────────────────────────
function SettingsPage({ title, sub, options, selected, setSelected }) {
  return (
    <div>
      <SectionHeader title={title} sub={sub} />
      <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 10 }}>
        {options.map(o => (
          <Card key={o.id} onClick={() => setSelected(o.id)} style={{
            cursor: "pointer", textAlign: "center", padding: "20px 16px",
            border: `1px solid ${selected === o.id ? T.accent : T.border}`,
            background: selected === o.id ? T.accentGlow : T.card,
          }}>
            <p style={{ margin: "0 0 8px", fontSize: 28 }}>{o.icon}</p>
            <p style={{ margin: "0 0 4px", fontSize: 13, fontWeight: 700, color: T.text }}>{o.label}</p>
            <p style={{ margin: 0, fontSize: 10, color: T.textDim }}>{o.sub}</p>
            {selected === o.id && <div style={{ marginTop: 10 }}><Badge color={T.success}>Active</Badge></div>}
          </Card>
        ))}
      </div>
    </div>
  );
}

// ─── Main App ──────────────────────────────────────────────────────────────────
export default function App() {
  const [active, setActive] = useState("dashboard");
  const [navOpen, setNavOpen] = useState(true);
  const [runtime, setRuntime] = useState("docker");
  const [storage, setStorage] = useState("local");
  const [database, setDatabase] = useState("postgresql");

  const groupedNav = Object.entries(GROUP_LABELS).map(([key, label]) => ({
    group: key, label, items: NAV.filter(n => n.group === key),
  }));

  const groupColor = { core: T.accent, commerce: T.commerce, integrations: T.sheets, settings: T.textDim };

  const renderPage = () => {
    switch (active) {
      case "dashboard": return <DashboardPage />;
      case "org-chart": return <OrgChartPage />;
      case "agents": return <AgentsPage />;
      case "goals": return <GoalsPage />;
      case "issues": return <IssuesPage />;
      case "approvals": return <ApprovalsPage />;
      case "marketplace-hub": return <MarketplaceHubPage />;
      case "content-studio": return <ContentStudioPage />;
      case "social-media": return <SocialMediaPage />;
      case "google-sheets": return <GoogleSheetsPage />;
      case "ai-providers": return <AIProvidersPage />;
      case "products": return <ProductsPage />;
      case "runtime": return <SettingsPage
        title="Runtime" sub="Choose where agents execute"
        options={[
          { id: "docker", icon: "🐳", label: "Docker", sub: "Container-based local runtime" },
          { id: "kubernetes", icon: "☸️", label: "Kubernetes", sub: "Scalable cluster orchestration" },
          { id: "railway", icon: "🚄", label: "Railway", sub: "Zero-config cloud deployment" },
          { id: "coolify", icon: "❄️", label: "Coolify", sub: "Self-hosted Heroku alternative" },
          { id: "vps", icon: "🖥️", label: "VPS", sub: "Raw virtual private server" },
          { id: "render", icon: "🌐", label: "Render", sub: "Managed cloud hosting" },
          { id: "fly", icon: "✈️", label: "Fly.io", sub: "Edge deployment network" },
          { id: "localhost", icon: "💻", label: "Localhost", sub: "Local development mode" },
        ]}
        selected={runtime} setSelected={setRuntime}
      />;
      case "storage": return <SettingsPage
        title="Storage" sub="Configure where files and assets are stored"
        options={[
          { id: "local", icon: "💾", label: "Local", sub: "Store files on server disk" },
          { id: "s3", icon: "☁️", label: "Amazon S3", sub: "AWS object storage" },
          { id: "r2", icon: "🟠", label: "Cloudflare R2", sub: "Zero-egress object storage" },
          { id: "minio", icon: "🪣", label: "MinIO", sub: "Self-hosted S3-compatible" },
          { id: "supabase", icon: "⚡", label: "Supabase Storage", sub: "Postgres-native file storage" },
        ]}
        selected={storage} setSelected={setStorage}
      />;
      case "database": return <SettingsPage
        title="Database" sub="Select the primary datastore for Paperclip Commerce"
        options={[
          { id: "postgresql", icon: "🐘", label: "PostgreSQL", sub: "Recommended — full features" },
          { id: "mysql", icon: "🐬", label: "MySQL", sub: "Wide compatibility" },
          { id: "mariadb", icon: "🦭", label: "MariaDB", sub: "MySQL-compatible fork" },
          { id: "sqlite", icon: "📄", label: "SQLite", sub: "Embedded — zero config" },
          { id: "supabase", icon: "⚡", label: "Supabase", sub: "PostgreSQL + realtime" },
          { id: "neon", icon: "🌙", label: "Neon", sub: "Serverless PostgreSQL" },
        ]}
        selected={database} setSelected={setDatabase}
      />;
      default: return <DashboardPage />;
    }
  };

  return (
    <div style={{ display: "flex", height: "100vh", background: T.bg, fontFamily: "'Inter', 'Segoe UI', system-ui, sans-serif", color: T.text, overflow: "hidden" }}>
      {/* Sidebar */}
      <div style={{
        width: navOpen ? 220 : 54, flexShrink: 0,
        background: T.surface, borderRight: `1px solid ${T.border}`,
        display: "flex", flexDirection: "column", overflow: "hidden",
        transition: "width 0.2s ease",
      }}>
        {/* Logo */}
        <div style={{
          padding: "16px 14px", borderBottom: `1px solid ${T.border}`,
          display: "flex", alignItems: "center", gap: 10, flexShrink: 0,
        }}>
          <div style={{
            width: 28, height: 28, borderRadius: 7,
            background: `linear-gradient(135deg, ${T.accent}, ${T.commerce})`,
            display: "flex", alignItems: "center", justifyContent: "center",
            fontSize: 14, color: "#fff", fontWeight: 900, flexShrink: 0,
          }}>◆</div>
          {navOpen && (
            <div>
              <p style={{ margin: 0, fontSize: 12, fontWeight: 800, color: T.text, letterSpacing: "-0.02em" }}>Paperclip</p>
              <p style={{ margin: 0, fontSize: 9, color: T.commerce, fontWeight: 700, letterSpacing: "0.08em", textTransform: "uppercase" }}>Commerce Edition</p>
            </div>
          )}
          <button
            onClick={() => setNavOpen(!navOpen)}
            style={{
              marginLeft: "auto", background: "none", border: "none", color: T.textDim,
              cursor: "pointer", fontSize: 14, padding: 4, flexShrink: 0,
            }}
          >≡</button>
        </div>
        {/* Nav */}
        <div style={{ flex: 1, overflowY: "auto", padding: "10px 8px" }}>
          {groupedNav.map(({ group, label, items }) => (
            <div key={group} style={{ marginBottom: 6 }}>
              {navOpen && (
                <p style={{
                  margin: "10px 6px 5px", fontSize: 9, color: groupColor[group] || T.textDim,
                  textTransform: "uppercase", letterSpacing: "0.1em", fontWeight: 700,
                }}>{label}</p>
              )}
              {items.map(item => (
                <button key={item.id} onClick={() => setActive(item.id)} style={{
                  display: "flex", alignItems: "center", gap: 9,
                  width: "100%", padding: navOpen ? "7px 8px" : "8px 0",
                  justifyContent: navOpen ? "flex-start" : "center",
                  background: active === item.id ? T.accentGlow : "transparent",
                  border: `1px solid ${active === item.id ? T.accent + "40" : "transparent"}`,
                  borderRadius: 6, cursor: "pointer",
                  color: active === item.id ? T.accent : T.textSub,
                  fontSize: 11, fontWeight: active === item.id ? 700 : 400,
                  transition: "all 0.1s", textAlign: "left",
                }}
                onMouseEnter={e => { if (active !== item.id) e.currentTarget.style.background = T.border + "40"; }}
                onMouseLeave={e => { if (active !== item.id) e.currentTarget.style.background = "transparent"; }}
                >
                  <span style={{ fontSize: 12, flexShrink: 0 }}>{item.icon}</span>
                  {navOpen && <span>{item.label}</span>}
                </button>
              ))}
              {navOpen && group !== "settings" && (
                <div style={{ height: 1, background: T.border, margin: "8px 6px 4px" }} />
              )}
            </div>
          ))}
        </div>
        {/* Footer */}
        {navOpen && (
          <div style={{ padding: "12px 14px", borderTop: `1px solid ${T.border}` }}>
            <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
              <div style={{
                width: 26, height: 26, borderRadius: "50%",
                background: T.marketplace + "30", display: "flex", alignItems: "center",
                justifyContent: "center", fontSize: 12, color: T.marketplace, flexShrink: 0,
              }}>◉</div>
              <div>
                <p style={{ margin: 0, fontSize: 11, color: T.text, fontWeight: 600 }}>Admin</p>
                <p style={{ margin: 0, fontSize: 9, color: T.textDim }}>admin@commerce.co</p>
              </div>
            </div>
          </div>
        )}
      </div>
      {/* Main content */}
      <div style={{ flex: 1, overflow: "hidden", display: "flex", flexDirection: "column" }}>
        {/* Topbar */}
        <div style={{
          padding: "0 20px", height: 52, borderBottom: `1px solid ${T.border}`,
          display: "flex", alignItems: "center", gap: 12,
          background: T.surface, flexShrink: 0,
        }}>
          <div style={{ flex: 1, display: "flex", alignItems: "center", gap: 8 }}>
            <span style={{ fontSize: 11, color: T.textDim }}>Paperclip Commerce</span>
            <span style={{ color: T.textDim }}>›</span>
            <span style={{ fontSize: 11, color: T.text, fontWeight: 600 }}>
              {NAV.find(n => n.id === active)?.label || "Dashboard"}
            </span>
          </div>
          <div style={{ display: "flex", gap: 8 }}>
            <Badge color={T.success}>v1.0.0-commerce</Badge>
            <Badge color={T.commerce}>8/12 Agents Active</Badge>
          </div>
        </div>
        {/* Page */}
        <div style={{ flex: 1, overflowY: "auto", padding: "24px 28px" }}>
          {renderPage()}
        </div>
      </div>
      <style>{`
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 5px; height: 5px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #252A38; border-radius: 3px; }
        input::placeholder { color: #4B5563; }
        textarea::placeholder { color: #4B5563; }
        @keyframes pulse { 0%,100% { opacity:1; } 50% { opacity:0.3; } }
      `}</style>
    </div>
  );
}

// Products sub-page (inline)
function ProductsPage() {
  return (
    <div>
      <SectionHeader title="Products" sub="All products across marketplace platforms" action={<Btn>+ Import Products</Btn>} />
      <div style={{ display: "flex", gap: 8, marginBottom: 14 }}>
        {["All", "Shopee", "Tokopedia", "TikTok Shop", "Lazada"].map(p => (
          <Btn key={p} size="sm" variant={p === "All" ? "primary" : "ghost"}>{p}</Btn>
        ))}
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {PRODUCTS_DATA.map(p => (
          <Card key={p.id} style={{ padding: "14px 16px" }}>
            <div style={{ display: "flex", gap: 14, alignItems: "center" }}>
              <div style={{ width: 48, height: 48, background: T.border, borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 28, flexShrink: 0 }}>{p.image}</div>
              <div style={{ flex: 1 }}>
                <p style={{ margin: "0 0 3px", fontSize: 13, fontWeight: 700, color: T.text }}>{p.name}</p>
                <p style={{ margin: 0, fontSize: 11, color: T.textDim }}>SKU: {p.sku} • {p.category}</p>
              </div>
              <span style={{ fontSize: 13, fontWeight: 700, color: T.commerce }}>{p.price}</span>
              <Badge color={T.marketplace}>{p.platform}</Badge>
              <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                <StatusDot status={p.status === "synced" ? "active" : p.status === "pending" ? "pending" : "error"} />
                <span style={{ fontSize: 10, color: T.textDim }}>{p.status}</span>
              </div>
              <div style={{ display: "flex", gap: 6 }}>
                <Btn size="sm" variant="ghost">✎ Edit</Btn>
                <Btn size="sm" variant="ghost">⇒ Push</Btn>
              </div>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}
