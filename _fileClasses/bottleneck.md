---
fileClass: true
limit: 20
mapWithTag: false
version: "2.1"
fields:
  - id: bn0001
    name: 상태
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        중요한 문제: 중요한 문제
        사소한 문제: 사소한 문제
        해결: 해결
        아카이브: 아카이브
  - id: bn0002
    name: 우선순위
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        높음: 높음
        보통: 보통
        낮음: 낮음
  - id: bn0003
    name: 해결일
    type: Date
    options: {}
icon: package
tagNames:
filesPaths:
bookmarksGroups:
excludes:
extends:
savedViews: []
favoriteView:
fieldsOrder:
  - bn0003
  - bn0002
  - bn0001
---
