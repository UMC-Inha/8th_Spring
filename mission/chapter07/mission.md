### @RestControllerAdvice

---

- 스프링 MVC에서 전역 예외 처리기를 선언할 때 사용하는 어노테이션
- 개별 컨트롤러에서 발생한 예외를 AOP 방식으로 가로채어, 한 곳에서 처리
- `ResponseEntity<>` 형태로 감싸서 응답을 반환

## 장점

---

- **중앙에서 예외 처리**
    - 모든 예외 처리 로직을 하나의 클래스(`GlobalExceptionHandler`)에 모아두어 중복 제거
    - 컨트롤러에서는 비즈니스 로직에만 집중 가능

    ```java
    // xxxController.java
    @RestController
    public class xxxController {
    
        @GetMapping("memeber/{id}")
        public ResponseEntity<ApiResponse<UserDto>> getUser(@PathVariable Long id) {
            // 예외 발생 시 GlobalExceptionHandler가 처리
            User user = userService.findById(id);
            return ResponseEntity.ok(ApiResponse.onSuccess(userMapper.toDto(user)));
        }
    }
    
    ```

- **일관된 응답 포맷**
    - 에러 코드, 메시지, 데이터 구조가 항상 동일
- **예외를 분류해서 처리 가능**
    - `@ExceptionHandler`를 여러 개 선언하여 예외별로 다른 응답을 반환할 수 있게 조정할 수 있다.

    ```java
    @ExceptionHandler(InvalidRequestException.class)
    public ResponseEntity<ApiResponse<?>> handleBadRequest(InvalidRequestException ex) {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ApiResponse.onFailure("REQ_001", ex.getMessage(), ex.getErrors()));
    }
    
    ```


## 단점

---

- 예외 발생 흐름 추적이 불편하다.
    - 예외 발생 지점과 처리 지점이 분리되어 있어 디버깅 시 호출 흐름을 따라가야 하는 불편함이 존재한다.
    - 로그 디버깅으로도 어떤 핸들러가 최종 처리했는지 파악이 어려울 수 있음
- **모든 예외가 전역으로 처리되는 오버헤드**
    - 예외가 발생할 때마다 AOP 방식으로 Advice가 개입하므로, 예외 발생 빈도가 높으면 성능 영향 우려

      **AOP 프록시 호출 비용**

        - Spring의 `@RestControllerAdvice`는 AOP 프록시 메커니즘 위에서 동작함
        - 예외 발생 시 프록시가 예외를 받아서 `@ExceptionHandler`로 이동시키는 ‘중개’ 역할을 함
        - 즉, 예외가 발생할 때마다 단순히 `throw`/`catch` 만 하는 것이 아니라, 프록시 진입 및 탈출, 리플렉션 호출, 핸들러 검색 등의 추가 비용이 발생함
    - 따라서, 매우 빈번한 예외는 별도의 필터나 인터셉터를 사용해 처리하는 것이 나을 수 있음
- **상황별 세밀 제어 제한**
    - 전역 처리기에서 공통 로직으로만 다뤄지므로, 특정 컨트롤러·메서드에서만 특별 처리해야 할 때 분기가 복잡해질 수 있음
    - 일부 예외만 로컬(`try-catch`)로 처리하고 싶다면 전역 Advice보다 우선순위 관리가 필요

### 없을 경우 불편한 점

- 애플리케이션에서 발생하는 모든 예외를 처리해주는 로직이 필요할 수 있음

---

**@RestControllerAdvice 유무로 달라지는  코드 (Controller 로직)**

```java
// 없을 때
// 모든 에러 발생 경우에 대해 try-catch 구문 사용해야 함
   public ResponseEntity<ApiResponse<UserDto>> getUser(Long id) {
       try {
           User user = userService.findById(id);
           return ResponseEntity.ok(ApiResponse.onSuccess(userMapper.toDto(user)));
       } catch (UserNotFoundException e) {
           return ResponseEntity.status(HttpStatus.NOT_FOUND)
               .body(ApiResponse.onFailure("USER_001", "사용자를 찾을 수 없습니다", null));
       } catch (Exception e) {
           return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
               .body(ApiResponse.onFailure("SYS_001", "서버 오류가 발생했습니다", null));
       }
   }
   
   
 
// 있을 때
   public ResponseEntity<ApiResponse<UserDto>> getUser(Long id) {
       User user = userService.findById(id); 
       // 예외 발생 시 자동으로 ExceptionAdvice가 처리해주기에
       return ResponseEntity.ok(ApiResponse.onSuccess(userMapper.toDto(user)));
   }
```

- Advice가 없을 때도 일관된 응답 형식을 반환할 수 있었지만, 발생할 수 있는 에러를 예측하고 모두 try-catch로 묶어야 했었음
- Advice가 있음으로써 발생된 예외에 대해서 중앙 처리를 해주는 기관이 생긴 셈

### **실제 작동 방식**

1. `@RestControllerAdvice` 어노테이션으로 전역 예외 처리기 선언
2. `@ExceptionHandler` 어노테이션으로 처리할 예외 유형 지정
3. 예외 발생 시 해당 타입의 핸들러 메소드가 자동으로 호출됨
4. 예외 정보를 API 응답 형식으로 가공
5. 적절한 HTTP 상태 코드와 함께 응답 반환