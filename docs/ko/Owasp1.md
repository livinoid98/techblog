## OWASP란?

OWASP(Open Web Application Security Project)는 웹 애플리케이션 보안을 강화하기 위한 비영리 단체이자 커뮤니티다.

#### 주요활동

- OWASP Top 10: 가장 흔하고 치명적인 웹 보안 취약점 10가지 (2~3년마다 업데이트)
- OWASP ASVS: 애플리케이션 보안 검증 기준
- OWASP ZAP: 웹 애플리케이션 자동화 취약점 스캐너

#### OWASP Top 10

| 순위 | 코드명 | 취약점 명칭 | 설명 |
| -- | -- | -- | -- |
| A01 |	Broken Access Control | 권한 부여 실패 | 권한 없는 사용자가 제한된 기능이나 데이터를 접근하거나 조작할 수 있음 |
| A02 |	Cryptographic Failures | 암호화 실패 | 민감 정보 보호를 위한 암호화가 부적절하거나 누락됨 (예: 평문 저장, 전송) |
| A03 |	Injection |	인젝션 공격 | SQL, NoSQL, OS 명령어 등 외부 입력이 검증 없이 실행되어 공격에 노출됨 |
| A04 |	Insecure Design | 불안전한 설계 | 보안이 고려되지 않은 설계 단계에서의 취약점 발생 (예: 인증, 권한, 로깅 미흡) |
| A05 | Security Misconfiguration |	보안 설정 오류 | 기본 설정, 불필요한 기능 활성화, 권한 과다 설정 등 잘못된 보안 설정 |
| A06 |	Vulnerable and Outdated Components | 취약하거나 구버전 컴포넌트 사용 | 알려진 취약점이 존재하는 라이브러리, 프레임워크, 서버 등 사용 |
| A07 |	Identification and Authentication Failures | 인증 및 식별 실패 | 로그인, 세션 관리, 다중 인증 등이 부적절하여 계정 탈취 위험 |
| A08 | Software and Data Integrity Failures | 소프트웨어 및 데이터 무결성 실패 | 신뢰할 수 없는 코드, 업데이트, 데이터가 실행되거나 사용됨 |
| A09 |	Security Logging and Monitoring Failures | 보안 로깅 및 모니터링 실패 | 침해 탐지, 사고 대응을 위한 로그가 부실하거나 누락됨 |
| A10 |	Server-Side Request Forgery (SSRF) | 서버 측 요청 위조	| 서버가 공격자가 지정한 임의의 외부 리소스에 요청을 보내게 함 |

## 검사 권장 방향

OWASP는 보안 진단 시 SAST + DAST 병행 사용(통합 접근 방식) 을 강력히 추천한다.

## 동적 분석 도구

런타임 시점에 분석하는 도구이다. 실제 사용 환경에서 공격 시뮬레이션을 통해 취약점을 탐지한다.

| 도구명 | 분석 방식 | 라이선스 | 특징 |
| -- | -- | -- | -- |
| OWASP ZAP | DAST | 무료 |	OWASP 공식, 자동화 우수 |
| Burp Suite | DAST | 무료/유료 | 전문가용, 수동 진단 우수 |
| Nikto | 웹서버 | 무료 | 간단한 HTTP 진단 |
| AppSpider | DAST | 유료 | 기업용, DevOps 연계 가능 |