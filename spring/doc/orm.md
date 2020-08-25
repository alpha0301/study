# hibernate

## 주의사항

### @Temporal
- java.util.Date -> java.sql.Date 변환
- milliseconds 제대로 안나오는거 해결 가능

### @Version

### Cascade.Remove vs with OrphanRemoval

### n+1
- 관계 객체 조회 시 미리 fetch 안하면 join 해도 proxy 가 넘어와서 접근할 때마다 포풍 select

### annotation field vs getter
- 캡슐화 허용 여부(캡슐화 위험성이 부각됨. mapping 이 필요한 경우 활용 가능)
- @Transient
- [참고링크](https://stackoverflow.com/questions/594597/hibernate-annotations-which-is-better-field-or-property-access)
