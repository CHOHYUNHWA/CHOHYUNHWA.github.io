---
layout: post
title:  "Spring과 Redis를 이용한 이메일 인증 기능 구현"
date:   2023-06-02 14:42:00 +0900
categories: spring
tags: spring
---

## Spring과 Redis를 이용한 이메일 인증 기능 구현

지금하고 있는 사이드 프로젝트 내 요구사항 정의서에는 이메일 인증기능이 정의되어있다.<br>
이번 포스팅에서는 이메일 인증구현 로직과 코드에 대해서 다뤄 보려고 한다.<br>
메일 발송을 위한 ```JavaMailSender```라이브러리와 인증번호를 임시로 저장할 캐시메모리 ```Redis```를 사용하여 구현하였다.

### 이메일 인증 기능 구현 로직

1. 사용자의 이메일을 Request Data로 받는다.
2. 알파벳 + 숫자를 이용하여 랜덤한 인증코드 6자리를 생성한다.
3. 사용자에게 이메일로 인증코드를 발송함과 동시에 Redis에 인증번호를 저장한다.
4. 사용자가 이메일로 받은 인증코드를 입력한다.
5. 입력된 인증코드를 Redis에 저장된 인증코드가 일치하는지 대조한다.

### 인증 기능 구현 코드

디렉토리 구조

* 레디스 패키지

redis<br>
├── config<br>
│   └── RedisConfig.class<br>
└── util<br>
      └── RedisUtil.class

* 메일 패키지

mail<br>
├── Dto<br>
│   ├── MailDto$AuthEmail.class<br>
│   ├── MailDto$FindEmail.class<br>
│   ├── MailDto$Key.class<br>
│   └── MailDto.class<br>
├── controller<br>
│   └── MailController.class<br>
└── service<br>
    └── MailService.class<br>


### 레디스 소스파일

레디스를 사용하기 위한 ```RedisConfig```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;

@Configuration
public class RedisConfig {
    @Value("${spring.redis.host}") //host 환경변수 설정
    private String host;  

    @Value("${spring.redis.port}") //port 환경변수 설정
    private int port; 

    @Bean 
    public RedisConnectionFactory redisConnectionFactory(){
        return new LettuceConnectionFactory(host,port);
    }

    @Bean
    public RedisTemplate<?,?> redisTemplate(){ //레디스 템플릿 사용
        RedisTemplate<?,?> redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

<br>

레디스에 데이터를 저장,조회,삭제하는 ```RedisUtil```

```java
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Service;

import java.time.Duration;

@RequiredArgsConstructor
@Service
public class RedisUtil {
    private final StringRedisTemplate template;
    public String getData(String key){  //String타입의 키값을 조회한 후 value값을 가져온다.
        ValueOperations<String,String> valueOperations = template.opsForValue();
        return valueOperations.get(key);
    }

    public boolean existData(String key){  //키값이 존재여부에 따라서 boolean타입으로 반환
        return Boolean.TRUE.equals(template.hasKey(key));
    }

    public void setDataExpire(String key, String value, long duration){ 
        // String 타입의 키,밸류와 Expire타임을 지정하여 레디스에 저장
        ValueOperations<String,String> valueOperations = template.opsForValue();
        Duration expriedDuration = Duration.ofSeconds(duration);
        valueOperations.set(key,value,expriedDuration);
    }

    public void deleteData(String key){ //레디스에 저장된 데이터를 키값을 받아 삭제
        template.delete(key);
    }
}
```

<br> 

다음으로 ```MailService```코드이다.

```java
import Doffy.server.global.exception.BusinessLogicException;
import Doffy.server.global.exception.ExceptionCode;
import Doffy.server.mail.Dto.MailDto;
import Doffy.server.redis.util.RedisUtil;
import Doffy.server.user.entity.User;
import Doffy.server.user.repository.UserRepository;
import Doffy.server.user.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class MailService {

    private final UserRepository userRepository;
    private final JavaMailSender javaMailSender;
    private final RedisUtil redisUtil;
    private final UserService userService;

    private static final String FROM_ADDRESS = "teamdoffy@gmail.com";

    public void sendEmail(MailDto.AuthEmail mailDto) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(mailDto.getEmail());
        message.setFrom(MailService.FROM_ADDRESS);
        message.setSubject(mailDto.getTitle());
        message.setText(mailDto.getMessage());
        javaMailSender.send(message);
    }

    @Transactional
    public MailDto.AuthEmail createAuthMail(MailDto.AuthEmail mailDto) {
        String createdKey = createKey();
        MailDto.AuthEmail createdMail = new MailDto.AuthEmail();
        createdMail.setEmail(mailDto.getEmail());
        createdMail.setTitle("Doffy 회원가입 이메일 인증번호 발송 메일 입니다.");
        createdMail.setMessage("Doffy에 오신것을 환영 합니다! 이메일 인증번호는 " + createdKey + " 입니다.");
        if(redisUtil.existData(mailDto.getEmail())){
            redisUtil.deleteData(mailDto.getEmail());
        }
        redisUtil.setDataExpire(mailDto.getEmail(), createdKey, 300);
        return createdMail;
    }

    public void verifyEmailCode(String email, String code){
        String foundedCode = redisUtil.getData(email);
        if(foundedCode == null) throw new BusinessLogicException(ExceptionCode.CODE_MISMATCH);
        if(!foundedCode.equals(code)) throw new BusinessLogicException(ExceptionCode.CODE_MISMATCH);
    }

    public String createKey() {
        char[] charSet = new char[]{
                '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
                'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'
        };
        String createdKey = "";
        int idx = 0;
        for (int i = 0; i < 6; i++) {
            idx = (int) (Math.random() * charSet.length);
            createdKey += charSet[idx];
        }
        return createdKey;
    }
}
```

* ```sendEmail```은 createAuthMail로 부터 만들어진 메일 객체를 바탕으로 이메일을 생성하여 발송해준다.
* ```createAuthMail```은 사용자에게서 받은 mailDto를 토대로 이메일을 객체를 만들어 준다.
* ```verifyEmailCode```는 레디스에 키값으로 저장된 유저의 email이 존재하는지 확인하고 만약 있는 경우 예외를 던진다.
* ```createKey```는 6자리 랜덤한 인증번호를 생성하고 리턴한다.

<br>

다음으로는 ```MailController```이다.

```java
import Doffy.server.mail.Dto.MailDto;
import Doffy.server.mail.service.MailService;
import Doffy.server.redis.util.RedisUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/mail")
@RequiredArgsConstructor
public class MailController {

    private final MailService mailService;
    private final RedisUtil redisUtil;

    @PostMapping("/signup-auth-email")
    public void signUpCheckEmailSend(@RequestBody MailDto.AuthEmail mailDto) {
        MailDto.AuthEmail createEmail = mailService.createAuthMail(mailDto);
        mailService.sendEmail(createEmail);
    }

    @PostMapping("signup-check-email")
    public ResponseEntity signUpCheckEmail(@RequestBody MailDto.Key key) {
        mailService.verifyEmailCode(key.getKey(), key.getValue());
        redisUtil.deleteData(key.getKey());
        return new ResponseEntity(HttpStatus.OK);
    }
}
```

* ```signUpCheckEmailSend``` 이메일을 발송해주는 API 컨트롤러 이다.
* ```signUpCheckEmail``` 사용자가 입력한 인증코드를 검증해주는 API 컨트롤러 이다.

<br>

---

## 정리

위와 같이 SpringBoot와 Redis를 이용하여 이메일 인증기능을 구현해 보았다.<br>
블로그를 작성하며, 비효율적인 코드도 많이 발견하고 리팩토링도 중간중간 해주었다.<br>
***이메일 인증기능이 아니라 다른 기능에서 발견되었지만..***
또한 레디스를 이용하면, DB에 별도의 테이블이나 컬럼을 추가하지 않더라도 간단하게 **Key와 Value**를저장했다가 소멸시킬 수 있는 편리함을 가지고 있다.<br>
물론 DB에 저장하는 방법 또한 가능하지만.. 휘발성 자료가 DB에 저장되는 것은.. 비효율적이고 해당 데이터를 삭제하기 위한 기능을 구현하고 또 데이터베이스에서 쿼리문이 실행되는 것은 리소스 낭비라고 생각한다:)