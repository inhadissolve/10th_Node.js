
# 2주차 미션 - SQL 쿼리 작성

이번 미션에서는 1주차에 설계한 데이터베이스를 기준으로, 실제 화면에 필요한 SQL 쿼리를 작성했다.  
테이블은 아래 구조를 사용했다.

- users
- regions
- stores
- missions
- user_missions
- reviews

또한 워크북 안내에 따라 join과 subquery는 상황에 맞게 선택했고, 필요하면 쿼리를 분리해서 작성할 수 있다는 점을 고려했다.

---

## 1. 내가 진행중 / 진행 완료한 미션 조회 쿼리

### 1-1.  제일 작은 쿼리부터

![alt text](images\image.png)

```sql
SELECT * 
FROM user_missions;
```
### [확인할 것]
- 어떤 user_id가 있는지
- 어떤 mission_id가 있는지
- status 값이 어떻게 들어있는지

### 1-2. 1번 사용자의 미션만 보기

![alt text](images\image-1.png)

```sql
SELECT *
FROM user_missions
WHERE user_id = 1;
```

- ### 의미

    1번 사용자가 가진 미션 관계 데이터만 보는 것

### 1-3. 1번 사용자의 진행중 미션만 보기

![alt text](images\image-2.png)

```sql
SELECT *
FROM user_missions
WHERE user_id = 1
  AND status = 'IN_PROGRESS';
```
- ### 의미

    진행중 상태만 필터링하는 것

### 1-4. 미션 제목 붙이기

![alt text](images\image-3.png)

```sql
SELECT *
FROM user_missions um
JOIN missions m
  ON um.mission_id = m.id
WHERE um.user_id = 1
  AND um.status = 'IN_PROGRESS';
```

- ### 이유?

    user_missions에는 미션 제목이 없고 mission_id만 있음

    그래서 missions 테이블과 연결해야 제목이 나온다

### 1-5. 가게 이름까지 붙이기

![alt text](images\image-4.png)

```sql
SELECT *
FROM user_missions um
JOIN missions m
  ON um.mission_id = m.id
JOIN stores s
  ON m.store_id = s.id
WHERE um.user_id = 1
  AND um.status = 'IN_PROGRESS';
```
- ### 왜 stores를 붙이는지?

    가게 이름은 stores에 있기 때문

### 1-6. 필요한 컬럼만 정리해서 최종 진행중 미션 쿼리

![alt text](images\image-5.png)

```sql
SELECT
    um.id AS user_mission_id,
    m.id AS mission_id,
    m.title,
    m.description,
    m.reward_point,
    s.name AS store_name,
    um.started_at
FROM user_missions um
JOIN missions m
  ON um.mission_id = m.id
JOIN stores s
  ON m.store_id = s.id
WHERE um.user_id = 1
  AND um.status = 'IN_PROGRESS'
ORDER BY um.created_at DESC
LIMIT 10 OFFSET 0;
```

### 1-7. 진행 완료 미션 조회

사용자가 완료한 미션 목록을 조회하는 쿼리이다.
완료 시점을 기준으로 최근 순으로 정렬했다.

![alt text](images\image-6.png)

```sql
SELECT
    um.id AS user_mission_id,
    m.id AS mission_id,
    m.title,
    m.description,
    m.reward_point,
    s.name AS store_name,
    um.completed_at
FROM user_missions um
JOIN missions m
  ON um.mission_id = m.id
JOIN stores s
  ON m.store_id = s.id
WHERE um.user_id = 1
  AND um.status = 'COMPLETED'
ORDER BY um.completed_at DESC
LIMIT 10 OFFSET 0;
```

### 1-8. 페이지 수 계산용 count 쿼리
페이지네이션을 위해 전체 개수 조회 쿼리도 같이 필요하다.

![alt text](images\image-7.png)

```sql
SELECT COUNT(*) AS total_count
FROM user_missions
WHERE user_id = 1
  AND status = 'IN_PROGRESS';
```
### 설명

- 진행중과 완료를 하나의 쿼리로도 만들 수 있지만, 화면 탭이 분리되어 있다고 보고 쿼리도 나누어 작성했다.
또한 사용자 기준 조회가 많다고 생각해서 user_id, status 조건이 핵심이라고 판단했다.
---

## 2. 리뷰 작성 쿼리
리뷰는 완료한 미션에 대해서만 작성 가능하다고 가정했다.
또한 한 사용자가 같은 미션에 중복 리뷰를 남기지 않도록 조건을 걸었다.

### 2-1 먼저 완료한 미션인지 확인

![alt text](images\image-8.png)

```sql
SELECT *
FROM user_missions
WHERE user_id = 1
  AND mission_id = 2
  AND status = 'COMPLETED';
```
- ### 의미

    1번 사용자가 2번 미션을 완료했는지 확인

### 2-2 가장 쉬운 리뷰 insert

```sql
INSERT INTO reviews (
    user_id,
    mission_id,
    rating,
    content,
    created_at,
    updated_at
)
VALUES (
    1,
    2,
    4,
    '생각보다 재미있었고 다시 도전해보고 싶다.',
    NOW(),
    NOW()
);
```
- ### 주의

    이건 이미 기존 리뷰가 있어서 중복이 될 수 있음

    그래서 실무적으로는 아래 방식이 더 좋음

### 2-3 완료 조건을 포함한 리뷰 작성 쿼리

```sql
INSERT INTO reviews (
    user_id,
    mission_id,
    rating,
    content,
    created_at,
    updated_at
)
SELECT
    um.user_id,
    um.mission_id,
    4,
    '생각보다 재미있었고 다시 도전해보고 싶다.',
    NOW(),
    NOW()
FROM user_missions um
WHERE um.user_id = 1
  AND um.mission_id = 2
  AND um.status = 'COMPLETED';
```
- ### 의미

    완료한 경우에만 insert 되게 하는 방식임

### 2-4 리뷰가 들어갔는지 확인

![alt text](images\image-9.png)

```sql
SELECT *
FROM reviews
WHERE user_id = 1;
```

### 설명

- 리뷰를 그냥 insert만 하면 아직 완료하지 않은 미션에도 리뷰를 쓰는 문제가 생길 수 있다.
그래서 user_missions에서 완료 상태를 확인한 후에만 insert 되도록 구성했다.
이번 미션에서는 사진은 제외라고 되어 있으므로 텍스트 리뷰만 저장했다.

---

## 3. 홈 화면 쿼리

홈 화면은 “현재 선택된 지역에서 도전 가능한 미션”이 핵심

여기서는 예를 들어 인천 지역 id = 1이라고 가정

### 3-1. 먼저 인천 지역의 미션만 보기

![alt text](images\image-10.png)

```sql
SELECT *
FROM missions m
JOIN stores s
  ON m.store_id = s.id
JOIN regions r
  ON s.region_id = r.id
WHERE r.id = 1;
```
- ### 의미

    1번 지역에 속한 가게들의 미션만 보기

### 3-2. 내가 이미 하고 있는 미션은 제외하기

![alt text](images\image-11.png)
![alt text](images\image-12.png)
![alt text](images\image-13.png)

```sql
SELECT *
FROM missions m
JOIN stores s
  ON m.store_id = s.id
JOIN regions r
  ON s.region_id = r.id
LEFT JOIN user_missions um
  ON um.mission_id = m.id
 AND um.user_id = 1
WHERE r.id = 1
  AND um.id IS NULL;
```

- ### 왜 ```LEFT JOIN + um.id IS NULL``` 인지?

    내가 아직 시작하지 않은 미션만 남기기 위해서야.

    - join해서 붙는 건 이미 시작한 미션
    - 안 붙는 건 아직 안 한 미션
    - 그래서 um.id IS NULL이 도전 가능한 미션

### 3-3. 최종 홈 화면 쿼리

![alt text](images\image-14.png)

```sql
SELECT
    m.id AS mission_id,
    m.title,
    m.description,
    m.reward_point,
    s.id AS store_id,
    s.name AS store_name,
    s.address,
    r.id AS region_id,
    r.name AS region_name
FROM missions m
JOIN stores s
  ON m.store_id = s.id
JOIN regions r
  ON s.region_id = r.id
LEFT JOIN user_missions um
  ON um.mission_id = m.id
 AND um.user_id = 1
WHERE r.id = 1
  AND um.id IS NULL
ORDER BY m.created_at DESC
LIMIT 10 OFFSET 0;
```

### 3-4. 홈 화면 count 쿼리

![alt text](images\image-15.png)

```sql
SELECT COUNT(*) AS total_count
FROM missions m
JOIN stores s
  ON m.store_id = s.id
JOIN regions r
  ON s.region_id = r.id
LEFT JOIN user_missions um
  ON um.mission_id = m.id
 AND um.user_id = 1
WHERE r.id = 1
  AND um.id IS NULL;
```

### 설명

- 지역을 기준으로 미션을 가져오고, 이미 사용자가 시작했거나 완료한 미션은 제외해야 한다고 생각했다.
그래서 LEFT JOIN으로 사용자 미션을 붙이고, um.id IS NULL 조건으로 아직 등록되지 않은 미션만 남겼다.

---

## 4. 마이페이지 화면 쿼리

마이페이지에서는 사용자 기본 정보와 현재 포인트, 작성한 리뷰 수, 완료한 미션 수를 함께 보여준다고 가정했다.

### 4-1. 가장 기본 정보만 조회

![alt text](images\image-16.png)

```sql
SELECT
    id,
    name,
    nickname,
    phone,
    gender,
    point
FROM users
WHERE id = 1;
```

### 4-2. 리뷰 수까지 같이 보기

![alt text](images\image-17.png)

```sql
SELECT
    u.id,
    u.name,
    u.nickname,
    u.point,
    COUNT(r.id) AS review_count
FROM users u
LEFT JOIN reviews r
  ON u.id = r.user_id
WHERE u.id = 1
GROUP BY u.id, u.name, u.nickname, u.point;
```

### 4-3. 최종 마이페이지 쿼리

![alt text](images\image-18.png)

```sql
SELECT
    u.id,
    u.name,
    u.nickname,
    u.phone,
    u.gender,
    u.status,
    u.point,
    u.created_at,
    COUNT(DISTINCT r.id) AS review_count,
    COUNT(DISTINCT CASE WHEN um.status = 'COMPLETED' THEN um.id END) AS completed_mission_count
FROM users u
LEFT JOIN reviews r
  ON u.id = r.user_id
LEFT JOIN user_missions um
  ON u.id = um.user_id
WHERE u.id = 1
GROUP BY
    u.id,
    u.name,
    u.nickname,
    u.phone,
    u.gender,
    u.status,
    u.point,
    u.created_at;
```

- ### 여기서 제일 어려운 부분
```sql 
COUNT(DISTINCT CASE WHEN um.status = 'COMPLETED' THEN um.id END)
```
쉽게 말하면,

완료 상태인 user_missions만 세겠다는 뜻임

### 설명

- 1주차에서 설계한 users 테이블에 이메일 컬럼은 없어서, 이름, 닉네임, 전화번호, 포인트를 기준으로 마이페이지를 구성했다.
추가로 작성한 리뷰 수와 완료한 미션 수를 같이 보여주면 마이페이지답다고 생각해서 집계 컬럼을 포함했다.