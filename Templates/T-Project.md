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
const FIXED_ID = "integrated-project-dashboard";
const ROSEWATER = "#e6aba9";
const today = dv.date("today");
const projectName = dv.current().file.name;
const TASKS_PATH = '"05. Tasks"';

// 1. 현재 프로젝트와 연결된 할 일 데이터 수집
const allTasks = dv.pages(TASKS_PATH).filter(t => {
    const prop = t.프로젝트;
    if (!prop) return false;
    const nameOnly = prop.path ? prop.fileName : String(prop).replace(/[\[\]]/g, "").trim();
    return nameOnly === projectName;
}).array();

// 2. 전체 레이아웃 (Grid: 왼쪽 4, 오른쪽 6 비율)
let html = `
<div id="${FIXED_ID}" style="display: grid; grid-template-columns: 380px 1fr; gap: 25px; font-family: var(--font-interface); padding: 10px 0;">
    
    <div id="left-task-panel">
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
            <h3 style="margin: 0; font-size: 1rem; color: ${ROSEWATER}; font-weight: 800;">할 일 목록 📋</h3>
        </div>
        
        <div style="display: flex; gap: 12px; margin-bottom: 15px; border-bottom: 1px solid rgba(0,0,0,0.05); padding-bottom: 10px;">
            <div class="sub-tab s-active" data-sub="todo" style="cursor: pointer; font-size: 0.75rem; color: ${ROSEWATER}; font-weight: 700;">진행 중</div>
            <div class="sub-tab" data-sub="done" style="cursor: pointer; font-size: 0.75rem; color: #8a81a3;">완료된 일</div>
        </div>

        <div id="list-display" style="display: grid; gap: 10px; max-height: 600px; overflow-y: auto; padding-right: 5px;">
            </div>
    </div>

    <div id="right-calendar-panel">
        <div style="display: flex; justify-content: flex-end; gap: 15px; margin-bottom: 20px;">
            <div class="cal-tab c-active" data-cal="weekly" style="cursor: pointer; font-size: 0.75rem; padding: 4px 12px; border-radius: 20px; border: 1.5px solid ${ROSEWATER}; color: ${ROSEWATER}; font-weight: 700;">Weekly</div>
            <div class="cal-tab" data-cal="monthly" style="cursor: pointer; font-size: 0.75rem; padding: 4px 12px; border-radius: 20px; border: 1.5px solid #eee; color: #8a81a3;">Monthly</div>
        </div>

        <div id="calendar-display" style="background: rgba(245, 234, 224, 0.4); border: 2px solid rgba(230, 171, 169, 0.3); border-radius: 15px; height: 550px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 30px;">
            <div id="cal-hint-icon" style="font-size: 2.5rem; margin-bottom: 15px;">🗓️</div>
            <h4 id="cal-hint-title" style="margin: 0; font-size: 0.9rem; color: #444;">주간 일정 계획 뷰</h4>
            <p style="font-size: 0.75rem; color: #8a81a3; margin-top: 10px; line-height: 1.5;">
                왼쪽 목록의 할 일을 확인하고<br>
                <b>Full Calendar</b> 플러그인의 'Unscheduled' 섹션에서<br>
                이곳(달력)으로 드래그하여 일정을 배정하세요.
            </p>
        </div>
    </div>
</div>

<style>
    .sub-tab.s-active { border-bottom: 2px solid ${ROSEWATER}; padding-bottom: 10px; }
    .cal-tab.c-active { background: ${ROSEWATER}1A; }
    #list-display::-webkit-scrollbar { width: 4px; }
    #list-display::-webkit-scrollbar-thumb { background: ${ROSEWATER}4D; border-radius: 10px; }
</style>
`;

dv.el("div", html);

// 리스트 렌더링 함수
const renderList = (subTarget) => {
    const listDisplay = document.getElementById("list-display");
    const tasks = subTarget === "done" 
        ? allTasks.filter(t => String(t.상태 || "").trim() === "완료") 
        : allTasks.filter(t => String(t.상태 || "").trim() !== "완료");

    let items = "";
    if (tasks.length === 0) {
        items = `<div style="font-size: 0.75rem; color: #999; text-align: center; padding: 40px 0;">항목이 없습니다.</div>`;
    } else {
        tasks.forEach(t => {
            const dateStr = t.날짜 ? `📅 ${t.날짜}` : "⚠️ 날짜 미지정";
            items += `
                <div style="background: var(--background-primary); border: 1px solid rgba(0,0,0,0.06); border-radius: 10px; padding: 14px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                    <div style="font-size: 0.8rem; font-weight: 700; margin-bottom: 6px;">
                        <a class="internal-link" href="${t.file.path}" style="color: var(--text-normal); text-decoration: none;">${t.file.name}</a>
                    </div>
                    <div style="display: flex; justify-content: space-between; align-items: center; font-size: 0.65rem; color: #888;">
                        <span>${dateStr}</span>
                        <span style="color: ${ROSEWATER}; font-weight: 600;">${t.상태 || '계획 전'}</span>
                    </div>
                </div>`;
        });
    }
    listDisplay.innerHTML = items;
};

// 메인 루프 및 이벤트
setTimeout(() => {
    const root = document.getElementById(FIXED_ID);
    if (!root) return;

    // 할 일 서브탭 이벤트
    root.querySelectorAll(".sub-tab").forEach(tab => {
        tab.addEventListener("click", () => {
            root.querySelectorAll(".sub-tab").forEach(t => { t.classList.remove("s-active"); t.style.color = "#8a81a3"; });
            tab.classList.add("s-active"); tab.style.color = ROSEWATER;
            renderList(tab.dataset.sub);
        });
    });

    // 캘린더 탭 전환 이벤트
    root.querySelectorAll(".cal-tab").forEach(tab => {
        tab.addEventListener("click", () => {
            root.querySelectorAll(".cal-tab").forEach(t => { 
                t.classList.remove("c-active"); t.style.borderColor = "#eee"; t.style.color = "#8a81a3"; t.style.background = "none";
            });
            tab.classList.add("c-active"); tab.style.borderColor = ROSEWATER; tab.style.color = ROSEWATER; tab.style.background = `${ROSEWATER}1A`;
            
            // 힌트 텍스트 변경
            document.getElementById("cal-hint-title").innerText = tab.dataset.cal === "weekly" ? "주간 일정 계획 뷰" : "월간 일정 계획 뷰";
            document.getElementById("cal-hint-icon").innerText = tab.dataset.cal === "weekly" ? "🗓️" : "📅";
        });
    });

    renderList("todo");
}, 150);
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
