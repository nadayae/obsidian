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

# <% tp.file.title %>

> [!goal] 목표 정보
> **Box:** *(연결할 Box 링크 입력)*
> **기간:** <% tp.date.now("YYYY") %> · 1Q
> **상태:** 시작 전
> **목표 달성일:** <% tp.date.now("YYYY") %>-03-31

```dataviewjs
const me = dv.current().file.name;
const allT = dv.pages('"05. Tasks"');
const projects = dv.pages('"04. Projects"').where(p => p.목표 === me || String(p.목표) === me);
let sumRate = 0;
for (let p of projects) {
  const t = allT.where(t => t.프로젝트 === p.file.name || String(t.프로젝트) === p.file.name);
  if (t.length > 0) sumRate += (t.where(t => t.완료여부 === true).length / t.length) * 100;
}
const rate = projects.length > 0 ? Math.round(sumRate / projects.length) : 0;
dv.el("div", `<progress value="${rate}" max="100" style="width:100%;"></progress><div style="font-size:0.8rem; margin-top:4px;">달성률 <b>${rate}%</b> (${projects.length}개 프로젝트 평균)</div>`);
```

---

## 이 목표를 달성하면

*이 목표가 달성됐을 때 어떤 상태가 되어 있을지, 왜 이 목표가 중요한지 적어보세요.*

---

## ➕ 이 목표에 프로젝트 추가

```button
name 🚀 새 프로젝트 만들기
type command
action QuickAdd: New Project
color default
```

---

## 🚀 연결된 프로젝트

```dataview
TABLE 상태 AS "상태", 달성률 + "%" AS "달성률", 날짜 AS "날짜"
FROM "04. Projects"
WHERE 목표 = this.file.link OR 목표 = this.file.name
SORT 날짜 ASC
```

## 📋 연결된 할 일

```dataview
TABLE 구분 AS "구분", 프로젝트 AS "프로젝트", 날짜 AS "날짜"
FROM "05. Tasks"
WHERE (목표 = this.file.link OR 목표 = this.file.name) AND 완료여부 != true
SORT 날짜 ASC
LIMIT 10
```

## 🔗 관련 리소스

```dataview
LIST
FROM "06. Resources"
WHERE contains(목표, this.file.link) OR contains(목표, this.file.name)
SORT file.mtime DESC
LIMIT 8
```

## 💭 메모

*이 목표와 관련된 인사이트, 중간 회고, 아이디어를 자유롭게 기록하세요.*
