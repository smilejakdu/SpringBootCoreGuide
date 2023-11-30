# 08 Query DSL

쿼리의 문자열이 잘못된 경우에는 애플리케이션이 실행된 후 로직이 실행되고 나서야 오류를 발견할 수 있습니다.



쿼리의 문자열이 잘못된 경우에는 애플리케이션이 실행된 후 로직이 실행되고 나서야 오류를 발견할 수 있습니다.

이러한 이유로 개발 환경에서는 문제가 없는 것처럼 보이다가 실제 운영 환경에 애플리케이션을 배포하고 나서 오류가 발견되는 리스크를 유발합니다.



위와 같은 문제로 QueryDSL이 나오게 됐습니다.

QueryDSL은 문자열이 아니라 코드로 쿼리를 작성할 수 있도록 도와준다.



## ✅ QueryDSL 이란 ?

정적 타입을 이용해 SQL과 같은 쿼리를 생성할 수 있도록 지원하는 프레임워크이다.



### 📌 QueryDSL 의 장점

QueryDSL을 사용하면 다음과 같은 장점이 있다.

* 코드 자동 완성 기능
* 문법적으로 잘못된 쿼리를 허용하지 않습니다.
* 고정된 SQL 쿼리를 작성하지 않기 때문에 동적으로 쿼리를 생성할 수 있습니다.
* 코드로 작성하므로 가독성 및 생산성 향상이 됩니다.
* 도메인 타입과 프로퍼티를 안전하게 참조할 수 있습니다.

### 📌 QueryDSL 설정

{% embed url="https://myvelop.tistory.com/213" %}

build.gradle 파일에

```java
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.0.1'
    id 'io.spring.dependency-management' version '1.1.0'
    // querydsl관련 명령어를 gradle탭에 생성해준다. (권장사항)
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
    maven { url 'https://repo.spring.io/snapshot' }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'mysql:mysql-connector-java:8.0.28'
    implementation 'org.springframework.boot:spring-boot-devtools'
    implementation 'org.hibernate.validator:hibernate-validator'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'	//for jpa
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.5'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.11.5'

    // QueryDSL Implementation
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
}

test {
    useJUnitPlatform()
}

/**
 * QueryDSL Build Options
 */
def querydslDir = "src/main/generated"

sourceSets {
    main.java.srcDirs += [ querydslDir ]
}


tasks.withType(JavaCompile) {
    options.getGeneratedSourceOutputDirectory().set(file(querydslDir))
}

clean.doLast {
    file(querydslDir).deleteDir()
}

tasks.named('test') {
    useJUnitPlatform()
}
```

위와 같이 build.gradle 을 추가한 후에

빌드를 진행한다.

<figure><img src=".gitbook/assets/스크린샷 2023-11-30 오후 10.32.11.png" alt=""><figcaption></figcaption></figure>

그러면 generated 폴더가 생기게된다.

<figure><img src=".gitbook/assets/스크린샷 2023-11-30 오후 10.32.41.png" alt="" width="351"><figcaption></figcaption></figure>

build.gradle 에서 경로는 src/main/generated 로 설정했기때무에 generated 폴더가 생기게 됐다.



이후에&#x20;

```java
package com.example.showmeyourability.teacher.application;

import com.example.showmeyourability.comments.domain.QComments;
import com.example.showmeyourability.comments.infrastructure.dto.FindCommentDto.CommentDto;
import com.example.showmeyourability.teacher.domain.QTeacher;
import com.example.showmeyourability.teacher.domain.Teacher;
import com.example.showmeyourability.teacher.infrastructure.dto.FindTeacherDto.FindTeacherByIdResponseDto;
import com.example.showmeyourability.teacher.infrastructure.dto.FindTeacherDto.FindTeacherResponseDto;
import com.example.showmeyourability.teacher.infrastructure.dto.FindTeacherDto.TeacherDto;
import com.example.showmeyourability.teacher.infrastructure.repository.TeacherRepository;
import com.querydsl.core.types.Projections;
import com.querydsl.jpa.impl.JPAQueryFactory;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class FindTeacherApplication {
    private final JPAQueryFactory queryFactory; // JPAQueryFactory 주입
    private final TeacherRepository teacherRepository;

    private final QTeacher qTeacher = QTeacher.teacher; // 클래스 수준의 QTeacher 인스턴스
    private final QComments qComments = QComments.comments; // 클래스 수준의 QComments 인스턴스


    @Transactional
    public FindTeacherResponseDto findAllTeacher(int page, int size) {
        // 교사 정보와 평균 점수를 함께 조회
        List<TeacherDto> teacherDtos = queryFactory
                .select(Projections.constructor(TeacherDto.class,
                        qTeacher.id,
                        qTeacher.career,
                        qTeacher.user.email, // Email 필드 추가
                        qTeacher.skill,
                        qTeacher.user.id,
                        qComments.likes.avg().coalesce(0.0) // 평균 점수 계산 및 null 처리
                ))
                .from(qTeacher)
                .leftJoin(qTeacher.comments, qComments)
                .groupBy(qTeacher.id)
                .orderBy(qTeacher.user.email.asc())
                .offset((long) page * size)
                .limit(size)
                .fetch();

        // 총 페이지 수 계산
        long total = queryFactory
                .selectFrom(qTeacher)
                .fetchCount();

        int lastPage = (int) Math.ceil((double) total / size);

        return convertToFindTeacherResponseDto(lastPage, teacherDtos);
    }

    private FindTeacherResponseDto convertToFindTeacherResponseDto(
            int lastPage,
            List<TeacherDto> teachers
    ) {
        return FindTeacherResponseDto.builder()
                .lastPage(lastPage)
                .teachers(teachers)
                .build();
    }

    @Transactional
    public FindTeacherByIdResponseDto findOneTeacherById(
            Long teacherId
    ) {
        Teacher teacher = queryFactory
                .selectFrom(QTeacher.teacher)
                // Teacher의 ID가 메서드 파라미터로 전달된 teacherId와 같은 경우를 조건으로 합니다.
                .where(QTeacher.teacher.id.eq(teacherId))
                .fetchOne();

        List<CommentDto> commentDtos = queryFactory.selectFrom(qComments)
                .where(qComments.teacher.eq(teacher))
                .fetch()
                .stream()
                .map(comment -> CommentDto.builder()
                        .id(comment.getId())
                        .content(comment.getContent())
                        .likes(comment.getLikes())
                        .userId(comment.getUser().getId())
                        .build())
                .collect(Collectors.toList());

        return FindTeacherByIdResponseDto.builder()
                .teacher(teacher)
                .commentDtoList(commentDtos)
                .build();
    }
}

```

