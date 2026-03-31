---
fileClass: true
limit: 20
mapWithTag: false
version: "2.2"
fields:
  - id: tk0001
    name: 완료여부
    type: Boolean
    options: {}
  - id: tk0002
    name: 구분
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        일반: 일반
        집중: 집중
        쉬운: 쉬운
        위임: 위임
        일정: 일정
        저널: 저널
        나중에: 나중에
  - id: tk0003
    name: 중요긴급
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        1. 중요O + 긴급O: 1. 중요O + 긴급O
        2. 중요O + 긴급X: 2. 중요O + 긴급X
        3. 중요X + 긴급O: 3. 중요X + 긴급O
        4. 중요X + 긴급X: 4. 중요X + 긴급X
        5. 습관|루틴: 5. 습관|루틴
  - id: tk0004
    name: 소요시간(분)
    type: Number
    options: {}
  - id: tk0005
    name: 뽀모도로
    type: Number
    options: {}
  - id: tk0006
    name: 날짜
    type: Date
    options: {}
  - id: tk0007
    name: 수임자
    type: Input
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
  - tk0007
  - tk0006
  - tk0005
  - tk0004
  - tk0003
  - tk0002
  - tk0001
---
