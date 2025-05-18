## - 시니어 미션

#### - 500 에러가 날 시 자동으로 디스코드(or 슬랙)에 올려보기

**1. 디스코드/슬랙 웹훅(Webhook) 설정**

다음과 같이 에러 메시지를 전송하기 위한 별도의 디스코드 채널을 설정했다.

<img src = "https://velog.velcdn.com/images/sm011212/post/853feb4d-2a01-492d-9e69-a83459971b69/image.png" width="500">

채널 설정에 들어가 웹후크 항목에 들어간 뒤,

<img src = "https://velog.velcdn.com/images/sm011212/post/1ef9491b-274e-4776-b941-7706a2d431aa/image.png" width="500">

다음과 같이 새로운 웹 후크를 생성해주었고, '웹후크 URL 복사' 버튼을 눌러 URL을 복사해주었다.

<img src = "https://velog.velcdn.com/images/sm011212/post/fc937292-c3bf-4ea3-99ef-79ca42fcd831/image.png" width="500">

application.yml 파일에 들어가 다음과 같이 주소를 붙여넣기 해주어 디스코드 웹후크 기본 설정을 완료해주었다.

``` yml
discord:
  webhook-url: 복사한 url 경로
```

**2. 디스코드/슬랙 웹훅(Webhook) 설정**

디스코드에 메시지를 보내기 위해 별도의 메시지 전송 객체를 생성해주었다.

``` java
public record DiscordWebhookPayload(
        String content,
        List<Embed> embeds
) {
    public record Embed(
            String title,
            String description,
            String timestamp,
            List<Field> fields
    ) {
        public record Field(String name, String value, boolean inline) {}
    }
}
```

기존 미션 코드에서 예외 처리를 담당하는 부분이 ExceptionAdvice.class 파일이기 때문에 거기에 Discord 메시지를 보내는 코드를 추가하기로 하였다.


먼저 application.yml 파일에 적어놓은 웹훅 url을 가져오기 위해 **@Value 어노테이션을 사용해 url을 가져오도록 설정했고**, 미션에서 로컬 환경과 운영 환경을 분리하도록 요청했기 때문에, @Profile 값을 받아올 수 있는 env 변수를 선언해주었다.

``` java
@Slf4j
@RestControllerAdvice(annotations = {RestController.class})
public class ExceptionAdvice extends ResponseEntityExceptionHandler {

    private final RestTemplate restTemplate = new RestTemplate();

    @Value("${discord.webhook-url}")
    private String discordWebhookUrl;

    @Autowired
    private Environment env;

    ...
}
```

다음으로 디스코드에 메시지를 보낼 수 있도록 **sendToDiscord 메서드를 생성해주었다.** (미션 요구사항을 만족하기 위해 프로필이 "prod"일 때만 메시지를 보내도록 구현했다.) 위에서 만들었던 DiscordWebhokPayload 객체에 메시지를 적절하게 담아 보내도록 코드가 구현되었다.

``` java
private void sendToDiscord(Exception ex, WebRequest request, HttpStatus status) {
        if (!env.acceptsProfiles(Profiles.of("prod"))) {
            log.debug("현재 active 프로파일이 prod가 아니므로 Discord 알림을 보내지 않습니다.");
            return;
        }

        String path      = ((ServletWebRequest) request).getRequest().getRequestURI();
        String timestamp = Instant.now().toString();

        // Embed 필드 구성
        DiscordWebhookPayload.Embed embed = new DiscordWebhookPayload.Embed(
                "🚨 서버 에러 발생",
                "```" + ex.getMessage() + "```",
                timestamp,
                List.of(
                        new DiscordWebhookPayload.Embed.Field("URL", path, false),
                        new DiscordWebhookPayload.Embed.Field("Status", status.toString(), true),
                        new DiscordWebhookPayload.Embed.Field("Time", timestamp, true),
                        new DiscordWebhookPayload.Embed.Field("Exception", ex.getClass().getSimpleName(), true)
                )
        );
        DiscordWebhookPayload payload = new DiscordWebhookPayload(null, List.of(embed));

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        try {
            restTemplate.postForEntity(
                    discordWebhookUrl,
                    new HttpEntity<>(payload, headers),
                    String.class
            );
            log.info("Discord Webhook 전송 완료");
        } catch (Exception sendEx) {
            log.warn("Discord Webhook 전송 실패", sendEx);
        }
    }
```

이후, 다음과 같이 예외를 처리하는 메서드에 sendToDiscord() 메서드를 통해 메시지를 디스코드에 전달하도록 코드를 추가해주었다.

``` java
@ExceptionHandler(value = GeneralException.class)
    public ResponseEntity onThrowException(GeneralException generalException, HttpServletRequest request) {
        ErrorReasonDTO errorReasonHttpStatus = generalException.getErrorReasonHttpStatus();

        WebRequest webRequest = new ServletWebRequest(request);
        sendToDiscord(generalException, webRequest, errorReasonHttpStatus.getHttpStatus());

        return handleExceptionInternal(generalException,errorReasonHttpStatus,null,request);
    }
```

**3.에러 발생 시 알림 테스트**

실제로 예외에 대해 메시지가 적절하게 발생하는지 확인하기 위해 기존에 작성되어 있던 TempRestController에 에러 전용 api 엔드포인트를 추가해주었다.

``` java
@RestController
@RequestMapping("/temp")
@RequiredArgsConstructor
public class TempRestController {

    private final TempQueryService tempQueryService;

    @GetMapping("/test")
    public ApiResponse<TempResponse.TempTestDTO> testAPI(){

        return ApiResponse.onSuccess(TempConverter.toTempTestDTO());
    }

    @GetMapping("/exception")
    public ApiResponse<TempResponse.TempExceptionDTO> exceptionAPI(@RequestParam Integer flag){
        tempQueryService.CheckFlag(flag);

        return ApiResponse.onSuccess(TempConverter.toTempExceptionDTO(flag));
    }

    @GetMapping("/error")
    public void errorAPI(){
        throw new GeneralException(ErrorStatus._BAD_REQUEST);
    }
}
```

이후, 다음과 같이 postman을 통해 요청을 보내보았고,

<img src = "https://velog.velcdn.com/images/sm011212/post/f8d4269f-c07b-49c4-ba2e-12e57cf2100a/image.png" width = "400">

메시지가 Discord에 잘 보내지는 것을 확인했다!

<img src = "https://velog.velcdn.com/images/sm011212/post/4aac7447-dfe2-4f2b-acd7-bba654a545f4/image.png" width = "400">

추가적으로, 다음과 같이 application.yml의 프로필을 다른 local로 변경하면, 요청을 보내도 디스코드에 알림이 가지 않는다!

``` yml
spring:
  profiles:
    active: local
```

