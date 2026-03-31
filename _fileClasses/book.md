---
fileClass: true
limit: 20
mapWithTag: false
version: "2.1"
fields:
  - id: bk0001
    name: 독서 상태
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        읽고 싶은 책: 읽고 싶은 책
        읽는 중: 읽는 중
        완독: 완독
        중단: 중단
  - id: bk0002
    name: 분야
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        소설: 소설
        인문: 인문
        경제/경영: 경제/경영
        자기계발: 자기계발
        정치/사회: 정치/사회
        역사/문화: 역사/문화
        과학: 과학
        예술/대중문화: 예술/대중문화
        건강: 건강
        만화: 만화
        잡지: 잡지
  - id: bk0003
    name: 진행률
    type: Number
    options: {}
  - id: bk0004
    name: 별점
    type: Number
    options: {}
  - id: bk0005
    name: 시작일
    type: Date
    options: {}
  - id: bk0006
    name: 종료일
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
  - bk0006
  - bk0005
  - bk0004
  - bk0003
  - bk0002
  - bk0001
---
