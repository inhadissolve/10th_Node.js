# 6주차 미션 README

이번 6주차 미션에서는 Prisma ORM을 이용해 기존 Repository 코드를 리팩토링하고, 목록 조회 및 미션 완료 API를 구현했다.  
기존 `mysql2` 기반 SQL 작성 방식에서 Prisma Client를 사용하는 방식으로 변경했다.

---

## 1. GitHub 정보

### 작업 브랜치

```text
feature/chapter-06
```

### GitHub 링크

```text
여기에 본인 GitHub PR 또는 Repository 링크 첨부
```

---

## 2. 미션 목표

이번 미션의 목표는 다음과 같다.

- 기존 Repository 함수들을 Prisma ORM 기반으로 변경
- Prisma Schema와 Migration을 이용해 DB 테이블 관리
- 목록 조회 API 구현
- Cursor 기반 페이지네이션 적용
- 진행 중인 미션 완료 처리 API 구현
- 사용하지 않는 `mysql2` 제거

---

## 3. 구현한 API 목록

| 번호 | API | Method | Endpoint |
| --- | --- | --- | --- |
| 1 | 내가 작성한 리뷰 목록 | GET | `/api/v1/users/:userId/reviews` |
| 2 | 특정 가게의 미션 목록 | GET | `/api/v1/stores/:storeId/missions` |
| 3 | 내가 진행 중인 미션 목록 | GET | `/api/v1/users/:userId/missions/in-progress` |
| 4 | 진행 중인 미션 완료 처리 | PATCH | `/api/v1/users/:userId/missions/:missionId/complete` |

---

## 4. Prisma ORM 적용

기존에는 Repository에서 `mysql2`의 `pool`을 사용해 SQL을 직접 작성했다.

```ts
import { pool } from "../../../db.config.js";

const [rows] = await conn.query(
  "SELECT * FROM stores WHERE id = ?",
  [storeId]
);
```

6주차에서는 Prisma Client를 사용하도록 변경했다.

```ts
import { prisma } from "../../../db.config.js";

export const findStoreById = async (storeId: number) => {
  return await prisma.store.findFirst({
    where: {
      id: storeId,
    },
  });
};
```

이를 통해 SQL 문자열을 직접 작성하지 않고, Prisma 모델을 기준으로 DB에 접근할 수 있었다.

---

## 5. Prisma Client 설정

`src/db.config.ts`에서 Prisma Client를 생성했다.

```ts
import "dotenv/config";
import { PrismaClient } from "../src/generated/prisma/client.js";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";

const adapter = new PrismaMariaDb({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  port: process.env.DB_PORT ? parseInt(process.env.DB_PORT, 10) : 3306,
  connectionLimit: 10,
});

export const prisma = new PrismaClient({
  adapter,
  log: ["query", "info", "error", "warn"],
});
```

---

## 6. 1번 API: 내가 작성한 리뷰 목록

### Request

```text
GET /api/v1/users/1/reviews
```

### 핵심 내용

특정 사용자가 작성한 리뷰 목록을 조회한다.  
리뷰 정보와 함께 해당 리뷰가 작성된 가게 정보도 함께 반환한다.

### Repository

```ts
export const getReviewsByUserId = async (
  userId: number,
  cursor: number
) => {
  return await prisma.review.findMany({
    where: {
      userId,
      id: {
        gt: cursor,
      },
    },
    select: {
      id: true,
      rating: true,
      content: true,
      createdAt: true,
      store: {
        select: {
          id: true,
          name: true,
          address: true,
        },
      },
    },
    orderBy: {
      id: "asc",
    },
    take: 5,
  });
};
```

### 응답 구조

```json
{
  "data": [
    {
      "reviewId": 1,
      "rating": 5,
      "content": "가게 분위기가 좋았습니다.",
      "store": {
        "storeId": 1,
        "name": "인하 맛집",
        "address": "인천 미추홀구 인하로 100"
      }
    }
  ],
  "pagination": {
    "cursor": 3
  }
}
```

---

## 7. 2번 API: 특정 가게의 미션 목록

### Request

```text
GET /api/v1/stores/1/missions
```

### 핵심 내용

특정 가게에 등록된 미션 목록을 조회한다.  
미션 정보와 함께 가게 정보를 같이 반환한다.

### Repository

```ts
export const getMissionsByStoreId = async (
  storeId: number,
  cursor: number
) => {
  return await prisma.mission.findMany({
    where: {
      storeId,
      id: {
        gt: cursor,
      },
    },
    select: {
      id: true,
      title: true,
      description: true,
      rewardPoint: true,
      createdAt: true,
      store: {
        select: {
          id: true,
          name: true,
          address: true,
        },
      },
    },
    orderBy: {
      id: "asc",
    },
    take: 5,
  });
};
```

### 응답 구조

```json
{
  "data": [
    {
      "missionId": 1,
      "title": "가게 방문하기",
      "description": "가게에 방문한 뒤 인증하기",
      "rewardPoint": 500,
      "store": {
        "storeId": 1,
        "name": "인하 맛집",
        "address": "인천 미추홀구 인하로 100"
      }
    }
  ],
  "pagination": {
    "cursor": 2
  }
}
```

---

## 8. 3번 API: 내가 진행 중인 미션 목록

### Request

```text
GET /api/v1/users/1/missions/in-progress
```

### 핵심 내용

사용자가 현재 진행 중인 미션 목록을 조회한다.  
`user_missions` 테이블에서 `IN_PROGRESS` 상태인 데이터만 조회한다.

### Repository

```ts
export const getInProgressMissionsByUserId = async (
  userId: number,
  cursor: number
) => {
  return await prisma.userMission.findMany({
    where: {
      userId,
      status: "IN_PROGRESS",
      id: {
        gt: cursor,
      },
    },
    select: {
      id: true,
      status: true,
      startedAt: true,
      mission: {
        select: {
          id: true,
          title: true,
          description: true,
          rewardPoint: true,
          store: {
            select: {
              id: true,
              name: true,
              address: true,
            },
          },
        },
      },
    },
    orderBy: {
      id: "asc",
    },
    take: 5,
  });
};
```

### 응답 구조

```json
{
  "data": [
    {
      "userMissionId": 1,
      "status": "IN_PROGRESS",
      "mission": {
        "missionId": 1,
        "title": "가게 방문하기",
        "rewardPoint": 500
      }
    }
  ],
  "pagination": {
    "cursor": 2
  }
}
```

---

## 9. 4번 API: 진행 중인 미션 완료 처리

### Request

```text
PATCH /api/v1/users/1/missions/1/complete
```

### 핵심 내용

사용자의 진행 중 미션을 완료 상태로 변경한다.  
해당 미션이 `IN_PROGRESS` 상태일 때만 완료 처리되도록 검증했다.

### Repository

```ts
export const findInProgressUserMission = async (
  userId: number,
  missionId: number
) => {
  return await prisma.userMission.findFirst({
    where: {
      userId,
      missionId,
      status: "IN_PROGRESS",
    },
  });
};

export const completeUserMission = async (userMissionId: number) => {
  return await prisma.userMission.update({
    where: {
      id: userMissionId,
    },
    data: {
      status: "COMPLETED",
      completedAt: new Date(),
      updatedAt: new Date(),
    },
  });
};
```

### Service 검증 로직

```ts
export const completeMission = async (
  userId: number,
  missionId: number
) => {
  const user = await findUserById(userId);

  if (!user) {
    throw new Error("존재하지 않는 사용자입니다.");
  }

  const mission = await findMissionById(missionId);

  if (!mission) {
    throw new Error("존재하지 않는 미션입니다.");
  }

  const userMission = await findInProgressUserMission(userId, missionId);

  if (!userMission) {
    throw new Error("진행 중인 미션이 아닙니다.");
  }

  const completedMission = await completeUserMission(userMission.id);

  return responseFromCompletedMission(completedMission);
};
```

### 응답 구조

```json
{
  "result": {
    "userMissionId": 1,
    "userId": 1,
    "missionId": 1,
    "status": "COMPLETED",
    "startedAt": "2026-05-07T00:31:58.489Z",
    "completedAt": "2026-05-06T16:09:00.874Z"
  }
}
```

---

## 10. Cursor 기반 페이지네이션

목록 조회 API에는 cursor 기반 페이지네이션을 적용했다.

### 적용 이유

- 데이터를 한 번에 모두 조회하지 않기 위해
- DB 부하를 줄이기 위해
- 무한 스크롤이나 더보기 방식에 적합하기 때문에

### 구현 방식

```ts
where: {
  id: {
    gt: cursor,
  },
},
orderBy: {
  id: "asc",
},
take: 5,
```

응답에서는 마지막 데이터의 id를 다음 cursor로 내려주었다.

```ts
pagination: {
  cursor: lastItem ? lastItem.id : null,
}
```

---

## 11. Controller → Service → Repository → DB 흐름

이번 미션 API들은 아래 흐름으로 처리되었다.

```text
Client(Postman)
→ Express Router
→ Controller
→ Service
→ Repository
→ Prisma Client
→ DB
→ Prisma Client
→ Repository
→ Service
→ Controller
→ Response
```

예를 들어 진행 중인 미션 완료 API는 다음과 같이 동작한다.

```text
PATCH /api/v1/users/1/missions/1/complete 요청
→ handleCompleteMission Controller 실행
→ userId, missionId 확인
→ completeMission Service 호출
→ 사용자 존재 여부 확인
→ 미션 존재 여부 확인
→ 진행 중인 미션인지 확인
→ completeUserMission Repository 호출
→ Prisma Client가 user_missions 테이블 업데이트
→ COMPLETED 상태 응답 반환
```

---

## 12. mysql2 제거

모든 Repository를 Prisma 기반으로 변경한 후 직접 사용하던 `mysql2`를 제거했다.

```bash
npm uninstall mysql2
```

`npm list mysql2`를 실행했을 때 Prisma 내부 의존성으로는 `mysql2`가 표시될 수 있었다.  
하지만 프로젝트의 직접 의존성에서는 제거되었으므로 정상으로 판단했다.

---

## 13. 느낀 점

이번 미션을 통해 ORM을 사용하면 SQL을 직접 작성하지 않고도 DB 작업을 할 수 있다는 점을 체감했다.  
특히 `findMany`, `findFirst`, `update`, `select`를 사용하니 Repository 코드가 더 읽기 쉬워졌다.

다만 ORM을 사용한다고 해서 SQL 개념이 필요 없는 것은 아니라고 느꼈다.  
관계 설정, 페이지네이션, 조회 조건, update 조건을 정확히 이해해야 원하는 데이터를 가져올 수 있었다.

또한 Migration을 통해 DB 테이블 구조를 코드로 관리하는 방식이 협업에서 중요하다는 것도 알게 되었다.
