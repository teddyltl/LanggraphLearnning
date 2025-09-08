<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>LangGraph 单线知识系统 · 交互侧栏（稳健修复版）</title>
<style>
  :root{--bg:#fff;--fg:#111;--muted:#666;--accent:#4b70ff;}
  @media (prefers-color-scheme: dark){
    :root{--bg:#0d1117;--fg:#e6edf3;--muted:#9aa5b1;--accent:#a3b3ff;}
    body{background:#0d1117;color:#e6edf3}
  }
  *{box-sizing:border-box}
  body{margin:0;font:14px/1.6 system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial}
  .wrap{display:grid;grid-template-columns:minmax(320px,1fr) minmax(320px,420px);gap:16px;min-height:100vh;padding:16px}
  .panel{border:1px solid #e5e7eb;border-radius:10px;padding:12px;background:var(--bg)}
  .title{margin:0 0 8px;font-weight:700}
  #chart svg{width:100%;height:auto}
  #side{position:sticky;top:12px;max-height:calc(100vh - 24px);overflow:auto}
  .hint{color:var(--muted);margin:4px 0 12px}
  ol{margin:0;padding-left:18px}
  li+li{margin-top:6px}
  .concept-title{color:var(--accent);font-weight:600;margin-top:12px;margin-bottom:6px;border-left:3px solid var(--accent);padding-left:8px}
  .knowledge-sublist{margin-top:4px;padding-left:16px;list-style-type:disc}
  .toolbar{display:flex;gap:8px;margin-bottom:8px;align-items:center}
  .toolbar input[type="text"]{flex:1;padding:6px 8px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg)}
  .toolbar textarea{flex:1;padding:6px 8px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg);min-height:120px;resize:vertical}
  .btn{padding:6px 10px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg);cursor:pointer}
  .btn:hover{border-color:var(--accent)}
  @media (max-width: 860px){ .wrap{grid-template-columns:1fr} #side{position:static;max-height:none} }
</style>
</head>
<body>
<div class="wrap">
  <section class="panel">
    <h2 class="title">主流程</h2>
    <!-- 重要：去掉 class="mermaid"，我们手动 render，避免自动扫描 & 二次解析 -->
    <div id="chart"></div>
    <p class="hint">提示：将鼠标悬停或点击节点，可在右侧查看该层所有知识点。</p>
    <details>
      <summary>可选：粘贴"知识点汇总_合并.mmd"文件内容，一键解析为侧栏数据</summary>
      <div class="toolbar" style="margin-top:8px">
        <textarea id="raw" rows="6" placeholder="粘贴知识点汇总_合并.mmd的内容后点『解析』，或点击『快速演示』加载示例数据"></textarea>
        <button class="btn" id="parseBtn">解析</button>
        <button class="btn" id="demoBtn">快速演示</button>
      </div>
      <pre style="white-space:pre-wrap;color:var(--muted);margin:0">
格式说明（按概念分类组织）：
C1["概念：环境配置与密钥管理"]
K_1_0__K13["[1.0] module_0 / basics：「环境准备 解决了 运行失败与配置缺失（做法：安装依赖+配置密钥）」"]
C1 --> L0
K_1_0__K13 --> C1
      </pre>
    </details>
  </section>

  <aside id="side" class="panel">
    <h2 id="side-title" class="title">悬停左侧节点查看知识点</h2>
    <div class="toolbar">
      <input id="filter" type="text" placeholder="输入关键字快速筛选（例如：Router / 记忆 / thread_id）"/>
      <button class="btn" id="pin">📌 置顶/取消</button>
    </div>
    <ol id="kp-list"></ol>
  </aside>
</div>

<!-- 不直接写 mermaid 的 <script src=...>，而是用 JS 按需加载并做多源回退 -->
<script>
(async () => {
  const chartEl = document.getElementById("chart");
  const showError = (msg) => {
    chartEl.innerHTML =
      `<pre style="white-space:pre-wrap;line-height:1.4;color:#ef4444">${msg}</pre>`;
  };

  // 多CDN回退加载器：jsDelivr → unpkg → staticfile
  async function ensureMermaidLoaded() {
    if (window.mermaid) return;
    const cdns = [
      "https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js",
      "https://unpkg.com/mermaid@10/dist/mermaid.min.js",
      "https://cdn.staticfile.org/mermaid/10.9.4/mermaid.min.js"
    ];
    let i = 0;
    await new Promise((resolve, reject) => {
      const tryNext = () => {
        if (window.mermaid) return resolve();
        if (i >= cdns.length) return reject(new Error("Mermaid 脚本加载失败（多源均不可达）"));
        const s = document.createElement("script");
        s.src = cdns[i++]; s.defer = true;
        s.onload = () => window.mermaid ? resolve() : tryNext();
        s.onerror = tryNext;
        document.head.appendChild(s);
      };
      tryNext();
    });
  }

  try {
    await ensureMermaidLoaded(); // 解决“脚本没加载导致页面空白”
  } catch (e) {
    showError("无法加载 Mermaid 库，请检查网络或更换 CDN。\n\n" + e.message +
      "\n\n（可将 mermaid.min.js 放到本地并用 <script src=\"./mermaid.min.js\"> 引入）");
    return;
  }

  // 初始化（关闭自动扫描；允许 HTML 标签；暗色主题自适应）
  mermaid.initialize({
    startOnLoad: false, // 阻止自动 run（官方推荐高级集成用 run/render） :contentReference[oaicite:3]{index=3}
    securityLevel: "loose", // 允许点击/标签（官方文档示例） :contentReference[oaicite:4]{index=4}
    theme: matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "default",
    flowchart: { curve: "linear", htmlLabels: true, rankSpacing: 60, nodeSpacing: 40 }
  });

  // 主流程：方括号全部用实体转义，避免被当作“矩形节点语法”的结束符（官方建议用实体编码规避“会破坏语法的字符”） :contentReference[oaicite:5]{index=5}
  const FLOW = `
flowchart TD
  L0["#91;0#93; 入口层：输入与上下文 (config)"]
  L1["#91;1#93; 状态层：State & Reducer (MessagesState / add_messages)"]
  L2["#91;2#93; 决策层：模型调用与工具调用循环（预构建 ReAct 或自建）"]
  L3["#91;3#93; 控制层：顺序/分支/循环 + 条件边/Router"]
  L4["#91;4#93; 并行层：Send API 做 Map-Reduce/动态扇出"]
  L5["#91;5#93; 交接层：Command API 跳转 + 状态更新（多代理 Handoff）"]
  L6["#91;6#93; 短期记忆：Checkpointer + thread_id（可中断/可回溯）"]
  L7["#91;7#93; HITL：静态/动态中断 + 人审恢复"]
  L8["#91;8#93; 时间旅行：从 checkpoint 回放/分叉"]
  L9["#91;9#93; 长期记忆：Store（命名空间/键）+ 语义检索"]
  L10["#91;10#93; 流式体验：values/updates/messages/custom/debug"]
  L11["#91;11#93; 可视化与部署：Studio/CLI/Platform + 监控与鉴权"]
  L0 --> L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> L7 --> L8 --> L9 --> L10 --> L11
  classDef layer fill:#eef2ff,stroke:#93c5fd,color:#1e3a8a;
  class L0,L1,L2,L3,L4,L5,L6,L7,L8,L9,L10,L11 layer;
`;

  // 推荐流程：parse 校验 → render（v10 起 render 为 async，必须 await） :contentReference[oaicite:6]{index=6}
  try {
    await mermaid.parse(FLOW); // 语法不通过会抛错（或在 suppressErrors 时返回 false） :contentReference[oaicite:7]{index=7}
    const { svg, bindFunctions } = await mermaid.render("graph-main", FLOW);
    chartEl.innerHTML = svg;
    bindFunctions?.(chartEl);
  } catch (err) {
    showError("Mermaid 解析/渲染失败：\n\n" + (err && err.message ? err.message : String(err)) +
      "\n\n若报与文本相关的错误，优先检查节点标题是否含未转义的特殊字符。");
    return;
  }

  // ===== 右侧侧栏（简化版显示与筛选） =====
  const KNOWLEDGE = { L0:{}, L1:{}, L2:{}, L3:{}, L4:{}, L5:{}, L6:{}, L7:{}, L8:{}, L9:{}, L10:{}, L11:{} };
  const listEl   = document.getElementById("kp-list");
  const titleEl  = document.getElementById("side-title");
  const filterEl = document.getElementById("filter");
  const pinBtn   = document.getElementById("pin");
  let pinned = false, currentLayer = null;

  function renderList(layerId){
    currentLayer = layerId;
    const text = ({
      L0:"[0] 入口层：输入与上下文 (config)",
      L1:"[1] 状态层：State & Reducer",
      L2:"[2] 决策层：模型/工具循环",
      L3:"[3] 控制层：条件边/Router",
      L4:"[4] 并行层：Send API",
      L5:"[5] 交接层：Command/Handoff",
      L6:"[6] 短期记忆：Checkpointer",
      L7:"[7] HITL 人在环",
      L8:"[8] 时间旅行：回放/分叉",
      L9:"[9] 长期记忆：Store/RAG",
      L10:"[10] 流式体验：streams",
      L11:"[11] 可视化与部署"
    })[layerId] || layerId;
    titleEl.textContent = text;

    const q = filterEl.value.trim().toLowerCase();
    listEl.innerHTML = "";
    const layerData = KNOWLEDGE[layerId] || {};
    const concepts = Object.keys(layerData);

    if (concepts.length === 0) {
      const li = document.createElement("li");
      li.innerHTML = `<span style="color:var(--muted)">（该层暂无内容，可在下方粘贴 .mmd 后点击“解析”导入）</span>`;
      listEl.appendChild(li);
      return;
    }
    concepts.forEach(conceptId => {
      const conceptData = layerData[conceptId];
      const conceptTitle = conceptData.title || conceptId;
      const points = conceptData.points || [];
      const filtered = points.filter(s => s.toLowerCase().includes(q) || conceptTitle.toLowerCase().includes(q));
      if (q && filtered.length === 0) return;

      const conceptLi = document.createElement("li");
      conceptLi.innerHTML = conceptTitle;
      conceptLi.className = "concept-title";
      conceptLi.style.listStyle = "none";
      listEl.appendChild(conceptLi);

      const subList = document.createElement("ul");
      subList.className = "knowledge-sublist";
      (q ? filtered : points).forEach(point => {
        const pointLi = document.createElement("li");
        pointLi.textContent = point;
        pointLi.style.fontSize = "13px";
        pointLi.style.lineHeight = "1.4";
        subList.appendChild(pointLi);
      });
      listEl.appendChild(subList);
    });
  }
  filterEl.addEventListener("input", ()=> currentLayer && renderList(currentLayer));
  function updatePinUI(){ pinBtn.textContent = pinned ? "📌 已置顶（点击取消）" : "📌 置顶/取消"; }
  pinBtn.addEventListener("click", ()=>{ pinned = !pinned; updatePinUI(); });
  updatePinUI();

  // 绑定 hover / click 到 SVG（事件委托）
  const svg = chartEl.querySelector("svg");
  function extractLayerIdFrom(el){
    if(!el) return null;
    const g = el.closest ? el.closest('g') : null;
    if(!g) return null;
    const cls = (g.className && (g.className.baseVal || g.className)) || "";
    if(!/\bnode\b/.test(cls)) return null;
    const gid = g.id || "";
    const m = gid.match(/flowchart-(L\d+)(?:-|$)/) || gid.match(/-(L\d+)-/);
    return m && m[1] ? m[1] : null;
  }
  if (svg) {
    svg.addEventListener("pointerover", (e)=>{ if(!pinned){ const lid = extractLayerIdFrom(e.target); if(lid && lid!==currentLayer) renderList(lid); }});
    svg.addEventListener("click",       (e)=>{ const lid = extractLayerIdFrom(e.target); if(lid){ renderList(lid); pinned = true; updatePinUI(); }});
  }

  // 解析“知识点汇总_合并.mmd”→ 侧栏（只支持 K↔C 边 与 简单标题）
  document.getElementById("parseBtn").addEventListener("click", ()=>{
    const raw = (document.getElementById("raw").value || "");
    Object.keys(KNOWLEDGE).forEach(k=>KNOWLEDGE[k]={});
    const knowledgeMap = {};
    raw.replace(/(K_[A-Za-z0-9_]+)\s*\[\s*"([^"]+)"\s*\]/g,(m,id,label)=> (knowledgeMap[id]=label, m));
    const lines = raw.split(/\r?\n/); let currentLayer = null; const layerConcepts = {}; const layerHeadings = {};
    const ensure = (o,k,v)=> (o[k]??=(v||{}), o[k]);
    lines.forEach(line=>{
      const b=line.match(/%%\s*----\s*Begin:\s*(L\d+)_/); if(b){currentLayer=b[1]; ensure(layerConcepts,currentLayer,{}); ensure(layerHeadings,currentLayer,[]); return;}
      const e=line.match(/%%\s*----\s*End:\s*(L\d+)_/); if(e && currentLayer===e[1]){currentLayer=null; return;}
      if(!currentLayer) return;
      const mC=line.match(/(C\d+)\s*\[\s*"([^"]+)"/); if(mC){ ensure(layerConcepts,currentLayer,{}); layerConcepts[currentLayer][mC[1]]=mC[2]; }
      const mH=line.match(/%%\s*#{1,3}\s*(概念[:：][^\n\r]+)/) || line.match(/^(概念[:：](?:C\d+_)?[^\n\r]+)$/);
      if(mH){ layerHeadings[currentLayer].push((mH[1]||mH[0]).trim()); }
    });

    const conceptTitleMap = {};
    function ensureEntry(layer, compId, title){ KNOWLEDGE[layer][compId] ??= { title, points: [] }; if(title && !KNOWLEDGE[layer][compId].title) KNOWLEDGE[layer][compId].title=title; }
    function pushPoint(layer, compId, kId){ const t=knowledgeMap[kId]||kId; ensureEntry(layer, compId, conceptTitleMap[compId]); KNOWLEDGE[layer][compId].points.push(t); }

    currentLayer=null; let headIdx=-1; let headTitle=null;
    lines.forEach(line=>{
      const b=line.match(/%%\s*----\s*Begin:\s*(L\d+)_/); if(b){currentLayer=b[1]; headIdx=-1; headTitle=null; return;}
      const e=line.match(/%%\s*----\s*End:\s*(L\d+)_/); if(e && currentLayer===e[1]){currentLayer=null; return;}
      if(!currentLayer) return;

      const mH=line.match(/%%\s*#{1,3}\s*(概念[:：][^\n\r]+)/) || line.match(/^(概念[:：](?:C\d+_)?[^\n\r]+)$/);
      if(mH){ headIdx++; headTitle=(mH[1]||mH[0]).trim(); return; }

      const mKC=line.match(/(K_[A-Za-z0-9_]+)\s*-->\s*(C\d+)/g);
      if(mKC){ mKC.forEach(edge=>{ const m=edge.match(/(K_[A-Za-z0-9_]+)\s*-->\s*(C\d+)/); if(!m) return; const [_,kId,cId]=m; const title=(layerConcepts[currentLayer]||{})[cId]; if(title){ const comp=`${currentLayer}#${cId}`; conceptTitleMap[comp]=title; pushPoint(currentLayer,comp,kId);} }); return; }

      const mCK=line.match(/(C\d+)\s*-->\s*(K_[A-Za-z0-9_]+)/g);
      if(mCK){ mCK.forEach(edge=>{ const m=edge.match(/(C\d+)\s*-->\s*(K_[A-Za-z0-9_]+)/); if(!m) return; const [_,cId,kId]=m; const title=(layerConcepts[currentLayer]||{})[cId]; if(title){ const comp=`${currentLayer}#${cId}`; conceptTitleMap[comp]=title; pushPoint(currentLayer,comp,kId);} }); return; }

      if(headTitle && (layerHeadings[currentLayer]||[]).length>0){
        const mK=line.trim().match(/^(K_[A-Za-z0-9_]+)\s*\[/);
        if(mK){ const kId=mK[1]; const comp=`${currentLayer}#H${headIdx}`; conceptTitleMap[comp]=headTitle; pushPoint(currentLayer,comp,kId); }
      }
    });

    // 去重
    Object.keys(KNOWLEDGE).forEach(layer=>{
      Object.values(KNOWLEDGE[layer]).forEach(x=>{
        const seen=new Set(); x.points=x.points.filter(p=>!seen.has(p)&&(seen.add(p),true));
      });
    });

    renderList(currentLayer || "L0");
    alert("解析完成！可在右侧查看该层概念与知识点。");
  });

  // 演示数据
  document.getElementById("demoBtn").addEventListener("click", ()=>{
    document.getElementById("raw").value = `
C1["概念：环境配置与密钥管理"]
K_1_0__K13["[1.0] module_0 / basics：「环境准备 解决了 运行失败与配置缺失（做法：安装依赖+配置密钥）」"]
K_1_0__K24["[1.0] module_0 / basics：「依赖安装 解决了 运行环境缺失（做法：pip 安装四个核心包）」"]
K_1_0__K25["[1.0] module_0 / basics：「安全配置密钥 解决了 明文泄露风险（做法：getpass 设置环境变量）」"]

C2["概念：状态管理与验证"]
K_1_6__K23["[1.6] module_2 / state_schema：「Pydantic 解决了 运行时验证缺失（做法：BaseModel+field_validator）」"]
K_1_6__K24["[1.6] module_2 / state_schema：「mood 校验 解决了 非法枚举值（做法：限定 'happy'/'sad'）」"]

C1 --> L0
C2 --> L1

K_1_0__K13 --> C1
K_1_0__K24 --> C1
K_1_0__K25 --> C1
K_1_6__K23 --> C2
K_1_6__K24 --> C2
    `;
    alert("已加载演示数据到文本框，点击『解析』按钮即可查看效果！");
  });
})();
</script>
</body>
</html>
