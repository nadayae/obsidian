---
type: box
fileClass: box
제목: <% tp.file.title %>
이모지: 🗂️
설명: ""
상태:
생성일: <% tp.date.now("YYYY-MM-DD") %>
---

# <% tp.file.title %>

> [!box] Box 정보
> **이모지:** 🗂️
> **설명:** *(이 영역에 대한 한 줄 설명)*
> **상태:** 일반
> **생성:** <% tp.date.now("YYYY-MM-DD") %>

---

## 비전 & 방향

*이 영역에서 내가 원하는 것, 나아가고 싶은 방향, 핵심 가치를 자유롭게 적어보세요.*

---

## ➕ 이 Box에 항목 추가

```button
name 🎯 새 Goal
type command
action QuickAdd: New Goal
color default
```
```button
name 🚀 새 Project
type command
action QuickAdd: New Project
color default
```

---

## 🎯 목표

```dataview
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율", 마감일 AS "마감"
FROM "03. Goals"
WHERE 박스 = this.file.link
SORT 마감일 ASC
```

## 🚀 프로젝트

```dataview
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율", 마감일 AS "마감일"
FROM "04. Projects"
WHERE 박스 = this.file.link
SORT 마감일 ASC
```

## 🔗 핵심 리소스

```dataview
LIST
FROM "06. Resources"
WHERE contains(박스, this.file.link)
SORT file.mtime DESC
LIMIT 10
```

## 💭 생각들

*이 Box와 관련된 자유로운 메모, 아이디어, 연결 고리들을 기록하세요.*
