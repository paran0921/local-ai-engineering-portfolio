# Codex, OpenCode, Ollama, Laravel 기반 Windows 개발 워크플로 검증

**검증일:** 2026년 11월 11일  
**상태:** 공개 초판 — 후속 검증 결과에 따라 개정 가능

## 1. 개요

이 문서는 Windows 개발 환경에서 Codex 내부 터미널을 통해 OpenCode CLI를 실행하고, 별도 노트북인 HP OMEN에서 구동하는 Ollama 추론 서버에 연결하여 언어 모델의 응답을 확인한 과정을 기록한다.

이번 검증의 목적은 Codex 내부 터미널과 OpenCode, 원격 Ollama 서버 사이에서 발생한 네트워크 접근 및 설정 경로 문제를 해결하고, OpenCode가 설정된 모델에 프롬프트를 전달하여 정상 응답을 받을 수 있는지 확인하는 것이었다.

추가로 Laravel 12 프로젝트의 Composer 메타데이터와 User 모델을 읽기 전용으로 분석했다. 소스 코드 수정이나 PHPUnit 실행은 수행하지 않았다.

## 2. 시스템 아키텍처

현재 시스템은 두 대의 노트북이 서로 다른 역할을 담당하는 구조다.

| 장비 | 역할 | 담당 기능 |
|---|---|---|
| Lenovo ThinkPad X1 Carbon | Development Client | Codex 및 OpenCode 실행, Laravel 프로젝트 관리 |
| HP OMEN MAX | Ollama Inference Server | Ollama 실행 및 LAN을 통한 모델 추론 제공 |

ThinkPad에서는 자체 Ollama 모델을 사용한 추론을 수행하지 않는다. ThinkPad는 Development Client 역할을 담당하며, 실제 모델 추론 요청은 HP OMEN의 Ollama Inference Server로 전달한다.

향후 사양이 더 높은 노트북으로 교체하여 로컬 추론을 사용하는 방안은 가능성으로만 남겨 둔다. 이는 이번 검증에 포함된 현재 구성은 아니다.

### 요청 흐름

~~~text
Development Client (ThinkPad)
        |
        | Codex 내부 터미널
        v
     OpenCode CLI
        |
        | OpenAI 호환 API (/v1)
        v
Ollama Inference Server (HP OMEN)
        |
        v
qwen3:8b-agent-32k
~~~

## 3. 검증 환경

| 구성 요소 | 확인된 값 |
|---|---|
| 개발 클라이언트 OS | Windows 11 Pro |
| 추론 서버 OS | Windows 11 Home |
| Codex CLI | 0.166.0 |
| OpenCode | 1.18.33 |
| Ollama | 0.40.16 |
| Laravel 프로젝트 | Laravel 12 |
| PHP 요구 버전 | ^8.2 |
| 추론 성공 모델 | qwen3:8b-agent-32k |

Ollama 서버는 OpenAI 호환 API를 제공하며, 개발 클라이언트는 LAN을 통해 해당 API에 접근한다.

qwen3:8b-agent-32k는 로컬 Ollama에 등록된 모델 태그다. 모델 태그의 이름만으로 실제 실행 시점의 Context Length가 확인되는 것은 아니다. 활성 Context Length는 별도의 런타임 검증이 필요하다.

## 4. 발생한 문제와 해결 과정

### 4.1 Codex 내부 터미널에서 Ollama 서버에 접근하지 못하는 문제

**관찰된 현상**

- 별도 PowerShell에서는 Ollama 서버의 TCP 포트 연결에 성공했다.
- Codex 내부 터미널에서는 동일한 연결 시도가 실패했다.
- Codex 상태 정보에서 Windows sandbox가 elevated 모드로 실행되고 있었으며, workspace network access가 활성화되어 있지 않은 것을 확인했다.

**해결**

사용자의 명시적인 승인 후 Codex 사용자 설정에 다음 항목을 추가했다.

~~~toml
[sandbox_workspace_write]
network_access = true
writable_roots = ['C:\projects\laravel-app']
~~~

Codex를 재시작한 뒤 내부 터미널에서 TCP 연결을 다시 확인했고, 연결에 성공했다.

**보안상 고려 사항**

network_access = true는 해당 workspace-write sandbox의 일반적인 외부 네트워크 접근을 허용한다. Ollama 서버 한 곳이나 특정 포트만 허용하는 제한 설정은 아니다.

따라서 다른 환경에 적용할 때는 허용 범위가 넓어진다는 점을 고려해야 한다.

설정 변경 전 기존 파일의 백업을 생성했다.

### 4.2 OpenCode 초기화 및 파일시스템 경로 오류

OpenCode는 초기 실행 과정에서 기본 설정 및 데이터 디렉터리를 생성하거나 접근할 때 파일시스템 오류를 발생시켰다.

관찰된 오류는 다음과 같다.

- 기본 OpenCode 설정 디렉터리 생성 과정의 EEXIST
- 기본 OpenCode 로그 경로 접근 과정의 FileSystem.open 실패

XDG_CONFIG_HOME만 변경하는 것으로는 모든 경로 문제가 해결되지 않았다.

**해결**

진단 세션에서 네 가지 XDG 경로를 모두 임시 디렉터리 아래로 분리했다.

~~~powershell
$base = Join-Path $env:TEMP 'opencode-isolated-test'

$env:XDG_CONFIG_HOME = Join-Path $base 'config'
$env:XDG_DATA_HOME   = Join-Path $base 'data'
$env:XDG_STATE_HOME  = Join-Path $base 'state'
$env:XDG_CACHE_HOME  = Join-Path $base 'cache'
~~~

이후 격리된 세션에서 OpenCode 초기화가 진행되었다.

이 설정은 진단 세션에 적용한 환경 변수이며, 시스템 전체에 영구 적용한 설정은 아니다.

### 4.3 OpenCode가 의도한 프로젝트 설정을 사용하지 못한 문제

XDG 경로를 격리한 후에도 OpenCode에서 Provider/Model 확인 오류가 발생했다. 모델 목록 조회 명령에서는 Provider not found: ollama가 반환되었다.

프로젝트별 OpenCode 설정 파일은 존재했지만, 해당 진단 실행에서는 그 파일이 사용되지 않고 있었다.

**해결**

Laravel 프로젝트 디렉터리에서 실행할 세션의 프로젝트 설정 파일을 명시적으로 지정했다.

~~~powershell
$env:OPENCODE_CONFIG = (Resolve-Path '.oc\config\opencode\opencode.json').Path
~~~

이후 다음 명령으로 Ollama 모델 목록을 조회했다.

~~~powershell
opencode models ollama
~~~

확인된 모델 목록:

~~~text
ollama/qwen3:8b
ollama/qwen3:8b-agent-32k
ollama/qwen7-cline:latest
~~~

이 결과는 해당 세션에서 OpenCode가 Ollama Provider와 모델을 인식했다는 의미다. 모델 목록에 표시되는 것만으로 실제 추론 성공을 의미하지는 않는다.

## 5. 실제 추론 응답 검증

XDG 경로를 격리하고 프로젝트 설정 파일을 명시한 상태에서 Codex 내부 터미널로 다음 명령을 실행했다.

~~~powershell
opencode run --model ollama/qwen3:8b-agent-32k "Reply with exactly: CODEX_OPENCODE_32K_OK"
~~~

실제 응답:

~~~text
> build · qwen3:8b-agent-32k
CODEX_OPENCODE_32K_OK
~~~

**종료 코드:** 0

이번 검증으로 다음 항목을 확인했다.

1. Codex 내부 터미널에서 원격 Ollama 서버로 네트워크 연결이 가능했다.
2. OpenCode가 명시적으로 지정한 프로젝트 설정 파일을 불러왔다.
3. OpenCode가 설정된 Ollama 모델을 인식했다.
4. OpenCode가 프롬프트를 전달하고 기대한 모델 응답을 수신했다.

이는 짧은 프롬프트를 이용한 추론 확인이다. 장문 Context 안정성, Tool Calling의 정확성, 자율적인 코드 개발 능력 또는 운영 환경 적합성까지 검증한 것은 아니다.

## 6. Laravel 프로젝트 읽기 전용 분석

Laravel 프로젝트의 소스 파일은 수정하지 않은 상태에서 분석했다.

### Composer 메타데이터

composer.json 확인 결과:

| 항목 | 확인된 값 |
|---|---|
| Package | laravel/laravel |
| PHP 요구 버전 | ^8.2 |
| Laravel 요구 버전 | ^12.0 |

Composer 메타데이터 조회 명령은 정상 종료했다.

### User 모델 분석

app/Models/User.php를 읽고 다음 내용을 확인했다.

- Namespace: App\Models
- Traits: HasFactory, Notifiable
- Fillable: name, email, password
- Hidden: password, remember_token
- Casts: email_verified_at은 datetime, password는 hashed
- Relationship: posts()는 HasMany 관계를 반환
- isAdmin()은 is_admin 값의 truthiness를 확인

isAdmin()에 대해서는 실제 코드에서 확인한 truthiness 검사만 기술했다. 별도의 타입 의미나 권한 정책을 추정하지 않았다.

## 7. 검증 범위 및 한계

| 검증 항목 | 결과 |
|---|---|
| Codex 내부 터미널 → Ollama TCP 연결 | 성공 |
| 격리된 OpenCode 초기화 | 성공 |
| 프로젝트 설정 파일 명시적 선택 | 성공 |
| OpenCode의 Ollama 모델 등록 확인 | 성공 |
| OpenCode 짧은 추론 요청 | 성공 |
| Composer 메타데이터 확인 | 성공 |
| Laravel User 모델 읽기 전용 분석 | 완료 |
| PHPUnit 테스트 작성 | 미수행 |
| PHPUnit 테스트 실행 | 미수행 |
| Laravel 소스 코드 수정 | 미수행 |
| 에이전트 기반 전체 개발 워크플로 | 미검증 |
| 장문 Context 동작 및 안정성 | 미검증 |

따라서 이번 결과는 연결, 설정, 짧은 추론 응답에 대한 검증으로 표현해야 한다. 전체 AI 개발 워크플로가 운영 준비를 마쳤다는 근거로 사용해서는 안 된다.

## 8. 후속 검증 계획

현재 검증 기준선을 유지하면서 다음 항목을 별도로 진행한다.

1. User::isAdmin()에 대한 PHPUnit 테스트를 작성하고 실행한다.
2. 기존 OpenCode 1.18.33 검증 환경을 대체하지 않고 별도 환경에서 OpenCode v2를 평가한다.
3. 별도 프로젝트 또는 격리 환경에서 Laravel 13을 평가한다.
4. 각 검증 결과에 근거가 확보된 후 비교 결과와 문서를 갱신한다.

## 9. 결론

2026년 11월 11일 검증에서는 Codex sandbox의 네트워크 접근 제한과 OpenCode의 설정 및 경로 문제를 해결하고, 원격 Ollama 서버를 통한 짧은 추론 요청을 성공적으로 완료했다.

현재 확인된 시스템 구조는 ThinkPad의 Development Client와 HP OMEN의 Ollama Inference Server로 구성된다.

Laravel 프로젝트 분석은 읽기 전용으로 수행했다. Laravel 소스 파일은 수정하지 않았고 PHPUnit 테스트도 실행하지 않았다. 따라서 자율적인 개발 작업 전체가 검증되었다고 주장하지 않는다.

**공개 안내:** 본 문서는 최초 공개 버전이다. 추가 검증이 완료되면 후속 개정에서 정정 및 증거를 보완할 수 있다.
