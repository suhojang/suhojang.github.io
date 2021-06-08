---
title: "Spring Validation Annotation"
layout: post
author: jsh
tags: Spring
cover: "/assets/cover.jpg"
categories: Spring
---

### Spring Validation Annotation
> @Valid를 통해 데이터의 유효성 검사를 수행 할 수 있는 javax.validation과 org.hibernate.validation 패키지가 제공하는 Validation Annotation

+ @AssertFalse
  + 주석이 달린 요소는 거짓어야 합니다.
  + 지원되는 타입은 boolean과 Boolean 입니다.
  + null 요소는 유효한 것으로 간주 됩니다.


+ @AssertTrue
  + 주석이 달린 요소는 참이어야 합니다.
  + 지원되는 타입은 boolean과 Boolean 입니다.
  + null 요소는 유효한 것으로 간주 됩니다.  


+ @DecimalMax(value*=, inclusive=)
  + 주석이 달린 요소는 value보다 작거나 같아야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, CharSequence, byte, shor, int, long 및 각각의 Wrapper Class
  + 반올림 오류로 인해 double, float은 지원되지 않습니다.
  + null 요소는 유효한 것으로 간주 됩니다.
  + inclusive가 true인 경우 주석이 달린 요소는 value보다 크거나 같아야 하고, false인 경우 value보다 커야 합니다.
  + inclusive의 default 값은 true 입니다.


+ @DecimalMin(value*=, inclusive=)
  + 주석이 달린 요소는 value보다 크거나 같아야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, CharSequence, byte, shor, int, long 및 각각의 Wrapper Class
  + 반올림 오류로 인해 double, float은 지원되지 않습니다.
  + null 요소는 유효한 것으로 간주 됩니다.
  + inclusive가 true인 경우 주석이 달린 요소는 value보다 크거나 같아야 하고, false인 경우 value보다 커야 합니다.
  + inclusive의 default 값은 true 입니다.  


+ @Digits(integer_=,fraction_=)
  + 주석이 달린 요소는 허용되는 범위 내의 숫자여야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, CharSequence, byte, shor, int, long 및 각각의 Wrapper Class
  + null 요소는 유효한 것으로 간주 됩니다.
  + integer는 허용된느 최대 정수 자릿수, fraction은 허용되는 최대 소수 자릿수 입니다.



+ @Emal(regexp=,flags=)
  + 주석이 달린 문자열은 올바른 형식의 이메일 주소여야 합니다.
  + 유효한 이메일 주소에 대한 정확한 의미는 빈 검증 제공자에게 달려 있습니다.
  + CharSequence 타입을 허용 합니다.


+ @Future
  + 주석이 달린 요소는 미래의 순간, 날짜 또는 시간이어야 합니다.
  + 현재는 Validator 또는 ValidatiorFactory에 첨부 된 ClockProvider에 의해 결정 됩니다.
  + 지원되는 타입은 Date, Calendar, Instant, LocalDate, LocalDateTime, LocalTime, MonthDay, OffsetDateTime,
    OffsetTime, Year, YearMonth, ZonedDateTime, HirahDate, JapaneseDate, MingiuoDate,ThaiBuddhisDate 입니다. 
  + null 요소는 유효한 것으로 간주 됩니다.


+ @Max(value*=)
  + 주석이 달린 요소는 value보다 작거나 같아야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + 반올림 오류로 인해 double, float은 지원되지 않습니다.
  + null 요소는 유효한 것으로 간주 됩니다.


+ @Min(value*=)
  + 주석이 달린 요소는 value보다 크거나 같아야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + 반올림 오류로 인해 double, float은 지원되지 않습니다.
  + null 요소는 유효한 것으로 간주 됩니다.


+ @Negative
  + 주석이 달린 요소는 음수여야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + null 요소는 유효한 것으로 간주 됩니다.


+ @NegativeOrZero
  + 주석이 달린 요소는 음수 또는 0이어야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + null 요소는 유효한 것으로 간주 됩니다.


+ @NotBlank
  + 주석이 달린 요소는 null이 아니어야 하며, 하나 이상의 공백이 아닌 문자를 포함해야 합니다.
  + CharSequence 타입을 허용 합니다.


+ @NotEmpty
  + 주석이 달린 요소는 null이거나 비어 있으면 안됩니다.
  + 지원되는 타입은 CharSequence, Collection, Map, 배열입니다.  


+ @NotNull
  + 주석이 달린 요소는 null이 아니어야 합니다.
  + 모든 타입을 허용 합니다.  


+ @Null
  + 주석이 달린 요소는 null이어야 합니다.
  + 모든 타입을 허용 합니다.


+ @Past
  + 주석이 달린 요소는 과거의 순간 날짜 또는 시간이어야 합니다.
  + 현재는 Validator 또는 ValidatiorFactory에 첨부 된 ClockProvider에 의해 결정 됩니다.
  + 지원되는 타입은 Date, Calendar, Instant, LocalDate, LocalDateTime, LocalTime, MonthDay, OffsetDateTime,
    OffsetTime, Year, YearMonth, ZonedDateTime, HirahDate, JapaneseDate, MingiuoDate,ThaiBuddhisDate 입니다.
  + null 요소는 유효한 것으로 간주 됩니다.  


+ @PastOrPresent
  + 주석이 달린 요소는 과거의 순간 날짜 또는 시간이어야 합니다.
  + 현재는 Validator 또는 ValidatiorFactory에 첨부 된 ClockProvider에 의해 결정 됩니다.
  + 여기는 현재의 개념은 제약 조건에 따라 상대적으로 정의 됩니다. 예를 들어 제약 조건이 Year에 있는 경우 현재는 현재 연도 전체를 의미 합니다.  
  + 지원되는 타입은 Date, Calendar, Instant, LocalDate, LocalDateTime, LocalTime, MonthDay, OffsetDateTime,
    OffsetTime, Year, YearMonth, ZonedDateTime, HirahDate, JapaneseDate, MingiuoDate,ThaiBuddhisDate 입니다.
  + null 요소는 유효한 것으로 간주 됩니다.   


+ @Pattern(regexp*=,flags=)
  + 주석이 달린 CharSequence는 지정 된 정규식과 일치해야 합니다.
  + 정규식은 Java 정규식 규칙을 따릅니다.
  + null 요소는 유효한 것으로 간주 됩니다.
  + 가능한 Regex flags
    + UNIX_LINES: 유닉스 라인 모드 활성화
    + CASE_INSENSITIVE: 대소문자 구분없이 일치 가능
    + COMMENTS: 패턴으로 공백 및 주석 허용
    + MULTILINE: 멀티 라인 모드 활성화
    + DOTALL: dotall 모드 활성화('\n' 포함 매칭)
    + UNICODE_CASE: 유니코드 인식 케이스 폴딩 지원
    + CANON_EQ: 규범적 동등성 활성화


+ @Positive
  + 주석이 달린 요소는 양수여야 합니다.(0은 잘못 된 값 입니다.)
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + null 요소는 유효한 것으로 간주 됩니다.  


+ @PositiveOrZero
  + 주석이 달린 요소는 양수 또는 0이어야 합니다.
  + 지원되는 타입은 BigDecimal, BigInteger, byte, shor, int, long 및 각각의 Wrapper Class
  + null 요소는 유효한 것으로 간주 됩니다.  


+ @Size(min=,max=)
  + 주석이 달린 요소의 크기는 지정 된 경계를 포함한 사이에 있어야 합니다.
  + 지원되는 타입은 CharSequence, Collection, Map, 배열 입니다.
  + null 요소는 유효한 것으로 간주 됩니다.
  + 요소의 크기는 min보다 크거나 같아야 하고 max보다 작거나 같아야 합니다.
    + min의 default 값은 0, max의 default 값은 2147483647 입니다.


+ @Range(min=,max=)
  + 주석이 달린 요소는 적절한 범위 내에 있어야 합니다.
  + 숫자 값 또는 숫자 값의 문자열 표현에 적용할 수 있습니다.
  + min의 default 값은 0, max의 default 값은 9223372036854775807L 입니다.


+ @UniqueElements
  + 주석이 달린 Collection의 모든 객체가 고유한지 확인 합니다. 
    + 즉, 동일한 요소를 찾을 수 없습니다.
    + 예를 들어 JAX-RS에서 유용합니다.
    + JAX-RS는 항상 컬렉션을 list로 비직렬화 합니다.
    + 그러므로 Set으로 변환 할 때 중복이 내재적으로 제거 됩니다.
  + 이 제약 조건을 사용하면 list에서 중복을 확인하고 대시 오류를 발생 시킬 수 있습니다.


+ @URL(protocol=,host=,port=,regexp=,flags=)
  + 주석이 달린 문자열이 URL인지 확인 합니다.
  + 매개 변수 protocol, host, port는 URL의 해당 부분과 매칭 됩니다.
  + 추가 정규식은 regexp와 flags를 사용하여 지정 할 수 있습니다.
  + 기본적으로 이 제약 조건에 대한 유효성 검증은 java.net.URL 생성자를 사용하여 문자열의 유효성을 검증 합니다.
  + 이는 일치하는 프로토콜 처리기를 사용할 수 있어야 함을 의미 합니다.
  + 다음 프로토콜에 대한 처리자는 기본 JVM 내에 존재하는 http, https, ftp, file 및 jar를 보장 합니다.


+ @CNPJ
  + CNPJ(브라질 법인 납세자 등록 번호)의 유효성을 검사합니다.


+ @CPF
  + CPF(브라질 개인 납세자 등록 번호)의 유효성을 검사합니다.


+ @TituloEleitoral
  + 브라질 유권자 ID 카드 번호의 유효성을 검사합니다.


+ @NIP
  + CharSequence가 NIP 번호(9자리 폴란드 VAT 식별 번호)인지 확인 합니다.


+ @PESEL
  + CharSequence가 PESEL(폴란드 국가 식별 번호)인지 확인 합니다.

  
+ @REGON
  + CharSequence가 REGON 번호(9/14 자리 폴란드 납세자 식별 번호)인지 확인 합니다.
