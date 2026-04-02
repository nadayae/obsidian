---
tags:
  - dashboard
cssclasses:
  - second-brain
---

```dataviewjs
const wrap = dv.el("div", "", {attr: {style: "display:flex; align-items:center; gap:16px; flex-wrap:wrap; margin-bottom:8px;"}});

// 프로필 카드
wrap.innerHTML += `
<div style="display:inline-flex; align-items:center; gap:12px; background:rgba(var(--ctp-surface0), 0.6); border:1px solid rgba(var(--ctp-surface1), 0.8); border-radius:14px; padding:10px 16px; flex-shrink:0;">
  <div style="width:42px; height:42px; border-radius:50%; background:linear-gradient(135deg, rgb(var(--ctp-rosewater)), rgb(var(--ctp-pink))); display:flex; align-items:center; justify-content:center; font-size:1.1rem; font-weight:800; color:rgb(var(--ctp-base)); box-shadow:0 2px 8px rgba(var(--ctp-rosewater),0.35); flex-shrink:0;">N</div>
  <div>
    <div style="font-weight:700; font-size:0.9rem; color:rgb(var(--ctp-text)); letter-spacing:-0.02em;">nadayae</div>
    <div style="font-size:0.62rem; color:rgb(var(--ctp-subtext0)); margin-top:1px;">나의 세컨드 브레인 ✨</div>
  </div>
</div>`;

// 버튼 목음
const buttons = [
  {icon:"🚀", label:"프로젝트", cmd:"quickadd:choice:New Project"},
  {icon:"✅", label:"할일", cmd:"quickadd:choice:New Task"},
  {icon:"🎯", label:"목표", cmd:"quickadd:choice:New Goal"},
  {icon:"📝", label:"노트", cmd:"quickadd:choice:New Resource"},
  {icon:"✍️", label:"캡처", cmd:"quickadd:choice:New Capture"},
];

const btnWrap = wrap.createEl("div", {attr: {style: "display:flex; gap:6px; flex-wrap:wrap; align-items:center;"}});
for (const b of buttons) {
  const btn = btnWrap.createEl("button", {attr: {style: `
    display:inline-flex; align-items:center; gap:5px;
    background:rgba(var(--ctp-surface0), 0.7);
    border:1px solid rgba(var(--ctp-surface2), 0.6);
    border-radius:8px; padding:5px 11px;
    font-size:0.72rem; font-weight:600;
    color:rgb(var(--ctp-text)); cursor:pointer;
    transition:all 0.15s;
  `}});
  btn.innerHTML = `<span>${b.icon}</span><span>${b.label}</span>`;
  btn.addEventListener("mouseenter", () => {
    btn.style.background = `rgba(var(--ctp-rosewater), 0.2)`;
    btn.style.borderColor = `rgba(var(--ctp-rosewater), 0.6)`;
  });
  btn.addEventListener("mouseleave", () => {
    btn.style.background = `rgba(var(--ctp-surface0), 0.7)`;
    btn.style.borderColor = `rgba(var(--ctp-surface2), 0.6)`;
  });
  btn.addEventListener("click", () => app.commands.executeCommandById(b.cmd));
}
```

```dataviewjs
const today = dv.date("today");
const startOfMonth = today.startOf("month");
const endOfMonth = today.endOf("month");
const startOfWeek = today.startOf("week");
const endOfWeek = today.endOf("week");

const calStart = startOfMonth.startOf("week");
const calEnd = endOfMonth.endOf("week");
let monthDays = [];
let curr = calStart;
while (curr <= calEnd) { monthDays.push(curr); curr = curr.plus({ days: 1 }); }
const weekDays = Array.from({length: 7}, (_, i) => startOfWeek.plus({days: i}));
const dayNames = ["월","화","수","목","금","토","일"];

const allTasks = dv.pages('"05. Tasks"');
const allResources = dv.pages('"06. Resources"');

// [변경] 이모지를 세련된 모노톤 기호로 교체
const taskTabs = [
  { id: "thisweek", icon: "⊙", label: "이번 주", filter: t => true, isDefault: true },
  { id: "total", icon: "▤", label: "전체(월간)", isMonth: true },
  { id: "done", icon: "▣", label: "이번 주 완료", filter: t => t.완료여부 === true },
  { id: "recent", icon: "▧", label: "최근 자료", isResource: true }
];

const uid = "mega-cal-" + Math.random().toString(36).substring(2, 7);

let html = `<div id="${uid}-container">`;
html += `<details open style="background: transparent !important; border: none !important; padding: 0;">`;
html += `<summary style="font-weight: 800; cursor: pointer; margin-bottom: 16px; color: rgb(var(--ctp-rosewater)); font-size: 1.1rem; list-style: none; opacity: 0.8;">캘린더</summary>`;

html += `<div class="sb-tabs" style="margin-bottom: 16px; display: flex; gap: 14px; border-bottom: 1px solid rgba(var(--ctp-rosewater), 0.2); padding-bottom: 8px;">`;
taskTabs.forEach((tab) => {
    const isActive = tab.isDefault;
    const color = isActive ? "rgb(var(--ctp-rosewater))" : "var(--text-muted)";
    const border = isActive ? `2px solid rgb(var(--ctp-rosewater))` : "none";
    html += `<div class="tab-btn" data-target="${tab.id}" style="font-size: 0.75rem; cursor: pointer; font-weight: 700; color: ${color}; border-bottom: ${border}; padding-bottom: 8px; letter-spacing: 0.02em;">${tab.icon} ${tab.label}</div>`;
});
html += `</div>`;

taskTabs.forEach((tab) => {
    const display = tab.isDefault ? "block" : "none";
    html += `<div class="tab-panel" id="${uid}-${tab.id}" style="display: ${display};">`;
    
    if (tab.isMonth) {
        html += `<div style="display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px;">`;
        dayNames.forEach(n => html += `<div style="text-align:center; font-size:0.6rem; color:var(--text-muted); opacity: 0.7;">${n}</div>`);
        monthDays.forEach(d => {
            const isToday = d.hasSame(today, "day");
            const opacity = d.month === today.month ? "1" : "0.2";
            const bg = isToday ? "background: rgba(var(--ctp-rosewater), 0.15);" : "background: rgba(var(--ctp-surface0), 0.2);";
            const dayTasks = allTasks.filter(t => t.날짜 && dv.date(t.날짜).hasSame(d, "day"));
            html += `<div style="min-height: 80px; max-height: 120px; overflow-y: auto; padding: 4px; ${bg} border-radius: 6px; opacity: ${opacity}; border: 1px solid rgba(var(--ctp-rosewater), ${isToday?0.5:0.05}); scrollbar-width: none;">`;
            html += `<div style="font-size: 0.6rem; color: ${isToday?'rgb(var(--ctp-rosewater))':'gray'}; font-weight: ${isToday?800:400}; margin-bottom: 4px;">${d.day}</div>`;
            dayTasks.forEach(t => {
                html += `<div style="font-size: 0.5rem; border-left: 2px solid rgb(var(--ctp-rosewater)); padding: 2px 4px; margin-bottom: 2px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; background: rgba(var(--ctp-surface1), 0.4); border-radius: 2px;">${t.제목 || t.file.name}</div>`;
            });
            html += `</div>`;
        });
        html += `</div>`;
    } 
    else if (tab.isResource) {
        const recentRes = allResources.sort(r => r.file.mtime, "desc").slice(0, 7);
        html += `<div style="display: grid; grid-template-columns: repeat(7, 1fr); gap: 8px;">`;
        recentRes.forEach(r => {
            html += `<div style="padding: 10px; border: 1px solid rgba(var(--ctp-rosewater), 0.3); border-radius: 12px; font-size: 0.65rem; min-height: 95px; background: rgba(var(--ctp-surface0), 0.5);">`;
            html += `<div style="color: rgb(var(--ctp-rosewater)); font-weight: 700; margin-bottom: 6px; opacity: 0.8;">${dv.date(r.file.mtime).toFormat("MM/dd")}</div>`;
            html += `<a class="internal-link" href="${r.file.path}" style="color: var(--text-normal); text-decoration: none;">${r.file.name}</a></div>`;
        });
        html += `</div>`;
    } 
    else {
        html += `<div style="display: grid; grid-template-columns: repeat(7, 1fr); gap: 8px;">`;
        dayNames.forEach((n, idx) => {
            const isToday = weekDays[idx].hasSame(today, "day");
            html += `<div style="text-align: center; font-size: 0.75rem; color: ${isToday?'rgb(var(--ctp-rosewater))':'var(--text-muted)'}; font-weight: ${isToday?800:400}; margin-bottom: 6px;">${weekDays[idx].month}/${weekDays[idx].day} ${n}</div>`;
        });
        const filteredTasks = allTasks.filter(t => {
            const d = dv.date(t.날짜);
            return d && d >= startOfWeek && d <= endOfWeek && tab.filter(t);
        });
        weekDays.forEach(d => {
            const dayTasks = filteredTasks.filter(t => dv.date(t.날짜).hasSame(d, "day"));
            const isToday = d.hasSame(today, "day");
            const bg = isToday ? "background: rgba(var(--ctp-rosewater), 0.12);" : "background: rgba(var(--ctp-surface0), 0.3);";
            html += `<div style="min-height: 120px; padding: 8px; ${bg} border: 1px solid rgba(var(--ctp-rosewater), ${isToday?0.6:0.15}); border-radius: 12px;">`;
            dayTasks.forEach(t => {
                const isDone = t.완료여부 === true;
                const isDoneTab = tab.id === "done";
                const textDecoration = (isDone && !isDoneTab) ? "line-through" : "none"; 
                const opacityValue = isDoneTab ? "1" : (isDone ? "0.5" : "1");
                const filterValue = (isDone && !isDoneTab) ? "grayscale(0.8)" : "none";
                html += `<div style="font-size: 0.65rem; padding: 4px 6px; margin-bottom: 5px; border-left: 3px solid rgb(var(--ctp-rosewater)); background: rgba(var(--ctp-surface1), 0.6); border-radius: 4px; text-decoration: ${textDecoration}; opacity: ${opacityValue}; filter: ${filterValue};">`;
                html += `<a class="internal-link" href="${t.file.path}" style="color: var(--text-normal) !important; text-decoration: inherit;">${t.제목 || t.file.name}</a></div>`;
            });
            html += `</div>`;
        });
        html += `</div>`;
    }
    html += `</div>`;
});
html += `</details></div>`;

dv.el("div", html);

setTimeout(() => {
    const container = document.getElementById(`${uid}-container`);
    if (!container) return;
    const buttons = container.querySelectorAll('.tab-btn');
    const panels = container.querySelectorAll('.tab-panel');
    buttons.forEach(btn => {
        btn.addEventListener('click', () => {
            const target = btn.getAttribute('data-target');
            buttons.forEach(b => { b.style.color = 'var(--text-muted)'; b.style.borderBottom = 'none'; });
            btn.style.color = 'rgb(var(--ctp-rosewater))';
            btn.style.borderBottom = '2px solid rgb(var(--ctp-rosewater))';
            panels.forEach(p => p.style.display = 'none');
            const targetPanel = container.querySelector(`#${uid}-${target}`);
            if (targetPanel) targetPanel.style.display = 'block';
        });
    });
}, 180);
```

```dataviewjs
// 1. 설정 및 데이터 로드
const today = dv.date("today");
const todayStr = today.toFormat("yyyy-MM-dd");
const templatePath = "Templates/Tasks.md";

// 05. Tasks 폴더에서 오늘 일정을 가져와 정렬 로직 적용
const allTasks = dv.pages('"05. Tasks"').filter(t => {
    if (!t.날짜) return false;
    const d = dv.date(t.날짜);
    return d && d.year === today.year && d.month === today.month && d.day === today.day;
}).sort(t => {
    // [정렬 1순위] 계획 시간 (가까운 순)
    const planTime = t.계획 || "99:99";
    // [정렬 2순위] 중요/긴급 (중요긴급 > 중요 > 긴급 > 일반 순으로 가중치 부여)
    const priorityMap = { "중요긴급": 1, "중요": 2, "긴급": 3, "일반": 4 };
    const priority = priorityMap[t["중요/긴급"]] || 5;
    // [정렬 3순위] 구분 (집중 > 일반 > 쉬운 순)
    const categoryMap = { "집중": 1, "일반": 2, "쉬운": 3, "쉬움": 3 };
    const category = categoryMap[t.구분] || 4;

    return `${planTime}-${priority}-${category}`;
}, 'asc');

const sections = [
    { id: "today_all", icon: "◈", label: "오늘",      filter: t => true },
    { id: "focus",     icon: "◎", label: "집중",      filter: t => t.구분 === "집중" },
    { id: "easy",      icon: "○", label: "쉬운",      filter: t => t.구분 === "쉬움" || t.구분 === "쉬운" },
    { id: "sched",     icon: "⊙", label: "일정",      filter: t => t.구분 === "일정" },
    { id: "delegate",  icon: "◇", label: "위임",      filter: t => t.구분 === "위임" },
    { id: "proj",      icon: "▣", label: "프로젝트별", filter: t => t.프로젝트 },
    { id: "delay",     icon: "▤", label: "지연",      filter: t => t.구분 === "지연" }
];

const uid = "today-matrix-" + Math.random().toString(36).substring(2, 7);

// 2. UI 생성
let html = `<div id="${uid}-container" style="border: 1px solid rgba(var(--ctp-rosewater), 0.2); border-radius: 12px; padding: 15px;">`;

// 헤더: 추가 버튼 포함
html += `<div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; border-bottom: 1px solid rgba(var(--ctp-rosewater), 0.1); padding-bottom: 10px;">
            <span style="color: rgb(var(--ctp-rosewater)); font-weight: 800; font-size: 0.85rem; letter-spacing: 0.05em;">DAILY MISSION CONTROL</span>
            <button class="add-btn-top" style="background: transparent; cursor: pointer; font-size: 0.6rem; color: rgb(var(--ctp-rosewater)); border: 1px solid rgba(var(--ctp-rosewater), 0.5); padding: 3px 10px; border-radius: 4px; font-weight: 800;">+ ADD TASK</button>
         </div>`;

// 탭 버튼
html += `<div style="display: grid; grid-template-columns: repeat(7, 1fr); gap: 5px; margin-bottom: 20px;">`;
sections.forEach((sec, i) => {
    const active = i === 0 ? "1" : "0.4";
    const bColor = i === 0 ? "rgb(var(--ctp-rosewater))" : "rgba(var(--ctp-rosewater), 0.1)";
    html += `<div class="sec-tab" data-target="${sec.id}" style="cursor: pointer; text-align: center; padding: 8px 2px; border-radius: 6px; border: 1.5px solid ${bColor}; opacity: ${active}; transition: 0.2s;">
                <div style="font-size: 0.9rem; color: rgb(var(--ctp-rosewater)); margin-bottom: 3px;">${sec.icon}</div>
                <div style="font-size: 0.55rem; font-weight: 700; color: var(--text-normal);">${sec.label}</div>
             </div>`;
});
html += `</div>`;

// 패널 생성 (표 형식)
sections.forEach((sec, i) => {
    const display = i === 0 ? "block" : "none";
    const tasks = allTasks.filter(sec.filter);
    html += `<div class="sec-panel" id="${uid}-${sec.id}" style="display: ${display};">`;
    
    if (tasks.length > 0) {
        html += `<table style="width: 100%; border-collapse: collapse; font-size: 0.7rem; color: var(--text-normal);">
                    <tr style="border-bottom: 1px solid rgba(var(--ctp-rosewater), 0.2); color: var(--text-muted); text-align: left;">
                        <th style="padding: 8px 4px; width: 15%;">계획</th>
                        <th style="padding: 8px 4px; width: 45%;">할 일 이름</th>
                        <th style="padding: 8px 4px; width: 15%;">구분</th>
                        <th style="padding: 8px 4px; width: 25%;">중요/긴급</th>
                    </tr>`;
        
        tasks.forEach(t => {
            const isDone = t.완료여부 === true;
            const opacity = isDone ? "opacity: 0.4; text-decoration: line-through;" : "";
            const prioColor = t["중요/긴급"] === "중요긴급" ? "color: rgb(var(--ctp-maroon)); font-weight: 800;" : "";
            
            html += `<tr style="${opacity} border-bottom: 1px solid rgba(var(--ctp-surface1), 0.3);">
                        <td style="padding: 10px 4px; font-family: monospace;">${t.계획 || "--:--"}</td>
                        <td style="padding: 10px 4px; font-weight: 600;">
                            <a class="internal-link" href="${t.file.path}" style="color: inherit; text-decoration: none;">${t.제목 || t.file.name}</a>
                        </td>
                        <td style="padding: 10px 4px; color: var(--text-muted);">${t.구분 || "-"}</td>
                        <td style="padding: 10px 4px; ${prioColor}">${t["중요/긴급"] || "일반"}</td>
                    </tr>`;
        });
        html += `</table>`;
    } else {
        html += `<div style="padding: 30px; text-align: center; font-size: 0.7rem; color: var(--text-faint); border: 1px dotted rgba(var(--ctp-rosewater), 0.15); border-radius: 8px;">NO MISSIONS FOUND</div>`;
    }
    html += `</div>`;
});

html += `</div>`;
dv.el("div", html);

// 3. 자바스크립트 바인딩 (탭 & 템플릿 추가)
setTimeout(() => {
    const container = document.getElementById(`${uid}-container`);
    if (!container) return;

    const tabs = container.querySelectorAll('.sec-tab');
    const panels = container.querySelectorAll('.sec-panel');
    tabs.forEach(tab => {
        tab.addEventListener('click', () => {
            const target = tab.getAttribute('data-target');
            tabs.forEach(t => { t.style.opacity = '0.4'; t.style.borderColor = 'rgba(var(--ctp-rosewater), 0.1)'; });
            tab.style.opacity = '1'; tab.style.borderColor = 'rgb(var(--ctp-rosewater))';
            panels.forEach(p => p.style.display = 'none');
            container.querySelector(`#${uid}-${target}`).style.display = 'block';
        });
    });

    const addBtn = container.querySelector('.add-btn-top');
    addBtn.addEventListener('click', async () => {
        const activeTab = container.querySelector('.sec-tab[style*="opacity: 1"]');
        const activeLabel = activeTab ? activeTab.querySelector('div:last-child').innerText : "오늘";
        const fileName = `05. Tasks/Task_${Date.now()}.md`;
        let fileContent = `---
날짜: ${todayStr}
계획: 09:00
구분: ${activeLabel}
중요/긴급: 일반
완료여부: false
---`;
        const templateFile = app.vault.getAbstractFileByPath(templatePath);
        if (templateFile) {
            const templateContent = await app.vault.read(templateFile);
            fileContent += "\n" + templateContent.replace(/---[\s\S]*?---/, '').trim();
        }
        try {
            const newFile = await app.vault.create(fileName, fileContent);
            app.workspace.getLeaf(false).openFile(newFile);
        } catch (e) { new Notice("파일 생성 실패: " + e.message); }
    });
}, 250);
```
---

## 🗓️ 프로젝트 타임라인

```dataviewjs
const pages = dv.pages('"04. Projects"')
  .where(p => p.상태 !== "아카이브" && p.상태 !== "완료");

const today = dv.date("today");
const startDate = today.minus({days: 14});
const totalDays = 42;

let html = `<div class="sb-timeline" style="position:relative; min-height:${(pages.length + 1) * 36 + 50}px;">`;

// 헤더 날짜
html += `<div class="sb-timeline-header" style="display:flex; margin-bottom:8px; padding-bottom:6px; border-bottom:1px solid rgba(0,0,0,0.1);">`;
for (let i = 0; i < totalDays; i++) {
  const d = startDate.plus({days: i});
  const isToday = d.equals(today);
  const show = d.day === 1 || d.day === 8 || d.day === 15 || d.day === 22 || i === 0;
  const label = show ? `${d.month}/${d.day}` : "";
  const style = isToday
    ? "color:#007aff; font-weight:700;"
    : "color:#636366;";
  html += `<div class="sb-tl-date" style="flex:1; text-align:center; font-size:0.55rem; ${style} min-width:20px;">${label}</div>`;
}
html += `</div>`;

// today 선
const todayOffset = 14;
const todayLeft = (todayOffset / totalDays) * 100;
html += `<div class="sb-tl-today-line" style="position:absolute; left:${todayLeft}%; top:30px; bottom:0; width:2px; background:#ff3b30; z-index:3;">`;
html += `<div class="sb-tl-today-dot" style="position:absolute; top:-4px; left:-3px; width:8px; height:8px; border-radius:50%; background:#ff3b30;"></div>`;
html += `</div>`;

// 프로젝트 바
let row = 0;
for (let p of pages) {
  const pDate = p.날짜 ? dv.date(p.날짜) : null;
  const created = p.생성일 ? dv.date(p.생성일) : today;

  const barStart = created;
  const barEnd = pDate || today.plus({days: 14});

  const startOff = Math.max(0, barStart.diff(startDate, "days").days);
  const endOff = Math.min(totalDays, barEnd.diff(startDate, "days").days);

  if (endOff <= 0 || startOff >= totalDays) { row++; continue; }

  const left = (startOff / totalDays) * 100;
  const width = ((endOff - startOff) / totalDays) * 100;

  const statusClass = p.상태 === "진행 중" ? "status-진행중"
    : p.상태 === "집중" ? "status-집중"
    : p.상태 === "시작 전" ? "status-시작전"
    : "default";

  // D-day 계산
  let ddayText = "";
  if (pDate) {
    const diff = pDate.diff(today, "days").days;
    ddayText = diff > 0 ? `D-${Math.round(diff)}` : diff === 0 ? "D-Day" : `${Math.abs(Math.round(diff))}일 지남`;
  }

  const topPx = 40 + row * 34;
  html += `<div class="sb-tl-bar ${statusClass}" style="position:absolute; left:${left}%; width:${Math.max(width, 2)}%; top:${topPx}px;">`;
  if (ddayText) html += `<span class="sb-tl-dday">${ddayText}</span>`;
  html += `<a class="internal-link" href="${p.file.name}" style="color:white; text-decoration:none; font-size:0.7rem;">${p.제목 || p.file.name}</a>`;
  html += `</div>`;
  row++;
}

html += `</div>`;
dv.el("div", html);
```

---

```dataviewjs
const header = dv.el("div", "", {cls: "sb-section-header"});
header.innerHTML = `<span class="sb-section-icon-title"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="vertical-align:-3px; margin-right:6px;"><path d="M4.5 16.5c-1.5 1.26-2 5-2 5s3.74-.5 5-2c.71-.84.7-2.13-.09-2.91a2.18 2.18 0 0 0-2.91-.09z"/><path d="m12 15-3-3a22 22 0 0 1 2-3.95A12.88 12.88 0 0 1 22 2c0 2.72-.78 7.5-6 11a22.35 22.35 0 0 1-4 2z"/><path d="M9 12H4s.55-3.03 2-4c1.62-1.08 5 0 5 0"/><path d="M12 15v5s3.03-.55 4-2c1.08-1.62 0-5 0-5"/></svg>프로젝트</span>`;
const btn = header.createEl("button", {text: "+ 추가", cls: "sb-add-btn"});
btn.addEventListener("click", () => app.commands.executeCommandById("quickadd:choice:New Project"));

const allTasks = dv.pages('"05. Tasks"');
const pages = dv.pages('"04. Projects"').where(p => p.상태 !== "아카이브");
const statuses = ["진행 중", "집중", "시작 전", "완료", "중단"];
const colors = {
  "진행 중": "green", "집중": "orange", "시작 전": "gray",
  "완료": "blue", "중단": "red"
};
const today = dv.date("today");

function calcInfo(projName) {
  const t = allTasks.where(t => t.프로젝트 === projName || String(t.프로젝트) === projName);
  const total = t.length;
  const done = t.where(t => t.완료여부 === true).length;
  const remain = total - done;
  const rate = total > 0 ? Math.round((done / total) * 100) : 0;
  return { total, done, remain, rate };
}

let html = `<div class="sb-kanban">`;
for (let status of statuses) {
  const items = pages.where(p => p.상태 === status);
  html += `<div class="sb-kanban-col">`;
  html += `<div class="sb-kanban-col-title">${status} (${items.length})</div>`;

  for (let p of items) {
    const badge = colors[status] || "gray";
    const info = calcInfo(p.file.name);

    // D-day
    let dday = "";
    if (p.날짜) {
      const diff = Math.round(dv.date(p.날짜).diff(today, "days").days);
      dday = diff > 0 ? `D-${diff}` : diff === 0 ? "D-Day" : `D+${Math.abs(diff)}`;
    }

    // 날짜 범위
    const startDate = p.생성일 ? dv.date(p.생성일).toFormat("yyyy/MM/dd") : "";
    const endDate = p.날짜 ? dv.date(p.날짜).toFormat("yyyy/MM/dd") : "";
    const dateRange = startDate && endDate ? `${startDate} → ${endDate}` : endDate || startDate || "";

    // 박스
    const box = p.박스 ? String(p.박스) : "";

    html += `<div class="sb-kanban-card">`;
    html += `<div class="sb-kanban-card-title"><a class="internal-link" href="${p.file.name}">${p.제목 || p.file.name}</a></div>`;

    if (dday) html += `<div style="font-size:0.65rem; font-weight:700; color:#007aff; margin:2px 0;">${dday}</div>`;

    html += `<div style="height:3px; background:#e0e0e0; border-radius:2px; margin:4px 0;"><div style="height:100%; width:${info.rate}%; background:#34c759; border-radius:2px;"></div></div>`;
    html += `<div style="font-size:0.65rem; color:#636366;">${info.rate.toFixed(2)}%</div>`;
    html += `<div style="font-size:0.62rem; color:#8e8e93; margin-top:2px;">총 작업: ${info.total} | 남은 작업: ${info.remain} | ${info.rate}% 달성</div>`;

    if (box) html += `<div style="font-size:0.62rem; color:#8e8e93; margin-top:2px;">📦 ${box}</div>`;
    if (dateRange) html += `<div style="font-size:0.62rem; color:#8e8e93; margin-top:1px;">📅 ${dateRange}</div>`;

    html += `</div>`;
  }
  if (items.length === 0) {
    html += `<div style="font-size:0.7rem; color:#8e8e93; text-align:center; padding:16px 0;">비어있음</div>`;
  }
  html += `</div>`;
}
html += `</div>`;
dv.el("div", html);
```

---

## 🎯 목표

```dataviewjs
const goalAllTasks = dv.pages('"05. Tasks"');
const goalAllProjects = dv.pages('"04. Projects"');
const pages = dv.pages('"03. Goals"').where(g => g.상태 !== "아카이브");
const colors = {
  "진행 중": "green", "집중": "orange", "시작 전": "gray",
  "완료": "blue", "중단": "red"
};

function goalRate(goalName) {
  const projs = goalAllProjects.where(p => p.목표 === goalName || String(p.목표) === goalName);
  if (projs.length === 0) return 0;
  let sumRate = 0;
  for (let p of projs) {
    const t = goalAllTasks.where(t => t.프로젝트 === p.file.name || String(t.프로젝트) === p.file.name);
    if (t.length > 0) sumRate += (t.where(t => t.완료여부 === true).length / t.length) * 100;
  }
  return Math.round(sumRate / projs.length);
}

let html = `<div class="sb-kanban">`;

const statuses = ["진행 중", "집중", "시작 전", "완료"];
for (let status of statuses) {
  const items = pages.where(g => g.상태 === status);
  html += `<div class="sb-kanban-col">`;
  html += `<div class="sb-kanban-col-title">🎯 ${status} (${items.length})</div>`;

  for (let g of items) {
    const badge = colors[status] || "gray";
    const rate = goalRate(g.file.name);

    html += `<div class="sb-kanban-card">`;
    html += `<div class="sb-kanban-card-title"><a class="internal-link" href="${g.file.name}">${g.제목 || g.file.name}</a></div>`;

    html += `<div style="height:3px; background:#e0e0e0; border-radius:2px; margin:6px 0;">`;
    html += `<div style="height:100%; width:${rate}%; background:#34c759; border-radius:2px;"></div>`;
    html += `</div>`;

    html += `<div class="sb-kanban-card-meta">`;
    html += `<span class="sb-kanban-badge ${badge}">${status}</span>`;
    html += `<span>${rate}%</span>`;
    if (g.분기) html += `<span>${g.분기}</span>`;
    if (g["목표 달성일"]) html += `<span>${dv.date(g["목표 달성일"]).toFormat("MM/dd")}</span>`;
    html += `</div>`;
    html += `</div>`;
  }
  if (items.length === 0) {
    html += `<div style="font-size:0.7rem; color:#8e8e93; text-align:center; padding:16px 0;">비어있음</div>`;
  }
  html += `</div>`;
}
html += `</div>`;
dv.el("div", html);
```

---

## 📝 자료

```dataview
TABLE WITHOUT ID
  file.link AS "이름",
  분류 AS "분류",
  중요도 AS "중요도",
  박스 AS "박스",
  프로젝트 AS "프로젝트",
  file.mtime AS "수정일"
FROM "06. Resources"
SORT file.mtime DESC
LIMIT 20
```

```button
name 새로 만들기
type command
action QuickAdd: New Resource
color default
```

---

## 📦 박스

```dataviewjs
const pages = dv.pages('"02. Boxes"').where(b => b.type === "box");

let html = `<div class="sb-gallery">`;
for (let b of pages) {
  const emoji = b.이모지 || "📦";
  const desc = b.설명 || "";
  const status = b.상태 || "일반";

  html += `<div class="sb-gallery-card">`;
  html += `<div class="sb-gallery-icon">${emoji}</div>`;
  html += `<div class="sb-gallery-title"><a class="internal-link" href="${b.file.name}">${b.제목 || b.file.name}</a></div>`;
  html += `<div class="sb-gallery-desc">${desc}</div>`;
  html += `<div class="sb-gallery-stats">`;
  html += `<span class="sb-kanban-badge ${status === "고정하기" ? "blue" : "gray"}">${status}</span>`;
  html += `</div>`;
  html += `</div>`;
}
if (pages.length === 0) {
  html += `<div style="color:#8e8e93; font-size:0.8rem; padding:20px; text-align:center;">박스가 없습니다. 새 박스를 만들어보세요.</div>`;
}
html += `</div>`;
dv.el("div", html);
```

```button
name 새로 만들기
type command
action QuickAdd: New Box
color default
```

---

## ⚙️ 관리도구

```dataviewjs
const items = [
  {icon: "📦", label: "박스 관리", link: "02. Boxes"},
  {icon: "🎯", label: "목표 관리", link: "03. Goals"},
  {icon: "🚀", label: "프로젝트 관리", link: "04. Projects"},
  {icon: "✅", label: "할일 관리", link: "05. Tasks"},
  {icon: "📝", label: "자료 관리", link: "06. Resources"},
  {icon: "📚", label: "도서 라이브러리", link: "07. Books"},
  {icon: "📅", label: "저널 관리", link: "08. Journal"},
  {icon: "🚧", label: "장애물 극복하기", link: "09. Bottlenecks"},
  {icon: "🗄️", label: "아카이브", link: "Archive"},
];

let html = `<div class="sb-mgmt-grid">`;
for (let item of items) {
  html += `<div class="sb-mgmt-item" onclick="app.workspace.openLinkText('', '${item.link}')">`;
  html += `<div class="sb-mgmt-icon">${item.icon}</div>`;
  html += `<div class="sb-mgmt-label">${item.label}</div>`;
  html += `</div>`;
}
html += `</div>`;
dv.el("div", html);
```

---

## 📊 DB 현황

| DB | 링크 |
|---|---|
| 📥 Captures | [[01. Captures\|바로가기]] |
| 📦 Boxes | [[02. Boxes]] |
| 🎯 Goals | [[03. Goals]] |
| 🚀 Projects | [[04. Projects]] |
| ✅ Tasks | [[05. Tasks]] |
| 📝 Resources | [[06. Resources]] |
| 📚 Books | [[07. Books]] |
| 📅 Daily Journal | [[08. Journal/Daily]] |
| 📆 Weekly Journal | [[08. Journal/Weekly]] |
| 🗓️ Monthly Journal | [[08. Journal/Monthly]] |
| 🚧 Bottleneck | [[09. Bottlenecks]] |
