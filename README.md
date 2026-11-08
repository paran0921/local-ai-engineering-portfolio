# Local AI Engineering Portfolio

로컬 AI 환경 구성과 검증, 코딩 에이전트 평가 방향 및 RAG 기초 실험을 기록하는 기술 포트폴리오입니다.

이 저장소는 실제 수행한 작업과 확인된 결과를 중심으로 구성하며, 검증하지 않은 항목과 향후 계획은 완료된 결과와 구분해 기록합니다.

## 공개 문서

### 1. 로컬 AI 변화 대응 전략 및 에이전트 평가 이력

[문서 보기](docs/2026-10-15_local-ai-adaptation-and-agent-evaluation-public-draft.md)

- Windows 네이티브 개발 환경, Ollama, Aider를 기준으로 한 기존 아키텍처 유지 전략
- Aider, cecli, OpenCode, Jan.ai 등 코딩 에이전트의 평가 방향
- 대체 도구 도입 전 검증 및 변경 통제 원칙
- 후보 도구의 실제 평가와 적용은 문서에 명시된 범위에서 미완료 상태

### 2. Open WebUI RAG 기본 검증 결과

[문서 보기](docs/2026-10-18_open-webui-rag-basic-validation-public-draft.md)

- Open WebUI 지식 기반 첨부를 이용한 단일 문서 RAG 기초 검증
- 세 가지 기본 시나리오 결과: 통과
- 검색 정확도 일반화, 다중 문서 처리, 권한 격리 및 운영 안정성 등은 미검증

### 3. Local AI Architecture Baseline

[문서 보기](docs/architecture-baseline-public-draft.md)

- Windows 네이티브 개발 클라이언트와 Ollama 추론 서버의 기준 구성
- ThinkPad에서 OMEN 서버로 수행한 원격 API 및 모델 추론 확인
- Aider, CECLI, OpenCode 등 코딩 에이전트의 역할과 검증 범위
- 미검증 항목과 아키텍처 변경 조건 명시

### 4. 로컬 AI에서 하이브리드 AI까지

[문서 보기](docs/local-ai-to-hybrid-ai-experience-revised-draft.md)

- 실제 개발 환경에서 로컬 AI를 구성하고 원격 추론을 확인한 경험
- 로컬 및 클라우드 실행 환경을 선택하기 위한 판단 기준
- 코딩 에이전트 평가와 검증 가능한 변경 절차
- 하이브리드 AI는 완성된 운영 구성이 아닌 향후 적용 방향으로 구분

### 5. Laravel Development History

[문서 보기](docs/laravel-development-history-public-draft-v7.md)

- Laravel 프로젝트의 주요 Git 개발 이력
- 관리자 권한 및 동시성 관련 구현·테스트 기록
- SQLite 기반 개별 동시 삭제 및 전체 테스트 결과
- 테스트 범위와 미검증 데이터베이스 환경을 명시

## 문서 작성 원칙

- 확인된 사실, 관찰 결과, 계획 및 미검증 항목을 구분합니다.
- 제한된 테스트 결과를 일반적인 성능이나 운영 신뢰성의 보증으로 확대 해석하지 않습니다.
- 아키텍처 및 도구 변경은 재현 가능한 근거를 바탕으로 별도 판단합니다.

## 범위 안내

이 저장소에는 공개를 위해 선별한 문서만 포함합니다. 로컬 프로젝트 전체, 실험용 임시 파일, 비공개 설정 및 별도 검토 자료는 포함하지 않습니다.

## License

별도의 라이선스 파일이 추가되기 전까지 이 저장소의 문서에 대한 재사용 권한을 명시적으로 부여하지 않습니다.
