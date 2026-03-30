---
type: goal
fileClass: goal
제목: Untitled
박스: ""
연도: 2026
분기:
상태:
완료율: 0
마감일: 2026-03-31
생성일: 2026-03-28
---

# Untitled

> [!goal] 목표 정보
> **Box:** *(연결할 Box 링크 입력)*
> **기간:** 2026 · Q1
> **상태:** 시작 전
> **마감:** 2026-03-31
>
> <progress value="0" max="100"></progress>
> 완료율 **0%**

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
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율", 마감일 AS "마감일"
FROM "04. Projects"
WHERE 목표 = this.file.link
SORT 마감일 ASC
```

## 📋 연결된 할 일

```dataview
TASK FROM "05. Tasks"
WHERE 목표 = this.file.link AND !completed
SORT 우선순위 DESC
LIMIT 10
```

## 🔗 관련 리소스

```dataview
LIST
FROM "06. Resources"
WHERE contains(목표, this.file.link)
SORT file.mtime DESC
LIMIT 8
```

## 💭 메모

*이 목표와 관련된 인사이트, 중간 회고, 아이디어를 자유롭게 기록하세요.*
