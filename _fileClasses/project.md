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
        계획 전: 계획 전
        계획: 계획
        진행중: 진행중
        집중: 집중
        중단: 중단
        완료: 완료
  - id: pj0002
    name: 달성률
    type: Number
    options: {}
  - id: pj0003
    name: 날짜
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
  - pj0003
  - pj0002
  - pj0001
---
