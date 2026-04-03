---
type: project
fileClass: project
제목: Untitled
박스: 자기관리
목표: 나만의 세컨드 브레인 구축하기
상태: 계획 전
달성률: 0
종료일: 2026-04-15
url: ""
시작일: 2026-04-03
---

# Untitled

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

```dataview
TABLE 구분 AS "구분", 중요긴급 AS "중요긴급", 날짜 AS "날짜"
FROM "05. Tasks"
WHERE (프로젝트 = this.file.link OR 프로젝트 = this.file.name) AND 완료여부 != true
SORT 날짜 ASC
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
