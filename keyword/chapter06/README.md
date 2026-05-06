# 6주차 핵심 키워드 정리

---

## 1. ORM

### 1-1. ORM이란?

ORM은 **Object-Relational Mapping**의 약자이다.  
말 그대로 객체와 관계형 데이터베이스를 연결해주는 기술이다.

조금 쉽게 말하면, 원래는 데이터베이스를 사용하려면 SQL을 직접 작성해야 한다.

```sql
SELECT * FROM user WHERE id = 1;
```

하지만 ORM을 사용하면 SQL을 직접 문자열로 작성하지 않고, TypeScript나 JavaScript 코드로 DB에 접근할 수 있다.

```ts
// Prisma ORM을 사용한 사용자 조회 예시
const user = await prisma.user.findUnique({
  where: {
    id: 1,
  },
});
```

즉, ORM은 SQL을 완전히 몰라도 된다는 뜻은 아니지만,  
반복적이고 긴 SQL을 더 읽기 쉬운 코드 형태로 작성할 수 있게 도와주는 도구라고 이해했다.

---

### 1-2. SQL 직접 작성 방식과 ORM 방식 비교

#### SQL 직접 작성 방식

```ts
// SQL을 직접 문자열로 작성하는 방식
// 장점: SQL이 그대로 보이므로 세밀한 제어가 가능함
// 단점: 쿼리가 길어지면 읽기 어렵고, 오타가 나도 실행 전까지 찾기 어려움

const [rows] = await pool.query(
  "SELECT * FROM user WHERE email = ?",
  [email]
);
```

#### ORM 사용 방식

```ts
// Prisma ORM을 사용하는 방식
// SQL을 직접 작성하지 않고 객체 형태로 조건을 표현함

const user = await prisma.user.findFirst({
  where: {
    email: email,
  },
});
```

위 두 코드는 모두 “이메일이 일치하는 사용자 찾기”라는 같은 목적을 가진다.  
하지만 ORM을 사용하면 SQL 문자열을 직접 작성하지 않아도 되고, IDE 자동완성 도움도 받을 수 있다.

---

### 1-3. ORM을 사용하는 이유

ORM을 사용하는 이유는 다음과 같다.

| 이유 | 설명 |
| --- | --- |
| 가독성 향상 | 긴 SQL보다 TypeScript 객체 형태가 더 읽기 쉽다. |
| 유지보수 편리 | 테이블이나 컬럼 이름을 코드에서 추적하기 쉽다. |
| 자동완성 지원 | Prisma Client를 생성하면 IDE에서 자동완성을 사용할 수 있다. |
| 타입 안정성 | TypeScript와 함께 사용하면 잘못된 필드 접근을 줄일 수 있다. |
| 반복 코드 감소 | INSERT, SELECT, JOIN 등을 더 간단히 작성할 수 있다. |

---

### 1-4. 내가 이해한 ORM

나는 ORM을 “SQL을 없애는 도구”라기보다,  
**SQL을 직접 문자열로 쓰는 부담을 줄이고, DB 작업을 코드 레벨에서 더 안전하게 관리하게 해주는 도구**라고 이해했다.

특히 5주차에는 Repository에서 직접 SQL을 작성했기 때문에, SQL이 길어질수록 실수하기 쉽다는 것을 느꼈다.  
그래서 6주차에서 ORM을 적용하면 코드가 더 깔끔해지고, 유지보수하기 쉬워진다는 점이 더 잘 와닿았다.

---

## 2. Prisma 문서 살펴보기

### 2-1. Prisma란?

Prisma는 Node.js와 TypeScript 환경에서 많이 사용하는 ORM 라이브러리이다.  
Prisma를 사용하면 `schema.prisma` 파일에 DB 테이블 구조를 정의하고, 그 구조를 바탕으로 Prisma Client를 생성해 DB에 접근할 수 있다.

Prisma를 사용할 때 핵심 파일은 다음과 같다.

| 파일 | 역할 |
| --- | --- |
| `prisma/schema.prisma` | DB 모델과 관계를 정의하는 파일 |
| `prisma.config.ts` | Prisma 설정 파일 |
| `.env` | DB 연결 정보를 저장하는 파일 |
| `src/generated/prisma` | Prisma Client가 생성되는 위치 |
| `src/db.config.ts` | Prisma Client 인스턴스를 만들어 export하는 파일 |

---

### 2-2. Prisma 설치

6주차 워크북에서는 Prisma와 MariaDB/MySQL 연결을 위해 아래 라이브러리를 설치한다.

```bash
npm install @prisma/client @prisma/adapter-mariadb dotenv
npm install -D prisma
```

각 라이브러리의 역할은 다음과 같다.

| 라이브러리 | 역할 |
| --- | --- |
| `@prisma/client` | 코드에서 DB에 접근할 때 사용하는 Prisma Client |
| `prisma` | Prisma CLI 도구 |
| `@prisma/adapter-mariadb` | MariaDB/MySQL 연결을 위한 Driver Adapter |
| `dotenv` | `.env` 환경 변수 사용 |

---

### 2-3. Prisma 초기 설정

Prisma 설정 파일을 만들기 위해 아래 명령어를 실행한다.

```bash
npm exec prisma init
```

그러면 보통 아래와 같은 구조가 생긴다.

```text
prisma
  schema.prisma
.env
```

Prisma 7부터는 `DATABASE_URL`을 `schema.prisma`에 직접 쓰는 것이 아니라, `prisma.config.ts`에서 관리한다.

```ts
// prisma.config.ts

import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  // Prisma Schema 파일 위치
  schema: "prisma/schema.prisma",

  // DB 연결 URL을 .env에서 가져옴
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

---

### 2-4. DATABASE_URL 설정

`.env` 파일에는 DB 연결 정보를 작성한다.

```env
DATABASE_URL="mysql://root:root1234!@localhost:3306/umc_week6"
```

형식은 다음과 같다.

```text
mysql://DB_USER:DB_PASSWORD@DB_HOST:DB_PORT/DB_NAME
```

주의할 점은 중괄호 `{}`를 넣으면 안 된다는 것이다.  
예를 들어 `{root1234!}`처럼 쓰면 중괄호까지 비밀번호로 인식될 수 있다.

---

### 2-5. schema.prisma 기본 구조

```prisma
generator client {
  // Prisma Client를 생성한다는 의미
  provider = "prisma-client"

  // 생성된 Prisma Client 코드가 저장될 위치
  output = "../src/generated/prisma"
}

datasource db {
  // 사용할 DB 종류
  provider = "mysql"
}
```

`schema.prisma`는 Prisma의 핵심 파일이다.  
이 안에 DB 테이블을 Model 형태로 정의하고, 이 파일을 기반으로 Prisma Client를 생성한다.

---

### 2-6. User 모델 예시

```prisma
model User {
  id            Int      @id @default(autoincrement())
  email         String   @unique(map: "email") @db.VarChar(255)
  name          String   @db.VarChar(100)
  gender        String   @db.VarChar(15)
  birth         DateTime @db.Date
  address       String   @db.VarChar(255)
  detailAddress String?  @map("detail_address") @db.VarChar(255)
  phoneNumber   String   @map("phone_number") @db.VarChar(15)

  @@map("user")
}
```

### 코드 설명

```prisma
id Int @id @default(autoincrement())
```

- `id`는 기본키이다.
- 자동으로 1씩 증가한다.

```prisma
email String @unique(map: "email") @db.VarChar(255)
```

- `email`은 문자열이다.
- 중복될 수 없도록 `@unique`를 붙인다.
- DB에서는 `VARCHAR(255)`로 저장된다.

```prisma
detailAddress String? @map("detail_address") @db.VarChar(255)
```

- `String?`는 null을 허용한다는 뜻이다.
- 코드에서는 `detailAddress`로 사용하지만, DB 컬럼명은 `detail_address`이다.
- 이를 연결하기 위해 `@map("detail_address")`를 사용한다.

```prisma
@@map("user")
```

- Prisma 모델명은 `User`이지만 실제 DB 테이블 이름은 `user`이다.
- 테이블 이름을 매핑할 때는 `@@map`을 사용한다.

---

### 2-7. @map과 @@map 차이

| 문법 | 의미 |
| --- | --- |
| `@map` | 특정 컬럼 이름을 DB 컬럼명과 매핑 |
| `@@map` | 모델 전체를 DB 테이블명과 매핑 |

예시:

```prisma
model User {
  detailAddress String? @map("detail_address")

  @@map("user")
}
```

- 코드에서는 `detailAddress`
- DB에서는 `detail_address`
- 코드에서는 `User`
- DB에서는 `user`

이렇게 서로 다른 이름을 연결할 수 있다.

---

### 2-8. Prisma Client 생성

`schema.prisma`를 작성하거나 수정한 뒤에는 Prisma Client를 생성해야 한다.

```bash
npm exec prisma generate
```

생성된 Prisma Client를 사용하면 코드에서 다음과 같이 DB에 접근할 수 있다.

```ts
import { PrismaClient } from "./generated/prisma/client.js";

const prisma = new PrismaClient();

const user = await prisma.user.findFirst({
  where: {
    email: "test@example.com",
  },
});
```

---

## 3. Prisma의 Connection Pool 관리 방법

### 3-1. Connection Pool이란?

Connection Pool은 DB 연결을 매번 새로 만들지 않고, 미리 만들어둔 연결을 재사용하는 방식이다.

DB 작업을 할 때마다 새 연결을 만들면 비용이 크다.  
그래서 미리 여러 개의 연결을 만들어두고, 요청이 들어올 때마다 그 연결을 빌려 쓰는 방식이 효율적이다.

---

### 3-2. 5주차 mysql2 방식

5주차에는 `mysql2`를 사용해서 직접 Pool을 만들었다.

```ts
import mysql from "mysql2/promise";

export const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  port: Number(process.env.DB_PORT),

  // 동시에 사용할 수 있는 connection 개수
  connectionLimit: 10,
});
```

이 방식에서는 직접 `pool.getConnection()`을 사용하고, 마지막에 `conn.release()`도 직접 호출해야 했다.

```ts
const conn = await pool.getConnection();

try {
  const [rows] = await conn.query("SELECT * FROM user");
  return rows;
} finally {
  // 빌린 connection을 반드시 반납해야 함
  conn.release();
}
```

---

### 3-3. Prisma 방식

Prisma를 사용하면 보통 Prisma Client를 하나 만들어서 프로젝트 전체에서 공유한다.

```ts
// src/db.config.ts

import "dotenv/config";
import { PrismaClient } from "../src/generated/prisma/client.js";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";

const adapter = new PrismaMariaDb({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,

  // DB_PORT는 문자열이므로 숫자로 바꿔준다.
  port: process.env.DB_PORT ? parseInt(process.env.DB_PORT, 10) : 3306,

  // 동시에 사용할 수 있는 connection 수
  connectionLimit: 10,
});

export const prisma = new PrismaClient({
  adapter,

  // 실제 Prisma가 실행하는 쿼리를 터미널에서 확인하기 위한 로그 설정
  log: ["query", "info", "error", "warn"],
});
```

### 중요한 점

Prisma Client는 요청이 들어올 때마다 새로 만들면 안 된다.  
보통 `db.config.ts`에서 한 번만 만들고, 필요한 Repository에서 import해서 사용한다.

```ts
// Repository에서 Prisma Client 사용
import { prisma } from "../../../db.config.js";

export const getUser = async (userId: number) => {
  return await prisma.user.findFirst({
    where: {
      id: userId,
    },
  });
};
```

---

## 4. Prisma의 Migration 관리 방법

### 4-1. Migration이란?

Migration은 DB 테이블 구조의 변경 이력을 관리하는 방법이다.

예를 들어 개발을 하다 보면 테이블에 컬럼을 추가하거나, 새로운 테이블을 만들거나, 관계를 수정해야 할 때가 있다.  
기존에는 직접 SQL을 작성해야 했다.

```sql
ALTER TABLE user ADD COLUMN password VARCHAR(255);
```

하지만 Prisma에서는 `schema.prisma`를 수정하고 migration 명령어를 실행하면, 변경 내용을 SQL 파일로 자동 생성하고 DB에도 반영할 수 있다.

---

### 4-2. migrate dev

개발 중 schema를 수정한 뒤 실행하는 명령어이다.

```bash
npx prisma migrate dev
```

역할은 다음과 같다.

- `schema.prisma` 변경 사항 확인
- migration SQL 파일 생성
- 로컬 DB에 변경 사항 적용
- Prisma Client 재생성

예시:

```bash
npx prisma migrate dev --name init_database
```

---

### 4-3. migrate deploy

운영 환경이나 테스트 서버에서 이미 만들어진 migration 파일을 적용할 때 사용한다.

```bash
npx prisma migrate deploy
```

`migrate dev`와 다르게 새 migration 파일을 만드는 용도가 아니라,  
이미 Git에 올라간 migration 파일을 DB에 적용하는 용도이다.

---

### 4-4. db push

`schema.prisma` 내용을 DB에 바로 반영하는 명령어이다.

```bash
npx prisma db push
```

빠르게 실험할 때는 편하지만, migration 파일이 생성되지 않는다.  
그래서 팀 프로젝트나 운영 환경에서는 변경 이력이 남지 않기 때문에 주의해야 한다.

---

### 4-5. migrate reset

개발 DB를 초기화할 때 사용하는 명령어이다.

```bash
npx prisma migrate reset
```

이 명령어는 DB 데이터를 모두 삭제하고 migration을 다시 적용한다.  
따라서 운영 DB에서는 절대 사용하면 안 된다.

---

### 4-6. 명령어 정리

| 명령어 | 사용 상황 | 특징 |
| --- | --- | --- |
| `npx prisma migrate dev` | 개발 중 schema 변경 | migration 생성 및 DB 반영 |
| `npx prisma migrate deploy` | 운영/배포 환경 | 이미 생성된 migration 적용 |
| `npx prisma db push` | 빠른 실험 | migration 파일 생성 안 함 |
| `npx prisma migrate reset` | 개발 DB 초기화 | 데이터 모두 삭제 |

---

### 4-7. 팀 프로젝트에서 주의할 점

팀 프로젝트에서는 migration 파일도 코드처럼 관리해야 한다.

- `schema.prisma`와 `prisma/migrations/*`를 함께 커밋해야 한다.
- 운영 환경에서는 `migrate reset`을 사용하면 안 된다.
- 운영 환경에서는 보통 `migrate deploy`를 사용한다.
- 서로 다른 팀원이 동시에 migration을 만들면 충돌이 생길 수 있다.

---

## 5. ORM, Prisma를 사용하여 좋은 점과 나쁜 점

### 5-1. 좋은 점

| 장점 | 설명 |
| --- | --- |
| SQL 직접 작성 감소 | 문자열 SQL을 직접 쓰지 않아도 된다. |
| 가독성 향상 | DB 작업을 객체 형태로 표현할 수 있다. |
| 자동완성 지원 | Prisma Client를 통해 필드 자동완성을 사용할 수 있다. |
| 타입 안정성 | TypeScript와 함께 사용하면 타입 오류를 줄일 수 있다. |
| 관계 조회 편리 | `include`를 이용해 연관 테이블을 쉽게 가져올 수 있다. |
| Migration 관리 | DB 변경 이력을 파일로 관리할 수 있다. |

---

### 5-2. 나쁜 점

| 단점 | 설명 |
| --- | --- |
| 학습 비용 | Prisma 문법과 schema 작성법을 새로 배워야 한다. |
| 복잡한 쿼리 한계 | 매우 복잡한 SQL은 직접 작성하는 것이 더 나을 수 있다. |
| 내부 쿼리 확인 필요 | ORM이 실제로 어떤 SQL을 실행하는지 확인해야 한다. |
| Migration 충돌 가능성 | 팀원 간 schema 변경이 겹치면 충돌이 날 수 있다. |
| 의도치 않은 DB 변경 | migration이나 reset 명령어 사용 시 주의가 필요하다. |

---

### 5-3. Prisma 쿼리 로그 확인

ORM을 쓰더라도 실제 어떤 SQL이 나가는지 확인하는 습관이 필요하다.

```ts
export const prisma = new PrismaClient({
  adapter,

  // Prisma가 실제 실행하는 쿼리와 에러 로그를 출력한다.
  log: ["query", "info", "warn", "error"],
});
```

이렇게 설정하면 Prisma가 내부적으로 어떤 SQL을 실행하는지 터미널에서 확인할 수 있다.

---

### 5-4. 내가 느낀 점

Prisma를 사용하면 Repository 코드가 훨씬 짧아지고 읽기 쉬워진다.  
하지만 SQL을 몰라도 된다는 뜻은 아니라고 생각한다.  
Prisma가 내부적으로 SQL을 만들어 실행하기 때문에, 성능 문제가 생기거나 복잡한 조회가 필요할 때는 결국 SQL의 기본 개념도 알고 있어야 한다.

즉, ORM은 SQL을 대체한다기보다, SQL을 더 안전하고 편하게 사용할 수 있게 도와주는 도구라고 느꼈다.

---

## 6. 다양한 ORM 라이브러리 살펴보기

Node.js 환경에서 사용할 수 있는 ORM은 Prisma 외에도 여러 가지가 있다.

| ORM | 특징 |
| --- | --- |
| Prisma | TypeScript 친화적이고 schema 기반으로 사용하기 쉽다. |
| TypeORM | 클래스와 데코레이터 기반으로 Entity를 정의한다. |
| Sequelize | 오래된 ORM으로 자료가 많고 JavaScript 프로젝트에서 많이 사용된다. |
| Drizzle ORM | 가볍고 SQL에 가까운 방식으로 작성할 수 있다. |
| MikroORM | TypeScript와 객체지향 스타일에 친화적인 ORM이다. |

---

### 6-1. Prisma 예시

```ts
// Prisma는 schema.prisma를 기반으로 생성된 Client를 사용한다.
const user = await prisma.user.findFirst({
  where: {
    id: 1,
  },
});
```

### 6-2. Sequelize 느낌의 예시

```ts
// Sequelize는 모델을 정의한 뒤 findOne 같은 메서드로 조회한다.
const user = await User.findOne({
  where: {
    id: 1,
  },
});
```

### 6-3. Drizzle 느낌의 예시

```ts
// Drizzle은 SQL에 가까운 느낌으로 타입 안전하게 작성하는 것을 목표로 한다.
const user = await db
  .select()
  .from(users)
  .where(eq(users.id, 1));
```

---

### 6-4. 내가 Prisma를 선택하는 이유

이번 UMC 워크북에서는 Prisma를 사용한다.  
개인적으로 Prisma는 `schema.prisma`로 DB 구조를 한눈에 볼 수 있고, TypeScript 자동완성이 강해서 초보자가 ORM 개념을 배우기에 좋다고 느꼈다.

다만 프로젝트 성격에 따라 ORM 선택은 달라질 수 있다.  
SQL에 가까운 제어가 필요하면 Drizzle, 객체지향적인 스타일이 익숙하면 TypeORM이나 MikroORM도 고려할 수 있다고 생각했다.

---

## 7. 페이지네이션을 사용하는 API

### 7-1. 페이지네이션이란?

페이지네이션은 데이터를 한 번에 모두 조회하지 않고, 일정 개수씩 나누어 조회하는 방식이다.

예를 들어 리뷰가 10,000개 있는데, API 한 번에 10,000개를 모두 내려주면 서버와 DB에 부담이 크다.  
그래서 보통 10개, 20개, 50개처럼 필요한 만큼만 나누어 조회한다.

---

### 7-2. 페이지네이션이 필요한 이유

| 이유 | 설명 |
| --- | --- |
| 성능 개선 | 한 번에 너무 많은 데이터를 조회하지 않는다. |
| 응답 속도 개선 | 클라이언트가 필요한 만큼만 빠르게 받을 수 있다. |
| DB 부하 감소 | 불필요한 조회를 줄일 수 있다. |
| 사용자 경험 개선 | 무한 스크롤이나 더보기 버튼 구현에 적합하다. |

---

### 7-3. 오프셋 기반 페이지네이션

오프셋 방식은 `page`, `limit` 또는 `offset`, `limit`을 사용하는 방식이다.

```text
GET /api/v1/stores/1/reviews?page=2&limit=10
```

SQL로 생각하면 다음과 비슷하다.

```sql
SELECT * FROM reviews
WHERE store_id = 1
ORDER BY id ASC
LIMIT 10 OFFSET 10;
```

### 장점

- 이해하기 쉽다.
- 페이지 번호 방식에 적합하다.

### 단점

- 데이터가 많아질수록 뒤 페이지 조회가 느려질 수 있다.
- 중간에 데이터가 추가되거나 삭제되면 페이지 결과가 밀릴 수 있다.

---

### 7-4. 커서 기반 페이지네이션

커서 방식은 마지막으로 조회한 데이터의 id를 기준으로 다음 데이터를 조회하는 방식이다.

```text
GET /api/v1/stores/1/reviews?cursor=5
```

의미는 “id가 5보다 큰 리뷰를 다음 5개만 가져와라”에 가깝다.

```ts
const reviews = await prisma.userStoreReview.findMany({
  where: {
    storeId: storeId,

    // cursor보다 id가 큰 데이터만 조회
    id: {
      gt: cursor,
    },
  },

  // id 오름차순 정렬
  orderBy: {
    id: "asc",
  },

  // 최대 5개만 조회
  take: 5,
});
```

---

### 7-5. 리뷰 목록 조회 API 예시

#### Controller

```ts
export const handleListStoreReviews = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    // Path Variable에서 storeId를 가져온다.
    const storeId = parseInt(req.params.storeId as string, 10);

    // Query String에서 cursor를 가져온다.
    // cursor가 없으면 첫 페이지이므로 0으로 처리한다.
    const cursor =
      typeof req.query.cursor === "string"
        ? parseInt(req.query.cursor, 10)
        : 0;

    // Service 호출
    const reviews = await listStoreReviews(storeId, cursor);

    res.status(StatusCodes.OK).json(reviews);
  } catch (err) {
    next(err);
  }
};
```

#### Service

```ts
export const listStoreReviews = async (
  storeId: number,
  cursor: number
): Promise<ReviewListResponse> => {
  // Repository에서 리뷰 목록 조회
  const reviews = await getAllStoreReviews(storeId, cursor);

  // DTO에서 응답 형식으로 변환
  return responseFromReviews(reviews);
};
```

#### Repository

```ts
import { prisma } from "../../../db.config.js";

export const getAllStoreReviews = async (
  storeId: number,
  cursor: number
) => {
  const reviews = await prisma.userStoreReview.findMany({
    select: {
      id: true,
      content: true,
      store: true,
      user: true,
    },

    where: {
      storeId,

      // cursor 값보다 큰 id만 가져온다.
      id: {
        gt: cursor,
      },
    },

    // 작은 id부터 순서대로 조회
    orderBy: {
      id: "asc",
    },

    // 한 번에 5개만 가져온다.
    take: 5,
  });

  return reviews;
};
```

#### DTO

```ts
export const responseFromReviews = (
  reviews: ReviewItem[]
): ReviewListResponse => {
  // 현재 응답 데이터 중 마지막 리뷰를 찾는다.
  const lastReview = reviews[reviews.length - 1];

  return {
    data: reviews,

    pagination: {
      // 다음 요청에서 사용할 cursor를 내려준다.
      // 리뷰가 없으면 더 이상 조회할 데이터가 없다는 뜻으로 null을 반환한다.
      cursor: lastReview ? lastReview.id : null,
    },
  };
};
```

---

### 7-6. API 호출 예시

첫 번째 요청은 cursor 없이 보낸다.

```text
GET /api/v1/stores/1/reviews
```

응답 예시:

```json
{
  "data": [
    {
      "id": 1,
      "content": "리뷰 1"
    },
    {
      "id": 2,
      "content": "리뷰 2"
    }
  ],
  "pagination": {
    "cursor": 2
  }
}
```

다음 요청은 응답으로 받은 cursor를 사용한다.

```text
GET /api/v1/stores/1/reviews?cursor=2
```

이렇게 하면 id가 2보다 큰 리뷰부터 다시 조회한다.

---

### 7-7. 페이지네이션을 사용하는 API 예시

페이지네이션은 다음과 같은 API에서 자주 사용된다.

| API 종류 | 페이지네이션이 필요한 이유 |
| --- | --- |
| 게시글 목록 조회 | 게시글이 많아질 수 있기 때문 |
| 댓글 목록 조회 | 댓글이 수백 개 이상 쌓일 수 있기 때문 |
| 리뷰 목록 조회 | 가게 리뷰가 많아질 수 있기 때문 |
| 알림 목록 조회 | 사용자의 알림을 한 번에 모두 가져오면 비효율적 |
| 검색 결과 조회 | 검색 결과가 많을 수 있기 때문 |
| 주문 내역 조회 | 기간이 길어지면 데이터가 많아질 수 있기 때문 |

---

### 7-8. 내가 선택할 페이지네이션 방식

이번 리뷰 목록 조회 API에서는 커서 기반 페이지네이션이 더 적합하다고 생각했다.

이유는 다음과 같다.

- 리뷰 목록은 무한 스크롤이나 더보기 방식으로 보여주기 좋다.
- 데이터가 많아져도 offset 방식보다 안정적으로 다음 데이터를 가져올 수 있다.
- 마지막으로 받은 리뷰 id를 기준으로 다음 리뷰를 조회하면 흐름이 단순하다.

따라서 이번 주차에서는 `cursor`를 Query String으로 받아서, 해당 cursor보다 큰 id를 가진 리뷰를 일정 개수만 조회하는 방식으로 구현하는 것이 좋다고 생각했다.

---

## 8. 6주차 핵심 정리

이번 6주차에서 가장 중요하게 느낀 점은 다음과 같다.

1. ORM은 SQL을 직접 작성하는 부담을 줄이고 DB 작업을 코드로 더 안전하게 관리하게 해준다.
2. Prisma는 `schema.prisma`를 중심으로 DB 구조와 Prisma Client를 관리한다.
3. Prisma Client를 사용하면 `findFirst`, `create`, `findMany`, `include` 같은 메서드로 DB 작업을 할 수 있다.
4. Migration을 사용하면 DB 테이블 변경 이력을 파일로 관리할 수 있다.
5. ORM은 편리하지만 실제 실행되는 SQL과 성능도 같이 신경 써야 한다.
6. 목록 API에서는 페이지네이션을 적용해 DB 부하와 응답 크기를 줄여야 한다.

이번 주차는 단순히 Prisma 문법을 배우는 것이 아니라,  
**DB를 어떻게 더 안전하고 유지보수하기 쉽게 다룰 수 있는지 배우는 주차**였다고 생각한다.