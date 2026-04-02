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

---

// 1. 기본 설정 및 데이터 수집
const today = dv.date("today");
const startOfWeek = today.startOf("week");
const endOfWeek = today.endOf("week");
const startOfMonth = today.startOf("month");
const endOfMonth = today.endOf("month");

const allTasks = dv.pages('"05. Tasks"');
const allResources = dv.pages('"06. Resources"');

// 2. 탭 정의 (요청하신 필터 5종)
const taskTabs = [
  { id: "week",   icon: "📅", label: "이번 주",     filter: t => t.완료여부 !== true && t.날짜 && dv.date(t.날짜) >= startOfWeek && dv.date(t.날짜) <= endOfWeek },
  { id: "month",  icon: "🗓️", label: "이번 달",     filter: t => t.완료여부 !== true && t.날짜 && dv.date(t.날짜) >= startOfMonth && dv.date(t.날짜) <= endOfMonth },
  { id: "sched",  icon: "⏰", label: "일정",       filter: t => t.완료여부 !== true && t.구분 === "일정" },
  { id: "done",   icon: "✅", label: "이번 주 완료", filter: t => t.완료여부 === true && t.날짜 && dv.date(t.날짜) >= startOfWeek && dv.date(t.날짜) <= endOfWeek },
  { id: "recent", icon: "📚", label: "최근 자료",   isResource: true }
];

// 3. 고유 ID 생성 (충돌 방지)
const uid = "task-cal-tabs-" + Math.random().toString(36).slice(2,8);

// 4. HTML 구조 생성
let html = `<details open style="background: rgba(var(--ctp-surface0), 0.3); border: 1px solid rgba(var(--ctp-surface1), 0.5); border-radius: 12px; padding: 12px;">`;
html += `<summary style="font-weight: 700; font-size: 1rem; cursor: pointer; margin-bottom: 10px; color: rgb(var(--ctp-text));">📅 캘린더 뷰 및 필터</summary>`;

// 4-1. 상단 탭 바
html += `<div class="sb-tabs" id="${uid}-bar" style="margin-bottom: 12px;">`;
taskTabs.forEach((tab, i) => {
  const cls = i === 0 ? "sb-tab active" : "sb-tab";
  html += `<div class="${cls}" data-tab="${tab.id}" style="font-size: 0.75rem;" onclick="
    this.parentElement.querySelectorAll('.sb-tab').forEach(t=>t.classList.remove('active'));
    this.classList.add('active');
    this.closest('details').querySelectorAll('.sb-tab-panel').forEach(p=>p.style.display='none');
    this.closest('details').querySelector('#${uid}-'+this.dataset.tab).style.display='block';
  ">${tab.icon} ${tab.label}</div>`;
});
html += `</div>`;

// 4-2. 탭별 패널 생성
taskTabs.forEach((tab, i) => {
  const display = i === 0 ? "block" : "none";
  html += `<div class="sb-tab-panel" id="${uid}-${tab.id}" style="display:${display};">`;

  if (tab.isResource) {
    // [최근 자료] 필터 - 리소스 테이블
    const recentRes = allResources.sort(r => r.file.mtime, "desc").slice(0, 8);
    html += `<table style="width:100%; font-size:0.8rem; border-collapse:collapse;">`;
    recentRes.forEach(r => {
      html += `<tr style="border-bottom:1px solid rgba(var(--ctp-surface2),0.3);">
        <td style="padding:6px 0;"><a class="internal-link" href="${r.file.name}">${r.file.name}</a></td>
        <td style="text-align:right; font-size:0.7rem; color:rgb(var(--ctp-subtext0));">${dv.date(r.file.mtime).toFormat("MM/dd HH:mm")}</td>
      </tr>`;
    });
    html += `</table>`;
  } else {
    [할 일 관련] 필터 - 할 일 리스트
    const filtered = allTasks.where(tab.filter).sort(t => t.날짜, "asc");
    if (filtered.length === 0) {
      html += `<div style="text-align:center; padding:20px; color:rgb(var(--ctp-subtext1)); font-size:0.8rem;">항목이 없습니다.</div>`;
    } else {
      html += `<table style="width:100%; font-size:0.8rem; border-collapse:collapse;">`;
      filtered.forEach(t => {
        const dateStr = t.날짜 ? dv.date(t.날짜).toFormat("MM/dd") : "-";
        const color = t.구분 === "집중" ? "#ff9500" : t.구분 === "일정" ? "#007aff" : "#34c759";
        html += `<tr style="border-bottom:1px solid rgba(var(--ctp-surface2),0.3);">
          <td style="padding:6px 0; border-left:3px solid ${color}; padding-left:8px;">
            <a class="internal-link" href="${t.file.name}">${t.제목 || t.file.name}</a>
          </td>
          <td style="text-align:right; font-size:0.7rem; color:rgb(var(--ctp-subtext0));">${dateStr}</td>
        </tr>`;
      });
      html += `</table>`;
    }
  }
  html += `</div>`;
});

html += `</details>`;
dv.el("div", html);

### 오늘

```dataviewjs
const today = dv.date("today");
const tomorrow = today.plus({days: 1});
const allTasks = dv.pages('"05. Tasks"').where(t => t.완료여부 !== true);

// 탭 정의
const tabs = [
  {id: "today",   icon: "🕐", label: "오늘",       filter: t => t.날짜 && dv.date(t.날짜).equals(today)},
  {id: "tomorrow", icon: "🕐", label: "내일",       filter: t => t.날짜 && dv.date(t.날짜).equals(tomorrow)},
  {id: "focus",   icon: "🔥", label: "집중",       filter: t => t.구분 === "집중"},
  {id: "easy",    icon: "✏️", label: "쉬운",       filter: t => t.구분 === "쉬운"},
  {id: "sched",   icon: "🗓", label: "일정",       filter: t => t.구분 === "일정"},
  {id: "deleg",   icon: "👥", label: "위임",       filter: t => t.구분 === "위임"},
  {id: "proj",    icon: "🚀", label: "프로젝트별",  filter: t => true, grouped: true},
  {id: "late",    icon: "⏰", label: "지연",       filter: t => t.날짜 && dv.date(t.날짜) < today},
  {id: "urgent",  icon: "🔴", label: "긴급+중요",   filter: t => t.중요긴급 === "1. 중요O + 긴급O"},
];

// 고유 ID
const uid = "sb-task-tabs-" + Math.random().toString(36).slice(2,8);

// 탭 바
let html = `<div class="sb-tabs" id="${uid}-bar">`;
tabs.forEach((tab, i) => {
  const cls = i === 0 ? "sb-tab active" : "sb-tab";
  html += `<div class="${cls}" data-tab="${tab.id}" onclick="
    this.parentElement.querySelectorAll('.sb-tab').forEach(t=>t.classList.remove('active'));
    this.classList.add('active');
    this.closest('.block-language-dataviewjs').querySelectorAll('.sb-tab-panel').forEach(p=>p.style.display='none');
    this.closest('.block-language-dataviewjs').querySelector('#${uid}-'+this.dataset.tab).style.display='block';
  ">${tab.icon} ${tab.label}</div>`;
});
html += `</div>`;

// 탭 패널들
tabs.forEach((tab, i) => {
  const display = i === 0 ? "block" : "none";
  const filtered = allTasks.where(tab.filter);

  html += `<div class="sb-tab-panel" id="${uid}-${tab.id}" style="display:${display}; margin-top:8px;">`;

  if (tab.grouped) {
    // 프로젝트별 그룹
    const projects = {};
    filtered.forEach(t => {
      const pName = t.프로젝트 ? String(t.프로젝트) : "미배정";
      if (!projects[pName]) projects[pName] = [];
      projects[pName].push(t);
    });

    for (let [proj, tasks] of Object.entries(projects)) {
      html += `<div style="margin-bottom:12px;">`;
      html += `<div style="font-size:0.7rem; font-weight:700; color:#636366; text-transform:uppercase; letter-spacing:0.04em; margin-bottom:4px;">🚀 ${proj} (${tasks.length})</div>`;
      html += `<table style="width:100%; border-collapse:collapse; font-size:0.8rem;">`;
      tasks.forEach(t => {
        const name = t.제목 || t.file.name;
        const date = t.날짜 ? dv.date(t.날짜).toFormat("MM/dd") : "-";
        html += `<tr style="border-bottom:1px solid rgba(0,0,0,0.06);">`;
        html += `<td style="padding:6px 8px;"><a class="internal-link" href="${t.file.name}">${name}</a></td>`;
        html += `<td style="padding:6px 8px; color:#636366; font-size:0.7rem;">${t.구분 || ""}</td>`;
        html += `<td style="padding:6px 8px; color:#636366; font-size:0.7rem; text-align:right;">${date}</td>`;
        html += `</tr>`;
      });
      html += `</table></div>`;
    }
  } else {
    // 일반 테이블
    if (filtered.length === 0) {
      html += `<div style="color:#8e8e93; font-size:0.8rem; padding:20px; text-align:center;">해당 항목이 없습니다</div>`;
    } else {
      html += `<table style="width:100%; border-collapse:collapse; font-size:0.8rem;">`;
      html += `<thead><tr>`;
      html += `<th style="text-align:left; padding:5px 8px; font-size:0.62rem; font-weight:600; text-transform:uppercase; color:#636366; border-bottom:1px solid rgba(0,0,0,0.1); letter-spacing:0.06em;">할 일</th>`;
      if (tab.id === "deleg") {
        html += `<th style="text-align:left; padding:5px 8px; font-size:0.62rem; font-weight:600; text-transform:uppercase; color:#636366; border-bottom:1px solid rgba(0,0,0,0.1);">수임자</th>`;
      } else {
        html += `<th style="text-align:left; padding:5px 8px; font-size:0.62rem; font-weight:600; text-transform:uppercase; color:#636366; border-bottom:1px solid rgba(0,0,0,0.1);">프로젝트</th>`;
      }
      html += `<th style="text-align:right; padding:5px 8px; font-size:0.62rem; font-weight:600; text-transform:uppercase; color:#636366; border-bottom:1px solid rgba(0,0,0,0.1);">날짜</th>`;
      html += `</tr></thead><tbody>`;

      for (let t of filtered.slice(0, 20)) {
        const name = t.제목 || t.file.name;
        const date = t.날짜 ? dv.date(t.날짜).toFormat("MM/dd") : "-";
        const col2 = tab.id === "deleg" ? (t.수임자 || "-") : (t.프로젝트 ? String(t.프로젝트) : "-");

        // 지연 표시
        let lateTag = "";
        if (tab.id === "late" && t.날짜) {
          const diff = Math.abs(Math.round(today.diff(dv.date(t.날짜), "days").days));
          lateTag = `<span style="font-size:0.6rem; background:#fce4e4; color:#c0392b; padding:1px 5px; border-radius:3px; margin-left:4px;">${diff}일 지남</span>`;
        }

        html += `<tr style="border-bottom:1px solid rgba(0,0,0,0.06);">`;
        html += `<td style="padding:6px 8px;"><a class="internal-link" href="${t.file.name}">${name}</a>${lateTag}</td>`;
        html += `<td style="padding:6px 8px; color:#636366; font-size:0.75rem;">${col2}</td>`;
        html += `<td style="padding:6px 8px; color:#636366; font-size:0.75rem; text-align:right;">${date}</td>`;
        html += `</tr>`;
      }
      html += `</tbody></table>`;
      if (filtered.length > 20) {
        html += `<div style="font-size:0.65rem; color:#8e8e93; text-align:center; padding:6px;">+${filtered.length - 20}개 더</div>`;
      }
    }
  }
  html += `</div>`;
});

dv.el("div", html);
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
