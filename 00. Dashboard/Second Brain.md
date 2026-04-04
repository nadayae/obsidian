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
html += `<summary style="font-weight: 800; cursor: pointer; margin-bottom: 16px; margin-top: 16px; color: rgb(var(--ctp-subtext0)); font-size: 1.3rem; letter-spacing: 0.06em; color:#d6827d; list-style: none;">캘린더</summary>`;

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
// 1. 데이터 로드 및 정렬 로직
const today = dv.date("today");
const todayStr = today.toFormat("yyyy-MM-dd");
const templatePath = "Templates/Tasks.md";

// [필터 A] 오늘 날짜의 할 일
const todayTasks = dv.pages('"05. Tasks"').filter(t => {
    if (!t.날짜) return false;
    const d = dv.date(t.날짜);
    return d && d.year === today.year && d.month === today.month && d.day === today.day;
});

// [필터 B] 지연된 할 일 (오늘 이전 날짜 && 완료X)
const delayedTasks = dv.pages('"05. Tasks"').filter(t => {
    if (!t.날짜 || t.완료여부 === true) return false;
    const d = dv.date(t.날짜);
    return d && d < today;
}).sort(t => dv.date(t.날짜), 'asc');

// [필터 C] 프로젝트용 (날짜 상관없이 완료X)
const totalProjectTasks = dv.pages('"05. Tasks"').filter(t => t.프로젝트 && t.완료여부 !== true);

const sortLogic = (t) => {
    let timeRaw = String(t.계획 || "99:99").replace(":", "");
    if (timeRaw.length === 3) timeRaw = "0" + timeRaw; 
    let prioWeight = 5;
    const pStr = String(t.중요긴급 || "");
    if (pStr.includes("중요O + 긴급O")) prioWeight = 1;
    else if (pStr.includes("중요X + 긴급O")) prioWeight = 2;
    else if (pStr.includes("중요O + 긴급X")) prioWeight = 3;
    const catMap = { "집중": 1, "일반": 2, "쉬운": 3, "위임": 4, "일정": 5, "나중에": 6 };
    return `${timeRaw}-${prioWeight}-${catMap[t.구분] || 7}`;
};

const sections = [
    { id: "today_all", icon: "◈", label: "오늘",      tasks: todayTasks.sort(sortLogic, 'asc') },
    { id: "focus",     icon: "◎", label: "집중",      tasks: todayTasks.filter(t => t.구분 === "집중").sort(sortLogic, 'asc') },
    { id: "easy",      icon: "○", label: "쉬운",      tasks: todayTasks.filter(t => t.구분 === "쉬운" || t.구분 === "쉬움").sort(sortLogic, 'asc') },
    { id: "proj",      icon: "▣", label: "프로젝트별", tasks: totalProjectTasks.sort(sortLogic, 'asc'), isProjectView: true },
    { id: "delay",     icon: "▤", label: "지연",      tasks: delayedTasks, isDelayView: true }
];

const uid = "hybrid-v4-final-" + Math.random().toString(36).substring(2, 7);

// 2. UI 생성
let html = `<div id="${uid}-container" class="dataviewjs-today-view">`;
html += `<div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
            <div style="font-weight: 800; font-size: 1.3rem;margin-top:20px;color:#d6827d;; letter-spacing: 0.06em;">할일</div>
            <button class="add-btn-top" style="background: rgba(var(--ctp-rosewater), 0.2); cursor: pointer; font-size: 0.65rem; color: rgb(var(--ctp-rosewater)); border: 1px solid rgba(var(--ctp-rosewater), 0.4); padding: 6px 14px; border-radius: 4px; font-weight: 700;">+ ADD TASK</button>
         </div>`;

html += `<div style="display: flex; gap: 18px; margin-bottom: 20px; border-bottom: 1px solid rgba(var(--ctp-rosewater), 0.2); padding-bottom: 10px;">`;
sections.forEach((sec, i) => {
    const isActive = i === 0;
    const count = sec.tasks.length;
    html += `<div class="sec-tab" data-target="${sec.id}" style="cursor: pointer; font-size: 0.8rem; color: ${isActive ? "rgb(var(--ctp-rosewater))" : "var(--text-muted)"}; font-weight: ${isActive ? "800" : "400"}; transition: 0.2s;">
                ${sec.icon} ${sec.label} ${count > 0 ? `<span style="font-size: 0.65rem; opacity: 0.6;">(${count})</span>` : ''}
            </div>`;
});
html += `</div>`;

sections.forEach((sec, i) => {
    html += `<div class="sec-panel" id="${uid}-${sec.id}" style="display: ${i === 0 ? "block" : "none"};">`;
    if (sec.tasks.length > 0) {
        if (sec.isProjectView) {
            const grouped = sec.tasks.groupBy(t => t.프로젝트);
            html += `<div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 12px;">`;
            grouped.forEach(group => {
                html += `<div style="background: rgba(var(--ctp-surface0), 0.5); border: 1px solid rgba(var(--ctp-rosewater), 0.15); border-radius: 10px; padding: 12px;">
                            <div style="font-weight: 800; font-size: 0.75rem; color: rgb(var(--ctp-rosewater)); margin-bottom: 10px; border-bottom: 1px solid rgba(var(--ctp-rosewater), 0.1); padding-bottom: 6px; display: flex; justify-content: space-between;">
                                <span>📦 ${group.key}</span>
                                <span style="font-size: 0.6rem; opacity: 0.5;">Active</span>
                            </div>`;
                group.rows.forEach(t => {
                    const pStr = String(t.중요긴급 || "");
                    let pStyle = "color: var(--text-muted); opacity: 0.6;"; 
                    if (pStr.includes("중요O + 긴급O")) pStyle = "color: #e64553; font-weight: 800;";
                    else if (pStr.includes("중요X + 긴급O")) pStyle = "color: #f2cdcd; font-weight: 700;";
                    html += `<div style="margin-bottom: 8px;">
                                <div style="display: flex; justify-content: space-between; font-size: 0.6rem;">
                                    <span>${t.날짜} | ${t.계획 || "--:--"}</span>
                                    <span style="${pStyle}">${pStr}</span>
                                </div>
                                <div style="font-size: 0.7rem; font-weight: 600;"><a class="internal-link" href="${t.file.path}">${t.제목 || t.file.name}</a></div>
                            </div>`;
                });
                html += `</div>`;
            });
            html += `</div>`;
        } else {
            html += `<table class="dataview table-view-table" style="width: 100%;">
                        <thead><tr class="table-view-tr">
                            <th class="table-view-th">${sec.isDelayView ? "지연일" : "계획"}</th>
                            <th class="table-view-th">이름</th>
                            <th class="table-view-th">구분</th>
                            <th class="table-view-th">중요긴급</th>
                        </tr></thead>
                        <tbody>`;
            sec.tasks.forEach(t => {
                const pStr = String(t.중요긴급 || "");
                let pStyle = "color: var(--text-muted);"; 
                if (pStr.includes("중요O + 긴급O")) pStyle = "color: #e64553; font-weight: 800;";
                else if (pStr.includes("중요X + 긴급O")) pStyle = "color: #f2cdcd; font-weight: 700;";
                
                let timeDisplay = t.계획 || "--:--";
                if (sec.isDelayView) {
                    // [수정 포인트] Luxon diff 기능을 사용하여 일수 계산
                    const taskDate = dv.date(t.날짜);
                    const diffDays = Math.floor(today.diff(taskDate, 'days').days);
                    timeDisplay = `<span style="color: #e64553; font-weight: 800;">D+${diffDays}</span>`;
                }

                html += `<tr class="table-view-tr" style="${t.완료여부 ? 'opacity: 0.4; text-decoration: line-through;' : ''}">
                            <td class="table-view-td">${timeDisplay}</td>
                            <td class="table-view-td"><a class="internal-link" href="${t.file.path}" style="font-weight: 600;">${t.제목 || t.file.name}</a></td>
                            <td class="table-view-td" style="color: var(--text-muted);">${t.구분 || "-"}</td>
                            <td class="table-view-td" style="${pStyle}">${pStr}</td>
                        </tr>`;
            });
            html += `</tbody></table>`;
        }
    } else {
        html += `<div style="padding: 40px; text-align: center; font-size: 0.8rem; color: var(--text-faint);">ALL CLEAR</div>`;
    }
    html += `</div>`;
});
html += `</div>`;
dv.el("div", html);

// JS 탭 바인딩 로직
setTimeout(() => {
    const container = document.getElementById(`${uid}-container`);
    if (!container) return;
    const tabs = container.querySelectorAll('.sec-tab');
    const panels = container.querySelectorAll('.sec-panel');
    tabs.forEach(tab => {
        tab.addEventListener('click', () => {
            tabs.forEach(t => { t.style.color = 'var(--text-muted)'; t.style.fontWeight = '400'; });
            tab.style.color = 'rgb(var(--ctp-rosewater))'; tab.style.fontWeight = '800';
            panels.forEach(p => p.style.display = 'none');
            container.querySelector(`#${uid}-${tab.getAttribute('data-target')}`).style.display = 'block';
        });
    });
}, 250);
```


```dataviewjs
const today = dv.date("today");

// [색상 정의] image_0.png 배경 및 AnuPpuccin 톤 반영
const BG_COLOR = "#EEE6DD"; // 아주 연한 아이보리 베이지
const ROSEWATER_BAR_BG = "#e4a6a3"; // 막대기 레일 (조금 더 진한 베이지)
const ROSEWATER_FILLED = "#f2cdcd"; // [핵심] 선명한 로즈워터 (진행률)
const ROSEWATER_ACCENT = "#D6817D"; // 월 표시/오늘 강조색 (조금 더 진한 로즈워터)

// 1. 데이터 수집: 04. Projects 폴더 기준
const projectPages = dv.pages('"04. Projects"').filter(p => 
    (p.type === "project" || p.file.path.toLowerCase().includes("project")) &&
    p.file.name !== "Project Dashboard"
).array();

// 타임라인 범위 설정 (AnuPpuccin 톤에 맞춰 4월~5월 고정)
let minDate = today.startOf("month");
let maxDate = today.endOf("month").plus({ months: 1 });

const timelineData = projectPages.map(p => {
    const parse = (val) => {
        if (!val) return null;
        let s = String(val).replace(/[\[\]]/g, "").trim();
        return dv.date(s);
    };
    const sDate = parse(p.시작일);
    const eDate = parse(p.목표일) || parse(p.종료일);
    if (!sDate || !eDate) return null;

    if (sDate < minDate) minDate = sDate.startOf("month");
    if (eDate > maxDate) maxDate = eDate.endOf("month");

    return {
        name: p.file.name, link: p.file.path,
        start: sDate, end: eDate,
        progress: p.달성률 || p.진행률 || 0,
        goal: p.목표 || "기타/미지정"
    };
}).filter(p => p !== null);

const totalDays = Math.ceil(maxDate.diff(minDate, 'days').days) + 1;
const dayWidth = 35; // 그리드 너비 축소 (슬림 디자인 어울림)
const labelWidth = 160; 
const fullTimelineWidth = labelWidth + (totalDays * dayWidth);

// 2. UI 생성
const containerId = "timeline-" + Date.now();
let html = `<div id="${containerId}" style="background: ${BG_COLOR}; border-radius: 8px; font-family: var(--font-interface); color: var(--text-normal); overflow: hidden;">`;

// 상단 헤더
html += `<div style="display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 20px; padding: 0px 0px 0;">
            <div style="font-weight: 800; font-size: 1.3rem; marjin-top=16px; color:#d6827d;; letter-spacing: 0.06em;">프로젝트 타임라인 <span style="font-weight: 400; font-size: 0.6rem; color: var(--text-faint); margin-left: 8px;">${today.year}.${today.toFormat('LL')}</span></div>
            <div style="font-size: 0.55rem; color: var(--text-faint); text-transform: uppercase;">Shift + Scroll to Navigate</div>
         </div>`;

// 가로 스크롤 영역
html += `<div style="overflow-x: auto; position: relative;">`;
html += `<div style="position: relative; min-width: ${fullTimelineWidth}px;">`;

// [배경 그리드] ◈ AnuPpuccin 톤에 맞춰 아주 연한 베이지색 라인
html += `<div style="position: absolute; top: 50px; left: ${labelWidth}px; right: 0; bottom: 0; display: flex; z-index: 0; pointer-events: none;">`;
for (let d = 0; d < totalDays; d++) {
    html += `<div style="width: ${dayWidth}px; border-right: 1px solid rgba(232, 216, 201, 0.5); height: 100%;"></div>`;
}
html += `</div>`;

// [날짜 헤더] ◈ AnuPpuccin 톤에 맞춰 로즈워터 딥 테두리
html += `<div style="display: flex; border-bottom: 2px solid ${ROSEWATER_ACCENT}; padding-bottom: 10px; position: sticky; top: 0; z-index: 30; background: ${BG_COLOR};">
            <div style="width: ${labelWidth}px; font-size: 0.6rem; font-weight: 900; color: var(--text-faint); align-self: flex-end; padding-left: 15px;">PROJECTS</div>
            <div style="display: flex; flex-direction: column;">
                <div style="display: flex; height: 18px; position: relative;">`;

// 월 표시
let currentMonth = -1;
for (let d = 0; d < totalDays; d++) {
    const curr = minDate.plus({ days: d });
    if (curr.month !== currentMonth) {
        currentMonth = curr.month;
        html += `<div style="position: absolute; left: ${d * dayWidth}px; font-weight: 800; color: ${ROSEWATER_ACCENT}; font-size: 0.7rem; padding-left: 5px;">${curr.month}M</div>`;
    }
}
html += `</div><div style="display: flex;">`;

// 일 표시
for (let d = 0; d < totalDays; d++) {
    const curr = minDate.plus({ days: d });
    const isToday = curr.hasSame(today, "day");
    html += `<div style="width: ${dayWidth}px; text-align: center; font-size: 0.5rem; color: ${isToday ? 'white' : 'var(--text-muted)'}; background: ${isToday ? ROSEWATER_ACCENT : 'transparent'}; border-radius: 4px; font-weight: ${isToday ? '900' : '400'}; line-height: 16px;">${curr.day}</div>`;
}
html += `</div></div></div>`;

// [프로젝트 리스트 렌더링]
const groups = timelineData.reduce((acc, cur) => {
    if (!acc[cur.goal]) acc[cur.goal] = [];
    acc[cur.goal].push(cur);
    return acc;
}, {});

Object.keys(groups).forEach(goalName => {
    html += `<details open style="margin-bottom: 0px; position: relative; z-index: 10;">
                <summary style="padding: 10px; font-size: 0.7rem; font-weight: 800; color: var(--text-normal); cursor: pointer; list-style: none; border-bottom: 1px solid rgba(232, 216, 201, 0.4); background: ${BG_COLOR}; width: 100%; display: flex; align-items: center; position: sticky; left: 0;">
                    <span style="color: ${ROSEWATER_ACCENT}; margin-right: 8px;">◈</span> ${goalName.toUpperCase()} 
                    <span style="font-size: 0.55rem; color: var(--text-faint); margin-left: 10px; font-weight: 400;">/ ${groups[goalName].length} projects</span>
                </summary>
                <div style="padding-top: 5px; border-bottom: 1px solid rgba(232, 216, 201, 0.2);">`;
    
    groups[goalName].forEach(proj => {
        const startDiff = Math.ceil(proj.start.diff(minDate, 'days').days);
        const duration = Math.ceil(proj.end.diff(proj.start, 'days').days) + 1;
        
        html += `<div style="display: flex; align-items: center; padding: 6px 0;">
                    <div style="width: ${labelWidth}px; padding-left: 20px; flex-shrink: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; position: sticky; left: 0; z-index: 5; background: ${BG_COLOR};">
                        <a class="internal-link" href="${proj.link}" style="font-size: 0.65rem; font-weight: 500; color: var(--text-muted); text-decoration: none;">${proj.name}</a>
                    </div>
                    <div style="display: flex; height: 16px; position: relative; flex-grow: 1;">
                        <div style="margin-left: ${startDiff * dayWidth}px; width: ${duration * dayWidth}px; background: ${ROSEWATER_BAR_BG}; border-radius: 100px; height: 8px; align-self: center; position: relative; z-index: 2; overflow: hidden; border: 1px solid rgba(232, 216, 201, 0.5);">
                            <div style="width: ${proj.progress}%; height: 100%; background: ${ROSEWATER_FILLED}; border-radius: 100px;"></div>
                        </div>
                    </div>
                 </div>`;
    });
    html += `</div></details>`;
});

html += `</div></div></div>`;
dv.el("div", html);
```
---

```dataviewjs
const FIXED_ID = "task-sync-project-kanban";
const projects = dv.pages('"04. Projects"').array();
// [수정] 폴더 경로를 05. Tasks로 정확히 지정
const allTasksFiles = dv.pages('"05. Tasks"'); 
const today = dv.date("today");
const ROSEWATER = "#e6aba9"; 

// [수정] 사용자님의 환경에 맞춰 '계획전' 상태 대응
const activeStatuses = ["계획전", "계획", "진행중", "집중"];
const archiveStatuses = ["중단", "완료"];

// 공통: task-project 매칭 함수
const getLinkedTasks = (p) => allTasksFiles.filter(t => {
    const prop = t.프로젝트;
    if (!prop || prop === "") return false;
    let name = (typeof prop === 'object' && prop.path)
        ? prop.path.split('/').pop().replace(/\.md$/, '')
        : String(prop).replace(/[\[\]"]/g, "").trim();
    return name === p.file.name;
});

// 공통: 프로젝트 카드 HTML 생성
const makeCard = (p) => {
    const linkedTasks = getLinkedTasks(p);
    const totalTasks = linkedTasks.length;
    const completedTasks = linkedTasks.filter(t =>
        t.완료여부 === true || String(t.완료여부).toLowerCase() === "true"
    ).length;
    const remainingTasks = totalTasks - completedTasks;
    const calcProgress = totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;
    const eDate = p.목표일 ? dv.date(String(p.목표일).replace(/[\[\]]/g, "")) : null;
    let dDayText = "D-Day 미정";
    if (eDate) {
        const diff = Math.ceil(eDate.diff(today, 'days').days);
        dDayText = diff === 0 ? "D-Day" : (diff > 0 ? `D-${diff}` : `D+${Math.abs(diff)}`);
    }
    return `<div class="k-card" style="background: var(--background-primary); border: 1px solid rgba(0,0,0,0.1); border-radius: 10px; padding: 16px; margin-bottom: 15px;">
        <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 12px;">
            <a class="internal-link" href="${p.file.path}" style="font-size: 0.85rem; font-weight: 700; color: var(--text-normal); text-decoration: none;">${p.file.name}</a>
        </div>
        <div style="display: inline-block; background: rgba(67, 123, 255, 0.1); color: #437bff; font-size: 0.65rem; font-weight: 800; padding: 2px 6px; border-radius: 4px; margin-bottom: 12px;">${dDayText}</div>
        <div style="margin-bottom: 12px;">
            <div style="display: flex; align-items: center; gap: 10px;">
                <span style="font-size: 0.7rem; font-weight: 700; color: #8a81a3; width: 35px;">${calcProgress}%</span>
                <div style="flex-grow: 1; background: #eee; height: 6px; border-radius: 10px; overflow: hidden;">
                    <div style="width: ${calcProgress}%; background: ${ROSEWATER}; height: 100%;"></div>
                </div>
            </div>
        </div>
        <div style="font-size: 0.65rem; color: #666; margin-bottom: 12px; font-weight: 500;">
            총 작업: ${totalTasks} | 남은 작업: ${remainingTasks} | ${calcProgress}% 달성
        </div>
        <div style="font-size: 0.7rem; color: #444; font-weight: 600;">${p.목표 || '미지정'}</div>
    </div>`;
};

// UI 생성
let html = `<div id="${FIXED_ID}" style="font-family: var(--font-interface); padding: 10px 0;">
    <div style="font-weight: 800; font-size: 1.3rem; color:#d6827d; letter-spacing: 0.06em; margin-bottom: 14px; margin-top: 16px;">프로젝트</div>
    <div style="display: flex; gap: 20px; border-bottom: 1px solid rgba(0,0,0,0.08); padding-bottom: 0px; margin-bottom: 25px;">
        <div class="ntab active" data-target="all"     style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:700; color:${ROSEWATER};">전체</div>
        <div class="ntab"        data-target="today"   style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">오늘</div>
        <div class="ntab"        data-target="bygoal"  style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">목표별</div>
        <div class="ntab"        data-target="archive" style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">아카이브</div>
    </div>
    <div id="display-area" style="gap: 18px; overflow-x: auto; padding-bottom: 15px;"></div>
</div>`;

dv.el("div", html);

const renderK = (target) => {
    const root = document.getElementById(FIXED_ID);
    if (!root) return;
    const display = root.querySelector("#display-area");
    const filtered = projects.filter(p => p.file.name !== "Project Dashboard");

    // ── 오늘 탭 (시작일 <= 오늘 <= 목표일 인 프로젝트) ───
    if (target === "today") {
        const todayProjects = filtered.filter(p => {
            const parse = v => v ? dv.date(String(v).replace(/[\[\]]/g, "").trim()) : null;
            const s = parse(p.시작일);
            const e = parse(p.목표일) || parse(p.종료일);
            if (!s || !e) return false;
            return s <= today && today <= e;
        });

        display.style.display = "grid";
        display.style.gridTemplateColumns = `repeat(${activeStatuses.length}, 1fr)`;
        if (todayProjects.length === 0) {
            display.innerHTML = `<div style="padding:40px; text-align:center; font-size:0.8rem; color:var(--text-faint);">오늘 진행 중인 프로젝트 없음</div>`;
            return;
        }
        display.innerHTML = activeStatuses.map(status => {
            const cards = todayProjects.filter(p => {
                const pStatus = String(p.상태 || "").replace(/\s/g, "") || "계획전";
                return pStatus === status.replace(/\s/g, "");
            });
            return `<div class="k-column" style="background:rgba(245,234,224,0.3); border:2px solid rgba(230,171,169,0.4); border-radius:12px; padding:15px;">
                <div style="font-size:0.7rem; font-weight:800; color:#8a81a3; margin-bottom:20px; display:flex; justify-content:space-between;">
                    <span>${status}</span>
                    <span style="background:${ROSEWATER}; color:white; padding:2px 8px; border-radius:100px;">${cards.length}</span>
                </div>
                ${cards.map(p => makeCard(p)).join("")}
            </div>`;
        }).join("");
        return;
    }

    // ── 목표별 탭 ─────────────────────────────────────────
    if (target === "bygoal") {
        const activeProjects = filtered.filter(p =>
            !archiveStatuses.includes(String(p.상태 || "").replace(/\s/g, ""))
        );
        const goalMap = {};
        activeProjects.forEach(p => {
            const g = String(p.목표 || "미지정").trim();
            if (!goalMap[g]) goalMap[g] = [];
            goalMap[g].push(p);
        });
        const goals = Object.keys(goalMap);
        display.style.display = "grid";
        display.style.gridTemplateColumns = `repeat(${Math.min(goals.length, 4)}, 1fr)`;
        display.innerHTML = goals.map(g =>
            `<div style="background:rgba(245,234,224,0.3); border:2px solid rgba(230,171,169,0.4); border-radius:12px; padding:15px;">
                <div style="font-size:0.7rem; font-weight:800; color:#8a81a3; margin-bottom:16px; display:flex; justify-content:space-between;">
                    <span>${g}</span>
                    <span style="background:${ROSEWATER}; color:white; padding:2px 8px; border-radius:100px;">${goalMap[g].length}</span>
                </div>
                ${goalMap[g].map(p => makeCard(p)).join("")}
            </div>`
        ).join("");
        return;
    }

    // ── 전체 / 아카이브 탭 ───────────────────────────────
    const currentStatuses = target === "archive" ? archiveStatuses : activeStatuses;
    display.style.display = "grid";
    display.style.gridTemplateColumns = `repeat(${currentStatuses.length}, 1fr)`;
    display.innerHTML = currentStatuses.map(status => {
        const cards = filtered.filter(p => {
            const pStatus = String(p.상태 || "").replace(/\s/g, "") || "계획전";
            return pStatus === status.replace(/\s/g, "");
        });
        return `<div class="k-column" style="background:rgba(245,234,224,0.3); border:2px solid rgba(230,171,169,0.4); border-radius:12px; padding:15px;">
            <div style="font-size:0.7rem; font-weight:800; color:#8a81a3; margin-bottom:20px; display:flex; justify-content:space-between;">
                <span>${status}</span>
                <span style="background:${ROSEWATER}; color:white; padding:2px 8px; border-radius:100px;">${cards.length}</span>
            </div>
            ${cards.map(p => makeCard(p)).join("")}
        </div>`;
    }).join("");
};

setTimeout(() => {
    const root = document.getElementById(FIXED_ID);
    if (root) {
        const tabs = root.querySelectorAll(".ntab");
        tabs.forEach(tab => {
            tab.addEventListener("click", () => {
                tabs.forEach(t => { t.classList.remove("active"); t.style.color = "#8a81a3"; t.style.fontWeight = "500"; });
                tab.classList.add("active");
                tab.style.color = ROSEWATER;
                tab.style.fontWeight = "700";
                renderK(tab.dataset.target);
            });
        });
        renderK("all");
    }
}, 150);
```
---

```dataviewjs
const GOAL_ID = "goal-kanban-" + Math.random().toString(36).substring(2, 7);
const ROSEWATER_G = "#e6aba9";
const allGoals   = dv.pages('"03. Goals"').where(g => g.상태 !== "아카이브").array();
const allProjects = dv.pages('"04. Projects"').array();
const allTasks    = dv.pages('"05. Tasks"').array();

// 목표 달성률: 연결된 프로젝트들의 task 완료율 평균
const goalRate = (goalName) => {
    const projs = allProjects.filter(p => String(p.목표 || "").trim() === goalName);
    if (projs.length === 0) return 0;
    let sum = 0;
    projs.forEach(p => {
        const t = allTasks.filter(t => {
            const prop = t.프로젝트;
            if (!prop || prop === "") return false;
            const name = (typeof prop === 'object' && prop.path)
                ? prop.path.split('/').pop().replace(/\.md$/, '')
                : String(prop).replace(/[\[\]"]/g, "").trim();
            return name === p.file.name;
        });
        if (t.length > 0) sum += (t.filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true").length / t.length) * 100;
    });
    return Math.round(sum / projs.length);
};

// 목표 카드 HTML
const makeGoalCard = (g) => {
    const goalName = String(g.제목 || g.file.name).trim();

    // 연결된 프로젝트
    const linkedProjects = allProjects.filter(p =>
        String(p.목표 || "").trim() === goalName || String(p.목표 || "").trim() === g.file.name
    );
    const totalProjects   = linkedProjects.length;
    const doneProjects    = linkedProjects.filter(p => String(p.상태 || "").trim() === "완료").length;
    const remainProjects  = totalProjects - doneProjects;
    const activeProjects  = linkedProjects.filter(p => ["진행 중","집중"].includes(String(p.상태 || "").trim()));

    // 연결된 전체 tasks
    const linkedTasks = allTasks.filter(t => {
        const prop = t.프로젝트;
        if (!prop || prop === "") return false;
        const name = (typeof prop === 'object' && prop.path)
            ? prop.path.split('/').pop().replace(/\.md$/, '')
            : String(prop).replace(/[\[\]"]/g, "").trim();
        return linkedProjects.some(p => p.file.name === name);
    });
    const totalTasks   = linkedTasks.length;
    const doneTasks    = linkedTasks.filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true").length;
    const remainTasks  = totalTasks - doneTasks;
    const rate         = totalTasks > 0 ? Math.round((doneTasks / totalTasks) * 100) : 0;

    // D-day
    let dDayText = "기한 미정";
    if (g["목표 달성일"]) {
        const eDate = dv.date(g["목표 달성일"]);
        if (eDate) {
            const diff = Math.ceil(eDate.diff(dv.date("today"), 'days').days);
            dDayText = diff === 0 ? "D-Day" : (diff > 0 ? `D-${diff}` : `D+${Math.abs(diff)}`);
        }
    }

    // 진행중 프로젝트 목록
    const activeProjHTML = activeProjects.length > 0
        ? activeProjects.map(p =>
            `<div style="font-size:0.65rem; padding:3px 6px; margin-bottom:3px; border-left:2px solid ${ROSEWATER_G}; background:rgba(230,171,169,0.08); border-radius:3px;">
                <a class="internal-link" href="${p.file.path}" style="color:var(--text-normal); text-decoration:none;">${p.file.name}</a>
            </div>`
          ).join("")
        : `<div style="font-size:0.65rem; color:var(--text-faint);">현재 진행중인 프로젝트가 없습니다.</div>`;

    return `<div style="background:var(--background-primary); border:1px solid rgba(0,0,0,0.1); border-radius:10px; padding:16px; margin-bottom:15px;">

        <a class="internal-link" href="${g.file.path}" style="font-size:0.85rem; font-weight:700; color:var(--text-normal); text-decoration:none; display:block; margin-bottom:8px;">${goalName}</a>

        <div style="display:inline-block; background:rgba(67,123,255,0.1); color:#437bff; font-size:0.65rem; font-weight:800; padding:2px 6px; border-radius:4px; margin-bottom:10px;">${dDayText}</div>

        <div style="display:flex; align-items:center; gap:8px; margin-bottom:12px;">
            <span style="font-size:0.7rem; font-weight:700; color:#8a81a3; width:30px;">${rate}%</span>
            <div style="flex-grow:1; background:#eee; height:5px; border-radius:10px; overflow:hidden;">
                <div style="width:${rate}%; background:${ROSEWATER_G}; height:100%;"></div>
            </div>
        </div>

        <div style="font-size:0.65rem; color:#666; margin-bottom:10px; line-height:1.8;">
            <div style="font-weight:700; color:#8a81a3; margin-bottom:4px;">지금까지</div>
            총 프로젝트: ${totalProjects}<br>
            완료된 프로젝트: ${doneProjects}<br>
            남은 프로젝트: ${remainProjects}
        </div>

        <div style="font-size:0.65rem; margin-bottom:10px;">
            <div style="font-weight:700; color:#8a81a3; margin-bottom:6px;">진행중인 프로젝트</div>
            ${activeProjHTML}
        </div>

        <div style="font-size:0.65rem; color:#666; font-weight:500; margin-bottom:10px;">
            총 작업: ${totalTasks} | 남은 작업: ${remainTasks} | 달성률: ${rate}%
        </div>

        <div style="display:flex; gap:6px; flex-wrap:wrap;">
            ${g.박스 ? `<span style="font-size:0.6rem; background:rgba(230,171,169,0.15); border:1px solid rgba(230,171,169,0.4); color:#8a81a3; padding:2px 7px; border-radius:100px;">${g.박스}</span>` : ""}
            ${g.분기 ? `<span style="font-size:0.6rem; background:rgba(230,171,169,0.15); border:1px solid rgba(230,171,169,0.4); color:#8a81a3; padding:2px 7px; border-radius:100px;">${g.분기}</span>` : ""}
        </div>
    </div>`;
};

// 칸반 컬럼 HTML
const makeColumn = (label, items) =>
    `<div style="background:rgba(245,234,224,0.3); border:2px solid rgba(230,171,169,0.4); border-radius:12px; padding:15px;">
        <div style="font-size:0.7rem; font-weight:800; color:#8a81a3; margin-bottom:20px; display:flex; justify-content:space-between;">
            <span>${label}</span>
            <span style="background:${ROSEWATER_G}; color:white; padding:2px 8px; border-radius:100px;">${items.length}</span>
        </div>
        ${items.length > 0 ? items.map(g => makeGoalCard(g)).join("") : `<div style="font-size:0.7rem; color:var(--text-faint); text-align:center; padding:20px 0;">비어있음</div>`}
    </div>`;

// UI
let html = `<div id="${GOAL_ID}" style="font-family:var(--font-interface); padding:10px 0;">
    <div style="font-weight:800; font-size:1.3rem; color:#d6827d; letter-spacing:0.06em; margin-top:16px; margin-bottom:14px;">목표</div>
    <div style="display:flex; gap:20px; border-bottom:1px solid rgba(0,0,0,0.08); padding-bottom:0; margin-bottom:25px;">
        <div class="gtab active" data-target="all"      style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:700; color:${ROSEWATER_G};">전체</div>
        <div class="gtab"        data-target="active"   style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">달성중</div>
        <div class="gtab"        data-target="byquarter" style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">분기별</div>
        <div class="gtab"        data-target="bybox"    style="cursor:pointer; padding:8px 4px; font-size:0.85rem; font-weight:500; color:#8a81a3;">박스별</div>
    </div>
    <div id="${GOAL_ID}-area" style="display:grid; gap:18px; overflow-x:auto; padding-bottom:15px;"></div>
</div>`;

dv.el("div", html);

const renderG = (target) => {
    const root = document.getElementById(GOAL_ID);
    if (!root) return;
    const area = root.querySelector(`#${GOAL_ID}-area`);

    // ── 전체 탭 (진행 중 / 집중 / 시작 전 / 완료 컬럼) ────
    if (target === "all") {
        const allCols = ["진행 중", "집중", "시작 전", "완료"];
        area.style.gridTemplateColumns = `repeat(${allCols.length}, 1fr)`;
        area.innerHTML = allCols.map(s =>
            makeColumn(s, allGoals.filter(g => String(g.상태 || "").trim() === s))
        ).join("");
        return;
    }

    // ── 달성중 탭 (진행 중 / 집중 컬럼만) ─────────────────
    if (target === "active") {
        const activeCols = ["진행 중", "집중"];
        area.style.gridTemplateColumns = `repeat(${activeCols.length}, 1fr)`;
        area.innerHTML = activeCols.map(s =>
            makeColumn(s, allGoals.filter(g => String(g.상태 || "").trim() === s))
        ).join("");
        return;
    }

    // ── 분기별 탭 ──────────────────────────────────────────
    if (target === "byquarter") {
        const qMap = {};
        allGoals.forEach(g => {
            const key = String(g.분기 || "미정").trim();
            if (!qMap[key]) qMap[key] = [];
            qMap[key].push(g);
        });
        const keys = Object.keys(qMap).sort();
        area.style.gridTemplateColumns = `repeat(${Math.min(keys.length, 4)}, 1fr)`;
        area.innerHTML = keys.map(k => makeColumn(k, qMap[k])).join("");
        return;
    }

    // ── 박스별 탭 ──────────────────────────────────────────
    if (target === "bybox") {
        const bMap = {};
        allGoals.forEach(g => {
            const key = String(g.박스 || "미지정").trim();
            if (!bMap[key]) bMap[key] = [];
            bMap[key].push(g);
        });
        const keys = Object.keys(bMap).sort();
        area.style.gridTemplateColumns = `repeat(${Math.min(keys.length, 4)}, 1fr)`;
        area.innerHTML = keys.map(k => makeColumn(k, bMap[k])).join("");
        return;
    }
};

setTimeout(() => {
    const root = document.getElementById(GOAL_ID);
    if (!root) return;
    const tabs = root.querySelectorAll(".gtab");
    tabs.forEach(tab => {
        tab.addEventListener("click", () => {
            tabs.forEach(t => { t.style.color = "#8a81a3"; t.style.fontWeight = "500"; });
            tab.style.color = ROSEWATER_G;
            tab.style.fontWeight = "700";
            renderG(tab.dataset.target);
        });
    });
    renderG("all");
}, 200);
```

---

```dataviewjs
dv.el("div", `<div style="font-weight: 800; font-size: 1.3rem; color: #d6827d; letter-spacing: 0.06em; margin-top: 16px; margin-bottom: 14px;">자료</div>`);
```

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

```dataviewjs
const pages = dv.pages('"02. Boxes"').where(b => b.type === "box");

let html = `<div style="font-weight: 800; font-size: 1.3rem; color: #d6827d; letter-spacing: 0.06em; margin-top: 16px; margin-bottom: 14px;">박스</div>`;
html += `<div class="sb-gallery">`;
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
