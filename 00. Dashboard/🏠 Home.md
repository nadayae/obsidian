---
tags:
  - dashboard
cssclasses:
  - second-brain
---

# 🧠 나의 세컨드 브레인

> [!card] 빠른 추가
> ```button
> name 🚀 프로젝트 추가
> type command
> action QuickAdd: New Project
> color default
> ```
> ```button
> name ✅ 할일 추가
> type command
> action QuickAdd: New Task
> color default
> ```
> ```button
> name 🎯 목표 추가
> type command
> action QuickAdd: New Goal
> color default
> ```
> ```button
> name 📝 노트 추가
> type command
> action QuickAdd: New Resource
> color default
> ```
> ```button
> name ✍️ 캡처 추가
> type command
> action QuickAdd: New Capture
> color default
> ```

---

## ✅ 할 일

> [!col]
> > [!card] 전체
> > ```dataview
> > TABLE WITHOUT ID
> >   file.link AS "할 일",
> >   프로젝트 AS "프로젝트",
> >   마감일 AS "마감",
> >   분류 AS "분류"
> > FROM "05. Tasks"
> > WHERE 상태 = "시작 전" OR 상태 = "진행 중" OR 상태 = "집중"
> > SORT 마감일 ASC
> > ```
>
> > [!card] 오늘 집중
> > ```dataview
> > TABLE WITHOUT ID
> >   file.link AS "할 일",
> >   프로젝트 AS "프로젝트",
> >   마감일 AS "마감"
> > FROM "05. Tasks"
> > WHERE (마감일 = date(today) OR 분류 = "집중") AND 상태 != "완료"
> > SORT 우선순위 DESC
> > LIMIT 10
> > ```

> [!card]- 집중 (Focused)
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감",
>   우선순위 AS "우선순위"
> FROM "05. Tasks"
> WHERE 분류 = "집중" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

> [!card]- 일반 (General)
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감"
> FROM "05. Tasks"
> WHERE 분류 = "일반" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

> [!card]- 쉬운 것 (Easy)
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감"
> FROM "05. Tasks"
> WHERE 분류 = "쉬운" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

> [!card]- 위임 (Delegated)
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감"
> FROM "05. Tasks"
> WHERE 분류 = "위임" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

> [!card]- 예약됨 (Scheduled)
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감"
> FROM "05. Tasks"
> WHERE 분류 = "일정" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

> [!card]- 긴급 + 중요
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "할 일",
>   프로젝트 AS "프로젝트",
>   마감일 AS "마감"
> FROM "05. Tasks"
> WHERE 매트릭스 = "중요+긴급" AND 상태 != "완료"
> SORT 마감일 ASC
> ```

---

## ⚡ 오늘

> [!card] 오늘 집중할 것
> ```dataview
> TASK FROM "05. Tasks"
> WHERE !completed
> WHERE 마감일 = date(today) OR 분류 = "집중"
> SORT 우선순위 DESC
> LIMIT 10
> ```

---

## 🗓️ 프로젝트 타임라인

> [!card] 일정순
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "프로젝트",
>   목표 AS "목표",
>   시작일 AS "시작",
>   마감일 AS "마감",
>   상태 AS "상태",
>   완료율 + "%" AS "완료율"
> FROM "04. Projects"
> WHERE 상태 != "완료" AND 상태 != "아카이브"
> SORT 마감일 ASC
> ```

---

## 🚀 프로젝트  ·  🎯 목표

> [!col]
> > [!card] 진행 중인 프로젝트
> > ```dataview
> > TABLE WITHOUT ID
> >   file.link AS "프로젝트",
> >   상태 AS "상태",
> >   완료율 + "%" AS "완료율",
> >   마감일 AS "마감"
> > FROM "04. Projects"
> > WHERE 상태 = "진행 중" OR 상태 = "집중"
> > SORT 마감일 ASC
> > ```
>
> > [!card] 활성 목표
> > ```dataview
> > TABLE WITHOUT ID
> >   file.link AS "목표",
> >   박스 AS "Box",
> >   분기 AS "분기",
> >   완료율 + "%" AS "완료율",
> >   마감일 AS "마감"
> > FROM "03. Goals"
> > WHERE 상태 != "완료" AND 상태 != "아카이브"
> > SORT 마감일 ASC
> > ```

> [!card]- 전체 프로젝트
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "프로젝트",
>   목표 AS "목표",
>   상태 AS "상태",
>   완료율 + "%" AS "완료율",
>   마감일 AS "마감"
> FROM "04. Projects"
> WHERE 상태 != "아카이브"
> SORT 상태 ASC, 마감일 ASC
> ```

> [!card]- Box별 목표 구조
> ```dataview
> TABLE WITHOUT ID
>   박스 AS "Box",
>   file.link AS "목표",
>   상태 AS "상태",
>   완료율 + "%" AS "완료율"
> FROM "03. Goals"
> WHERE 상태 != "완료" AND 상태 != "아카이브"
> SORT 박스 ASC, 마감일 ASC
> ```

---

## 📝 자료

> [!card] 전체 자료
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "자료",
>   유형 AS "유형",
>   중요도 AS "중요도",
>   프로젝트 AS "프로젝트",
>   file.mtime AS "수정일"
> FROM "06. Resources"
> SORT file.mtime DESC
> LIMIT 20
> ```

---

## 📦 박스

> [!card] 전체 박스
> ```dataview
> TABLE WITHOUT ID
>   file.link AS "Box",
>   설명 AS "설명",
>   상태 AS "상태"
> FROM "02. Boxes"
> WHERE type = "box"
> SORT file.name ASC
> ```

---

## ⚙️ 관리도구

> [!card] 바로가기
> ```button
> name 📦 박스 관리
> type link
> action 02. Boxes
> color default
> ```
> ```button
> name 🎯 목표 관리
> type link
> action 03. Goals
> color default
> ```
> ```button
> name 🚀 프로젝트 관리
> type link
> action 04. Projects
> color default
> ```
> ```button
> name ✅ 할일 관리
> type link
> action 05. Tasks
> color default
> ```
> ```button
> name 📝 자료 관리
> type link
> action 06. Resources
> color default
> ```
> ```button
> name 📚 도서 라이브러리
> type link
> action 07. Books
> color default
> ```
> ```button
> name 📅 저널 관리
> type link
> action 08. Journal
> color default
> ```
> ```button
> name 🚧 장애물
> type link
> action 09. Bottlenecks
> color default
> ```
> ```button
> name 🗄️ 아카이브
> type link
> action Archive
> color default
> ```

---

## 📊 DB 현황

> [!card] 데이터베이스 목록
>
> | DB | 링크 |
> |---|---|
> | 📥 Captures | [[01. Captures\|바로가기]] |
> | 📦 Boxes | [[02. Boxes\|바로가기]] |
> | 🎯 Goals | [[03. Goals\|바로가기]] |
> | 🚀 Projects | [[04. Projects\|바로가기]] |
> | ✅ Tasks & 할일 | [[05. Tasks\|바로가기]] |
> | 📝 Resources | [[06. Resources\|바로가기]] |
> | 📚 도서 라이브러리 | [[07. Books\|바로가기]] |
> | 📅 Daily Journal DB | [[08. Journal/Daily\|바로가기]] |
> | 📆 WM Journal DB | [[08. Journal/Weekly\|바로가기]] |
> | ⚙️ Adjust & Management DB | [[08. Journal/Monthly\|바로가기]] |
> | 🚧 Bottleneck DB | [[09. Bottlenecks\|바로가기]] |
> | 🗄️ 아카이브 | [[Archive\|바로가기]] |
