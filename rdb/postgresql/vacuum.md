# vacuum 이란

vacuum은 제거해야할 데이터를 삭제 진행과 데이터베이스의 통계 정보를 갱신하는 작업이다.

# vacuum 이 필요한 이유
## 제거해야할 데이터를 삭제 진행

postgresql의 mvcc는 mysql의 mvcc와 달리 데이터를 삭제하면 실제로 데이터를 삭제하지 않고 삭제된 데이터를 표시하는 방식으로 동작한다.
이렇게 사용하지 않는 데이터를 삭제하지않고 남겨두면 데이터베이스의 크기가 계속해서 커지게 되고, 이로 인해 데이터베이스의 성능이 저하될 수 있다.

## 데이터베이스의 통계 정보를 갱신




출처:
- https://www.postgresql.org/docs/current/sql-vacuum.html
- https://techblog.woowahan.com/9478/ 