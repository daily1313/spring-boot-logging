## Logging

### 사용 이유
  - 에러가 발생할 때, 해당 내용을 재현하기 위해 사용
  - 로그 레벨에 따라 남기고 싶은 로그를 별도로 지정할 수 있습니다.
  - 출력 형식을 지정할 수 있습니다. 
  - 콘솔 뿐만 아니라, 파일, 네트워크 등 로그를 별도의 위치에 남길 수 있습니다.
  - 이러한 이유 때문에 성능이 System.out.println보다 좋습니다.
  
### application.properties 설정 (log 작업 환경 분리 및 Package 별로 Log Level 설정)

    spring.config.activate.on-profile=local
    logging.level.org.springframework.boot.autoconfigure.logging=INFO
    logging.level.com.example.springbootlogging.LoggingController=debug
    logging.config=classpath:logback-spring.xml

### logback-local.properties (로그 파일 저장 경로 및 파일 이름 설정)
    
    log.config.path=./logs
    log.config.filename=local_log

### VM Option

    -Dspring.profiles.active=local
### logback-spring.xml (logging 작업을 위한 xml 파일)
    
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration scan="true" scanPeriod="60 seconds">
        <springProfile name="local">
            <property resource="logback-local.properties"/>
        </springProfile>
        <springProfile name="dev">
            <property resource="logback-dev.properties"/>
        </springProfile>
        <springProperty scope="context" name="LOG_LEVEL" source="logging.level.root"/>
    
        <property name="LOG_PATH" value="${log.config.path}"/>
        <property name="LOG_FILE_NAME" value="${log.config.filename}"/>
        <property name="ERR_LOG_FILE_NAME" value="err_log"/>
        <property name="LOG_PATTERN" value="%-5level %d{yy-MM-dd HH:mm:ss}[%thread] [%logger{0}:%line] - %msg%n"/>
    
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>${LOG_PATTERN}</pattern>
            </encoder>
        </appender>
    
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    
            <file>${LOG_PATH}/${LOG_FILE_NAME}.log</file>
    
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>${LOG_PATTERN}</pattern>
            </encoder>
    
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
                <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                    <maxFileSize>10MB</maxFileSize>
                </timeBasedFileNamingAndTriggeringPolicy>
               
                <maxHistory>30</maxHistory>
                <!--<MinIndex>1</MinIndex>
                <MaxIndex>10</MaxIndex>-->
            </rollingPolicy>
        </appender>
    
        <appender name="Error" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>error</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            
            <file>${LOG_PATH}/${ERR_LOG_FILE_NAME}.log</file>
            
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>${LOG_PATTERN}</pattern>
            </encoder>
            
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${ERR_LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
                <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                    <maxFileSize>10MB</maxFileSize>
                </timeBasedFileNamingAndTriggeringPolicy>
                <maxHistory>60</maxHistory>
            </rollingPolicy>
        </appender>
    
        <root level="${LOG_LEVEL}">
            <appender-ref ref="CONSOLE"/>
    <!--        <appender-ref ref="FILE"/>-->
    <!--        <appender-ref ref="Error"/>-->
        </root>
    </configuration>

### log-level 
- TRACE: 추적 레벨은 Debug보다 훨씬 상세한 정보를 나타냅니다.
- DEBUG: 프로그램을 디버깅하기 위한 정보를 표시
- INFO: 요구사항에 따라 시스템 동작을 확인, 상태 변경과 같은 정보성 로그 표시
- WARN: 에러가 될 수도 있는 잠재적 가능성이 있는 경우
- ERROR: 요청을 처리하는 중 오류가 발생
- 출력 레벨의 설정에 따라 설정 레벨 이상의 로그를 출력 

### log pattern

    %logger: 패키지 포함 클래스 정보
    %logger{0}: 패키지를 제외한 클래스 이름만 출력
    %logger{length}: Logger name을 축약할 수 있음. {length}는 최대 자리 수, ex)logger{35}
    %-5level: 로그 레벨, -5는 출력의 고정폭 값(5글자), 로깅레벨이 Info일 경우 빈칸 하나 추가
    ${PID:-}: 프로세스 아이디
    %d: 로그 기록시간 출력
    %p: 로깅 레벨 출력
    %wex: 예외 출력
    %F: 로깅이 발생한 프로그램 파일명 출력
    %M: 로깅일 발생한 메소드의 명 출력
    %line: 로깅이 발생한 호출지의 라인
    %L: 로깅이 발생한 호출지의 라인
    %thread: 현재 Thread 명
    %t: 로깅이 발생한 Thread 명
    %c: 로깅이 발생한 카테고리
    %C: 로깅이 발생한 클래스 명 (%C{2}는 somePackage.SomeClass 가 출력됩니다.)
    %m: 로그 메시지
    %msg: - 로그 메시지 (=%message)
    %n: 줄바꿈(new line)
    %%: %를 출력
    %r : 애플리케이션 시작 이후부터 로깅이 발생한 시점까지의 시간(ms)
    %d{yyyy-MM-dd-HH:mm:ss:sss}: %d는 date를 의미하며 중괄호에 들어간 문자열은 dateformat을 의미
    %-4relative: %relative는 초 아래 단위 시간(밀리초)을 나타냄. -4를하면 4칸의 출력폼을 고정으로 가지고 출력

### appender, logger, layout

#### appender 
- 콘솔, 파일 DB 등 로그를 출력하는 방법 지정 (어디에 기록할지 지정)

#### logger
- 실제로 로깅 작업이 이루어지는 곳

#### layout
- 로그를 어떤 형식으로 출력할 것인지 결정

### Output Log

- %-5level %d{yy-MM-dd HH:mm:ss}[%thread][%logger{0}:%line] - %msg%n
- DEBUG 24-02-08 17:17:24[http-nio-8080-exec-1][LoggingController:16] - A DEBUG Message

#### Console
<img width="600" alt="스크린샷 2024-02-08 오후 5 20 56" src="https://github.com/daily1313/spring-boot-logging/assets/88074556/a94565a6-43f6-49fb-bc70-ac3575e58458">


#### File

<img width="600" alt="스크린샷 2024-02-08 오후 5 19 56" src="https://github.com/daily1313/spring-boot-logging/assets/88074556/5f87a9e4-25e3-45a9-998a-a88841652e4f">
<img width="600" alt="스크린샷 2024-02-08 오후 5 20 18" src="https://github.com/daily1313/spring-boot-logging/assets/88074556/66d1fb5b-fe6f-4098-a753-94877e8444dd">
