<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>LangGraph 单线知识系统 · 交互侧栏（增强解析版）</title>
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
  .debug-info{font-size:12px;color:var(--muted);margin-top:10px;padding:8px;border:1px solid #e5e7eb;border-radius:4px;background:rgba(0,0,0,0.02)}
  @media (max-width: 860px){ .wrap{grid-template-columns:1fr} #side{position:static;max-height:none} }
</style>
</head>
<body>
<div class="wrap">
  <section class="panel">
    <h2 class="title">主流程</h2>
    <div id="chart"></div>
    <p class="hint">提示：将鼠标悬停或点击节点，可在右侧查看该层所有知识点。</p>
    <details>
      <summary>粘贴"知识点汇总_合并_提示词版.mmd"文件内容，一键解析为侧栏数据</summary>
      <div class="toolbar" style="margin-top:8px">
        <textarea id="raw" rows="6" placeholder="粘贴知识点汇总_合并_提示词版.mmd的内容后点『解析』，或点击『快速演示』加载示例数据"></textarea>
        <button class="btn" id="parseBtn">解析</button>
        <button class="btn" id="demoBtn">快速演示</button>
      </div>
      <div class="debug-info" id="debug-info" style="display:none">
        <strong>解析调试信息：</strong>
        <div id="parse-debug"></div>
      </div>
    </details>
  </section>

  <aside id="side" class="panel">
    <h2 id="side-title" class="title">悬停左侧节点查看知识点</h2>
    <div class="toolbar">
      <input id="filter" type="text" placeholder="输入关键字快速筛选（例如：Router / 记忆 / thread_id）"/>
      <button class="btn" id="pin">📌 置顶/取消</button>
      <button class="btn" id="debug-toggle">🔧 调试</button>
    </div>
    <ol id="kp-list"></ol>
  </aside>
</div>

<script>
(async () => {
  const chartEl = document.getElementById("chart");
  const showError = (msg) => {
    chartEl.innerHTML =
      `<pre style="white-space:pre-wrap;line-height:1.4;color:#ef4444">${msg}</pre>`;
  };

  // 多CDN回退加载器
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
    await ensureMermaidLoaded();
  } catch (e) {
    showError("无法加载 Mermaid 库，请检查网络或更换 CDN。\n\n" + e.message);
    return;
  }

  // 初始化Mermaid
  mermaid.initialize({
    startOnLoad: false,
    securityLevel: "loose",
    theme: matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "default",
    flowchart: { curve: "linear", htmlLabels: true, rankSpacing: 60, nodeSpacing: 40 }
  });

  // 主流程图
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

  // 渲染主流程图
  try {
    await mermaid.parse(FLOW);
    const { svg, bindFunctions } = await mermaid.render("graph-main", FLOW);
    chartEl.innerHTML = svg;
    bindFunctions?.(chartEl);
  } catch (err) {
    showError("Mermaid 解析/渲染失败：\n\n" + (err && err.message ? err.message : String(err)));
    return;
  }

  // ===== 侧栏数据结构和逻辑 =====
  const KNOWLEDGE = { L0:{}, L1:{}, L2:{}, L3:{}, L4:{}, L5:{}, L6:{}, L7:{}, L8:{}, L9:{}, L10:{}, L11:{} };
  const listEl   = document.getElementById("kp-list");
  const titleEl  = document.getElementById("side-title");
  const filterEl = document.getElementById("filter");
  const pinBtn   = document.getElementById("pin");
  const debugToggle = document.getElementById("debug-toggle");
  const debugInfo = document.getElementById("debug-info");
  const parseDebug = document.getElementById("parse-debug");
  let pinned = false, currentLayer = null, debugMode = false;

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
      li.innerHTML = `<span style="color:var(--muted)">（该层暂无内容，可在下方粘贴 .mmd 后点击"解析"导入）</span>`;
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

  // 事件处理
  filterEl.addEventListener("input", ()=> currentLayer && renderList(currentLayer));
  function updatePinUI(){ pinBtn.textContent = pinned ? "📌 已置顶（点击取消）" : "📌 置顶/取消"; }
  pinBtn.addEventListener("click", ()=>{ pinned = !pinned; updatePinUI(); });
  debugToggle.addEventListener("click", ()=>{
    debugMode = !debugMode;
    debugInfo.style.display = debugMode ? "block" : "none";
    debugToggle.textContent = debugMode ? "🔧 调试中" : "🔧 调试";
  });
  updatePinUI();

  // 绑定 hover/click 事件
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

  // ===== 增强版解析器 =====
  function parseEnhanced(rawText) {
    const debugLog = [];
    const layers = new Map();
    
    // 清空现有数据
    Object.keys(KNOWLEDGE).forEach(k => KNOWLEDGE[k] = {});

    // 按层分别提取和解析，避免跨层ID冲突
    const layerContents = new Map();
    
    // 首先分割出各层内容
    const layerRegex = /%%\s*----\s*Begin:\s*(L\d+)_[\s\S]*?%%\s*----\s*End:\s*\1_/g;
    let layerMatch;
    while ((layerMatch = layerRegex.exec(rawText)) !== null) {
      const layerId = layerMatch[0].match(/Begin:\s*(L\d+)_/)[1];
      layerContents.set(layerId, layerMatch[0]);
      debugLog.push(`提取到 ${layerId} 层内容`);
    }
    
    // 按层解析每一层
    for (const [layerId, layerContent] of layerContents) {
      debugLog.push(`\n=== 开始解析 ${layerId} 层 ===`);
      
      // 为当前层创建局部知识点映射，避免跨层污染
      const localKnowledgeMap = new Map();
      
      // 提取当前层的知识点（包括KC节点）
      const knowledgeRegex = /(K(?:C?_)?[A-Za-z0-9_]+|KC\d+)\s*\[\s*"([^"]+(?:「[^」]*」[^"]*)?(?:[^"]*「[^」]*」)*[^"]*)"\s*\]/g;
      let kMatch;
      while ((kMatch = knowledgeRegex.exec(layerContent)) !== null) {
        const [fullMatch, kId, content] = kMatch;
        localKnowledgeMap.set(kId, content);
        debugLog.push(`${layerId}: ${kId} -> ${content.substring(0, 50)}...`);
      }
      
      debugLog.push(`${layerId} 层提取到 ${localKnowledgeMap.size} 个知识点`);
      
      // 解析当前层的结构
      const lines = layerContent.split('\n');
      let currentConcept = null;
      let conceptCounter = 0;
      const layerConceptMap = new Map();

      for (let i = 0; i < lines.length; i++) {
        const line = lines[i].trim();
        if (!line) continue;

        // Mermaid概念节点
        const mermaidConceptMatch = line.match(/^(C\d+)\s*\[\s*"([^"]+)"\s*\]/);
        if (mermaidConceptMatch) {
          const [, cId, title] = mermaidConceptMatch;
          currentConcept = `${layerId}#${cId}`;
          KNOWLEDGE[layerId][currentConcept] = { title, points: [] };
          layerConceptMap.set(cId, currentConcept);
          debugLog.push(`${layerId}: Mermaid概念 ${cId} -> ${title}`);
          continue;
        }

        // 文本概念标题
        const textConceptMatch = line.match(/^概念[:：](.+)$/);
        if (textConceptMatch) {
          conceptCounter++;
          const title = textConceptMatch[1].trim();
          currentConcept = `${layerId}#T${conceptCounter}`;
          KNOWLEDGE[layerId][currentConcept] = { title: `概念：${title}`, points: [] };
          debugLog.push(`${layerId}: 文本概念 -> 概念：${title}`);
          continue;
        }

        // L2层等：按注释识别概念分组
        if (layerId === 'L2') {
          const conceptGroupMatch = line.match(/%%\s*概念(\d+)[:：]/);
          if (conceptGroupMatch) {
            const conceptNum = conceptGroupMatch[1];
            const cId = `C${conceptNum}`;
            currentConcept = layerConceptMap.get(cId);
            if (currentConcept) {
              debugLog.push(`${layerId}: 切换到概念组 ${cId}`);
            }
            continue;
          }
        }

        // L10层等：K节点到C节点的连接关系 K_C1__ALL --> C1（必须在一般知识点处理前）
        if (layerId === 'L10') {
          const kcConnectionMatch = line.match(/^(K_C\d+__[A-Za-z0-9_]+)\s*-->\s*(C\d+)/);
          if (kcConnectionMatch) {
            const [, kId, cId] = kcConnectionMatch;
            const content = localKnowledgeMap.get(kId);
            const targetConcept = layerConceptMap.get(cId);
            if (content && targetConcept) {
              KNOWLEDGE[layerId][targetConcept].points.push(content);
              debugLog.push(`${layerId}: K节点连接关系 ${kId} --> ${cId} 添加知识点到 ${targetConcept}`);
            }
            continue;
          }
        }

        // 知识点节点
        const knowledgeMatch = line.match(/^(K(?:C?_)?[A-Za-z0-9_]+)\s*\[/);
        if (knowledgeMatch) {
          const kId = knowledgeMatch[1];
          const content = localKnowledgeMap.get(kId); // 使用当前层的知识点映射
          if (content) {
            // L10层：跳过已在连接关系中处理的K_C*__ALL节点
            if (layerId === 'L10' && kId.match(/^K_C\d+__/)) {
              continue;
            }
            
            // 对于L2层，根据K节点的前缀推断概念归属
            let targetConcept = currentConcept;
            if (layerId === 'L2') {
              const kPrefixMatch = kId.match(/K_(\d+)__/);
              if (kPrefixMatch) {
                const conceptNum = kPrefixMatch[1];
                const cId = `C${conceptNum}`;
                targetConcept = layerConceptMap.get(cId);
              }
            }

            if (targetConcept && KNOWLEDGE[layerId][targetConcept]) {
              KNOWLEDGE[layerId][targetConcept].points.push(content);
              debugLog.push(`${layerId}: 知识点 ${kId} 添加到 ${targetConcept}`);
            } else if (currentConcept) {
              KNOWLEDGE[layerId][currentConcept].points.push(content);
              debugLog.push(`${layerId}: 知识点 ${kId} 添加到当前概念 ${currentConcept}`);
            } else {
              // 默认概念
              const defaultConcept = `${layerId}#DEFAULT`;
              if (!KNOWLEDGE[layerId][defaultConcept]) {
                KNOWLEDGE[layerId][defaultConcept] = { title: "其他知识点", points: [] };
              }
              KNOWLEDGE[layerId][defaultConcept].points.push(content);
              debugLog.push(`${layerId}: 知识点 ${kId} 添加到默认概念`);
            }
          }
          continue;
        }

        // L8层特殊处理：连接关系语法 C1 --> KC1["内容"]
        if (layerId === 'L8') {
          const connectionMatch = line.match(/^(C\d+)\s*-->\s*(KC\d+)\s*\[/);
          if (connectionMatch) {
            const [, cId, kcId] = connectionMatch;
            const content = localKnowledgeMap.get(kcId); // 使用当前层的知识点映射
            const targetConcept = `${layerId}#${cId}`;
            if (content && KNOWLEDGE[layerId][targetConcept]) {
              KNOWLEDGE[layerId][targetConcept].points.push(content);
              debugLog.push(`${layerId}: 连接关系 ${cId} --> ${kcId} 添加知识点到 ${targetConcept}`);
            }
            continue;
          }
        }
      }
    }

    // 去重处理
    Object.keys(KNOWLEDGE).forEach(layer => {
      Object.values(KNOWLEDGE[layer]).forEach(conceptData => {
        if (conceptData.points) {
          const seen = new Set();
          conceptData.points = conceptData.points.filter(point => {
            if (seen.has(point)) return false;
            seen.add(point);
            return true;
          });
        }
      });
    });

    // 统计信息
    let totalConcepts = 0, totalPoints = 0;
    Object.keys(KNOWLEDGE).forEach(layer => {
      const layerConcepts = Object.keys(KNOWLEDGE[layer]).length;
      const layerPoints = Object.values(KNOWLEDGE[layer]).reduce((sum, c) => sum + (c.points ? c.points.length : 0), 0);
      totalConcepts += layerConcepts;
      totalPoints += layerPoints;
      if (layerConcepts > 0) {
        debugLog.push(`${layer}: ${layerConcepts} 个概念, ${layerPoints} 个知识点`);
      }
    });
    debugLog.push(`总计: ${totalConcepts} 个概念, ${totalPoints} 个知识点`);

    return debugLog;
  }

  // 解析按钮事件
  document.getElementById("parseBtn").addEventListener("click", () => {
    const rawText = document.getElementById("raw").value || "";
    if (!rawText.trim()) {
      alert("请先粘贴文件内容");
      return;
    }

    const debugLog = parseEnhanced(rawText);
    parseDebug.innerHTML = debugLog.join('<br>');

    renderList(currentLayer || "L0");
    alert(`解析完成！共解析 ${debugLog[debugLog.length - 1]}。可在右侧查看各层知识点。`);
  });

  // 演示数据
  document.getElementById("demoBtn").addEventListener("click", () => {
    document.getElementById("raw").value = `%% ---- Begin: L0_入口层_概念分类.mmd ----
C1["概念：环境配置与密钥管理"]
K_C1__ALL["[合并] 环境/密钥/鉴权/模型参数：「统一依赖安装与环境变量(.env/.gitignore)管理密钥，设置 OPENAI_API_KEY / TAVILY_API_KEY 等；按需选择模型与温度——解决 运行失败 与 凭据泄露」"]

C2["概念：状态管理与验证"]  
K_C2__ALL["[合并] 输入校验与清洗：「以 TypedDict / Pydantic 定义 state；在节点入参处执行 运行时校验，覆盖枚举/规则/默认值/类型转换；捕获 ValidationError 阻断无效输入传播——解决 不可信或脏输入」"]
%% ---- End: L0_入口层_概念分类.mmd ----

%% ---- Begin: L8_时间旅行_概念分类.mmd ----
C1["概念：检查点管理"]
C2["概念：重放机制"]
C3["概念：分叉控制"]
C4["概念：执行回滚"]

C1 --> KC1["[合并] 检查点管理：「创建/恢复/一致性——按 thread_id 记录并加载历史 state；以服务端 checkpoint 为准对齐中断/取消后的状态；失败可回滚至最近 checkpoint」"]
C2 --> KC2["[合并] 重放机制：「重放语义——input=None + 原config 在所选 checkpoint 继续；checkpoint 前严格复现、之后重执行并产生新分支；有 input 视为从该点开启新分支；终态重放：next 为空即结束；支持云端：checkpoint_id + None」"]
%% ---- End: L8_时间旅行_概念分类.mmd ----`;
    alert("已加载演示数据到文本框，点击『解析』按钮即可查看效果！");
  });
})();
</script>
</body>
</html>
