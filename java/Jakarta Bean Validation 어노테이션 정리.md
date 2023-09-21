# Jakarta Bean Validation 어노테이션 정리

유효성 검증을 위한 Jakarta Bean Validation 어노테이션 정리

:gear: environments

- Jakarta Bean Validation 3.0
- Hibernate Validator 8.0.1.Final

## TOC

- [constraint annotations](#constraint-annotations)
  - [@NotNull](#notnull)
  - [@NotEmpty](#notempty)
  - [@NotBlank](#notblank)
  - [@Pattern](#pattern)
  - [@Email](#email)
  - [@Size](#size)

<br>

## constraint annotations

### @NotNull

`null`이 아니어야 한다.

대상 타입

- 모든 타입

### @NotEmpty

`null`이 아니면서 비어있지 않아야 한다.

대상 타입

- `CharSequence` → 문자열 길이가 1 이상이어야 함
- `Collection` → collection size가 1 이상이어야 함
- `Map` → map size가 1 이상이어야 함
- `Array` → 배열 길이가 1 이상이어야 함

### @NotBlank

문자열이 `null`이 아니면서 길이가 1 이상이어야 한다. 또한 후행 공백은 무시된다. 따라서 공백이 아닌 문자가 최소 1개는 포함되어야 한다.

대상 타입

- `CharSequence`

### @Pattern

문자열이 지정된 정규표현식과 일치해야 한다. 정규표현식은 자바 정규표현식 규칙을 따른다. 문자열이 `null`인 경우 유효한 것으로 처리한다.

속성

- `regexp` → 정규표현식 문자열
- `flags` (option) → 정규표현식 플래그 `Flag`[^1] 배열

대상 타입

- `CharSequence`

### @Email

문자열이 유효한 email 주소여야 한다. 정규표현식과 플래그를 지정할 경우 기본 email 검사와 더불어 추가한 정규표현식과도 일치해야 한다. 문자열이 `null`인 경우 유효한 것으로 처리한다.

속성

- `regexp` (option) → 추가로 검사할 정규표현식 문자열
- `flags` (option) → 정규표현식 플래그 `Flag`[^1] 배열

대상 타입

- `CharSequence`

### @Size

대상의 크기가 지정된 `min`, `max`(포함) 범위내에 있어야 한다. 대상이 `null`일 경우 유효한 것으로 처리한다.

속성

- `min` → 최소 크기, 기본값 0
- `max` → 최대 크기, 기본값 `Integer.MAX_VALUE`

대상 타입

- `CharSequence` → 문자열 길이가 범위내에 있어야 함
- `Collection` → collection size가 범위내에 있어야 함
- `Map` → map size가 범위내에 있어야 함
- `Array` → 배열 길이가 범위내에 있어야 함

## References

- [Jakarta Bean Validation specification 3.0 #8](https://jakarta.ee/specifications/bean-validation/3.0/jakarta-bean-validation-spec-3.0.html#builtinconstraints)
- [Hibernate Validator 8.0.1.Final - Jakarta Bean Validation Reference Implementation: Reference Guide
  #2.3.1](https://docs.jboss.org/hibernate/validator/8.0/reference/en-US/html_single/#validator-defineconstraints-spec)

[^1]: Flag enum은 java.util.regex.Pattern의 상수값을 wrapping함
