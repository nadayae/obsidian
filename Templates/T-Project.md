---
type: project
fileClass: project
제목: <% tp.file.title %>
박스: ""
목표: ""
상태:
달성률: 0
목표일: ""
url: ""
시작일: <% tp.date.now("YYYY-MM-DD") %>
---

# <% tp.file.title %>

> [!project] 프로젝트 정보
> **Box:** *(연결할 Box 링크)*
> **Goal:** *(연결할 Goal 링크)*
> **상태:** 시작 전
> **날짜:** *(마감일 입력)*

```dataviewjs
const tasks = dv.pages('"05. Tasks"').where(t => t.프로젝트 === dv.current().file.name || String(t.프로젝트) === dv.current().file.name);
const total = tasks.length;
const done = tasks.where(t => t.완료여부 === true).length;
const rate = total > 0 ? Math.round((done / total) * 100) : 0;
dv.el("div", `<progress value="${rate}" max="100" style="width:100%;"></progress><div style="font-size:0.8rem; margin-top:4px;">달성률 <b>${rate}%</b> (${done}/${total})</div>`);
```

---

## 목적 & 범위

*이 프로젝트를 통해 무엇을 달성하려 하는지, 어디까지가 범위인지 정의해주세요.*

---

## ➕ 이 프로젝트에 할 일 추가

```button
name ✅ 새 할 일 만들기
type command
action QuickAdd: New Task
color default
```

---

## 📋 할 일
```dataviewjs
// 1. 기본 설정 및 데이터 수집
const ROSEWATER = "#e6aba9";
const today = dv.date("today");
const projectName = dv.current().file.name;
const TASKS_PATH = '"05. Tasks"';

// 현재 프로젝트와 연결된 할 일 데이터 수집
const allTasks = dv.pages(TASKS_PATH).filter(t => {
    const prop = t.프로젝트;
    if (!prop) return false;
    const nameOnly = prop.path ? prop.fileName : String(prop).replace(/[\[\]]/g, "").trim();
    return nameOnly === projectName;
}).array();

// 2. 전체 레이아웃 (DOM 요소를 직접 생성하여 null 에러 방지)
const container = dv.el("div", "");
container.style.display = "grid";
container.style.gridTemplateColumns = "380px 1fr";
container.style.gap = "25px";
container.style.padding = "10px 0";

// --- [왼쪽 패널: 할 일 리스트] ---
const leftPanel = container.createDiv();
leftPanel.innerHTML = `
    <h3 style="margin: 0 0 20px 0; font-size: 1rem; color: ${ROSEWATER}; font-weight: 800;">할 일 목록 📋</h3>
    <div style="display: flex; gap: 12px; margin-bottom: 15px; border-bottom: 1px solid rgba(0,0,0,0.05); padding-bottom: 10px;">
        <span id="btn-todo" style="cursor: pointer; font-size: 0.75rem; color: ${ROSEWATER}; font-weight: 700; border-bottom: 2px solid ${ROSEWATER}; padding-bottom: 10px;">진행 중</span>
        <span id="btn-done" style="cursor: pointer; font-size: 0.75rem; color: #8a81a3; padding-bottom: 10px;">완료된 일</span>
    </div>
`;
const listArea = leftPanel.createDiv({ style: "display: grid; gap: 10px; max-height: 600px; overflow-y: auto;" });

// --- [오른쪽 패널: 캘린더 가이드] ---
const rightPanel = container.createDiv();
rightPanel.innerHTML = `
    <div style="display: flex; justify-content: flex-end; gap: 10px; margin-bottom: 20px;">
        <div id="btn-weekly" style="cursor: pointer; font-size: 0.7rem; padding: 4px 12px; border-radius: 20px; border: 1.5px solid ${ROSEWATER}; color: ${ROSEWATER}; background: ${ROSEWATER}1A; font-weight: 700;">Weekly</div>
        <div id="btn-monthly" style="cursor: pointer; font-size: 0.7rem; padding: 4px 12px; border-radius: 20px; border: 1.5px solid #eee; color: #8a81a3;">Monthly</div>
    </div>
    <div id="cal-box" style="background: rgba(245, 234, 224, 0.4); border: 2px solid rgba(230, 171, 169, 0.3); border-radius: 15px; height: 500px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 20px;">
        <div id="cal-icon" style="font-size: 2rem; margin-bottom: 10px;">🗓️</div>
        <h4 id="cal-title" style="margin: 0; font-size: 0.85rem; color: #444;">주간 일정 계획 뷰</h4>
        <p style="font-size: 0.7rem; color: #8a81a3; margin-top: 10px; line-height: 1.6;">
            왼쪽의 할 일을 <b>Full Calendar</b> 사이드바의<br>
            'Unscheduled' 목록에서 이 영역으로 드래그하세요.
        </p>
    </div>
`;

// 3. 리스트 렌더링 함수 (listArea 객체를 직접 참조)
const updateView = (subTarget) => {
    const tasks = subTarget === "done" 
        ? allTasks.filter(t => String(t.상태 || "").trim() === "완료") 
        : allTasks.filter(t => String(t.상태 || "").trim() !== "완료");

    let content = "";
    if (tasks.length === 0) {
        content = `<div style="font-size: 0.75rem; color: #999; text-align: center; padding: 40px 0;">항목이 없습니다.</div>`;
    } else {
        tasks.forEach(t => {
            const dateStr = t.날짜 ? `📅 ${t.날짜}` : "⚠️ 날짜 미지정";
            content += `
                <div style="background: var(--background-primary); border: 1px solid rgba(0,0,0,0.06); border-radius: 8px; padding: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                    <div style="font-size: 0.8rem; font-weight: 700; margin-bottom: 5px;">
                        <a class="internal-link" href="${t.file.path}" style="color: var(--text-normal); text-decoration: none;">${t.file.name}</a>
                    </div>
                    <div style="display: flex; justify-content: space-between; align-items: center; font-size: 0.65rem; color: #888;">
                        <span>${dateStr}</span>
                        <span style="color: ${ROSEWATER}; font-weight: 600;">${t.상태 || '계획 전'}</span>
                    </div>
                </div>`;
        });
    }
    listArea.innerHTML = content;
};

// 4. 이벤트 바인딩 (querySelector 대신 직접 객체 탐색)
const bTodo = leftPanel.querySelector("#btn-todo");
const bDone = leftPanel.querySelector("#btn-done");
const bWeekly = rightPanel.querySelector("#btn-weekly");
const bMonthly = rightPanel.querySelector("#btn-monthly");
const cTitle = rightPanel.querySelector("#cal-title");
const cIcon = rightPanel.querySelector("#cal-icon");

bTodo.onclick = () => {
    bTodo.style.color = ROSEWATER; bTodo.style.borderBottom = `2px solid ${ROSEWATER}`;
    bDone.style.color = "#8a81a3"; bDone.style.borderBottom = "none";
    updateView("todo");
};
bDone.onclick = () => {
    bDone.style.color = ROSEWATER; bDone.style.borderBottom = `2px solid ${ROSEWATER}`;
    bTodo.style.color = "#8a81a3"; bTodo.style.borderBottom = "none";
    updateView("done");
};

bWeekly.onclick = () => {
    bWeekly.style.background = `${ROSEWATER}1A`; bWeekly.style.color = ROSEWATER; bWeekly.style.borderColor = ROSEWATER;
    bMonthly.style.background = "none"; bMonthly.style.color = "#8a81a3"; bMonthly.style.borderColor = "#eee";
    cTitle.innerText = "주간 일정 계획 뷰"; cIcon.innerText = "🗓️";
};
bMonthly.onclick = () => {
    bMonthly.style.background = `${ROSEWATER}1A`; bMonthly.style.color = ROSEWATER; bMonthly.style.borderColor = ROSEWATER;
    bWeekly.style.background = "none"; bWeekly.style.color = "#8a81a3"; bWeekly.style.borderColor = "#eee";
    cTitle.innerText = "월간 일정 계획 뷰"; cIcon.innerText = "📅";
};

// 초기화
updateView("todo");
```

## ✅ 완료된 할 일

```dataview
TABLE 구분 AS "구분", 날짜 AS "날짜"
FROM "05. Tasks"
WHERE (프로젝트 = this.file.link OR 프로젝트 = this.file.name) AND 완료여부 = true
SORT file.mtime DESC
LIMIT 5
```

## 🚧 장애물

```dataview
LIST
FROM "09. Bottlenecks"
WHERE (프로젝트 = this.file.link OR 프로젝트 = this.file.name) AND 장애물 상태 != "해결"
```

## 🔗 관련 리소스

```dataview
LIST
FROM "06. Resources"
WHERE contains(프로젝트, this.file.link) OR contains(프로젝트, this.file.name)
SORT file.mtime DESC
LIMIT 8
```

## 📝 작업 기록

*작업하면서 생긴 결정사항, 배운 것, 메모를 남겨두세요.*
