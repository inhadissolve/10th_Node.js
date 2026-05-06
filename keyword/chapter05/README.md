# 5주차 핵심 키워드 정리

---

## 1. 환경 변수

환경 변수는 프로젝트에서 사용하는 설정값이나 민감한 정보를 코드와 분리해서 관리하기 위한 방법이다.

예를 들어 서버 포트 번호, DB 주소, DB 비밀번호 같은 값은 코드에 직접 적으면 위험하다.  
이런 값들은 `.env` 파일에 따로 저장하고, 코드에서는 `process.env`를 통해 불러와 사용한다.

### 예시

```env
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=비밀번호
DB_NAME=umc_week5
```

### 코드에서 사용하는 방법

```ts
import dotenv from "dotenv";

dotenv.config();

const port = process.env.PORT || 3000;
```

### 환경 변수를 쓰는 이유

환경 변수를 사용하면 개발 환경과 배포 환경의 설정을 쉽게 분리할 수 있다.  
또 DB 비밀번호나 API Key 같은 민감한 정보를 코드에 직접 노출하지 않을 수 있다.

### 주의할 점

`.env` 파일은 절대 GitHub에 올라가면 안 된다.  
그래서 `.gitignore`에 반드시 아래 내용을 추가해야 한다.

```gitignore
.env
.env.*
node_modules/
```

---

## 2. CORS

CORS는 Cross-Origin Resource Sharing의 약자이다.  
브라우저에서 서로 다른 출처의 서버에 요청을 보낼 때 발생할 수 있는 보안 정책과 관련된 개념이다.

예를 들어 프론트엔드가 `localhost:5173`에서 실행되고, 백엔드 서버가 `localhost:3000`에서 실행된다면 두 주소는 서로 다른 origin으로 인식될 수 있다.  
이때 백엔드에서 CORS를 허용하지 않으면 브라우저가 요청을 막을 수 있다.

### Express에서 CORS 설정

```ts
import cors from "cors";

app.use(cors());
```

### 내가 이해한 CORS

CORS는 서버가 “이 출처에서 온 요청은 허용해도 된다”고 알려주는 설정이라고 이해했다.  
프론트엔드와 백엔드가 분리된 프로젝트에서는 거의 필수로 고려해야 하는 부분이라고 느꼈다.

---

## 3. DB Connection

DB Connection은 서버와 데이터베이스가 데이터를 주고받기 위해 맺는 연결이다.

API 서버가 DB에서 데이터를 조회하거나 저장하려면 먼저 DB와 연결되어 있어야 한다.  
이번 주차에서는 `mysql2/promise`를 사용해서 MySQL과 연결했다.

### 예시

```ts
import mysql from "mysql2/promise";
import dotenv from "dotenv";

dotenv.config();

export const pool = mysql.createPool({
  host: process.env.DB_HOST || "localhost",
  user: process.env.DB_USER || "root",
  port: parseInt(process.env.DB_PORT || "3306"),
  database: process.env.DB_NAME || "umc_week5",
  password: process.env.DB_PASSWORD || "password",
});
```

DB 연결 정보는 코드에 직접 적는 것이 아니라 `.env`에서 가져오는 방식이 더 안전하다.

---

## 4. DB Connection Pool

Connection Pool은 DB 연결을 매번 새로 만들고 끊는 것이 아니라, 미리 여러 개의 연결을 만들어두고 필요할 때 빌려 쓰는 방식이다.

### Connection을 매번 새로 만들면 생길 수 있는 문제

API 요청이 들어올 때마다 DB 연결을 새로 만들면 비효율적이다.  
요청이 많아질수록 DB 연결 생성과 종료 비용이 커지고, 서버 성능에도 부담이 생길 수 있다.

### Connection Pool의 장점

미리 만들어둔 연결을 재사용할 수 있다.  
DB 연결 비용을 줄일 수 있다.  
많은 요청이 들어와도 더 안정적으로 처리할 수 있다.

### 내가 이해한 비유

매번 택시를 새로 부르는 것보다, 대기 중인 택시를 바로 타는 것이 더 빠른 것과 비슷하다고 이해했다.

---

## 5. 비동기 처리와 async / await

Node.js 서버에서는 DB 조회, 파일 읽기, 외부 API 호출처럼 시간이 걸리는 작업이 많다.  
이런 작업을 기다리는 동안 서버 전체가 멈추면 안 되기 때문에 비동기 처리가 필요하다.

`async / await`은 비동기 코드를 동기 코드처럼 읽기 쉽게 작성할 수 있게 해준다.

### 예시

```ts
export const getUser = async (userId: number) => {
  const user = await userRepository.findById(userId);
  return user;
};
```

### async

함수 앞에 붙이면 해당 함수는 Promise를 반환한다.

### await

Promise가 처리될 때까지 기다린 뒤 결과를 반환한다.  
단, `await`은 `async` 함수 안에서 사용해야 한다.

### 내가 느낀 점

DB 작업은 대부분 비동기로 처리되기 때문에, Express와 DB를 연결하는 프로젝트에서는 `async / await`이 거의 필수라고 느꼈다.

---

## 6. try / catch / finally

`try / catch / finally`는 오류 처리를 위한 문법이다.

DB 작업이나 외부 API 호출은 언제든 실패할 수 있기 때문에, 오류 상황을 고려해서 코드를 작성해야 한다.

### 예시

```ts
try {
  const [rows] = await pool.query("SELECT * FROM user");
  return rows;
} catch (err) {
  throw new Error(`오류가 발생했습니다: ${err}`);
} finally {
  conn.release();
}
```

### try

정상적으로 실행할 코드를 작성한다.

### catch

오류가 발생했을 때 실행된다.

### finally

성공하든 실패하든 무조건 실행된다.

### DB 작업에서 finally가 중요한 이유

DB connection을 빌려왔다면 반드시 반납해야 한다.  
반납하지 않으면 connection이 계속 쌓여서 서버가 느려지거나 문제가 생길 수 있다.

그래서 DB 작업에서는 `finally`에서 `conn.release()`를 호출하는 것이 중요하다.

---

## 7. Interface

Interface는 TypeScript에서 객체의 구조를 정의하는 문법이다.

이번 주차에서는 회원가입 요청 데이터의 구조를 정의하기 위해 Interface를 사용했다.

### 예시

```ts
export interface UserSignUpRequest {
  email: string;
  name: string;
  gender: string;
  birth: string;
  address?: string;
  detailAddress?: string;
  phoneNumber: string;
  preferences: number[];
}
```

### 의미

`email`, `name`, `gender`, `birth`, `phoneNumber`, `preferences`는 필수값이다.  
`address`, `detailAddress`는 `?`가 붙어 있으므로 선택값이다.

### Interface를 쓰는 이유

요청 데이터가 어떤 형태인지 명확하게 알 수 있다.  
오타나 타입 오류를 개발 단계에서 미리 잡을 수 있다.  
Controller, Service 사이에서 데이터 구조를 통일할 수 있다.

---

## 8. Type Assertion과 as 키워드

Type Assertion은 TypeScript에게 “이 값은 특정 타입으로 봐도 된다”고 알려주는 문법이다.

Express의 `req.body`는 기본적으로 타입이 명확하지 않다.  
그래서 TypeScript 입장에서는 어떤 값이 들어오는지 알 수 없다.

이럴 때 `as` 키워드를 사용해 타입을 지정할 수 있다.

### 예시

```ts
const userData = req.body as UserSignUpRequest;
```

또는 DTO 변환 함수에 바로 넘길 수도 있다.

```ts
const user = await userSignUp(bodyToUser(req.body as UserSignUpRequest));
```

### 주의할 점

Type Assertion은 실제 검증을 해주는 기능은 아니다.  
TypeScript에게 타입을 알려주는 역할에 가깝다.

즉, `as UserSignUpRequest`를 썼다고 해서 실제 요청 body가 반드시 올바른 값이라는 뜻은 아니다.  
그래서 실무에서는 추가적인 유효성 검사도 필요하다.

---

## 9. Controller, DTO, Service, Repository 역할 정리

이번 주차에서 가장 중요하게 느낀 부분은 계층별 역할 분리였다.

### Controller

클라이언트의 요청을 받는다.  
요청 데이터를 확인한다.  
Service를 호출한다.  
응답을 반환한다.

### DTO

요청 데이터나 응답 데이터를 정리한다.  
데이터의 형태를 명확하게 만든다.  
Controller와 Service 사이의 데이터 전달을 깔끔하게 만든다.

### Service

실제 비즈니스 로직을 처리한다.  
예를 들어 회원가입에서는 이메일 중복 확인, 사용자 저장, 선호 카테고리 연결 같은 흐름을 담당한다.

### Repository

DB와 직접 소통한다.  
SQL 쿼리를 실행하고 결과를 반환한다.

---