---
type: monthly
기간: <% tp.date.now("YYYY-MM") %>
연도: <% tp.date.now("YYYY") %>
월: <% tp.date.now("MM") %>
만족도: 0
생성일: <% tp.date.now("YYYY-MM-DD") %>
---

# 🗓️ <% tp.date.now("YYYY년 MM월") %> 먼슬리

> [!journal] 월간 정보
> **기간:** <% tp.date.now("YYYY년 MM월") %>
> **만족도:** ★☆☆☆☆ &nbsp; (1–5점)

---

## 🏆 이번 달 성과 & 하이라이트

*이번 달 완료한 것, 성장한 것, 기억에 남는 것들.*

-
-
-

---

## 🎯 목표 달성 현황

```dataview
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율", 마감일 AS "마감"
FROM "03. Goals"
WHERE 상태 != "아카이브"
SORT 마감일 ASC
```

---

## 🚀 프로젝트 월간 리뷰

```dataview
TABLE 상태 AS "상태", 완료율 + "%" AS "완료율"
FROM "04. Projects"
WHERE 상태 != "아카이브" AND 상태 != "완료"
SORT 마감일 ASC
```

---

## 📚 독서 현황

```dataview
TABLE 저자 AS "저자", 상태 AS "상태", 완료율 + "%" AS "진도"
FROM "07. Books"
WHERE 상태 = "읽는 중" OR 상태 = "완독"
SORT file.mtime DESC
```

---

## 💡 이번 달 핵심 인사이트

*이번 달 배운 것 중 가장 중요한 것들. 관련 [[리소스]]가 있다면 링크하세요.*

---

## 😤 아쉬웠던 것 & 다음 달 개선점

---

## 🚧 미해결 장애물

```dataview
LIST
FROM "09. Bottlenecks"
WHERE 상태 != "해결" AND 상태 != "아카이브"
```

---

## 🎯 다음 달 목표

- [ ]
- [ ]
- [ ]

---

## 🙏 이번 달 감사한 것

*이번 달 감사했던 것들.*

1.
2.
3.
