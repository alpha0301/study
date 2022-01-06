## 용도

언어에 구애받지 않는 RESTful API 표준 인터페이스를 정의하기 위함

## Path Templating

- 매개변수는 {}

## HTTP Status Code

- RFC7231 참고

## Version

- Semantic Versioning 을 따름
- MAJOR.MINOR.PATCH
  - MAJOR: 호환안되는 변경
  - MINOR: 이전 버전과 호환되는 방식으로 기능 추가
  - PATCH: 이전 버전과 호환되는 버그 픽스
- 음이 아닌 정수 사용
- 선행 0을 포함해서는 안됨
- 수치적으로 증가해야
- [자세한 내용은 공식문서 참고](https://semver.org/spec/v2.0.0.html)

## Format

- 문서 자체가 JSON Object
- YAML 형식도 가능(1.2 권장)
- case sensitive
- api request, response, 기타 컨텐츠는 굳이 json, yaml 일 필요는 없음
- 여러 문서로 쪼갤 경우, 참조는 $ref 필드를 사용해야 함
  - 문서 경로 루트에 openapi.json, openapi.yaml 로 만들어서 쓰는게 좋음

## Data Types

### json object

- 6개의 primitive type 을 가질 수 있음
- null, boolean, object, array, number, string
- 예외) 같은 키에 두개의 값이 정의되면 하나는 undefined 처리
- equality
  - type -> value 순으로 비교
  - strings: same codepoint for codepoint
  - numbers: methematical
  - arrays: 순서와 내용 모두 같아야(길이도 당연히 같아짐)
  - objects: key 구성과 key 에 따른 value 가 모두 같아야(키 순서 무관)
  - 공백, 콤마 위치, trailing zero 무관(ex 12.0000)

### OAS format

![image](https://user-images.githubusercontent.com/39113923/148327484-4eb9d660-4721-4b6d-817e-52f0f01611ce.png)


