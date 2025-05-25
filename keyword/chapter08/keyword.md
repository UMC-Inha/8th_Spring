# 💡 8주차 핵심 키워드 💡
<br>

## 1. java의 Exception 종류들

[Exception. 즉, 예외란❓]<br>프로그램 실행 중에 발생하는 비정상적인 상황.

[자바의 예외 종류]<br>
1. `Checked Exception (검사 예외)`
2. `Unchecked Exception (비검사 예외)`

<br>

### 1-1. Checked Exception

[정의]<br>반드시 예외 처리를 강제하는 예외로, try-catch 또는 throws를 하지 않으면 컴파일 에러가 남.

[예시]<br>
```java
FileReader reader = new FileReader("file.txt");  // FileNotFoundException 발생 가능

⬇️
        
try {
    FileReader reader = new FileReader("file.txt");
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

[종류]<br>
1. `IOException` : 파일, 네트워크 작업 중 입출력 문제
2. `FileNotFoundException` : 파일이 없을 때
3. `SQLException` : DB 처리 중 문제 발생
4. `ParseException` : 날짜/숫자 포맷 파싱 실패 등

<br>

### 1-2. Unchecked Exception

[정의]<br>RuntimeException을 상속받은 예외로, 컴파일러가 강제하지는 않지만 필요하면 catch할 수 있음.

[예시]<br>
```java
// ArrayIndexOutOfBoundsException
int[] arr = new int[3];
System.out.println(arr[5]);

// NullPointerException
String s = null;
s.length();
```

[종류]<br>
1. `NullPointerException` : null 객체를 접근할 때
2. `IllegalArgumentException` : 잘못된 파라미터 전달
3. `ArrayIndexOutOfBoundsException` : 배열 인덱스 범위 초과
4. `ClassCastException` : 잘못된 형변환
5. `NumberFormatException` : 숫자 형식 파싱 실패
6. `ArithmeticException` : 0으로 나누는 경우

<br>

### 1-3. Error와는 뭐가 다른가?

[Error의 정의]<br>예외와는 구분되는 심각한 시스템 문제.

[종류]<br>
1. `OutOfMemoryError` : 메모리 부족
2. `StackOverflowError` : 재귀 호출 무한 루프

-> 보통 try-catch로 처리하는 것이 아닌, **시스템이 중단된다.**

<br>

## 2. @Valid

[정의]<br>사용자가 보낸 데이터가 요구 조건에 맞는지 자동으로 검사해주는 어노테이션.

[언제 사용할까?]<br>클라이언트가 보낸 요청에서 DTO에 있는 필수값, 문자열 길이, 숫자 범위, 이메일 형식 등을 자동으로 검사하고, 틀리면 에러 응답을 보낸다.

[예시]<br>
```java
// 컨트롤러에 @Valid 적용
@PostMapping("/")
    public ApiResponse<MemberResponseDTO.JoinResultDTO> join(@RequestBody @Valid MemberRequestDTO.JoinDTO request){
        return null;
    }
```

[작동 흐름]<br>
1. 사용자가 요청을 보내면
2. @Valid가 DTO의 필드에 붙은 유효성 조건을 검사해서
3. 조건을 통과하지 못하면 `MethodArgumentNotValidException` 예외 발생
4. 이를 @RestControllerAdvice로 받아서 JSON 응답 가능

[검사 항목 어노테이션 종류]<br>
1. `@NotNull` : null이면 안됨
2. `@NotBlank` : null/공백/빈문자열 안됨
3. `@Size(min, max)` : 문자열 길이 제한
4. `@Email` : 이메일 형식
5. `@Min, @Max` : 숫자 범위 제한
6. `@Pattern` : 정규표현식 패턴 제한

**예외 응답은 커스터마이징도 가능하다**👍