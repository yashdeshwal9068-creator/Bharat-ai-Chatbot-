import { useState, useRef, useEffect } from "react";

// ═══════════════════════════════════════════════════════════════
// PERSONA CONFIGS
// ═══════════════════════════════════════════════════════════════
const PERSONAS = {
  nexus: {
    id: "nexus", name: "NEXUS", tag: "Balanced Intelligence",
    icon: "◈", color: "#00d4ff", glow: "rgba(0,212,255,0.15)",
    gradient: "linear-gradient(135deg,#00d4ff,#0066ff)",
    system: `You are NEXUS, an exceptionally intelligent and balanced AI assistant. You are thoughtful, helpful, and precise. Format your responses using markdown: **bold** for key terms, \`backticks\` for inline code, triple-backtick fenced code blocks with a language tag (e.g. \`\`\`python) for multi-line code, ## and ### for section headers, - for bullet lists, and 1. 2. 3. for numbered steps.`,
  },
  oracle: {
    id: "oracle", name: "ORACLE", tag: "Creative Insight",
    icon: "◉", color: "#c084fc", glow: "rgba(192,132,252,0.15)",
    gradient: "linear-gradient(135deg,#c084fc,#ec4899)",
    system: `You are ORACLE, a visionary and creative AI. You think expansively, forge surprising connections, and inspire new perspectives. Respond with both beauty and substance. Use vivid language and unexpected angles. Use markdown formatting when helpful.`,
  },
  axiom: {
    id: "axiom", name: "AXIOM", tag: "Precision Analysis",
    icon: "◇", color: "#34d399", glow: "rgba(52,211,153,0.15)",
    gradient: "linear-gradient(135deg,#34d399,#059669)",
    system: `You are AXIOM, an analytical AI optimized for precision and structure. Prioritize accuracy, brevity, and clear organization. Use markdown heavily: ## headers, bullet lists, numbered steps, **bold** for important terms, tables when comparing data.`,
  },
  kernel: {
    id: "kernel", name: "KERNEL", tag: "Code Expert",
    icon: "⬡", color: "#fbbf24", glow: "rgba(251,191,36,0.15)",
    gradient: "linear-gradient(135deg,#fbbf24,#f97316)",
    system: `You are KERNEL, an elite software engineer AI. Provide expert-level code, debugging help, and architectural guidance. Always use proper fenced markdown code blocks with the correct language identifier (e.g. \`\`\`python, \`\`\`javascript, \`\`\`bash). Write clean, well-commented, production-quality code. Explain your reasoning thoroughly.`,
  },
};

const SUGGESTIONS = [
  "Explain quantum computing in simple terms",
  "Write a Python web scraper with requests",
  "Pros and cons of microservices architecture",
  "Write a short noir detective story",
  "Solve: derivative of x³sin(x)",
  "Best practices for React performance",
];

// ═══════════════════════════════════════════════════════════════
// CODE BLOCK
// ═══════════════════════════════════════════════════════════════
function CodeBlock({ lang, code, color }) {
  const [copied, setCopied] = useState(false);
  const copy = () => {
    navigator.clipboard.writeText(code.trim());
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };
  return (
    <div style={{ margin: "14px 0", borderRadius: 10, overflow: "hidden", border: "1px solid rgba(255,255,255,0.07)", background: "#030710" }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "7px 14px", background: "rgba(255,255,255,0.03)", borderBottom: "1px solid rgba(255,255,255,0.06)" }}>
        <span style={{ fontFamily: "'JetBrains Mono',monospace", fontSize: 11, color, letterSpacing: "0.1em", textTransform: "uppercase" }}>
          {lang || "code"}
        </span>
        <button onClick={copy} style={{ background: "none", border: `1px solid ${copied ? color : "rgba(255,255,255,0.12)"}`, color: copied ? color : "rgba(255,255,255,0.4)", cursor: "pointer", padding: "2px 10px", borderRadius: 5, fontSize: 10, fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.08em", transition: "all 0.2s" }}>
          {copied ? "✓ COPIED" : "⧉ COPY"}
        </button>
      </div>
      <pre style={{ margin: 0, padding: "16px 18px", overflowX: "auto", fontFamily: "'JetBrains Mono',monospace", fontSize: 13, lineHeight: 1.8, color: "#dde8f5" }}>
        <code>{code.trim()}</code>
      </pre>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════
// MARKDOWN RENDERER
// ═══════════════════════════════════════════════════════════════
function Markdown({ text, color }) {
  const segments = text.split(/(```[\s\S]*?```)/g);
  return (
    <div style={{ fontSize: 14.5, lineHeight: 1.8 }}>
      {segments.map((seg, i) => {
        if (seg.startsWith("```") && seg.endsWith("```")) {
          const inner = seg.slice(3, -3);
          const nl = inner.indexOf("\n");
          const lang = nl > -1 ? inner.slice(0, nl).trim() : "";
          const code = nl > -1 ? inner.slice(nl + 1) : inner;
          return <CodeBlock key={i} lang={lang} code={code} color={color} />;
        }
        const html = seg
          .replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;")
          .replace(/^### (.+)$/gm, `<div style="color:${color};font-size:15px;font-weight:700;margin:14px 0 6px;font-family:'Oxanium',sans-serif;letter-spacing:0.05em">$1</div>`)
          .replace(/^## (.+)$/gm, `<div style="color:${color};font-size:17px;font-weight:700;margin:18px 0 8px;font-family:'Oxanium',sans-serif;letter-spacing:0.05em">$1</div>`)
          .replace(/^# (.+)$/gm, `<div style="color:${color};font-size:20px;font-weight:800;margin:20px 0 10px;font-family:'Oxanium',sans-serif">$1</div>`)
          .replace(/\*\*\*(.+?)\*\*\*/g, "<b><i>$1</i></b>")
          .replace(/\*\*(.+?)\*\*/g, `<b style="color:#f0f4ff;font-weight:600">$1</b>`)
          .replace(/\*(.+?)\*/g, `<i style="color:rgba(240,244,255,0.75)">$1</i>`)
          .replace(/`([^`\n]+)`/g, `<code style="font-family:'JetBrains Mono',monospace;font-size:0.84em;background:rgba(255,255,255,0.07);border:1px solid rgba(255,255,255,0.1);padding:1px 7px;border-radius:4px;color:${color}">$1</code>`)
          .replace(/^[-*] (.+)$/gm, `<div style="display:flex;gap:8px;margin:4px 0;padding-left:2px"><span style="color:${color};margin-top:4px;font-size:9px;flex-shrink:0">▸</span><span>$1</span></div>`)
          .replace(/^(\d+)\. (.+)$/gm, `<div style="display:flex;gap:8px;margin:4px 0;padding-left:2px"><span style="color:${color};font-weight:700;min-width:18px;font-size:13px">$1.</span><span>$2</span></div>`)
          .replace(/^&gt; (.+)$/gm, `<div style="border-left:3px solid ${color};padding:6px 14px;background:rgba(255,255,255,0.03);margin:8px 0;color:rgba(255,255,255,0.65);border-radius:0 6px 6px 0;font-style:italic">$1</div>`)
          .replace(/\n\n+/g, `</p><p style="margin:0.55em 0">`)
          .replace(/\n/g, "<br/>");
        return (
          <div key={i} dangerouslySetInnerHTML={{ __html: `<p style="margin:0">${html}</p>` }} />
        );
      })}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════
// MESSAGE BUBBLE
// ═══════════════════════════════════════════════════════════════
function MessageBubble({ msg, persona, isTyping = false, typingText = "" }) {
  const [copied, setCopied] = useState(false);
  const [hovered, setHovered] = useState(false);
  const isUser = msg.role === "user";
  const content = isTyping ? typingText : msg.content;
  const copy = () => {
    navigator.clipboard.writeText(msg.content || "");
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };
  const time = msg.ts ? new Date(msg.ts).toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" }) : "";

  return (
    <div className="msg-anim" style={{ display: "flex", flexDirection: isUser ? "row-reverse" : "row", gap: 10, padding: "6px 0", alignItems: "flex-start" }}>
      {/* Avatar */}
      <div style={{
        width: 30, height: 30, borderRadius: "50%", flexShrink: 0, marginTop: 2,
        background: isUser ? "rgba(255,255,255,0.07)" : persona.glow,
        border: `1px solid ${isUser ? "rgba(255,255,255,0.14)" : persona.color + "55"}`,
        display: "flex", alignItems: "center", justifyContent: "center",
        fontSize: isUser ? 10 : 13, color: isUser ? "rgba(255,255,255,0.55)" : persona.color,
        fontFamily: "'Oxanium',sans-serif", fontWeight: 700,
      }}>
        {isUser ? "YOU" : persona.icon}
      </div>

      {/* Bubble + timestamp */}
      <div style={{ maxWidth: "78%", display: "flex", flexDirection: "column", gap: 3, alignItems: isUser ? "flex-end" : "flex-start" }}
        onMouseEnter={() => setHovered(true)}
        onMouseLeave={() => setHovered(false)}>
        <div style={{
          background: isUser ? "rgba(255,255,255,0.055)" : "rgba(255,255,255,0.028)",
          border: `1px solid ${isUser ? "rgba(255,255,255,0.1)" : "rgba(255,255,255,0.065)"}`,
          borderRadius: isUser ? "12px 2px 12px 12px" : "2px 12px 12px 12px",
          padding: "11px 16px", position: "relative",
        }}>
          {isUser
            ? <span style={{ fontSize: 14.5, lineHeight: 1.75, color: "#f0f4ff" }}>{content}</span>
            : <><Markdown text={content || " "} color={persona.color} />{isTyping && <span className="cursor" style={{ color: persona.color }} />}</>
          }
          {/* Copy btn */}
          {!isTyping && (
            <button onClick={copy} style={{
              position: "absolute", top: 7, [isUser ? "left" : "right"]: 7,
              background: "rgba(8,12,24,0.9)", border: `1px solid ${copied ? persona.color : "rgba(255,255,255,0.1)"}`,
              borderRadius: 5, color: copied ? persona.color : "rgba(255,255,255,0.35)",
              fontSize: 10, padding: "2px 7px", cursor: "pointer",
              fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.06em",
              opacity: hovered ? 1 : 0, transition: "opacity 0.15s, color 0.15s, border-color 0.15s",
            }}>
              {copied ? "✓" : "⧉"}
            </button>
          )}
        </div>
        {time && !isTyping && (
          <span style={{ fontSize: 10, color: "rgba(255,255,255,0.2)", fontFamily: "'JetBrains Mono',monospace", padding: "0 4px" }}>
            {time}
          </span>
        )}
      </div>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════
// LOADING DOTS
// ═══════════════════════════════════════════════════════════════
function ThinkingDots({ color }) {
  return (
    <div style={{ display: "flex", gap: 10, padding: "8px 0", alignItems: "center" }}>
      <div style={{ width: 30, height: 30, borderRadius: "50%", background: `rgba(${color},0.12)`, border: `1px solid rgba(${color},0.3)`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 13, color: `rgb(${color})`, flexShrink: 0 }}>
      </div>
      <div style={{ display: "flex", gap: 5, padding: "12px 16px", background: "rgba(255,255,255,0.028)", border: "1px solid rgba(255,255,255,0.065)", borderRadius: "2px 12px 12px 12px" }}>
        {[0, 1, 2].map(i => (
          <div key={i} style={{ width: 7, height: 7, borderRadius: "50%", background: `rgb(${color})`, animation: `dot-pulse 1.4s ${i * 0.2}s ease-in-out infinite` }} />
        ))}
      </div>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════
// MAIN APP
// ═══════════════════════════════════════════════════════════════
let _uid = 1000;
const uid = () => ++_uid;

export default function App() {
  const [personaKey, setPersonaKey] = useState("nexus");
  const [chats, setChats] = useState([{ id: 1, title: "New Conversation", persona: "nexus", messages: [], ts: Date.now() }]);
  const [activeId, setActiveId] = useState(1);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [typingText, setTypingText] = useState(null);
  const [sidebarOpen, setSidebarOpen] = useState(true);
  const [renaming, setRenaming] = useState(null);
  const [renameVal, setRenameVal] = useState("");
  const [error, setError] = useState(null);
  const [inputFocused, setInputFocused] = useState(false);

  const endRef = useRef(null);
  const inputRef = useRef(null);
  const timerRef = useRef(null);
  const activeIdRef = useRef(activeId);
  activeIdRef.current = activeId;

  const chat = chats.find(c => c.id === activeId) || chats[0];
  const P = PERSONAS[chat?.persona || personaKey];

  useEffect(() => {
    endRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [chat?.messages?.length, typingText]);

  const updateChat = (id, patch) => setChats(prev => prev.map(c => c.id === id ? { ...c, ...patch } : c));

  const switchChat = id => {
    clearInterval(timerRef.current);
    setTypingText(null);
    setLoading(false);
    setError(null);
    setActiveId(id);
  };

  const newChat = () => {
    const id = uid();
    setChats(prev => [{ id, title: "New Conversation", persona: personaKey, messages: [], ts: Date.now() }, ...prev]);
    switchChat(id);
  };

  const deleteChat = (id, e) => {
    e.stopPropagation();
    const rest = chats.filter(c => c.id !== id);
    if (rest.length === 0) {
      const fresh = { id: uid(), title: "New Conversation", persona: personaKey, messages: [], ts: Date.now() };
      setChats([fresh]);
      setActiveId(fresh.id);
    } else {
      setChats(rest);
      if (activeId === id) switchChat(rest[0].id);
    }
  };

  const changePersona = key => {
    setPersonaKey(key);
    updateChat(activeId, { persona: key });
  };

  const exportChat = () => {
    const md = [`# ${chat.title}`, `**Persona:** ${P.name}`, "", "---", ""]
      .concat(chat.messages.map(m => `**${m.role === "user" ? "You" : P.name}:**\n\n${m.content}`).join("\n\n---\n\n"))
      .join("\n");
    const a = document.createElement("a");
    a.href = URL.createObjectURL(new Blob([md], { type: "text/markdown" }));
    a.download = `${chat.title.replace(/\s+/g, "_")}.md`;
    a.click();
  };

  const send = async () => {
    if (!input.trim() || loading || typingText !== null) return;
    setError(null);
    const content = input.trim();
    setInput("");
    if (inputRef.current) { inputRef.current.style.height = "auto"; }

    const userMsg = { id: uid(), role: "user", content, ts: Date.now() };
    const prevMsgs = chat.messages;
    const newMsgs = [...prevMsgs, userMsg];
    const autoTitle = prevMsgs.length === 0 ? content.slice(0, 44) + (content.length > 44 ? "…" : "") : chat.title;
    const chatPersona = chat.persona;

    updateChat(activeId, { messages: newMsgs, title: autoTitle });
    setLoading(true);

    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: PERSONAS[chatPersona].system,
          messages: newMsgs.map(m => ({ role: m.role, content: m.content })),
        }),
      });

      if (!res.ok) {
        const errData = await res.json().catch(() => ({}));
        throw new Error(errData?.error?.message || `API error ${res.status}`);
      }

      const data = await res.json();
      const fullText = data.content?.[0]?.text || "I couldn't generate a response. Please try again.";

      setLoading(false);
      let i = 0;
      const batchSize = Math.max(3, Math.ceil(fullText.length / 100));
      setTypingText("");

      clearInterval(timerRef.current);
      timerRef.current = setInterval(() => {
        i = Math.min(i + batchSize, fullText.length);
        setTypingText(fullText.slice(0, i));
        if (i >= fullText.length) {
          clearInterval(timerRef.current);
          const aiMsg = { id: uid(), role: "assistant", content: fullText, ts: Date.now() };
          setChats(prev => prev.map(c => c.id === activeIdRef.current ? { ...c, messages: [...c.messages, aiMsg] } : c));
          setTypingText(null);
        }
      }, 18);

    } catch (err) {
      setLoading(false);
      setError(err.message || "Network error. Please try again.");
    }
  };

  const onKeyDown = e => {
    if (e.key === "Enter" && (e.ctrlKey || e.metaKey)) { e.preventDefault(); send(); }
  };

  const allMsgs = chat?.messages || [];

  // ── Render ───────────────────────────────────────────────────
  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Oxanium:wght@300;400;500;600;700;800&family=IBM+Plex+Sans:ital,wght@0,300;0,400;0,500;0,600;1,400&family=JetBrains+Mono:wght@400;500&display=swap');
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
        body { overflow: hidden; background: #060a12; }
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.2); }
        @keyframes fadeUp { from { opacity:0; transform:translateY(10px); } to { opacity:1; transform:translateY(0); } }
        @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }
        @keyframes dot-pulse { 0%,80%,100%{transform:scale(0.7);opacity:0.4} 40%{transform:scale(1);opacity:1} }
        @keyframes spin-slow { to { transform: rotate(360deg); } }
        @keyframes glow-pulse { 0%,100%{opacity:0.6} 50%{opacity:1} }
        .msg-anim { animation: fadeUp 0.26s ease forwards; }
        .cursor { display:inline-block; width:2px; height:0.9em; background:currentColor; margin-left:2px; vertical-align:text-bottom; animation:blink 0.7s step-end infinite; border-radius:1px; }
        textarea { resize:none; outline:none; border:none; }
        button { cursor:pointer; }
        .chat-item:hover { background: rgba(255,255,255,0.04) !important; }
        .chip:hover { transform: translateY(-1px); }
        .persona-btn:hover { filter: brightness(1.25); }
        .send-btn:hover:not(:disabled) { filter: brightness(1.15); transform: scale(1.06); }
        .send-btn:active:not(:disabled) { transform: scale(0.95); }
        .icon-btn:hover { background: rgba(255,255,255,0.08) !important; color: rgba(255,255,255,0.7) !important; }
        .noise-bg::before {
          content: '';
          position: fixed; inset: 0; pointer-events: none; z-index: 0;
          background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
          opacity: 0.4;
        }
      `}</style>

      <div className="noise-bg" style={{ display: "flex", height: "100vh", background: "linear-gradient(160deg,#060a12 0%,#080d1a 50%,#060a12 100%)", color: "#dde4f0", fontFamily: "'IBM Plex Sans',sans-serif", overflow: "hidden", position: "relative", zIndex: 1 }}>

        {/* ── SIDEBAR ─────────────────────────────────────── */}
        <div style={{
          width: sidebarOpen ? 260 : 0, minWidth: sidebarOpen ? 260 : 0,
          overflow: "hidden", transition: "width 0.3s ease, min-width 0.3s ease",
          background: "rgba(4,8,18,0.95)", borderRight: "1px solid rgba(255,255,255,0.055)",
          display: "flex", flexDirection: "column", flexShrink: 0,
        }}>
          {/* Sidebar header */}
          <div style={{ padding: "16px 14px 12px", borderBottom: "1px solid rgba(255,255,255,0.055)", flexShrink: 0 }}>
            <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 14 }}>
              <div style={{ width: 32, height: 32, borderRadius: "50%", background: P.gradient, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 14, boxShadow: `0 0 16px ${P.glow}`, animation: "glow-pulse 3s ease-in-out infinite" }}>
                {P.icon}
              </div>
              <div>
                <div style={{ fontFamily: "'Oxanium',sans-serif", fontWeight: 800, fontSize: 15, color: P.color, letterSpacing: "0.12em" }}>AI CHAT</div>
                <div style={{ fontSize: 10, color: "rgba(255,255,255,0.3)", letterSpacing: "0.06em" }}>POWERED BY CLAUDE</div>
              </div>
            </div>
            <button onClick={newChat} style={{
              width: "100%", padding: "8px 0", background: P.glow, border: `1px solid ${P.color}44`,
              borderRadius: 8, color: P.color, fontFamily: "'Oxanium',sans-serif",
              fontSize: 11, letterSpacing: "0.1em", fontWeight: 700, transition: "all 0.2s",
            }}
            onMouseEnter={e => { e.target.style.background = P.glow.replace("0.15", "0.25"); }}
            onMouseLeave={e => { e.target.style.background = P.glow; }}>
              ＋ NEW CONVERSATION
            </button>
          </div>

          {/* Chat list */}
          <div style={{ flex: 1, overflowY: "auto", padding: "8px 8px" }}>
            {chats.map(c => (
              <div key={c.id} className="chat-item" onClick={() => switchChat(c.id)} style={{
                padding: "9px 10px", borderRadius: 8, marginBottom: 3, cursor: "pointer",
                background: c.id === activeId ? P.glow : "transparent",
                border: `1px solid ${c.id === activeId ? P.color + "44" : "transparent"}`,
                display: "flex", alignItems: "center", gap: 6, transition: "all 0.15s",
              }}>
                <div style={{ width: 6, height: 6, borderRadius: "50%", background: c.id === activeId ? P.color : "rgba(255,255,255,0.2)", flexShrink: 0, transition: "background 0.2s" }} />
                {renaming === c.id ? (
                  <input value={renameVal}
                    onChange={e => setRenameVal(e.target.value)}
                    onBlur={() => { updateChat(c.id, { title: renameVal || c.title }); setRenaming(null); }}
                    onKeyDown={e => { if (e.key === "Enter") { updateChat(c.id, { title: renameVal || c.title }); setRenaming(null); } e.stopPropagation(); }}
                    onClick={e => e.stopPropagation()}
                    autoFocus
                    style={{ flex: 1, background: "transparent", border: "none", outline: "none", color: "#dde4f0", fontSize: 12, fontFamily: "'IBM Plex Sans',sans-serif" }}
                  />
                ) : (
                  <span onDoubleClick={e => { e.stopPropagation(); setRenaming(c.id); setRenameVal(c.title); }}
                    style={{ flex: 1, fontSize: 12, color: c.id === activeId ? P.color : "rgba(255,255,255,0.55)", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>
                    {c.title}
                  </span>
                )}
                <button onClick={e => deleteChat(c.id, e)} style={{ background: "none", border: "none", color: "rgba(255,255,255,0.18)", fontSize: 15, padding: "0 2px", flexShrink: 0, transition: "color 0.15s" }}
                  onMouseEnter={e => { e.currentTarget.style.color = "rgba(255,80,80,0.7)"; }}
                  onMouseLeave={e => { e.currentTarget.style.color = "rgba(255,255,255,0.18)"; }}>
                  ×
                </button>
              </div>
            ))}
          </div>

          {/* Sidebar footer */}
          <div style={{ padding: "10px 14px", borderTop: "1px solid rgba(255,255,255,0.055)", fontSize: 10, color: "rgba(255,255,255,0.2)", fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.08em", textAlign: "center" }}>
            {chats.length} CONVERSATION{chats.length !== 1 ? "S" : ""} · DOUBLE-CLICK TO RENAME
          </div>
        </div>

        {/* ── MAIN PANEL ──────────────────────────────────── */}
        <div style={{ flex: 1, display: "flex", flexDirection: "column", overflow: "hidden", minWidth: 0 }}>

          {/* Top bar */}
          <div style={{ padding: "10px 16px", borderBottom: "1px solid rgba(255,255,255,0.055)", display: "flex", alignItems: "center", gap: 10, background: "rgba(0,0,0,0.25)", flexShrink: 0, flexWrap: "wrap" }}>
            <button className="icon-btn" onClick={() => setSidebarOpen(s => !s)} style={{ background: "rgba(255,255,255,0.05)", border: "1px solid rgba(255,255,255,0.09)", borderRadius: 7, padding: "6px 10px", color: "rgba(255,255,255,0.45)", fontSize: 13, transition: "all 0.2s" }}>
              {sidebarOpen ? "◀" : "▶"}
            </button>

            {/* Persona pills */}
            <div style={{ display: "flex", gap: 5, flexWrap: "wrap" }}>
              {Object.values(PERSONAS).map(p => (
                <button key={p.id} className="persona-btn" onClick={() => changePersona(p.id)} style={{
                  padding: "5px 13px", borderRadius: 20, fontSize: 11,
                  fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.09em", fontWeight: 700,
                  background: chat.persona === p.id ? p.glow : "transparent",
                  border: `1px solid ${chat.persona === p.id ? p.color : "rgba(255,255,255,0.09)"}`,
                  color: chat.persona === p.id ? p.color : "rgba(255,255,255,0.38)",
                  transition: "all 0.2s",
                }}>
                  {p.icon} {p.name}
                </button>
              ))}
            </div>

            <div style={{ flex: 1 }} />

            {/* Action buttons */}
            {allMsgs.length > 0 && (
              <>
                <button className="icon-btn" onClick={exportChat} style={{ background: "rgba(255,255,255,0.04)", border: "1px solid rgba(255,255,255,0.09)", borderRadius: 7, padding: "6px 12px", color: "rgba(255,255,255,0.38)", fontSize: 11, fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.08em", transition: "all 0.2s" }}>
                  ⬇ EXPORT
                </button>
                <button className="icon-btn" onClick={() => updateChat(activeId, { messages: [] })} style={{ background: "rgba(255,255,255,0.04)", border: "1px solid rgba(255,60,60,0.2)", borderRadius: 7, padding: "6px 12px", color: "rgba(255,80,80,0.45)", fontSize: 11, fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.08em", transition: "all 0.2s" }}>
                  ⌫ CLEAR
                </button>
              </>
            )}

            {/* Mode indicator */}
            <div style={{ display: "flex", alignItems: "center", gap: 6, padding: "5px 12px", background: P.glow, border: `1px solid ${P.color}44`, borderRadius: 20 }}>
              <div style={{ width: 6, height: 6, borderRadius: "50%", background: P.color, animation: "glow-pulse 2s ease-in-out infinite" }} />
              <span style={{ fontFamily: "'Oxanium',sans-serif", fontSize: 11, color: P.color, letterSpacing: "0.1em", fontWeight: 700 }}>{P.name}</span>
            </div>
          </div>

          {/* ── MESSAGE AREA ──────────────────────────── */}
          <div style={{ flex: 1, overflowY: "auto", padding: "20px 20px 8px" }}>

            {/* Welcome screen */}
            {allMsgs.length === 0 && !loading && typingText === null && (
              <div style={{ display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "70%", gap: 22, padding: "20px" }}>
                {/* Animated orb */}
                <div style={{ position: "relative", width: 80, height: 80 }}>
                  <div style={{ position: "absolute", inset: 0, borderRadius: "50%", background: P.gradient, animation: "spin-slow 8s linear infinite", opacity: 0.8 }} />
                  <div style={{ position: "absolute", inset: 3, borderRadius: "50%", background: "#060a12", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 30, color: P.color }}>
                    {P.icon}
                  </div>
                  <div style={{ position: "absolute", inset: -8, borderRadius: "50%", background: P.gradient, opacity: 0.12, filter: "blur(12px)" }} />
                </div>

                <div style={{ textAlign: "center" }}>
                  <div style={{ fontFamily: "'Oxanium',sans-serif", fontSize: 28, fontWeight: 800, color: P.color, letterSpacing: "0.15em", marginBottom: 6 }}>{P.name}</div>
                  <div style={{ fontSize: 13, color: "rgba(255,255,255,0.35)", letterSpacing: "0.08em", fontFamily: "'Oxanium',sans-serif" }}>{P.tag.toUpperCase()}</div>
                </div>

                <p style={{ maxWidth: 420, textAlign: "center", fontSize: 14, lineHeight: 1.75, color: "rgba(255,255,255,0.5)", fontStyle: "italic" }}>
                  Ask me anything — deep analysis, creative writing, code, math, or just a conversation. Switch personas anytime for a different thinking style.
                </p>

                {/* Suggestion chips */}
                <div style={{ display: "flex", flexWrap: "wrap", gap: 8, justifyContent: "center", maxWidth: 520 }}>
                  {SUGGESTIONS.map(s => (
                    <button key={s} className="chip" onClick={() => { setInput(s); inputRef.current?.focus(); }} style={{
                      padding: "7px 15px", borderRadius: 20, background: "rgba(255,255,255,0.04)",
                      border: "1px solid rgba(255,255,255,0.1)", color: "rgba(255,255,255,0.5)",
                      fontSize: 12, fontFamily: "'IBM Plex Sans',sans-serif", transition: "all 0.2s",
                    }}
                    onMouseEnter={e => { e.currentTarget.style.borderColor = P.color; e.currentTarget.style.color = P.color; e.currentTarget.style.background = P.glow; }}
                    onMouseLeave={e => { e.currentTarget.style.borderColor = "rgba(255,255,255,0.1)"; e.currentTarget.style.color = "rgba(255,255,255,0.5)"; e.currentTarget.style.background = "rgba(255,255,255,0.04)"; }}>
                      {s}
                    </button>
                  ))}
                </div>
              </div>
            )}

            {/* Messages */}
            {allMsgs.map(msg => (
              <MessageBubble key={msg.id} msg={msg} persona={P} />
            ))}

            {/* AI thinking */}
            {loading && (
              <div className="msg-anim" style={{ display: "flex", gap: 10, padding: "6px 0", alignItems: "center" }}>
                <div style={{ width: 30, height: 30, borderRadius: "50%", background: P.glow, border: `1px solid ${P.color}55`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 13, color: P.color, flexShrink: 0 }}>{P.icon}</div>
                <div style={{ padding: "12px 18px", background: "rgba(255,255,255,0.028)", border: "1px solid rgba(255,255,255,0.065)", borderRadius: "2px 12px 12px 12px", display: "flex", gap: 5, alignItems: "center" }}>
                  {[0, 1, 2].map(i => (
                    <div key={i} style={{ width: 7, height: 7, borderRadius: "50%", background: P.color, animation: `dot-pulse 1.4s ${i * 0.2}s ease-in-out infinite` }} />
                  ))}
                </div>
              </div>
            )}

            {/* Typewriter message */}
            {typingText !== null && (
              <MessageBubble
                msg={{ id: "typing", role: "assistant", content: "", ts: null }}
                persona={P}
                isTyping={true}
                typingText={typingText}
              />
            )}

            {/* Error */}
            {error && (
              <div className="msg-anim" style={{ margin: "10px 0", padding: "12px 16px", background: "rgba(239,68,68,0.09)", border: "1px solid rgba(239,68,68,0.25)", borderRadius: 10, color: "#fca5a5", fontSize: 13, display: "flex", gap: 10, alignItems: "center" }}>
                <span>⚠</span>
                <span>{error}</span>
                <button onClick={() => setError(null)} style={{ marginLeft: "auto", background: "none", border: "none", color: "rgba(252,165,165,0.5)", fontSize: 16 }}>×</button>
              </div>
            )}

            <div ref={endRef} style={{ height: 8 }} />
          </div>

          {/* ── INPUT AREA ────────────────────────────── */}
          <div style={{ padding: "12px 18px 16px", borderTop: "1px solid rgba(255,255,255,0.055)", background: "rgba(0,0,0,0.2)", flexShrink: 0 }}>
            <div style={{
              display: "flex", gap: 10, alignItems: "flex-end",
              background: "rgba(255,255,255,0.03)",
              border: `1px solid ${inputFocused ? P.color + "66" : "rgba(255,255,255,0.09)"}`,
              borderRadius: 14, padding: "10px 12px",
              boxShadow: inputFocused ? `0 0 24px ${P.glow}` : "none",
              transition: "border-color 0.2s, box-shadow 0.2s",
            }}>
              <textarea
                ref={inputRef}
                value={input}
                onChange={e => setInput(e.target.value)}
                onKeyDown={onKeyDown}
                onFocus={() => setInputFocused(true)}
                onBlur={() => setInputFocused(false)}
                onInput={e => { e.target.style.height = "auto"; e.target.style.height = Math.min(e.target.scrollHeight, 140) + "px"; }}
                placeholder={`Message ${P.name}… (Ctrl+Enter to send)`}
                disabled={loading || typingText !== null}
                rows={1}
                style={{
                  flex: 1, background: "transparent", color: "#f0f4ff", fontSize: 14.5,
                  fontFamily: "'IBM Plex Sans',sans-serif", lineHeight: 1.65,
                  maxHeight: 140, minHeight: 26, overflow: "auto",
                  opacity: loading || typingText !== null ? 0.5 : 1,
                }}
              />
              <button className="send-btn" onClick={send}
                disabled={!input.trim() || loading || typingText !== null}
                style={{
                  width: 38, height: 38, borderRadius: "50%", flexShrink: 0,
                  background: input.trim() && !loading && typingText === null ? P.gradient : "rgba(255,255,255,0.07)",
                  border: "none",
                  color: input.trim() && !loading && typingText === null ? "#000" : "rgba(255,255,255,0.2)",
                  fontSize: 17, display: "flex", alignItems: "center", justifyContent: "center",
                  transition: "all 0.2s",
                  boxShadow: input.trim() && !loading && typingText === null ? `0 0 18px ${P.glow}` : "none",
                }}>
                ↑
              </button>
            </div>

            {/* Status bar */}
            <div style={{ display: "flex", justifyContent: "space-between", marginTop: 7, padding: "0 4px" }}>
              <span style={{ fontSize: 10, color: "rgba(255,255,255,0.2)", fontFamily: "'JetBrains Mono',monospace" }}>
                {input.length > 0 ? `${input.length} chars` : "Ctrl+Enter · Send"}
              </span>
              <span style={{ fontSize: 10, color: P.color + "99", fontFamily: "'Oxanium',sans-serif", letterSpacing: "0.08em" }}>
                {P.name} · {allMsgs.filter(m => m.role === "assistant").length} responses
              </span>
            </div>
          </div>
        </div>
      </div>
    </>
  );
}
