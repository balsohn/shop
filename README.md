# 이커머스 플랫폼

Spring Boot 기반의 온라인 쇼핑몰 웹 애플리케이션입니다. 계층형 카테고리, 상품 관리, 회원 활동 기능을 갖춘 종합 이커머스 플랫폼입니다.

## 프로젝트 개요

- **개발 기간**: 2024.08 - 2024.10
- **개발 인원**: 백엔드 개발 담당
- **기술 스택**: Java, Spring Boot, MyBatis, MariaDB, JSP, JavaScript, jQuery, HTML/CSS, Spring Scheduler

## 주요 기능

### 1. 계층형 카테고리 관리 시스템
- 대분류/중분류/소분류 3단계 계층형 카테고리 구조
- 카테고리별 상품 조회 및 필터링 기능
- 동적 카테고리 메뉴 구현

```java
// 계층형 카테고리 조회 SQL 쿼리
<select id="getCate" resultType="kr.co.shop.dto.CateDTO"> 
SELECT 
    d.name AS daeName, 
    j.name AS jungName, 
    s.name AS soName
FROM 
    dae d
LEFT JOIN 
    jung j ON d.code = j.daecode
LEFT JOIN 
    so s ON CONCAT(d.code, j.code) = s.daejung
ORDER BY 
    d.id, j.id, s.id;
</select>
```

### 2. 상품 관리 시스템
- 상품 카탈로그 관리 (이미지, 가격, 할인율, 적립금 등)
- 상품 검색 및 정렬 기능 (최신순, 가격순, 인기순, 평점순)
- 타임세일 상품 관리 및 스케줄링

```java
// 타임세일 상품 조회
<select id="getProduct1" resultType="kr.co.shop.dto.ProductDTO">
    select * from product
    where now() &lt; salesDay
    order by rand() desc limit 4
</select>
```

### 3. 회원 활동 기능
- 회원 가입 및 로그인/로그아웃
- 장바구니 관리 (추가, 수량 조절, 삭제)
- 찜하기 기능
- 적립금 관리 시스템

### 4. 주문 및 결제 시스템
- 다양한 결제 수단 지원 (카드, 무통장입금, 간편결제 등)
- 배송지 관리 (기본 배송지, 배송 요청사항 등)
- 주문 내역 조회 및 상태 변경

### 5. 리뷰 및 문의 시스템
- 상품별 별점 및 리뷰 관리
- 평균 별점 자동 계산 기능
- 상품 Q&A 기능

## 기술적 특징

### 1. 비로그인 사용자를 위한 쿠키 기반 장바구니 구현
```java
// 비로그인 사용자를 위한 쿠키 기반 장바구니
if(session.getAttribute("userid")==null) {
    Cookie cookie=WebUtils.getCookie(request, "pcode");
    String newPro=pcode+"-"+su+"/";
    
    String newPcode=null;
    if(cookie==null || cookie.getValue().isEmpty()) {
        newPcode=newPro;
        cartNum="1";
    } else {
        // 기존 쿠키에 상품 추가 로직
        // ...
    }
    
    Cookie newCookie=new Cookie("pcode",newPcode);
    newCookie.setMaxAge(600);
    newCookie.setPath("/");
    response.addCookie(newCookie);
    
    return cartNum;
}
```

### 2. 예상 배송일 계산 알고리즘
```java
// 예상 배송일 계산
LocalDate today=LocalDate.now();
LocalDate xday=today.plusDays(pdto.getBaeday());
String yoil=MyUtils.getYoil(xday);

String baeEx=null;
if(pdto.getBaeday()==1) {
    baeEx="내일("+yoil+") 도착예정";
} else if(pdto.getBaeday()==2) {
    baeEx="모레("+yoil+") 도착예정";
} else {
    int m=xday.getMonthValue();
    int d=xday.getDayOfMonth();
    baeEx=m+"/"+d+"("+yoil+") 도착예정";
}
```

### 3. 스케줄링을 활용한 자동화
```java
@Service
public class MyScheduler {
    // fixedRate, fixedDelay, cron 표현식 활용
    @Scheduled(fixedRate = 1000)
    public void test() {
        // 정기적 작업 수행 로직
    }
}

// Spring Boot 애플리케이션에 스케줄링 활성화
@SpringBootApplication
@EnableScheduling
public class ShopApplication {
    //...
}
```

### 4. SiteMesh를 활용한 일관된 레이아웃 관리
```java
public class SitemeshConfig extends ConfigurableSiteMeshFilter {
    @Override
    protected void applyCustomConfiguration(SiteMeshFilterBuilder builder) {
        builder.addDecoratorPath("*", "/default.jsp");
        builder.addExcludedPath("/product/jusoWrite");
        builder.addExcludedPath("/product/jusoList");
        builder.addExcludedPath("/product/jusoUpdate");
    }
}
```

### 5. Lombok을 활용한 간결한 DTO 관리
```java
@Data
public class ProductDTO {
    private String pcode,pimg,dimg,title,writeday;
    private int id, price, halin, su, baeprice, baeday, juk, pansu, review;
    private double star;
    
    private String baeEx;
    private int halinPrice, jukPrice;
    
    private int ystar,gstar,hstar;
    
    // 타임세일 관련
    private int sales;
    private String salesDay;
}
```

## 프로젝트 구조
```
shop/
├── src/main/java/kr/co/shop/
│   ├── controller/
│   │   ├── LoginController.java
│   │   ├── MainController.java
│   │   ├── MemberController.java
│   │   └── ProductController.java
│   ├── dto/
│   │   ├── ProductDTO.java
│   │   ├── MemberDTO.java
│   │   ├── CateDTO.java
│   │   └── ... (기타 DTO 클래스)
│   ├── mapper/
│   │   ├── ProductMapper.java/xml
│   │   ├── MemberMapper.java/xml
│   │   └── ... (기타 Mapper 클래스)
│   ├── service/
│   │   ├── ProductService.java
│   │   ├── ProductServiceImpl.java
│   │   └── ... (기타 Service 클래스)
│   ├── utils/
│   │   └── MyUtils.java
│   ├── MyScheduler.java
│   ├── ShopApplication.java
│   └── SitemeshConfig.java
└── src/main/resources/
    ├── application.properties
    └── mapper/
        ├── ProductMapper.xml
        ├── MemberMapper.xml
        └── ... (기타 Mapper XML 파일)
```

## 배운 점 및 향후 개선 방향

### 배운 점
- 계층형 카테고리 DB 설계 및 조회 최적화
- 쿠키와 세션을 활용한 장바구니 구현
- MyBatis를 활용한 동적 쿼리 작성
- 스케줄링을 활용한 타임세일 상품 관리
- SiteMesh를 통한 프론트엔드 레이아웃 관리

### 향후 개선 방향
- 장바구니 동시성 제어 강화 (트랜잭션 처리)
- REST API 기반 리팩토링
- 검색 기능 고도화 (검색어 자동완성, 필터링 등)
- 실시간 재고 관리 시스템 구현
- 모니터링 및 로깅 시스템 강화
