# Ant-style Pattern

## 💡 핵심 요약 💡

>   - **한 줄 정의** : 파일 경로(Path)나 URL을 매칭하기 위해 Apache Ant 빌드 도구에서 고안된 직관적인 **와일드카드 패턴** 입니다.
>
>   - **핵심 키워드** : `와일드카드`, `경로 매칭(Path Matching)`, `Double Asterisk(**)`, `Glob Pattern`
>
>   - **왜 중요한가?** : 정규표현식보다 문법이 훨씬 단순하여 디렉터리 구조 탐색에 최적화되어 있습니다. 특히 **Spring Framework** 의 URL 설정, 파일 검색 등에서 표준으로 사용됩니다.

## 1. 개념

**Ant-style Pattern** 은 파일 시스템의 경로(Path)나 URL을 쉽고 간단하게 표현하기 위해 만들어진 규칙입니다.

복잡한 문자열 처리를 위한 '정규표현식'과 달리, **'디렉터리(폴더)와 파일의 계층 구조'**를 매칭하는 데 특화되어 있습니다. 우리 수업에서는 Spring Framework에서 리소스 위치나 URL 패턴(`@RequestMapping`)을 정의할 때 주로 사용했습니다.

-----

## 2. 왜 필요한가?

### 정규표현식의 복잡성 해소

  - 파일 경로나 URL은 `/` (슬래시)로 구분된 계층 구조를 가집니다.
  - 이를 정규표현식으로 표현하려면 이스케이프 처리(`\/`)와 복잡한 수량자 조합이 필요해 가독성이 떨어집니다.
  - Ant 패턴은 `*`, `**`, `?` 세 가지만으로 직관적인 경로 표현이 가능합니다.

### 디렉터리 재귀 탐색 (Recursive Search)

  - 가장 강력한 기능인 `**`를 통해 하위 디렉터리의 깊이(Depth)에 상관없이 모든 경로를 포함할 수 있습니다.
  - 이는 웹 애플리케이션에서 "특정 경로 하위의 모든 페이지에 보안 필터를 적용"하는 등의 시나리오에 필수적입니다.

-----

## 3. 컴퓨터 과학 내에서 Ant 패턴의 위치

### Glob Pattern의 일종

Ant 패턴은 컴퓨터 과학에서 **Glob Pattern**의 확장된 형태입니다.

  - **Glob Pattern**: 유닉스 셸(Shell)에서 파일 이름 확장을 위해 사용하는 패턴 (예: `ls *.txt`)
  - **Ant Pattern**: Glob 패턴에 **디렉터리 재귀 탐색(`**`)** 개념을 추가하여 확장한 것

### 정규표현식과의 관계

Ant 패턴은 정규표현식의 **부분집합(Subset)**처럼 동작하지만, 내부적으로는 정규표현식으로 변환되어 처리되거나 별도의 파서(Parser)를 통해 해석됩니다.

| 특성 | Ant Pattern | 정규표현식 (Regex) |
| :--- | :--- | :--- |
| **주 목적** | 경로(Path), 파일, URL 매칭 | 범용 문자열 검색 및 조작 |
| **복잡도** | 매우 낮음 (단순) | 높음 (강력함) |
| **주요 심볼** | `?`, `*`, `**` | `.`, `*`, `+`, `^`, `$`, `[]` 등 다수 |
| **경계 기준** | `/` (디렉터리 구분자) 기준 | 문자 단위 기준 |

-----

## 4. Ant 패턴의 주요 구성 요소 (Syntax)

Ant 패턴은 다음 3가지 특수 문자를 조합하여 사용합니다.

### 4.1 `?` (물음표)

**한 글자**와 매칭됩니다. (디렉터리 구분자 `/` 제외)

```java
// 예시: t?st.txt
"test.txt"  (O) // ? = e
"tast.txt"  (O) // ? = a
"tst.txt"   (X) // 문자가 있어야 함
"teest.txt" (X) // 한 글자만 가능
```

### 4.2 `*` (별표)

**0개 이상의 문자** 와 매칭됩니다. 단, **디렉터리 구분자(`/`)를 넘어갈 수 없습니다.** (현재 디렉터리/파일 이름 내에서만 유효)

```java
// 예시: *.txt
"file.txt"       (O)
"a.txt"          (O)
".txt"           (O) // 이름 없는 경우도 매칭 (구현체에 따라 다름)
"dir/file.txt"   (X) // / 를 넘어갈 수 없음
```

### 4.3 `**` (더블 애스터리스크)

**0개 이상의 디렉터리(패스)** 와 매칭됩니다.

```java
// 예시: /project/**/test
"/project/test"             (O) // 중간 디렉터리 0개
"/project/a/test"           (O) // 중간 디렉터리 1개
"/project/a/b/c/d/test"     (O) // 중간 디렉터리 다수

// 예시: /static/**
"/static/css/style.css"     (O)
"/static/images/logo.png"   (O)
```

-----

## 5. 실전 비교: Ant Pattern vs Regex

Spring Framework의 `AntPathMatcher`가 내부적으로 어떻게 동작하는지 이해하기 위해 같은 의도를 가진 패턴을 비교해 봅니다.

### 시나리오 1: 모든 .jsp 파일 찾기

  - **Ant Pattern**: `/**/*.jsp`
  - **Regex**: `^/.*.jsp$` (또는 경로 구분자를 명확히 하려면 `^/.*[^/].jsp$`)
      - *해석*: Ant 패턴이 훨씬 직관적입니다.

### 시나리오 2: /admin 하위의 모든 URL (깊이 무관)

  - **Ant Pattern**: `/admin/**`
  - **Regex**: `^/admin(/.*)?$`
      - *해석*: Regex는 하위 경로가 아예 없는 경우와 있는 경우를 모두 고려하는 그룹화가 필요합니다.

-----

## 6. Java(Spring)에서의 활용 예시

Java Spring 생태계에서는 `AntPathMatcher` 유틸리티 클래스를 통해 이 패턴을 지원합니다.

```java
import org.springframework.util.AntPathMatcher;

public class AntPatternExample {
    public static void main(String[] args) {
        AntPathMatcher matcher = new AntPathMatcher();

        // 1. ? 매칭
        System.out.println(matcher.match("t?st.jsp", "test.jsp")); // true

        // 2. * 매칭 (같은 레벨)
        System.out.println(matcher.match("*.jsp", "hello.jsp"));   // true
        System.out.println(matcher.match("*.jsp", "a/hello.jsp")); // false (*은 /를 못 넘음)

        // 3. ** 매칭 (하위 경로 포함)
        System.out.println(matcher.match("/**/api", "/v1/api"));       // true
        System.out.println(matcher.match("/app/**/*.html", "/app/views/home.html")); // true
    }
}
```

### 주요 활용처

1.  **URL 매핑**: `@RequestMapping("/api/**")`, `<mvc:resources location="..." mapping="..." />`
2.  **Spring Security**: `antMatchers("/admin/**").hasRole("ADMIN")`
3.  **파일 검색**: `PathMatchingResourcePatternResolver`를 통한 클래스패스 내 파일 로드
