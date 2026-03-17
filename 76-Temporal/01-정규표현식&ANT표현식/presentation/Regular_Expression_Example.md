## 💼 1. 실용 예제

### 1.1 유효성 검증 (Validation)

### 이메일 검증

```java
/**
 * 이메일 검증 패턴
 * - 로컬 파트: 영문, 숫자, 특수문자(.-_+) 허용
 * - @
 * - 도메인: 영문, 숫자, 하이픈 허용
 * - 최상위 도메인: 2자 이상의 영문
 */
public class EmailValidator {

    // 간단한 버전
    private static final String EMAIL_SIMPLE =
        "^[\\\\w.-]+@[\\\\w.-]+\\\\.[a-zA-Z]{2,}$";

    // RFC 5322 준수 (더 엄격한) 버전
    private static final String EMAIL_STRICT =
        "^[a-zA-Z0-9_+&*-]+(?:\\\\.[a-zA-Z0-9_+&*-]+)*@" +
        "(?:[a-zA-Z0-9-]+\\\\.)+[a-zA-Z]{2,7}$";

    public static boolean isValidEmail(String email) {
        if (email == null || email.isEmpty()) {
            return false;
        }
        return email.matches(EMAIL_SIMPLE);
    }

    // 테스트
    public static void main(String[] args) {
        String[] emails = {
            "user@example.com",           // true
            "user.name@example.com",      // true
            "user+tag@example.co.kr",     // true
            "user@",                      // false
            "@example.com",               // false
            "user@.com",                  // false
            "user name@example.com"       // false (공백)
        };

        for (String email : emails) {
            System.out.printf("%s: %b%n", email, isValidEmail(email));
        }
    }
}

```

### 비밀번호 검증

```java
/**
 * 비밀번호 강도 검증
 * 요구사항:
 * - 8-20자
 * - 최소 1개의 대문자
 * - 최소 1개의 소문자
 * - 최소 1개의 숫자
 * - 최소 1개의 특수문자
 */
public class PasswordValidator {

    // Lookahead를 사용한 복잡한 검증
    private static final String PASSWORD_PATTERN =
        "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\\\d)(?=.*[@$!%*?&])[A-Za-z\\\\d@$!%*?&]{8,20}$";

    public static ValidationResult validatePassword(String password) {
        if (password == null || password.isEmpty()) {
            return new ValidationResult(false, "비밀번호를 입력해주세요.");
        }

        // 길이 체크
        if (password.length() < 8 || password.length() > 20) {
            return new ValidationResult(false, "비밀번호는 8-20자여야 합니다.");
        }

        // 소문자 포함 체크
        if (!password.matches(".*[a-z].*")) {
            return new ValidationResult(false, "소문자를 최소 1개 포함해야 합니다.");
        }

        // 대문자 포함 체크
        if (!password.matches(".*[A-Z].*")) {
            return new ValidationResult(false, "대문자를 최소 1개 포함해야 합니다.");
        }

        // 숫자 포함 체크
        if (!password.matches(".*\\\\d.*")) {
            return new ValidationResult(false, "숫자를 최소 1개 포함해야 합니다.");
        }

        // 특수문자 포함 체크
        if (!password.matches(".*[@$!%*?&].*")) {
            return new ValidationResult(false, "특수문자(@$!%*?&)를 최소 1개 포함해야 합니다.");
        }

        return new ValidationResult(true, "안전한 비밀번호입니다.");
    }

    // 통합 패턴 사용 (한 번에 체크)
    public static boolean isValidPasswordQuick(String password) {
        return password != null && password.matches(PASSWORD_PATTERN);
    }

    static class ValidationResult {
        boolean valid;
        String message;

        ValidationResult(boolean valid, String message) {
            this.valid = valid;
            this.message = message;
        }
    }

    // 테스트
    public static void main(String[] args) {
        String[] passwords = {
            "Abc123!@",           // true
            "Password123!",       // true
            "weak",               // false (너무 짧음)
            "alllowercase123!",   // false (대문자 없음)
            "ALLUPPERCASE123!",   // false (소문자 없음)
            "NoSpecialChar123",   // false (특수문자 없음)
            "NoNumber!@#",        // false (숫자 없음)
        };

        for (String pw : passwords) {
            ValidationResult result = validatePassword(pw);
            System.out.printf("%s: %b - %s%n", pw, result.valid, result.message);
        }
    }
}

```

### 한국 전화번호 검증

```java
public class PhoneValidator {

    // 휴대폰: 010-XXXX-XXXX
    private static final String MOBILE_PATTERN = "^010-\\\\d{4}-\\\\d{4}$";

    // 일반 전화: 0XX-XXX(X)-XXXX
    private static final String PHONE_PATTERN = "^0\\\\d{1,2}-\\\\d{3,4}-\\\\d{4}$";

    // 통합 패턴
    private static final String ALL_PHONE_PATTERN =
        "^(010-\\\\d{4}-\\\\d{4}|0\\\\d{1,2}-\\\\d{3,4}-\\\\d{4})$";

    public static boolean isValidMobile(String phone) {
        return phone != null && phone.matches(MOBILE_PATTERN);
    }

    public static boolean isValidPhone(String phone) {
        return phone != null && phone.matches(ALL_PHONE_PATTERN);
    }

    // 하이픈 자동 추가
    public static String formatPhone(String phone) {
        if (phone == null) return null;

        // 숫자만 추출
        String numbers = phone.replaceAll("[^0-9]", "");

        // 휴대폰 (11자리)
        if (numbers.matches("^010\\\\d{8}$")) {
            return numbers.replaceAll("(\\\\d{3})(\\\\d{4})(\\\\d{4})", "$1-$2-$3");
        }

        // 서울 (10자리: 02-XXXX-XXXX)
        if (numbers.matches("^02\\\\d{8}$")) {
            return numbers.replaceAll("(\\\\d{2})(\\\\d{4})(\\\\d{4})", "$1-$2-$3");
        }

        // 기타 지역 (10자리: 0XX-XXX-XXXX)
        if (numbers.matches("^0\\\\d{9}$")) {
            return numbers.replaceAll("(\\\\d{3})(\\\\d{3})(\\\\d{4})", "$1-$2-$3");
        }

        // 기타 지역 (11자리: 0XX-XXXX-XXXX)
        if (numbers.matches("^0\\\\d{10}$")) {
            return numbers.replaceAll("(\\\\d{3})(\\\\d{4})(\\\\d{4})", "$1-$2-$3");
        }

        return phone; // 포맷할 수 없으면 원본 반환
    }

    // 테스트
    public static void main(String[] args) {
        System.out.println(formatPhone("01012345678"));   // 010-1234-5678
        System.out.println(formatPhone("0212345678"));    // 02-1234-5678
        System.out.println(formatPhone("0311234567"));    // 031-123-4567
        System.out.println(formatPhone("03112345678"));   // 031-1234-5678
    }
}

```

### 1.2 데이터 파싱 (Parsing)

### 로그 파일 파싱

```java
/**
 * 로그 파일에서 정보 추출
 * 로그 형식: [2025-11-18 23:43:21] [ERROR] [UserService] User login failed - IP: 192.168.1.100
 */
public class LogParser {

    private static final Pattern LOG_PATTERN = Pattern.compile(
        "\\\\[(\\\\d{4}-\\\\d{2}-\\\\d{2} \\\\d{2}:\\\\d{2}:\\\\d{2})\\\\]\\\\s+" +  // 타임스탬프
        "\\\\[(\\\\w+)\\\\]\\\\s+" +                                         // 로그 레벨
        "\\\\[([\\\\w]+)\\\\]\\\\s+" +                                       // 서비스명
        "(.+?)(?:\\\\s+-\\\\s+IP:\\\\s+(\\\\d+\\\\.\\\\d+\\\\.\\\\d+\\\\.\\\\d+))?"     // 메시지 및 IP (옵션)
    );

    public static class LogEntry {
        String timestamp;
        String level;
        String service;
        String message;
        String ipAddress;

        @Override
        public String toString() {
            return String.format("LogEntry{time='%s', level='%s', service='%s', message='%s', ip='%s'}",
                timestamp, level, service, message, ipAddress);
        }
    }

    public static LogEntry parse(String logLine) {
        Matcher m = LOG_PATTERN.matcher(logLine);
        if (!m.find()) {
            return null;
        }

        LogEntry entry = new LogEntry();
        entry.timestamp = m.group(1);
        entry.level = m.group(2);
        entry.service = m.group(3);
        entry.message = m.group(4);
        entry.ipAddress = m.group(5); // null일 수 있음

        return entry;
    }

    // 특정 IP 주소 추출
    public static List<String> extractIPs(String text) {
        List<String> ips = new ArrayList<>();
        Pattern ipPattern = Pattern.compile("\\\\b(?:\\\\d{1,3}\\\\.){3}\\\\d{1,3}\\\\b");
        Matcher m = ipPattern.matcher(text);

        while (m.find()) {
            ips.add(m.group());
        }

        return ips;
    }

    // 에러 로그만 필터링
    public static List<String> filterErrorLogs(List<String> logs) {
        return logs.stream()
            .filter(log -> log.matches(".*\\\\[ERROR\\\\].*"))
            .collect(Collectors.toList());
    }

    // 테스트
    public static void main(String[] args) {
        String log1 = "[2025-11-18 23:43:21] [ERROR] [UserService] User login failed - IP: 192.168.1.100";
        String log2 = "[2025-11-18 23:43:22] [INFO] [PaymentService] Payment processed successfully";

        LogEntry entry1 = parse(log1);
        LogEntry entry2 = parse(log2);

        System.out.println(entry1);
        System.out.println(entry2);

        // IP 추출
        String text = "Connections from 192.168.1.1, 10.0.0.1, and 172.16.0.1";
        List<String> ips = extractIPs(text);
        System.out.println("추출된 IP: " + ips);
    }
}

```

### URL 파싱

```java
public class URLParser {

    private static final Pattern URL_PATTERN = Pattern.compile(
        "^(?<protocol>https?)://" +                      // 프로토콜
        "(?<domain>[^:/]+)" +                            // 도메인
        "(?::(?<port>\\\\d+))?" +                          // 포트 (옵션)
        "(?<path>/[^?#]*)?" +                            // 경로 (옵션)
        "(?:\\\\?(?<query>[^#]*))?" +                      // 쿼리 스트링 (옵션)
        "(?:#(?<fragment>.*))?$"                         // 프래그먼트 (옵션)
    );

    public static class URLInfo {
        String protocol;
        String domain;
        String port;
        String path;
        String query;
        String fragment;
        Map<String, String> queryParams;

        @Override
        public String toString() {
            return String.format("URL{protocol='%s', domain='%s', port='%s', path='%s', query='%s', fragment='%s', params=%s}",
                protocol, domain, port, path, query, fragment, queryParams);
        }
    }

    public static URLInfo parse(String url) {
        Matcher m = URL_PATTERN.matcher(url);
        if (!m.matches()) {
            return null;
        }

        URLInfo info = new URLInfo();
        info.protocol = m.group("protocol");
        info.domain = m.group("domain");
        info.port = m.group("port");
        info.path = m.group("path");
        info.query = m.group("query");
        info.fragment = m.group("fragment");

        // 쿼리 파라미터 파싱
        if (info.query != null && !info.query.isEmpty()) {
            info.queryParams = parseQueryString(info.query);
        }

        return info;
    }

    private static Map<String, String> parseQueryString(String query) {
        Map<String, String> params = new HashMap<>();
        Pattern paramPattern = Pattern.compile("([^&=]+)=([^&]*)");
        Matcher m = paramPattern.matcher(query);

        while (m.find()) {
            params.put(m.group(1), m.group(2));
        }

        return params;
    }

    // 테스트
    public static void main(String[] args) {
        String url = "<https://example.com:8080/api/users?id=123&name=john#section1>";
        URLInfo info = parse(url);
        System.out.println(info);

        // 출력:
        // URL{protocol='https', domain='example.com', port='8080',
        //     path='/api/users', query='id=123&name=john',
        //     fragment='section1', params={id=123, name=john}}
    }
}

```

### CSV/TSV 파싱

```java
public class CSVParser {

    // 기본 CSV (쉼표로 구분, 따옴표 처리)
    private static final Pattern CSV_PATTERN = Pattern.compile(
        ",(?=(?:[^\\"]*\\"[^\\"]*\\")*[^\\"]*$)" // 따옴표 외부의 쉼표만 매칭
    );

    // RFC 4180 준수 CSV 파서
    public static List<String> parseCSVLine(String line) {
        List<String> fields = new ArrayList<>();

        // 복잡한 케이스 처리: "field1","field with, comma","field with ""quotes"""
        Pattern pattern = Pattern.compile(
            "\\"([^\\"]*(?:\\"\\"[^\\"]*)*)\\"|([^,]+)|(?<=,)(?=,)|^(?=,)|(?<=,)$"
        );

        Matcher m = pattern.matcher(line);
        while (m.find()) {
            String field;
            if (m.group(1) != null) {
                // 따옴표로 감싸진 필드
                field = m.group(1).replace("\\"\\"", "\\"");
            } else if (m.group(2) != null) {
                // 일반 필드
                field = m.group(2);
            } else {
                // 빈 필드
                field = "";
            }
            fields.add(field);
        }

        return fields;
    }

    // 간단한 버전 (따옴표 없는 경우)
    public static List<String> parseSimpleCSV(String line) {
        return Arrays.asList(line.split(","));
    }

    // 테스트
    public static void main(String[] args) {
        // 복잡한 케이스
        String line1 = "John,Doe,\\"123 Main St, Apt 4\\",\\"He said \\"\\"Hello\\"\\"\\"";
        List<String> fields1 = parseCSVLine(line1);
        System.out.println(fields1);
        // [John, Doe, 123 Main St, Apt 4, He said "Hello"]

        // 간단한 케이스
        String line2 = "apple,banana,cherry";
        List<String> fields2 = parseSimpleCSV(line2);
        System.out.println(fields2);
        // [apple, banana, cherry]
    }
}

```

---

## 🔧 2. Spring Boot 실무 통합 예제

### 2.1 Validation with Bean Validation

```java
import jakarta.validation.Constraint;
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import jakarta.validation.Payload;
import java.lang.annotation.*;

// 커스텀 어노테이션 정의
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
@Documented
public @interface PhoneNumber {
    String message() default "유효하지 않은 전화번호 형식입니다";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator 구현
public class PhoneNumberValidator implements ConstraintValidator<PhoneNumber, String> {

    private static final String PHONE_PATTERN = "^(010-\\\\d{4}-\\\\d{4}|0\\\\d{1,2}-\\\\d{3,4}-\\\\d{4})$";

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null || value.isEmpty()) {
            return true; // @NotNull과 함께 사용
        }
        return value.matches(PHONE_PATTERN);
    }
}

// DTO 사용
public class UserRegistrationDTO {

    @NotBlank(message = "이름은 필수입니다")
    @Size(min = 2, max = 50, message = "이름은 2-50자여야 합니다")
    private String name;

    @NotBlank(message = "이메일은 필수입니다")
    @Email(message = "유효한 이메일 주소를 입력하세요")
    private String email;

    @NotBlank(message = "전화번호는 필수입니다")
    @PhoneNumber  // 커스텀 검증
    private String phone;

    @NotBlank(message = "비밀번호는 필수입니다")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\\\d)(?=.*[@$!%*?&])[A-Za-z\\\\d@$!%*?&]{8,20}$",
        message = "비밀번호는 8-20자이며, 대소문자, 숫자, 특수문자를 각각 1개 이상 포함해야 합니다"
    )
    private String password;

    // getters, setters...
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody UserRegistrationDTO dto) {
        // @Valid가 자동으로 검증 수행
        // 검증 실패 시 MethodArgumentNotValidException 발생

        // 검증 통과 후 로직
        return ResponseEntity.ok("등록 성공");
    }
}

// Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        return ResponseEntity.badRequest().body(errors);
    }
}

```

### 2.2 로깅 및 모니터링

```java
@Service
public class LogAnalysisService {

    private static final Pattern ERROR_PATTERN = Pattern.compile(
        "\\\\[(\\\\d{4}-\\\\d{2}-\\\\d{2} \\\\d{2}:\\\\d{2}:\\\\d{2})\\\\].*\\\\[ERROR\\\\].*"
    );

    private static final Pattern IP_PATTERN = Pattern.compile(
        "\\\\b(?:\\\\d{1,3}\\\\.){3}\\\\d{1,3}\\\\b"
    );

    /**
     * 로그 파일에서 에러 발생 시간대 분석
     */
    public Map<Integer, Long> analyzeErrorsByHour(List<String> logs) {
        Pattern hourPattern = Pattern.compile("\\\\d{4}-\\\\d{2}-\\\\d{2} (\\\\d{2}):");

        return logs.stream()
            .filter(log -> log.contains("[ERROR]"))
            .map(log -> {
                Matcher m = hourPattern.matcher(log);
                return m.find() ? Integer.parseInt(m.group(1)) : -1;
            })
            .filter(hour -> hour != -1)
            .collect(Collectors.groupingBy(
                hour -> hour,
                Collectors.counting()
            ));
    }

    /**
     * 의심스러운 IP 주소 탐지 (단시간 내 많은 요청)
     */
    public List<String> detectSuspiciousIPs(List<String> logs, int threshold) {
        Map<String, Long> ipCounts = logs.stream()
            .flatMap(log -> {
                Matcher m = IP_PATTERN.matcher(log);
                List<String> ips = new ArrayList<>();
                while (m.find()) {
                    ips.add(m.group());
                }
                return ips.stream();
            })
            .collect(Collectors.groupingBy(
                ip -> ip,
                Collectors.counting()
            ));

        return ipCounts.entrySet().stream()
            .filter(entry -> entry.getValue() > threshold)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}

```

---
