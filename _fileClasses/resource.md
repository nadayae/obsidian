---
fileClass: true
limit: 20
mapWithTag: false
version: "2.1"
fields:
  - id: rs0001
    name: 분류
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        간단한 메모: 간단한 메모
        노트: 노트
        스크랩: 스크랩
        인사이트: 인사이트
        문장수집: 문장수집
  - id: rs0002
    name: 중요도
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        중요 자료: 중요 자료
        정리자료: 정리자료
        일반 자료: 일반 자료
        아카이브: 아카이브
  - id: rs0003
    name: 고정
    type: Boolean
    options: {}
  - id: rs0004
    name: 나중에 보기
    type: Boolean
    options: {}
  - id: rs0005
    name: 요약
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
  - rs0005
  - rs0004
  - rs0003
  - rs0002
  - rs0001
---
