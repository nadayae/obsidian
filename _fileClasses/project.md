---
fileClass: true
limit: 20
mapWithTag: false
version: "2.1"
fields:
  - id: pj0001
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
  - id: pj0002
    name: 우선순위
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        높음: 높음
        보통: 보통
        낮음: 낮음
  - id: pj0003
    name: 완료율
    type: Number
    options: {}
  - id: pj0004
    name: 시작일
    type: Date
    options: {}
  - id: pj0005
    name: 마감일
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
  - pj0005
  - pj0004
  - pj0003
  - pj0002
  - pj0001
---
