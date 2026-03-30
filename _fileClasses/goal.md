---
fileClass: true
limit: 20
mapWithTag: false
version: "2.0"
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
        Q1: Q1
        Q2: Q2
        Q3: Q3
        Q4: Q4
  - id: gl0003
    name: 완료율
    type: Number
    options: {}
  - id: gl0004
    name: 마감일
    type: Date
    options: {}
---
