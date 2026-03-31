---
fileClass: true
limit: 20
mapWithTag: false
version: "2.1"
fields:
  - id: gl0001
    name: 상태
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        시작 전: 시작 전
        진행 중: 진행 중
        집중: 집중
        중단: 중단
        완료: 완료
        아카이브: 아카이브
  - id: gl0002
    name: 분기
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        1Q: 1Q
        2Q: 2Q
        3Q: 3Q
        4Q: 4Q
  - id: gl0003
    name: 달성률
    type: Number
    options: {}
  - id: gl0004
    name: 목표 달성일
    type: Date
    options: {}
  - id: gl0005
    name: 년도
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        "2025": "2025"
        "2026": "2026"
        "2027": "2027"
icon: package
tagNames:
filesPaths:
bookmarksGroups:
excludes:
extends:
savedViews: []
favoriteView:
fieldsOrder:
  - gl0005
  - gl0004
  - gl0003
  - gl0002
  - gl0001
---
