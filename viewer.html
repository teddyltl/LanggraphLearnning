<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>LangGraph 单线知识系统 · 交互侧栏</title>
<style>
  :root{--bg:#fff;--fg:#111;--muted:#666;--accent:#4b70ff;}
  @media (prefers-color-scheme: dark){
    :root{--bg:#0d1117;--fg:#e6edf3;--muted:#9aa5b1;--accent:#a3b3ff;}
    body{background:#0d1117;color:#e6edf3}
  }
  *{box-sizing:border-box}
  body{margin:0;font:14px/1.6 system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial}
  .wrap{display:grid;grid-template-columns: minmax(320px,1fr) minmax(320px,420px);
        gap:16px;min-height:100vh;padding:16px}
  .panel{border:1px solid #e5e7eb;border-radius:10px;padding:12px;background:var(--bg)}
  .title{margin:0 0 8px;font-weight:700}
  #chart svg{width:100%;height:auto}
  #side{position:sticky;top:12px;max-height:calc(100vh - 24px);overflow:auto}
  .hint{color:var(--muted);margin:4px 0 12px}
  ol{margin:0;padding-left:18px}
  li+li{margin-top:6px}
  .concept-title{color:var(--accent);font-weight:600;margin-top:12px;margin-bottom:6px;border-left:3px solid var(--accent);padding-left:8px}
  .knowledge-sublist{margin-top:4px;padding-left:16px;list-style-type:disc}
  .badge{display:inline-block;padding:.1em .45em;border-radius:6px;background:#eef2ff;color:#3730a3;font-size:12px;margin-left:.25rem}
  .toolbar{display:flex;gap:8px;margin-bottom:8px;align-items:center}
  .toolbar input[type="text"]{flex:1;padding:6px 8px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg)}
  .toolbar textarea{flex:1;padding:6px 8px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg);min-height:120px;resize:vertical}
  .btn{padding:6px 10px;border:1px solid #d1d5db;border-radius:8px;background:transparent;color:var(--fg);cursor:pointer}
  .btn:hover{border-color:var(--accent)}
  /* 小屏堆叠 */
  @media (max-width: 860px){ .wrap{grid-template-columns: 1fr} #side{position:static;max-height:none} }
</style>
</head>
<body>
<div class="wrap">
  <section class="panel">
    <h2 class="title">主流程</h2>
    <div id="chart" class="mermaid"></div>
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

<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs";
  mermaid.initialize({
    startOnLoad: false,
    securityLevel: "loose",
    theme: (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "default"),
    flowchart: { curve: "linear", htmlLabels: true, rankSpacing: 60, nodeSpacing: 40 }
  });

  // 1) 固定主流程 Mermaid（只画 L0..L11）
  const FLOW = `
flowchart TD
  L0["[0] 入口层：输入与上下文 (config)"]
  L1["[1] 状态层：State & Reducer (MessagesState / add_messages)"]
  L2["[2] 决策层：模型调用与工具调用循环（预构建 ReAct 或自建）"]
  L3["[3] 控制层：顺序/分支/循环 + 条件边/Router"]
  L4["[4] 并行层：Send API 做 Map-Reduce/动态扇出"]
  L5["[5] 交接层：Command API 跳转 + 状态更新（多代理 Handoff）"]
  L6["[6] 短期记忆：Checkpointer + thread_id（可中断/可回溯）"]
  L7["[7] HITL：静态/动态中断 + 人审恢复"]
  L8["[8] 时间旅行：从 checkpoint 回放/分叉"]
  L9["[9] 长期记忆：Store（命名空间/键）+ 语义检索"]
  L10["[10] 流式体验：values/updates/messages/custom/debug"]
  L11["[11] 可视化与部署：Studio/CLI/Platform + 监控与鉴权"]
  L0 --> L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> L7 --> L8 --> L9 --> L10 --> L11
  classDef layer fill:#eef2ff,stroke:#93c5fd,color:#1e3a8a;
  class L0,L1,L2,L3,L4,L5,L6,L7,L8,L9,L10,L11 layer;
`;

  // 2) 知识点数据结构：按层级分组，每层下按概念分类
  const KNOWLEDGE = {
    L0: {},
    L1: {},
    L2: {},
    L3: {},
    L4: {},
    L5: {},
    L6: {},
    L7: {},
    L8: {},
    L9: {},
    L10: {},
    L11: {}
  };

  // 3) 渲染主流程
  const chartEl = document.getElementById("chart");
  chartEl.textContent = FLOW;
  await mermaid.run({ nodes: [chartEl] });
  const svg = chartEl.querySelector("svg");

  // 4) 侧栏：显示与筛选
  const listEl = document.getElementById("kp-list");
  const titleEl = document.getElementById("side-title");
  const filterEl = document.getElementById("filter");
  const pinBtn = document.getElementById("pin");
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
      li.innerHTML = `<span style="color:var(--muted)">（该层暂无内容，可通过下方解析功能导入）</span>`;
      listEl.appendChild(li);
      return;
    }
    
    concepts.forEach(conceptId => {
      const conceptData = layerData[conceptId];
      const conceptTitle = conceptData.title || conceptId;
      const knowledgePoints = conceptData.points || [];
      
      // 过滤知识点
      const filteredPoints = knowledgePoints.filter(s => 
        s.toLowerCase().includes(q) || conceptTitle.toLowerCase().includes(q)
      );
      
      if (q && filteredPoints.length === 0) return; // 如果有搜索词但没匹配项，跳过这个概念
      
      // 创建概念标题
      const conceptLi = document.createElement("li");
      conceptLi.innerHTML = conceptTitle;
      conceptLi.className = "concept-title";
      conceptLi.style.listStyle = "none";
      listEl.appendChild(conceptLi);
      
      // 创建知识点子列表
      const subList = document.createElement("ul");
      subList.className = "knowledge-sublist";
      
      const pointsToShow = q ? filteredPoints : knowledgePoints;
      pointsToShow.forEach(point => {
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
  function updatePinUI(){
    pinBtn.textContent = pinned ? "📌 已置顶（点击取消）" : "📌 置顶/取消";
  }
  pinBtn.addEventListener("click", ()=>{ pinned = !pinned; updatePinUI(); });

  // 5) 绑定事件（事件委托到 svg，兼容不同 Mermaid DOM 结构）
  function extractLayerIdFrom(el){
    if(!el) return null;
    const g = el.closest ? el.closest('g') : null;
    if(!g) return null;
    const className = (g.className && (g.className.baseVal || g.className)) || "";
    if(!/\bnode\b/.test(className)) return null; // 只响应节点，不响应边
    const gid = g.id || "";
    // 兼容 id：flowchart-L0-xxx / flowchart-L10 / ... 或其它包含 -L0- 的形式
    const m = gid.match(/flowchart-(L\d+)(?:-|$)/) || gid.match(/-(L\d+)-/);
    return m && m[1] ? m[1] : null;
  }

  svg.addEventListener("pointerover", (e)=>{
    if(pinned) return;
    const lid = extractLayerIdFrom(e.target);
    if(lid && lid !== currentLayer) renderList(lid);
  });
  svg.addEventListener("click", (e)=>{
    const lid = extractLayerIdFrom(e.target);
    if(lid){
      renderList(lid);
      pinned = true;
      updatePinUI();
    }
  });
  updatePinUI();

  // 6) 可选：解析"知识点汇总合并图"→ 侧栏数据（分层解析，避免跨层混入）
  const rawInput = document.getElementById("raw");
  document.getElementById("parseBtn").addEventListener("click", ()=>{
    const raw = rawInput.value || "";

    // 重置结构
    Object.keys(KNOWLEDGE).forEach(layerId => { KNOWLEDGE[layerId] = {}; });

    // 全局 K 文本表（先收集，后引用）
    const knowledgeMap = {};
    raw.replace(/(K_[A-Za-z0-9_]+)\s*\[\s*"([^"]+)"\s*\]/g, (m, id, label) => {
      knowledgeMap[id] = label;
      return m;
    });

    // 层内概念别名与标题表，概念ID按层隔离：形如 "L3#C1"、"L1#H2"
    const conceptAliasWithinLayer = {}; // { Lx: { C1: 'Lx#C1', ... } }
    const conceptLabelMap = {};        // { 'Lx#C1': '概念：xxx', ... }

    function ensureAlias(layer){ if(!conceptAliasWithinLayer[layer]) conceptAliasWithinLayer[layer] = {}; }
    function ensureConceptEntry(layer, compId, title){
      if(!KNOWLEDGE[layer][compId]){
        KNOWLEDGE[layer][compId] = { title: title || compId, points: [] };
      } else if(title && !KNOWLEDGE[layer][compId].title){
        KNOWLEDGE[layer][compId].title = title;
      }
    }
    function pushPoint(layer, compId, kId){
      const text = knowledgeMap[kId] || kId;
      ensureConceptEntry(layer, compId, conceptLabelMap[compId]);
      KNOWLEDGE[layer][compId].points.push(text);
    }

    // 行级流式解析：跟随 "Begin: Lx" 段落定位当前层
    const lines = raw.split(/\r?\n/);
    let currentLayer = null;
    
    // 为每个层收集概念映射
    const layerConcepts = {}; // { L0: { C1: '概念：xxx', C2: '概念：yyy' }, ... }
    const layerHeadingConcepts = {}; // { L1: ['概念：状态定义与结构', ...], ... }
    
    // 第一遍：收集所有层内的概念定义
    lines.forEach((line, idx) => {
      const mBegin = line.match(/%%\s*----\s*Begin:\s*(L\d+)_/);
      if(mBegin){
        currentLayer = mBegin[1];
        if(!layerConcepts[currentLayer]) layerConcepts[currentLayer] = {};
        if(!layerHeadingConcepts[currentLayer]) layerHeadingConcepts[currentLayer] = [];
        return;
      }
      
      const mEnd = line.match(/%%\s*----\s*End:\s*(L\d+)_/);
      if(mEnd && currentLayer === mEnd[1]){
        currentLayer = null;
        return;
      }
      
      if(!currentLayer) return;
      
      // 收集概念节点（格式1：C1["概念：xxx"]）
      const mCN = line.match(/(C\d+)\s*\[\s*"([^"]+)"/);
      if(mCN){
        layerConcepts[currentLayer][mCN[1]] = mCN[2];
      }
      
      // 收集概念标题（支持多种格式）
      // 格式2：%% ## 概念：xxx （L1, L3, L4, L6）
      // 格式3：%% ### 概念：xxx （L11）
      // 格式4：概念：xxx （L5）
      // 格式5：概念：C1_xxx （L7）
      const mHeading = line.match(/%%\s*#{1,3}\s*(概念[:：][^\n\r]+)/) || 
                       line.match(/^(概念[:：](?:C\d+_)?[^\n\r]+)$/);
      if(mHeading){
        const conceptTitle = (mHeading[1] || mHeading[0]).trim();
        layerHeadingConcepts[currentLayer].push(conceptTitle);
      }
    });
    
    // 第二遍：处理两种格式
    currentLayer = null;
    let currentHeadingConcept = null;
    let currentHeadingIndex = -1;
    
    lines.forEach(line => {
      const mBegin = line.match(/%%\s*----\s*Begin:\s*(L\d+)_/);
      if(mBegin){
        currentLayer = mBegin[1];
        currentHeadingConcept = null;
        currentHeadingIndex = -1;
        return;
      }
      
      const mEnd = line.match(/%%\s*----\s*End:\s*(L\d+)_/);
      if(mEnd && currentLayer === mEnd[1]){
        currentLayer = null;
        currentHeadingConcept = null;
        return;
      }
      
      if(!currentLayer) return;
      
      // 检测概念标题（支持多种格式）
      const mHeading = line.match(/%%\s*#{1,3}\s*(概念[:：][^\n\r]+)/) || 
                       line.match(/^(概念[:：](?:C\d+_)?[^\n\r]+)$/);
      if(mHeading){
        currentHeadingIndex++;
        currentHeadingConcept = (mHeading[1] || mHeading[0]).trim();
        return;
      }
      
      // 格式1：处理 K -> C 边
      const mKC = line.match(/(K_[A-Za-z0-9_]+)\s*-->\s*(C\d+)/g);
      if(mKC){
        mKC.forEach(edge => {
          const m = edge.match(/(K_[A-Za-z0-9_]+)\s*-->\s*(C\d+)/);
          if(m){
            const kId = m[1];
            const cId = m[2];
            const conceptTitle = layerConcepts[currentLayer][cId];
            if(conceptTitle){
              const compId = `${currentLayer}#${cId}`;
              ensureConceptEntry(currentLayer, compId, conceptTitle);
              pushPoint(currentLayer, compId, kId);
            }
          }
        });
        return; // 有边关系的行，不再当作格式2处理
      }
      
      // 格式1：处理 C -> K 边（有些地方是反向的）
      const mCK = line.match(/(C\d+)\s*-->\s*(K_[A-Za-z0-9_]+)/g);
      if(mCK){
        mCK.forEach(edge => {
          const m = edge.match(/(C\d+)\s*-->\s*(K_[A-Za-z0-9_]+)/);
          if(m){
            const cId = m[1];
            const kId = m[2];
            const conceptTitle = layerConcepts[currentLayer][cId];
            if(conceptTitle){
              const compId = `${currentLayer}#${cId}`;
              ensureConceptEntry(currentLayer, compId, conceptTitle);
              pushPoint(currentLayer, compId, kId);
            }
          }
        });
        return; // 有边关系的行，不再当作格式2处理
      }
      
      // 格式2：没有边关系，但有概念标题的层，将K节点归入当前概念
      if(currentHeadingConcept && layerHeadingConcepts[currentLayer].length > 0){
        // 支持缩进的K节点（L5格式）和非缩进的K节点
        const mKNode = line.trim().match(/^(K_[A-Za-z0-9_]+)\s*\[/);
        if(mKNode){
          const kId = mKNode[1];
          const compId = `${currentLayer}#H${currentHeadingIndex}`;
          ensureConceptEntry(currentLayer, compId, currentHeadingConcept);
          pushPoint(currentLayer, compId, kId);
        }
      }
    });

    // 去重
    Object.keys(KNOWLEDGE).forEach(layerId => {
      Object.keys(KNOWLEDGE[layerId]).forEach(conceptId => {
        const seen = new Set();
        KNOWLEDGE[layerId][conceptId].points = KNOWLEDGE[layerId][conceptId].points.filter(p => {
          if(seen.has(p)) return false; seen.add(p); return true;
        });
      });
    });

    // 调试信息
    let debugInfo = [];
    Object.keys(KNOWLEDGE).forEach(layerId => {
      const conceptCount = Object.keys(KNOWLEDGE[layerId]).length;
      if(conceptCount > 0) {
        let pointCount = 0;
        Object.keys(KNOWLEDGE[layerId]).forEach(conceptId => {
          pointCount += KNOWLEDGE[layerId][conceptId].points.length;
        });
        debugInfo.push(`${layerId}: ${conceptCount}个概念, ${pointCount}个知识点`);
      }
    });

    // 默认显示当前层或 L0
    renderList(currentLayer || "L0");
    alert(`解析完成！\n\n${debugInfo.join('\n')}\n\n提示：只处理有明确 K->C 或 C->K 边关系的知识点。`);
  });

  // 7) 快速演示功能
  document.getElementById("demoBtn").addEventListener("click", ()=>{
    // 示例数据
    const demoData = `
C1["概念：环境配置与密钥管理"]
K_1_0__K13["[1.0] module_0 / basics：「环境准备 解决了 运行失败与配置缺失（做法：安装依赖+配置密钥）」"]
K_1_0__K24["[1.0] module_0 / basics：「依赖安装 解决了 运行环境缺失（做法：pip 安装四个核心包）」"]
K_1_0__K25["[1.0] module_0 / basics：「安全配置密钥 解决了 明文泄露风险（做法：getpass 设置环境变量）」"]

C2["概念：状态管理与验证"]
K_1_6__K23["[1.6] module_2 / state_schema：「Pydantic 解决了 运行时验证缺失（做法：BaseModel+field_validator）」"]
K_1_6__K24["[1.6] module_2 / state_schema：「mood 校验 解决了 非法枚举值（做法：限定 'happy'/'sad'）」"]

C3["概念：MessagesState预构建状态"]
K_1_2__K6["[1.2] module_1 / chain：「MessagesState 定义 解决了 状态统一与类型安全（做法：messages字段绑定add_messages）」"]
K_1_3__K15["[1.3] module_1 / router：「MessagesState状态 解决了 维护对话历史（做法：StateGraph(MessagesState)）」"]

C4["概念：图构建与编译"]
K_1_1__K15["[1.1] module_1 / simple_graph：「StateGraph 构建器 解决了 组装节点与边（做法：StateGraph(State)作为容器）」"]
K_1_1__K16["[1.1] module_1 / simple_graph：「添加节点 解决了 节点纳入图的方式（做法：builder.add_node注册节点函数）」"]

C5["概念：路由与条件控制"]
K_1_3__K14["[1.3] module_1 / router：「tools_condition路由 解决了 自动决定是否进工具（做法：检测最后AI消息是否含tool_calls）」"]
K_1_3__K16["[1.3] module_1 / router：「条件边连接 解决了 将LLM与工具节点衔接（做法：add_conditional_edges配tools_condition）」"]

C6["概念：Checkpointer存储器类型"]
K_1_5__K11["[1.5] module_1 / agent_memory：「Checkpointer 解决了 执行过程状态易丢失（做法：每步后自动保存graph state）」"]
K_1_5__K13["[1.5] module_1 / agent_memory：「MemorySaver 解决了 开发阶段快速持久化需求（做法：内存key-value保存状态）」"]

C7["概念：基础中断机制"]
K_1_0__K10["[1.0] module_0 / basics：「HITL 人机协作 解决了 全自动失控风险（做法：人工中断与审批）」"]
K_3_7__K10["[3.7] / Pro版本学习_Langgraph初步：「中断与继续 解决了 在关键点暂停再恢复（做法：节点可设计等待确认再前进）」"]

C8["概念：检查点管理"]
K_2_7__K8["[2.7] module_3 / time_travel：「重放 解决了结果可复现（做法：None输入+旧config）」"]
K_2_7__K9["[2.7] module_3 / time_travel：「重放边界 解决了路径一致性（做法：checkpoint前重现后续重执行）」"]

C9["概念：记忆系统基础架构"]
K_1_0__K23["[1.0] module_0 / basics：「Agent 记忆系统 解决了 上下文延续（做法：引入记忆模块）」"]
K_3_2__K1["[3.2] module_5 / memory_store：「分层命名空间设计 解决混存难检索（做法：按(user_id,类型)分层）」"]

C10["概念：基础流式处理"]
K_1_0__K12["[1.0] module_0 / basics：「流式处理 解决了 过程不可见与体验差（做法：实时流式显示过程）」"]
K_1_0__K35["[1.0] module_0 / basics：「stream 流式调用 解决了 等待焦虑与长文本体验（做法：增量输出）」"]

C11["概念：基础学习框架"]
K_1_0__K6["[1.0] module_0 / basics：「独立设计 解决了 框架耦合限制（做法：LangGraph 独立于 LangChain）」"]
K_1_0__K7["[1.0] module_0 / basics：「模块化课程 解决了 学习跨度大难吸收（做法：按模块拆分学习）」"]

C1 --> L0
C2 --> L0
C3 --> L1
C4 --> L3
C5 --> L3
C6 --> L6
C7 --> L7
C8 --> L8
C9 --> L9
C10 --> L10
C11 --> L11

K_1_0__K13 --> C1
K_1_0__K24 --> C1
K_1_0__K25 --> C1
K_1_6__K23 --> C2
K_1_6__K24 --> C2
K_1_2__K6 --> C3
K_1_3__K15 --> C3
K_1_1__K15 --> C4
K_1_1__K16 --> C4
K_1_3__K14 --> C5
K_1_3__K16 --> C5
K_1_5__K11 --> C6
K_1_5__K13 --> C6
K_1_0__K10 --> C7
K_3_7__K10 --> C7
K_2_7__K8 --> C8
K_2_7__K9 --> C8
K_1_0__K23 --> C9
K_3_2__K1 --> C9
K_1_0__K12 --> C10
K_1_0__K35 --> C10
K_1_0__K6 --> C11
K_1_0__K7 --> C11
    `;
    
    rawInput.value = demoData;
    alert("已加载演示数据到文本框，点击『解析』按钮即可查看效果！");
  });
</script>
</body>
</html>
