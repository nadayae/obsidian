---
type: monthly
구분: 월간
기간: <% tp.date.now("YYYY-MM-DD") %>
이름: <% tp.date.now("YYYY년 MM월") %> 먼슬리
만족도: 0
월간 목표: ""
성과/배운점: ""
문제/고민: ""
개선 방향: ""
다음 월간 목표: ""
감사한 일: ""
생성일: <% tp.date.now("YYYY-MM-DD") %>
---

# 🗓️ <% tp.date.now("YYYY년 MM월") %> 먼슬리

> [!journal] 월간 정보
> **기간:** <% tp.date.now("YYYY년 MM월") %>
> **만족도:** ★☆☆☆☆ &nbsp; (1–5점)

---

## 🎯 이번 달 목표

*이번 달에 집중하려 했던 것들.*

---

## 🏆 성과 & 배운점

*이번 달 완료한 것, 성장한 것, 기억에 남는 것들.*

-
-
-

---

## 🎯 목표 달성 현황

```dataview
TABLE 상태 AS "상태", 달성률 + "%" AS "달성률", 목표 달성일 AS "달성일"
FROM "03. Goals"
WHERE 상태 != "아카이브"
SORT 목표 달성일 ASC
```

---

## 🚀 프로젝트 월간 리뷰

```dataview
TABLE 상태 AS "상태", 달성률 + "%" AS "달성률"
FROM "04. Projects"
WHERE 상태 != "아카이브" AND 상태 != "완료"
SORT 날짜 ASC
```

---

## 📚 독서 현황

```dataview
TABLE 저자 AS "저자", 독서 상태 AS "상태", 진행률 + "%" AS "진도"
FROM "07. Books"
WHERE 독서 상태 = "읽는 중" OR 독서 상태 = "완독"
SORT file.mtime DESC
```

---

## 😤 문제 & 고민

*이번 달 막혔던 것, 고민됐던 것.*

---

## 💡 개선 방향

*다음 달에 개선할 것들.*

---

## 🎯 다음 달 목표

- [ ]
- [ ]
- [ ]

---

## 🙏 이번 달 감사한 일

*이번 달 감사했던 것들.*

1.
2.
3.
