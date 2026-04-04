---
type: goal
fileClass: goal
제목: <% tp.file.title %>
박스: ""
년도: <% tp.date.now("YYYY") %>
분기:
상태:
달성률: 0
목표 달성일: <% tp.date.now("YYYY") %>-03-31
생성일: <% tp.date.now("YYYY-MM-DD") %>
---

```dataviewjs
(async () => {
const ROSE = "#d6827d";
const g = dv.current();
if (!g || !g.file) return;
const goalName = g.file.name;

function resolveName(val) {
    if (!val) return "-";
    if (typeof val === "object" && val.path)
        return val.path.split("/").pop().replace(/\.md$/, "");
    return String(val).replace(/\[/g,"").replace(/\]/g,"").replace(/"/g,"").trim() || "-";
}

const allProjects = dv.pages('"04. Projects"').filter(p => resolveName(p.목표) === goalName).array();
const allTasks = dv.pages('"05. Tasks"');

const totalProj = allProjects.length;
const doneProj = allProjects.filter(p => String(p.상태 || "").trim() === "완료").length;

let rateSum = 0;
allProjects.forEach(p => {
    const t = allTasks.filter(t => resolveName(t.프로젝트) === p.file.name);
    const total = t.length;
    const done = t.filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true").length;
    rateSum += total > 0 ? (done / total) * 100 : 0;
});
const rate = totalProj > 0 ? Math.round(rateSum / totalProj) : 0;

const 상태 = String(g.상태 || "-").trim();
const 박스 = resolveName(g.박스);
const 분기 = String(g.분기 || "-").trim();
const 년도 = String(g.년도 || "-").trim();
const 달성일 = g["목표 달성일"] ? String(g["목표 달성일"]).substring(0, 10) : "-";

const statusColorMap = { "진행 중": "#4caf7d", "집중": ROSE, "완료": "#8a81a3", "중단": "#aaa", "시작 전": "#aaa", "아카이브": "#8a81a3" };
const statusColor = statusColorMap[상태] || "#aaa";

const _wrap = dv.el("div", "");
_wrap.innerHTML = `
<div style="font-family:var(--font-interface); padding:10px 0 20px;">
    <div style="font-weight:800; font-size:1.5rem; color:var(--text-normal); letter-spacing:0.02em; margin-bottom:20px;">${goalName}</div>
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
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">기간</div>
            <div style="font-size:0.78rem; font-weight:600; color:var(--text-normal);">${년도} · ${분기}</div>
        </div>
        <div style="background:var(--background-secondary); border-radius:10px; padding:12px;">
            <div style="font-size:0.58rem; font-weight:700; color:var(--text-faint); letter-spacing:0.05em; margin-bottom:5px;">목표 달성일</div>
            <div style="font-size:0.78rem; font-weight:600; color:var(--text-normal);">${달성일}</div>
        </div>
    </div>
    <div style="background:var(--background-secondary); border-radius:10px; padding:14px 16px;">
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:8px;">
            <div style="font-size:0.62rem; font-weight:700; color:var(--text-faint); letter-spacing:0.04em;">달성률 (프로젝트 평균)</div>
            <div style="font-size:0.78rem; font-weight:800; color:${ROSE};">${rate}% <span style="font-size:0.62rem; color:var(--text-faint); font-weight:400;">(${doneProj} / ${totalProj} 완료)</span></div>
        </div>
        <div style="background:rgba(0,0,0,0.08); border-radius:100px; height:5px; overflow:hidden;">
            <div style="background:${ROSE}; width:${rate}%; height:100%; border-radius:100px;"></div>
        </div>
    </div>
</div>
`;
})();
```

```dataviewjs
(async () => {
const UID = "gproj-" + Math.random().toString(36).substring(2, 7);
const ROSE = "#d6827d";
const g = dv.current();
if (!g || !g.file) return;
const goalName = g.file.name;

function resolveName(val) {
    if (!val) return "";
    if (typeof val === "object" && val.path)
        return val.path.split("/").pop().replace(/\.md$/, "");
    return String(val).replace(/\[/g,"").replace(/\]/g,"").replace(/"/g,"").trim();
}

const allTasks = dv.pages('"05. Tasks"');

const allProjects = dv.pages('"04. Projects"').filter(p => resolveName(p.목표) === goalName).array();

const today = dv.date("today");

const activeStatuses = ["계획전", "계획", "진행중", "집중"];
const archiveStatuses = ["완료", "중단"];

const tabs = [
    { id: "all",     label: "전체" },
    { id: "active",  label: "진행중" },
    { id: "archive", label: "완료/중단" },
];

function getLinkedTasks(p) {
    return allTasks.filter(t => resolveName(t.프로젝트) === p.file.name);
}

function makeProjectCard(p) {
    const linkedTasks = getLinkedTasks(p);
    const total = linkedTasks.length;
    const done = linkedTasks.filter(t => t.완료여부 === true || String(t.완료여부).toLowerCase() === "true").length;
    const rate = total > 0 ? Math.round((done / total) * 100) : 0;
    const remaining = total - done;

    const eDate = p.목표일 ? dv.date(String(p.목표일).replace(/[\[\]]/g,"").trim()) : null;
    let dDayText = "D-Day 미정";
    let dDayColor = "#8a81a3";
    if (eDate) {
        const diff = Math.ceil(eDate.diff(today, "days").days);
        if (diff === 0) { dDayText = "D-Day"; dDayColor = ROSE; }
        else if (diff > 0) { dDayText = `D-${diff}`; dDayColor = "#437bff"; }
        else { dDayText = `D+${Math.abs(diff)}`; dDayColor = "#e64553"; }
    }

    const 상태 = String(p.상태 || "계획전").trim();
    const statusColorMap = { "진행중": "#4caf7d", "집중": ROSE, "완료": "#8a81a3", "중단": "#aaa", "계획전": "#aaa", "계획 전": "#aaa" };
    const statusColor = statusColorMap[상태.replace(/\s/g,"")] || "#aaa";

    const 시작일 = p.시작일 ? String(p.시작일).substring(0, 10) : "-";
    const 목표일 = p.목표일 ? String(p.목표일).substring(0, 10) : "-";

    return `<div style="background:var(--background-primary); border:1px solid rgba(0,0,0,0.08);
        border-radius:12px; padding:16px; display:flex; flex-direction:column; gap:10px;">
        <div style="display:flex; justify-content:space-between; align-items:flex-start;">
            <a class="internal-link" href="${p.file.path}"
                style="font-size:0.85rem; font-weight:800; color:var(--text-normal); text-decoration:none;
                line-height:1.3; flex:1; margin-right:8px;">${p.file.name}</a>
            <span style="font-size:0.6rem; font-weight:800; padding:2px 8px; border-radius:100px;
                background:rgba(0,0,0,0.05); color:${dDayColor}; white-space:nowrap;">${dDayText}</span>
        </div>
        <div style="display:flex; align-items:center; gap:6px;">
            <span style="font-size:0.6rem; font-weight:700; color:${statusColor}; min-width:32px;">${상태}</span>
            <div style="flex:1; background:rgba(0,0,0,0.08); border-radius:100px; height:4px; overflow:hidden;">
                <div style="width:${rate}%; background:${ROSE}; height:100%; border-radius:100px;"></div>
            </div>
            <span style="font-size:0.6rem; font-weight:700; color:${ROSE}; min-width:28px; text-align:right;">${rate}%</span>
        </div>
        <div style="font-size:0.6rem; color:var(--text-faint);">
            총 ${total}개 · 남은 할일 ${remaining}개
        </div>
        <div style="font-size:0.6rem; color:var(--text-faint); border-top:1px solid rgba(0,0,0,0.06); padding-top:8px;">
            ${시작일} ~ ${목표일}
        </div>
    </div>`;
}

function filterProjects(tabId) {
    if (tabId === "active")  return allProjects.filter(p => !archiveStatuses.includes(String(p.상태 || "").trim()));
    if (tabId === "archive") return allProjects.filter(p => archiveStatuses.includes(String(p.상태 || "").trim()));
    return allProjects;
}

const html = `
<div id="${UID}" style="font-family:var(--font-interface); padding:10px 0;">
    <div style="display:flex; justify-content:space-between; align-items:center;
        padding:12px 0; border-bottom:1px solid rgba(0,0,0,0.07); margin-bottom:16px;">
        <div style="font-weight:800; font-size:1.3rem; color:${ROSE}; letter-spacing:0.06em;">프로젝트</div>
        <button id="${UID}-add-btn" style="background:rgba(214,130,125,0.15); border:1px solid rgba(214,130,125,0.4);
            border-radius:6px; padding:4px 12px; font-size:0.65rem; font-weight:700;
            color:${ROSE}; cursor:pointer;">+ 추가</button>
    </div>
    <div style="display:flex; gap:0; border-radius:8px; overflow:hidden; background:var(--background-secondary);
        width:fit-content; margin-bottom:16px;">
        ${tabs.map((t, i) => `<div class="${UID}-tab" data-id="${t.id}"
            style="cursor:pointer; font-size:0.72rem; font-weight:${i===0?"700":"500"};
            color:${i===0?"var(--background-primary)":"var(--text-muted)"};
            background:${i===0?ROSE:"transparent"}; padding:5px 16px;">${t.label}</div>`).join("")}
    </div>
    <div id="${UID}-area" style="display:grid; grid-template-columns:repeat(auto-fill, minmax(260px, 1fr)); gap:14px;"></div>
</div>`;

const _wrap = dv.el("div", "");
_wrap.innerHTML = html;
const _root = _wrap.querySelector("#" + UID);

function renderProjects(tabId) {
    const area = _root.querySelector("#" + UID + "-area");
    const filtered = filterProjects(tabId);
    if (filtered.length === 0) {
        area.innerHTML = `<div style="grid-column:1/-1; padding:40px; text-align:center; font-size:0.8rem; color:var(--text-faint);">프로젝트가 없습니다</div>`;
        return;
    }
    area.innerHTML = filtered.map(p => makeProjectCard(p)).join("");
}

renderProjects("all");

setTimeout(() => {
    if (!_root) return;

    const addBtn = _root.querySelector("#" + UID + "-add-btn");
    if (addBtn) addBtn.addEventListener("click", () => {
        app.commands.executeCommandById("quickadd:choice:sb-new-project");
    });

    const tabEls = _root.querySelectorAll("." + UID + "-tab");
    tabEls.forEach(el => {
        el.addEventListener("click", () => {
            tabEls.forEach(t => {
                t.style.background = "transparent";
                t.style.color = "var(--text-muted)";
                t.style.fontWeight = "500";
            });
            el.style.background = ROSE;
            el.style.color = "var(--background-primary)";
            el.style.fontWeight = "700";
            renderProjects(el.dataset.id);
        });
    });
}, 200);
})();
```
