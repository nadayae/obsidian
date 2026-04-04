---
type: project
fileClass: project
제목: 무제 1
박스: ""
목표: ""
상태:
달성률: 0
목표일: ""
url: ""
시작일: 2026-04-04
---

```dataviewjs
const ROSE = "#d6827d";
const p = dv.current();
const projectName = p.file.name;

function resolveName(val) {
    if (!val) return "-";
    if (typeof val === "object" && val.path)
        return val.path.split("/").pop().replace(/\.md$/, "");
    return String(val).replace(/\[/g,"").replace(/\]/g,"").replace(/"/g,"").trim() || "-";
}

const allTasks = dv.pages('"05. Tasks"').filter(t => {
    const prop = t.프로젝트;
    if (!prop) return false;
    return resolveName(prop) === projectName;
});

const total = allTasks.length;
const done = allTasks.filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true").length;
const rate = total > 0 ? Math.round((done / total) * 100) : 0;

const 상태 = String(p.상태 || "-").trim();
const 박스 = resolveName(p.박스);
const 목표 = resolveName(p.목표);
const 시작일 = p.시작일 ? String(p.시작일).substring(0, 10) : "-";
const 목표일 = p.목표일 ? String(p.목표일).substring(0, 10) : "-";

const statusColorMap = { "진행중": "#4caf7d", "집중": ROSE, "완료": "#8a81a3", "보류": "#f0a500", "계획전": "#aaa", "계획 전": "#aaa" };
const statusColor = statusColorMap[상태.replace(/\s/g,"")] || "#aaa";

const _infoWrap = dv.el("div", "");
_infoWrap.innerHTML = `
<div style="font-family:var(--font-interface); padding:10px 0 20px;">
    <div style="font-weight:800; font-size:1.5rem; color:var(--text-normal); letter-spacing:0.02em; margin-bottom:20px;">${projectName}</div>
    <div style="display:grid; grid-template-columns:repeat(auto-fill, minmax(150px, 1fr)); gap:10px; margin-bottom:14px;">
        <div style="background:var(--background-secondary); border-radius:10px; padding:12px;">
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">상태</div>
            <div style="font-size:0.8rem; font-weight:800; color:${statusColor};">${상태}</div>
        </div>
        <div style="background:var(--background-secondary); border-radius:10px; padding:12px;">
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">박스</div>
            <div style="font-size:0.78rem; font-weight:600; color:var(--text-normal); white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">${박스}</div>
        </div>
        <div style="background:var(--background-secondary); border-radius:10px; padding:12px;">
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">목표</div>
            <div style="font-size:0.78rem; font-weight:600; color:var(--text-normal); white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">${목표}</div>
        </div>
        <div style="background:var(--background-secondary); border-radius:10px; padding:12px;">
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">기간</div>
            <div style="font-size:0.72rem; font-weight:600; color:var(--text-normal);">${시작일}<br><span style="color:var(--text-faint);">~ ${목표일}</span></div>
        </div>
    </div>
    <div style="background:var(--background-secondary); border-radius:10px; padding:14px 16px;">
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:8px;">
            <div style="font-size:0.62rem; font-weight:700; color:var(--text-faint); letter-spacing:0.04em;">달성률</div>
            <div style="font-size:0.78rem; font-weight:800; color:${ROSE};">${rate}% <span style="font-size:0.62rem; color:var(--text-faint); font-weight:400;">(${done} / ${total})</span></div>
        </div>
        <div style="background:rgba(0,0,0,0.08); border-radius:100px; height:5px; overflow:hidden;">
            <div style="background:${ROSE}; width:${rate}%; height:100%; border-radius:100px;"></div>
        </div>
    </div>
</div>
`;
```

```dataviewjs
const UID = "ptask-" + Math.random().toString(36).substring(2, 7);
const ROSE = "#d6827d";
const projectName = dv.current().file.name;

function resolveName(val) {
    if (!val) return "";
    if (typeof val === "object" && val.path)
        return val.path.split("/").pop().replace(/\.md$/, "");
    return String(val).replace(/\[/g,"").replace(/\]/g,"").replace(/"/g,"").trim();
}

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

const allProjTasks = dv.pages('"05. Tasks"').filter(t => {
    const prop = t.프로젝트;
    if (!prop) return false;
    return resolveName(prop) === projectName;
}).array();

const pendingTasks = allProjTasks
    .filter(t => !(t.완료여부 === true || String(t.완료여부).toLowerCase() === "true"))
    .sort((a, b) => sortLogic(a).localeCompare(sortLogic(b)));

const doneTasks = allProjTasks
    .filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true")
    .sort((a, b) => b.file.mtime - a.file.mtime);

function makeTaskRow(t, isDone) {
    const catColors = { "집중": ROSE, "일반": "#4caf7d", "쉬운": "#8a81a3", "위임": "#f0a500", "일정": "#437bff", "나중에": "#999" };
    const catColor = catColors[String(t.구분 || "").trim()] || "var(--text-faint)";
    const title = t.제목 || t.file.name;
    const dateStr = t.날짜 ? String(t.날짜).substring(0, 10) : "-";
    const timeStr = t.계획 || "-";
    return `<div style="display:grid; grid-template-columns:2fr 0.6fr 0.6fr 0.9fr; gap:8px; align-items:center;
        padding:8px 10px; border-radius:8px; margin-bottom:4px; background:var(--background-secondary);
        opacity:${isDone ? "0.5" : "1"}; ${isDone ? "text-decoration:line-through;" : ""}">
        <div style="font-size:0.72rem; font-weight:600; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;">
            <a class="internal-link" href="${t.file.path}" style="color:var(--text-normal); text-decoration:none;">${title}</a>
        </div>
        <div style="font-size:0.62rem; font-weight:700; color:${catColor};">${t.구분 || "-"}</div>
        <div style="font-size:0.62rem; color:var(--text-faint);">${timeStr}</div>
        <div style="font-size:0.62rem; color:var(--text-faint);">${dateStr}</div>
    </div>`;
}

const colHeader = `<div style="display:grid; grid-template-columns:2fr 0.6fr 0.6fr 0.9fr; gap:8px;
    padding:2px 10px; margin-bottom:6px;">
    ${["제목","구분","계획","날짜"].map(h =>
        `<div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.04em;">${h}</div>`
    ).join("")}
</div>`;

const html = `
<div id="${UID}" style="font-family:var(--font-interface); padding:10px 0;">
    <div id="${UID}-toggle" style="display:flex; justify-content:space-between; align-items:center;
        cursor:pointer; padding:12px 0; border-bottom:1px solid rgba(0,0,0,0.07); user-select:none;">
        <div style="font-weight:800; font-size:1.3rem; color:${ROSE}; letter-spacing:0.06em;">할일</div>
        <div style="display:flex; align-items:center; gap:10px;">
            <span style="font-size:0.62rem; color:var(--text-faint);">${pendingTasks.length}개 진행중 · ${doneTasks.length}개 완료</span>
            <span id="${UID}-chevron" style="font-size:0.7rem; color:${ROSE}; transition:transform 0.25s; display:inline-block;">▼</span>
        </div>
    </div>
    <div id="${UID}-body" style="padding-top:16px;">
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
            <div style="display:flex; gap:0; border-radius:8px; overflow:hidden; background:var(--background-secondary);">
                <div class="${UID}-stab" data-panel="${UID}-pending-panel"
                    style="cursor:pointer; font-size:0.72rem; font-weight:700; color:var(--background-primary);
                    background:${ROSE}; padding:5px 14px;">
                    할일 <span style="opacity:0.7;">(${pendingTasks.length})</span>
                </div>
                <div class="${UID}-stab" data-panel="${UID}-done-panel"
                    style="cursor:pointer; font-size:0.72rem; font-weight:500; color:var(--text-muted);
                    background:transparent; padding:5px 14px;">
                    완료한 일 <span style="opacity:0.7;">(${doneTasks.length})</span>
                </div>
            </div>
            <div style="display:flex; gap:6px;">
                <button id="${UID}-add-btn"
                    style="background:rgba(214,130,125,0.15); border:1px solid rgba(214,130,125,0.4);
                    border-radius:6px; padding:4px 12px; font-size:0.65rem; font-weight:700;
                    color:${ROSE}; cursor:pointer;">+ 추가</button>
                <button id="${UID}-cal-btn"
                    style="background:rgba(214,130,125,0.15); border:1px solid rgba(214,130,125,0.4);
                    border-radius:6px; padding:4px 12px; font-size:0.65rem; font-weight:700;
                    color:${ROSE}; cursor:pointer;">📅 계획</button>
            </div>
        </div>
        ${colHeader}
        <div id="${UID}-pending-panel">
            ${pendingTasks.length > 0
                ? pendingTasks.map(t => makeTaskRow(t, false)).join("")
                : `<div style="padding:36px; text-align:center; font-size:0.8rem; color:var(--text-faint);">진행 중인 할 일이 없습니다 ✅</div>`}
        </div>
        <div id="${UID}-done-panel" style="display:none;">
            ${doneTasks.length > 0
                ? doneTasks.map(t => makeTaskRow(t, true)).join("")
                : `<div style="padding:36px; text-align:center; font-size:0.8rem; color:var(--text-faint);">완료된 할 일이 없습니다</div>`}
        </div>
    </div>
</div>
`;

const _wrap = dv.el("div", "");
_wrap.innerHTML = html;
const _root = _wrap.querySelector("#" + UID);

setTimeout(() => {
    const root = _root || dv.container.querySelector("#" + UID);
    if (!root) return;

    const toggleBtn = root.querySelector("#" + UID + "-toggle");
    const body = root.querySelector("#" + UID + "-body");
    const chevron = root.querySelector("#" + UID + "-chevron");
    if (toggleBtn && body && chevron) {
        toggleBtn.addEventListener("click", () => {
            const isOpen = body.style.display !== "none";
            body.style.display = isOpen ? "none" : "block";
            chevron.style.transform = isOpen ? "rotate(-90deg)" : "rotate(0deg)";
        });
    }

    const addBtn = root.querySelector("#" + UID + "-add-btn");
    if (addBtn) {
        addBtn.addEventListener("click", (e) => {
            e.stopPropagation();
            app.commands.executeCommandById("quickadd:choice:sb-new-task");
        });
    }

    const calBtn = root.querySelector("#" + UID + "-cal-btn");
    if (calBtn) {
        calBtn.addEventListener("click", (e) => {
            e.stopPropagation();
            app.commands.executeCommandById("obsidian-full-calendar:full-calendar-open");
        });
    }

    const subTabs = root.querySelectorAll("." + UID + "-stab");
    subTabs.forEach(tab => {
        tab.addEventListener("click", () => {
            subTabs.forEach(t => {
                t.style.background = "transparent";
                t.style.color = "var(--text-muted)";
                t.style.fontWeight = "500";
            });
            tab.style.background = ROSE;
            tab.style.color = "var(--background-primary)";
            tab.style.fontWeight = "700";
            subTabs.forEach(t => {
                const panelId = t.dataset.panel;
                const panel = root.querySelector("#" + panelId);
                if (panel) panel.style.display = (panelId === tab.dataset.panel) ? "block" : "none";
            });
        });
    });
}, 200);
```


```dataviewjs
const UID = "dvcal-" + Math.random().toString(36).substring(2, 7);
const ROSE = "#d6827d";
const projectName = dv.current().file.name;

function resolveName(val) {
    if (!val) return "";
    if (typeof val === "object" && val.path)
        return val.path.split("/").pop().replace(/\.md$/, "");
    return String(val).replace(/\[/g,"").replace(/\]/g,"").replace(/"/g,"").trim();
}

const catColors = { "집중": ROSE, "일반": "#4caf7d", "쉬운": "#8a81a3", "위임": "#f0a500", "일정": "#437bff", "나중에": "#999" };

const allProjTasks = dv.pages('"05. Tasks"').filter(t => {
    const prop = t.프로젝트;
    if (!prop) return false;
    return resolveName(prop) === projectName;
}).filter(t => !(t.완료여부 === true || String(t.완료여부).toLowerCase() === "true")).array();

// 인메모리 상태 (드롭 시 변경됨)
const taskState = allProjTasks.map(t => ({
    path: t.file.path,
    name: t.제목 || t.file.name,
    date: t.날짜 ? String(t.날짜).substring(0, 10) : null,
    구분: String(t.구분 || "").trim()
}));

const today = new Date();
let viewYear = today.getFullYear();
let viewMonth = today.getMonth();
const todayStr = `${today.getFullYear()}-${String(today.getMonth()+1).padStart(2,"0")}-${String(today.getDate()).padStart(2,"0")}`;

const _wrap = dv.el("div", "");
_wrap.innerHTML = `
<div id="${UID}" style="font-family:var(--font-interface); padding:10px 0;">
    <div style="font-weight:800; font-size:1.3rem; color:${ROSE}; letter-spacing:0.06em;
        padding:12px 0; border-bottom:1px solid rgba(0,0,0,0.07); margin-bottom:16px;">계획</div>
    <div style="display:grid; grid-template-columns:190px 1fr; gap:16px; align-items:start;">

        <!-- 왼쪽: 미배정 할일 -->
        <div>
            <div style="font-size:0.6rem; font-weight:700; color:var(--text-faint);
                letter-spacing:0.05em; margin-bottom:8px;">미배정 할일 — 날짜 셀에 드래그</div>
            <div id="${UID}-unscheduled" style="display:flex; flex-direction:column; gap:6px; min-height:60px;
                border:1.5px dashed rgba(214,130,125,0.25); border-radius:10px; padding:8px;"></div>
        </div>

        <!-- 오른쪽: 캘린더 -->
        <div>
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
                <button id="${UID}-prev" style="background:none; border:1px solid rgba(214,130,125,0.3);
                    border-radius:6px; padding:2px 10px; cursor:pointer; font-size:0.85rem; color:${ROSE};">‹</button>
                <div id="${UID}-month-label" style="font-size:0.85rem; font-weight:700; color:var(--text-normal);"></div>
                <button id="${UID}-next" style="background:none; border:1px solid rgba(214,130,125,0.3);
                    border-radius:6px; padding:2px 10px; cursor:pointer; font-size:0.85rem; color:${ROSE};">›</button>
            </div>
            <div id="${UID}-cal-grid"></div>
        </div>
    </div>
</div>
`;

const root = _wrap.querySelector("#" + UID);
const unscheduledEl = root.querySelector("#" + UID + "-unscheduled");
const calGrid = root.querySelector("#" + UID + "-cal-grid");
const monthLabel = root.querySelector("#" + UID + "-month-label");

function renderUnscheduled() {
    const tasks = taskState.filter(t => !t.date);
    if (tasks.length === 0) {
        unscheduledEl.innerHTML = `<div style="font-size:0.7rem; color:var(--text-faint); padding:10px; text-align:center;">모두 배정됨 ✅</div>`;
        return;
    }
    unscheduledEl.innerHTML = tasks.map(t => `
        <div class="${UID}-task-card" draggable="true" data-path="${t.path}" data-name="${t.name}"
            style="background:var(--background-secondary); border:1px solid rgba(214,130,125,0.25);
            border-radius:8px; padding:7px 10px; cursor:grab; user-select:none;">
            <div style="font-size:0.7rem; font-weight:600; color:var(--text-normal);
                white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">${t.name}</div>
            ${t.구분 ? `<div style="font-size:0.58rem; font-weight:700; color:${catColors[t.구분] || "var(--text-faint)"}; margin-top:2px;">${t.구분}</div>` : ""}
        </div>
    `).join("");

    unscheduledEl.querySelectorAll("." + UID + "-task-card").forEach(card => {
        card.addEventListener("dragstart", e => {
            e.dataTransfer.setData("text/plain", JSON.stringify({ path: card.dataset.path, name: card.dataset.name }));
            setTimeout(() => card.style.opacity = "0.35", 0);
        });
        card.addEventListener("dragend", () => { card.style.opacity = "1"; });
    });
}

function renderCalendar() {
    const monthNames = ["1월","2월","3월","4월","5월","6월","7월","8월","9월","10월","11월","12월"];
    monthLabel.textContent = `${viewYear}년 ${monthNames[viewMonth]}`;

    const firstDay = new Date(viewYear, viewMonth, 1).getDay();
    const daysInMonth = new Date(viewYear, viewMonth + 1, 0).getDate();
    const monthStr = `${viewYear}-${String(viewMonth+1).padStart(2,"0")}`;
    const monthTasks = taskState.filter(t => t.date && t.date.startsWith(monthStr));

    let html = `<div style="display:grid; grid-template-columns:repeat(7,1fr); gap:3px;">`;

    ["일","월","화","수","목","금","토"].forEach((d, i) => {
        html += `<div style="text-align:center; font-size:0.58rem; font-weight:700; padding:4px 0;
            color:${i===0?"#e64553":i===6?"#437bff":"var(--text-faint)"};">${d}</div>`;
    });

    for (let i = 0; i < firstDay; i++) html += `<div></div>`;

    for (let d = 1; d <= daysInMonth; d++) {
        const dateStr = `${viewYear}-${String(viewMonth+1).padStart(2,"0")}-${String(d).padStart(2,"0")}`;
        const isToday = dateStr === todayStr;
        const dow = (firstDay + d - 1) % 7;
        const isSun = dow === 0, isSat = dow === 6;
        const dayTasks = monthTasks.filter(t => t.date === dateStr);

        html += `<div class="${UID}-cal-cell" data-date="${dateStr}"
            style="min-height:58px; padding:4px 3px; border-radius:7px;
            border:1.5px solid ${isToday ? ROSE : "rgba(0,0,0,0.06)"};
            background:${isToday ? "rgba(214,130,125,0.07)" : "var(--background-primary)"};
            transition:background 0.12s; box-sizing:border-box;">
            <div style="font-size:0.6rem; font-weight:${isToday?"800":"500"}; text-align:right; margin-bottom:2px;
                color:${isToday?ROSE:isSun?"#e64553":isSat?"#437bff":"var(--text-muted)"};">${d}</div>
            ${dayTasks.map(t => `
                <div class="${UID}-cal-task" draggable="true" data-path="${t.path}" data-name="${t.name}"
                    style="font-size:0.55rem; font-weight:700; padding:2px 4px; border-radius:4px; margin-bottom:2px;
                    background:rgba(214,130,125,0.18); color:${ROSE};
                    white-space:nowrap; overflow:hidden; text-overflow:ellipsis; cursor:grab;">${t.name}</div>
            `).join("")}
        </div>`;
    }
    html += `</div>`;
    calGrid.innerHTML = html;

    // 셀 드롭 이벤트
    calGrid.querySelectorAll("." + UID + "-cal-cell").forEach(cell => {
        const dateStr = cell.dataset.date;
        const isToday = dateStr === todayStr;

        cell.addEventListener("dragover", e => {
            e.preventDefault();
            cell.style.background = "rgba(214,130,125,0.18)";
        });
        cell.addEventListener("dragleave", () => {
            cell.style.background = isToday ? "rgba(214,130,125,0.07)" : "var(--background-primary)";
        });
        cell.addEventListener("drop", async e => {
            e.preventDefault();
            cell.style.background = isToday ? "rgba(214,130,125,0.07)" : "var(--background-primary)";
            let data;
            try { data = JSON.parse(e.dataTransfer.getData("text/plain")); } catch { return; }

            const task = taskState.find(t => t.path === data.path);
            if (!task) return;
            task.date = dateStr;

            const file = app.vault.getAbstractFileByPath(data.path);
            if (file) {
                await app.fileManager.processFrontMatter(file, fm => { fm["날짜"] = dateStr; });
            }

            renderUnscheduled();
            renderCalendar();
        });
    });

    // 캘린더 안 할일 → 다른 날로 재배정 드래그
    calGrid.querySelectorAll("." + UID + "-cal-task").forEach(taskEl => {
        taskEl.addEventListener("dragstart", e => {
            e.stopPropagation();
            e.dataTransfer.setData("text/plain", JSON.stringify({ path: taskEl.dataset.path, name: taskEl.dataset.name }));
            setTimeout(() => taskEl.style.opacity = "0.35", 0);
        });
        taskEl.addEventListener("dragend", () => { taskEl.style.opacity = "1"; });
    });
}

setTimeout(() => {
    renderUnscheduled();
    renderCalendar();

    root.querySelector("#" + UID + "-prev").addEventListener("click", () => {
        viewMonth--;
        if (viewMonth < 0) { viewMonth = 11; viewYear--; }
        renderCalendar();
    });
    root.querySelector("#" + UID + "-next").addEventListener("click", () => {
        viewMonth++;
        if (viewMonth > 11) { viewMonth = 0; viewYear++; }
        renderCalendar();
    });
}, 200);
```

## 목적 & 범위

*이 프로젝트를 통해 무엇을 달성하려 하는지, 어디까지가 범위인지 정의해주세요.*

---

## 작업 기록

*작업하면서 생긴 결정사항, 배운 것, 메모를 남겨두세요.*
