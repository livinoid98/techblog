## CWE란?

- CWE(Common Weakness Enumeration)는 소프트웨어에서 발생할 수 있는 보안 취약점과 약점을 체계적으로 정리한 목록이다.
- MITRE Corporation에서 관리하며, 개발자와 보안 전문가가 취약점을 식별하고 해결하는 데 사용할 공통 언어를 제공한다.

#### 주요특징

- 각 항목은 고유 식별자(CWE-ID)를 가짐 (예: CWE-79 (Cross-Site Scripting))
- 보안 툴, 교육, 인증, 코드 품질 분석 도구 등과 연동

#### CWE Top 25

| 순위 | CWE-ID	| 취약점 명칭 | 설명 |
| -- | -- | -- | -- |
| 1 | CWE-79 | 웹 페이지 생성 시 입력 중화 처리 미흡 (Cross-site Scripting) | 사용자 입력을 적절히 중화하지 않아 악성 스크립트가 실행될 수 있음 |
| 2 | CWE-787 | 버퍼 경계 외부 쓰기 (Out-of-bounds Write) | 할당된 메모리 경계를 넘어 데이터를 쓰는 오류로, 시스템 충돌이나 코드 실행 가능성 존재 |
| 3 | CWE-89 | SQL 명령어에 사용된 특수 요소의 중화 처리 미흡 (SQL Injection) | 사용자 입력을 적절히 처리하지 않아 악의적인 SQL 명령어가 실행될 수 있음 |
| 4 | CWE-352 | 교차 사이트 요청 위조 (Cross-Site Request Forgery) | 	사용자의 인증 정보를 이용해 원하지 않는 요청을 수행하게 함 |
| 5 | CWE-22 | 제한된 디렉터리에 대한 경로 이름 제한 미흡 (Path Traversal) | 디렉터리 경로를 조작하여 접근이 제한된 파일에 접근 가능 |
| 6 | CWE-125 | 버퍼 경계 외부 읽기 (Out-of-bounds Read) | 할당된 메모리 경계를 넘어 데이터를 읽는 오류로, 민감 정보 노출 가능성 존재 |
| 7 | CWE-78 | OS 명령어에 사용된 특수 요소의 중화 처리 미흡 (OS Command Injection) | 사용자 입력을 통해 시스템 명령어가 실행될 수 있음 |
| 8	| CWE-416 | 해제 후 사용 (Use After Free) | 메모리 해제 후 해당 메모리를 사용하는 오류로, 예기치 않은 동작이나 보안 취약점 발생 가능 |
| 9 | CWE-862 | 권한 부여 누락 (Missing Authorization) | 사용자의 권한을 확인하지 않고 민감한 기능에 접근 허용 |
| 10 | CWE-434 | 위험한 유형의 파일 업로드 제한 미흡 (Unrestricted Upload of File with Dangerous Type) | 악성 파일을 서버에 업로드하여 실행 가능성 존재 |
| 11 | CWE-94 | 코드 생성 제어 미흡 (Code Injection) | 사용자 입력을 통해 동적으로 생성된 코드가 실행될 수 있음 |
| 12 | CWE-20 | 입력 검증 미흡 (Improper Input Validation) | 입력 데이터를 적절히 검증하지 않아 다양한 취약점 발생 가능 |
| 13 | CWE-77 | 명령어에 사용된 특수 요소의 중화 처리 미흡 (Command Injection) | 시스템 명령어에 악의적인 입력이 포함되어 실행될 수 있음 |
| 14 | CWE-287 | 인증 미흡 (Improper Authentication) | 사용자의 신원을 적절히 확인하지 않아 접근이 허용될 수 있음 |
| 15 | CWE-269 | 권한 관리 미흡 (Improper Privilege Management) | 	사용자 권한을 적절히 관리하지 않아 권한 상승 가능성 존재 |
| 16 | CWE-502 | 신뢰되지 않은 데이터의 역직렬화 (Deserialization of Untrusted Data) | 악의적인 객체를 역직렬화하여 코드 실행 가능성 존재 |
| 17 | CWE-200 | 민감 정보의 비인가된 액터에게 노출 (Exposure of Sensitive Information to an Unauthorized Actor) | 민감한 정보가 비인가된 사용자에게 노출될 수 있음 |
| 18 | CWE-863 | 잘못된 권한 부여 (Incorrect Authorization) | 권한 확인 로직의 오류로 인해 접근이 허용될 수 있음 |
| 19 | CWE-918 | 서버 측 요청 위조 (Server-Side Request Forgery) | 서버가 악의적인 요청을 외부로 전송하게 함 |
| 20 | CWE-119 | 메모리 버퍼 내 작업 제한 미흡 (Improper Restriction of Operations within the Bounds of a Memory Buffer) | 메모리 버퍼 경계를 적절히 제한하지 않아 데이터 손상이나 코드 실행 가능성 존재 |
| 21 | CWE-476 | 널 포인터 역참조 (NULL Pointer Dereference) | 널 포인터를 역참조하여 시스템 충돌이나 예기치 않은 동작 발생 가능 |
| 22 | CWE-798 | 하드코딩된 자격 증명 사용 (Use of Hard-coded Credentials) | 코드 내에 고정된 자격 증명을 사용하여 보안 취약점 발생 가능 |
| 23 | CWE-190 | 정수 오버플로우 또는 래퍼라운드 (Integer Overflow or Wraparound) | 정수 연산 시 범위를 초과하여 예기치 않은 동작 발생 가능 |
| 24 | CWE-400 | 제어되지 않은 자원 소비 (Uncontrolled Resource Consumption) | 자원을 과도하게 소비하여 서비스 거부 상태 발생 가능 |
| 25 | CWE-306 | 중요한 기능에 대한 인증 누락 (Missing Authentication for Critical Function) | 중요한 기능에 대한 인증 절차가 없어 비인가된 접근 가능성 존재 |

## 정적 분석 도구(SAST, Static Application Security Testing)

소스코드나 바이너리를 실행하지 않고 코드 구조를 분석하여 잠재적 취약점을 찾는 도구이다. CWE에 명시된 다양한 약점 유형을 탐지할 수 있다.

| 도구명 | 주요 특징 | CWE 지원 여부 및 매핑 | 대표 사례 및 용도 |
| -- | -- | -- | -- |
| SonarQube (무료+유료) | 오픈소스 기반 코드 품질 및 보안 분석 플랫폼 | CWE 기반 취약점 탐지 및 리포트 제공	 | 자바, C#, C++ 등 다수 언어 지원, CI/CD 통합 가능 |
| Coverity (Synopsys) (유료) | 산업용 정적 분석 도구 | CWE ID와 매핑된 광범위한 취약점 탐지 | 대규모 프로젝트, 금융·항공 등 민감 분야 활용 |
| Checkmarx (유료) | 보안 중심 SAST 도구 | CWE 분류 기반 상세 분석 제공 | 웹, 모바일 애플리케이션 보안 점검에 특화 |
| Fortify Static Code Analyzer (Micro Focus) (유료) | 대기업용 정적 분석 도구	 | CWE와 OWASP 취약점 매핑 보고서 제공 | 금융권, 공공기관 등 대규모 환경에서 사용 |
| Veracode Static Analysis (유료) |	SaaS 형태의 보안 분석 서비스 | CWE 분류 체계와 연동 | 빠른 분석, 클라우드 환경에 최적화 |

## 동적 분석 도구 (DAST, Dynamic Application Security Testing)

실행 중인 애플리케이션을 테스트하며 취약점을 찾는 도구이다. CWE 기반 매핑보다는 OWASP Top10에 초점이 맞춰져있다.

## 도구 선택 시 고려사항

- 지원 언어 및 플랫폼
- CWE 취약점 탐지 커버리지
- 통합 및 자동화 가능성 (CI/CD 연동)
- 가격 및 라이선스 (오픈소스/상용)
- 결과 리포팅의 명확성 및 대응 가이드 제공 여부