---
fileClass: true
limit: 20
mapWithTag: false
version: "2.0"
fields:
  - id: tk0001
    name: 상태
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        시작 전: 시작 전
        진행 중: 진행 중
        집중: 집중
        완료: 완료
        아카이브: 아카이브
  - id: tk0002
    name: 분류
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
    name: 매트릭스
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        중요+긴급: 중요+긴급
        중요: 중요
        긴급: 긴급
        해당없음: 해당없음
        습관|루틴: 습관|루틴
  - id: tk0004
    name: 우선순위
    type: Select
    options:
      sourceType: ValuesList
      valuesList:
        높음: 높음
        보통: 보통
        낮음: 낮음
  - id: tk0005
    name: 마감일
    type: Date
    options: {}
  - id: tk0006
    name: 예상시간
    type: Number
    options: {}
  - id: tk0007
    name: 뽀모도로
    type: Number
    options: {}
---
