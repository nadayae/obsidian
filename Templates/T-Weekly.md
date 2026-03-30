---
type: weekly
기간: <% tp.date.now("YYYY") %>-W<% tp.date.now("WW") %>
시작일: <% tp.date.monday("YYYY-MM-DD") %>
종료일: <% tp.date.now("YYYY-MM-DD", 6) %>
만족도: 0
생성일: <% tp.date.now("YYYY-MM-DD") %>
---

# 📆 <% tp.date.now("YYYY년") %> <% tp.date.now("WW") %>주차 위클리

> [!journal] 주간 정보
> **기간:** <% tp.date.monday("MM/DD") %> – <% tp.date.now("MM/DD", 6) %>
> **만족도:** ★☆☆☆☆ &nbsp; (1–5점)

---

## 🏆 이번 주 성과

*이번 주에 완료한 것, 잘한 것들을 기록하세요.*

-
-
-

---

## 📊 프로젝트 진행 현황

```dataview
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율", 마감일 AS "마감일"
FROM "04. Projects"
WHERE 상태 = "진행 중" OR 상태 = "집중"
SORT 마감일 ASC
```

---

## 💡 이번 주 배운 것

*이번 주에 배운 것, 깨달은 것. 관련 [[리소스]]나 [[노트]]가 있다면 링크하세요.*

---

## 😤 아쉬웠던 것 & 개선점

*이번 주 아쉬웠던 점과 다음 주에 개선할 것.*

---

## 🚧 해결 못한 장애물

```dataview
LIST
FROM "09. Bottlenecks"
WHERE 상태 != "해결" AND 상태 != "아카이브"
SORT 우선순위 DESC
LIMIT 5
```

---

## 🎯 다음 주 목표

*다음 주에 집중할 것들.*

- [ ]
- [ ]
- [ ]

---

## 🙏 감사한 것

*이번 주 감사했던 것들 3가지.*

1.
2.
3.
