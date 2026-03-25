ERD에디터 사용방법: https://leeeeeyeon-dev.tistory.com/79

<!-- 캔버스 기본화면  -->
![alt text](image.png)

<!-- 테이블 생성 -->
![alt text](image-1.png)
![alt text](image-2.png)
![alt text](image-3.png)

<!-- ERD.vuerd.json파일에서 erd설계를 진행해보았습니다 -->
![alt text](image-4.png)

# 1주차 미션 DB 설계

## 1. 서비스 개요
이번 미션은 각 지역별로 가게가 존재하고, 사용자가 가게와 연결된 미션을 수행하며 포인트를 모으는 서비스를 기준으로 데이터베이스를 설계하는 것이다.  
사용자는 로그인 및 회원가입 이후 홈 화면에서 자신의 정보와 진행 중인 미션을 확인할 수 있고, 미션을 수행한 뒤 완료 처리 및 리뷰 작성이 가능하다고 가정하였다.

## 2. 설계 범위
이번 설계에서는 아래 기능을 중심으로 데이터베이스를 구성하였다.

- 로그인 및 회원가입
- 홈 화면에서 사용자 정보 조회
- 지역별 가게 조회
- 미션 조회 및 수행 상태 관리
- 미션 완료 후 리뷰 작성

## 3. 설계 제외 항목
워크북 안내에 따라 아래 기능은 설계 대상에서 제외하였다.

- 지도 검색 기능
- 알림 기능
- 포인트 상세 내역 관리
- 사장님 점포 관리 기능

## 4. ERD 설계 초안
아래는 ERD를 설계하기 전에 관계를 정리한 구조이다.

![alt text](unnamed-2026-03-17T16_30_05.png)
![alt text](image-6.png)
```graphql
type Mission {
  Id: ID!
  Title: String
  Description: String
  RewardPoint: Int
  CreatedAt: String
  UpdateAt: String
  Store: Store
  UserMissionList: [UserMission!]!
  ReviewList: [Review!]!
}

type Region {
  Id: ID!
  Name: String
  CreatedAt: String
  UpdateAt: String
  DeletedAt: String
  StoreList: [Store!]!
}

type Review {
  Id: ID!
  Rating: Int
  Content: String
  CreatedAt: String
  UpdateAt: String
  User: User
  Mission: Mission
}

type Store {
  Id: ID!
  Name: String
  Address: String
  CreatedAt: String
  UpdateAt: String
  Region: Region
  MissionList: [Mission!]!
}

type User {
  Id: ID!
  Provider: String
  Providerid: String
  Name: String
  Nickname: String
  Phone: String
  Gender: String
  Status: String
  Point: Int
  CreatedAt: String
  UpdateAt: String
  DeletedAt: String
  UserMissionList: [UserMission!]!
  ReviewList: [Review!]!
}

type UserMission {
  Id: ID!
  Status: String
  StartedAt: String
  CompletedAt: String
  CreatedAt: String
  UpdateAt: String
  User: User
  Mission: Mission
}
```

## 5. 테이블별 설명
- User

사용자 정보를 저장하는 테이블이다.
카카오 소셜 로그인을 고려하여 provider와 providerId를 두었고, 회원 탈퇴는 바로 삭제하지 않고 status와 deletedAt으로 관리하도록 설계하였다.
또한 홈 화면에서 현재 포인트를 보여주기 위해 point 컬럼을 두었다.

- Region

가게가 속한 지역 정보를 저장하는 테이블이다.
하나의 지역에는 여러 가게가 존재할 수 있다.

## 각 컬럼을 이렇게 둔 이유

Id
가게를 고유하게 식별하기 위한 기본키야.

Name
가게 이름을 저장하는 컬럼이야.

Address
사용자가 어떤 가게인지 구분하려면 주소 정보가 필요해.
특히 같은 이름의 가게가 여러 군데 있을 수 있으므로 주소는 중요한 정보야.

CreatedAt / UpdatedAt
가게 정보 생성 및 수정 시점을 관리하기 위해 필요해.

Region
가게가 어느 지역에 속하는지 나타내는 관계야.
실제 DB로 만들 때는 region_id 외래키로 들어간다고 생각하면 돼.

MissionList
한 가게는 여러 개의 미션을 가질 수 있으므로 Store와 Mission은 일대다 관계가 돼.
그래서 가게 하나에 여러 미션이 연결된다는 걸 표현한 거야.

- Store

가게 정보를 저장하는 테이블이다.
각 가게는 하나의 지역에 속한다고 가정하였다.

- Mission

가게와 연결된 미션 정보를 저장하는 테이블이다.
한 가게는 여러 개의 미션을 가질 수 있으며, 미션마다 제목, 설명, 보상 포인트를 가진다.

- UserMission

사용자와 미션의 관계를 관리하는 중간 테이블이다.
사용자와 미션은 다대다 관계이므로 별도의 테이블로 분리하였다.
여기서 미션 진행 상태와 완료 여부를 관리한다.

- Review

사용자가 완료한 미션에 대해 작성하는 리뷰를 저장하는 테이블이다.
리뷰는 사용자와 미션에 각각 연결되도록 설계하였다.

## 6. 관계 정리
- Region 1 : N Store

- Store 1 : N Mission

- User 1 : N UserMission

- Mission 1 : N UserMission

- User 1 : N Review

- Mission 1 : N Review

## 7. 설계 의도

이번 설계에서는 먼저 서비스에서 꼭 필요한 기능인 회원가입, 홈, 미션 중심으로 구조를 단순하게 잡는 것을 목표로 하였다.
특히 사용자와 미션의 관계는 한 사용자가 여러 미션을 수행할 수 있고, 하나의 미션도 여러 사용자에게 연결될 수 있으므로 다대다 관계로 보고 UserMission 테이블로 분리하였다.
또한 리뷰는 미션 완료 이후 작성된다고 가정하여 User와 Mission 모두를 참조하도록 설계하였다.

## 8. 느낀 점

이번 미션을 진행하면서 단순히 테이블을 나누는 것보다, 서비스 화면에서 실제로 어떤 데이터가 필요하고 어떤 관계가 필요한지 먼저 생각하는 것이 중요하다는 점을 느꼈다.
특히 다대다 관계를 어떻게 풀 것인지, 그리고 어떤 기능을 이번 설계 범위에서 제외할 것인지 명확하게 정하는 과정이 필요하다고 생각했다.