# sqltune — SQL Server 실시간 모니터 (무료)

![.NET 11](https://img.shields.io/badge/.NET-11.0-512BD4)
![C# 15](https://img.shields.io/badge/C%23-15-239120)
![Blazor Hybrid](https://img.shields.io/badge/Blazor-Hybrid-5C2D91)
![SQL Server 2019+](https://img.shields.io/badge/SQL%20Server-2019%2B-CC2927)
![Windows x64](https://img.shields.io/badge/Windows-x64-0078D6)

**[최신 배포본 내려받기](https://github.com/Doni-Kim/sqltune-release/releases/latest)** ·
[English](README.md) · [매뉴얼 (HTML)](sqltune.html)

SQL Server 상태를 실시간으로 보는 데스크톱 모니터입니다. 창 하나 · 실행 파일 하나이고, 서버에는 아무것도 설치하지 않습니다 —
DMV · Query Store · 시스템 카탈로그만 읽습니다. 서버를 바꾸는 것은 세션 KILL 하나뿐이고, 그것도 확인을 누를 때만입니다.
부담 없이 쓰시라고 공유드립니다.

## 화면

| 실시간 대시보드 | Top SQL (Query Store) |
|---|---|
| ![대시보드](screenshots/dashboard.jpg) | ![Top SQL](screenshots/top-sql.jpg) |

| Lock Chain | 세션 상세 |
|---|---|
| ![Lock Chain](screenshots/locks.jpg) | ![세션 상세](screenshots/session-detail.jpg) |

| Health Check | 데드락 |
|---|---|
| ![Health Check](screenshots/health-check.jpg) | ![데드락](screenshots/deadlocks.jpg) |

| 알림 | 인덱스 진단 |
|---|---|
| ![알림](screenshots/alerts.jpg) | ![인덱스](screenshots/indexes.jpg) |

화면은 촬영용으로 잠깐 띄운 시험 서버(Docker)의 AdventureWorks 예제 DB 입니다.

## 설치·설정

- 압축을 풀고 폴더째 둔 뒤 `sqltune.exe` 를 실행하면 됩니다 (Single File Publishing).
- .NET 설치 불필요 — 런타임이 실행 파일에 포함되어 있습니다.
- 설정은 같은 폴더의 접속 파일입니다. zip 에 `sqltuneDB1.json` · `sqltuneDB2.json` 두 개가 들어 있습니다 — 서버 하나에 파일 하나.
  서버가 하나면 한 파일만 두고 나머지는 지우세요. 파일을 고쳐도 되고(아래 예시), 그냥 실행해도 됩니다 —
  들어 있는 값으로 붙지 못하면 그 값이 채워진 접속 창이 뜹니다. 실제로 접속된 뒤에 입력한 값을 저장합니다(비밀번호는 암호화해서 저장).

## 주요 기능

- **실시간 대시보드**: 버퍼 캐시 적중률 · SQL Server CPU · PLE · 메모리 · 메모리 부여 대기 · tempdb · 초당 배치 / 트랜잭션 / 컴파일 / 재컴파일 / 페이지 분할 / 전체 스캔 / 락 대기 ·
  디스크 처리량과 지연 · 대기 · 추이 그래프 · 세션 표. 초당 값은 늘 직전 주기 구간 값입니다(서버 시작 이후 누적이 아님).
  SQL Server CPU 는 **SQL Server 가 쓸 수 있는 CPU 대비**입니다 — affinity 나 에디션 코어 제한으로 호스트 CPU 일부만 받은 서버(예: 8개 중 4개)는
  그 4개가 다 차면 100% 이고, 게이지 아래에 "4 of 8 CPUs" 와 호스트 전체 대비 값을 함께 보여 줍니다.
- **진짜 뿌리를 찾는 Lock Chain**: 트랜잭션을 연 채 쉬는 세션(`idle-tx`)까지 막힘의 뿌리로 잡습니다 — 실행 중인 요청만 보는 방식이 놓치는 흔한 원인입니다.
  `F5` 는 막는 세션과 막힌 세션을 함께 보여 줍니다.
- **세션 상세**(`Enter`): 문장 · 배치 전문, 지금까지의 실제 행 수가 든 라이브 실행 계획(없으면 캐시된 계획) — **SSMS 처럼 그림으로**(상자와 화살표, 상자에 마우스를 올리면 같은 속성 툴팁) 또는 글로 — 락.
  `Ctrl+K` 로 KILL(로그인 시각까지 다시 확인해 번호가 재사용된 다른 세션은 끊지 않습니다), `Ctrl+X` 로 Excel 저장 — 묶음 시트에 계획 그림과 연산자 표(다른 프로그램 없이 읽힘, `.sqlplan` 파일도 옆에) · 락 · 플랜에 나온 테이블의 크기 · 인덱스 · 통계까지.
- **팝업**: Server(`I`) · Connections(`C`) · Locks(`A`) · Disk(`D`) · Waits(`W`) · Deadlocks(`E`, system_health 에서, 최근 것은 풀어서) ·
  Statistics(`U`, UPDATE STATISTICS 스크립트) · Indexes(`X`: 누락(CREATE) · 미사용(DROP) · 중복 · 겹침, 조각화는 버튼으로).
- **Top SQL**(`T`): Query Store 에서 시간 · CPU · 읽기 · 실행 수 · 종합 점수 순으로 1시간 / 24시간 / 7일, 플랜이 여러 개인 문장,
  느려진 쿼리(앞 7일보다 2배 이상), 문장 검색, 머리글을 눌러 화면에서 다시 정렬, 쿼리별 플랜 이력과 플랜 강제 스크립트(보여 주기만 하고 실행하지 않음).
  상세에서 플랜을 그림으로 보고, **Object Info** 로 플랜이 읽는 테이블마다 컬럼 · 인덱스 · 통계를 봅니다 — 조건에 나온 컬럼 표시, 컬럼의 `CONVERT_IMPLICIT` 경고(Query Store 에서 읽을 뿐 다시 실행하지 않음).
- **Health Check**(`G`): 10개 범주(서버 설정 · 성능 · DB 설정 · 인덱스 · 통계 · tempdb · 파일 · 백업 · Agent 작업 · 로그)를
  점수 · 등급 · 먼저 고칠 것으로 보여 주고 Excel 로 저장. MAXDOP 은 NUMA 노드 기준, PLE 는 버퍼 풀 크기 기준, 백업은 서버 시계 기준으로 판정합니다.
- **임계값 알림** 12종: CPU(SQL Server 가 쓸 수 있는 CPU 대비) · Lock Chain · 막힌 요청 · idle in transaction · 긴 요청 · 버퍼 캐시 적중률 · PLE · Memory Grants Pending ·
  tempdb 사용률 · 디스크 읽기 / 쓰기 지연 · 데드락.
- **History**: `L` 로 로컬 SQLite 에 모니터링 데이터를 쌓고, `H` 로 지난 흐름을 되짚습니다.
  - 구간은 1시간 / 6시간 / 24시간 / 1주 / 1개월 / 전체, 지표는 20가지입니다.
  - 오래된 기록은 자동으로 지웁니다(기본 지표 30일 · 세션 7일, `O` 설정 창에서 조정).
- Excel 저장: ClosedXML 기반이라 Excel 이 없어도 xlsx 파일이 저장됩니다.
- **테마 12종**: 밝은 6 · 어두운 6, 기본은 GitHub Light. 상단 바에서 고릅니다. 접속이 끊기면 스스로 다시 붙습니다.
- **2.1 에서 더한 것**: Jobs 팝업(`J`) — SQL Server Agent 의 지금 도는 잡 · 실패한 단계와 오류 글 · 잡마다 마지막 결과와 다음 실행. Windows 인증(접속 창의 체크박스 또는 `"windowsAuth": true`). Top SQL 에 쿼리별 주된 대기.
- **찾기**: `/` 로 세션을 글자로 거릅니다.
- **알림이 켜지는 순간**: 막힘 트리 · 문장(`.txt`)이나 데드락 그래프(`.xdl`)를 `captures\` 아래에 남기고, 위험 알림은 창이 앞에 없을 때 작업 표시줄 깜빡임 · Windows 알림으로 알려 줍니다.
- History 에서 한 시점을 누르면 그때 기록된 세션이 나옵니다.
- **Most read tables**(Indexes, `X`): 디스크 읽기(PAGEIOLATCH)를 기다린 테이블 — 전체 대비 비율과 `Δ delta`. **Scan buffer pool** 은 지금 메모리를 차지한 테이블. 두 스캔 버튼은 경고를 먼저 띄우고, 제한 시간은 `indexes.fragmentationTimeoutSec` · `indexes.bufferPoolTimeoutSec`.
- **System Stored Procedures**: `F1` 의 두 번째 탭에서 서버의 시스템 저장 프로시저(master · msdb — SQL Server 2025 에서 1,600개 넘게)를
  글자를 칠 때마다 찾습니다. 목록과 매개변수는 접속한 서버에서 읽어 버전과 늘 맞고, 700개쯤에 한 줄 설명, 자주 쓰는 169개에 복사해 쓰는 샘플이 있습니다 —
  프로그램에 들어 있어 인터넷이 필요 없습니다. sqltune 은 실행하지 않습니다.
- **DBCC 명령**: 같은 탭에 DBCC 명령 35개가 있습니다(`dbcc` 로 찾기 — Microsoft 문서 목록의 32개 전부를 문서의 네 갈래대로, 그리고 `MEMORYSTATUS` · `LOGINFO` · `PAGE`) — `SQLPERF(LOGSPACE)` · `OPENTRAN` · `INPUTBUFFER` · `SHOW_STATISTICS` · `CHECKDB` · `CHECKIDENT` ·
  `SHRINKFILE` · `FREEPROCCACHE` · `TRACEON` … 인자 · 필요한 권한 · 복사해 쓰는 샘플과 함께, 폐기 예정 · 문서 없음도 표시합니다.
- **서버마다 설정 파일 하나**: exe 옆에 둘 이상이면 시작할 때 어느 것으로 붙을지 고르는 창이 뜹니다. 손으로 고치다 깨진 파일은 몇째 줄 몇째 글자가 틀렸는지와 함께 빨갛게 보입니다.
- **설정 창**: `O` 로 수집 주기(3~60초, 기본 5초) · 로그 보관 · Indexes 스캔 제한 시간 · Excel · 알림 임계값 · `L` 로깅이 남길 세션을 화면에서 고칩니다.
  값을 검사한 뒤 지금 쓰는 설정 파일에 저장하고 바로 적용합니다. 접속 정보는 시작 접속 창에서만 바꿉니다.
- **시작할 때 서버가 응답하지 않으면**: 30초 동안 빈 화면 대신, 1초 뒤 누구에게 몇 초째 붙는 중인지 보이는 작은 창이 뜨고 Cancel 로 접속 정보를 고칠 수 있습니다.
- `Ctrl+B` 로 DB 단위 팝업이 볼 데이터베이스를 바꿉니다. `F1` 을 누르면 단축키 도움말이, 한 번 더 누르면 System Procedures & DBCC 탭이 나옵니다.

자세한 사용법은 첨부한 `sqltune.html`(스크린샷이 든 매뉴얼)을 참고해 주세요. 사용상 제한 없습니다.

## 지원 범위 · 제약사항

- UI 가 Web 기반(Blazor Hybrid)이라 **Windows 전용**입니다. Linux·macOS 에서는 단독 실행되지 않습니다.
- **SQL Server 2019 이상** — 온프레미스 · VM · Managed Instance. SQL Server 2025 에서 개발 · 확인했습니다.
- **Azure SQL Database 는 지원하지 않습니다** — 접속하면 접속 창에서 알려 줍니다.
- **Top SQL 은 Query Store 가 필요합니다**(보는 DB 에). SQL Server 2022 부터 새 DB 는 기본으로 켜져 있고,
  꺼져 있으면 켜는 문장을 팝업이 보여 줍니다.
- 코드는 ConfuserEx 로 난독화(무료 툴이라 강력한 수준은 아닙니다).

## 모니터링 전용 계정

SQL Server 2025 에서 확인했습니다 — 아래 권한이면 모든 팝업과 Health Check 가 `sa` 로 볼 때와 같습니다.

```sql
USE master;
CREATE LOGIN sqltune WITH PASSWORD = '...';
GRANT VIEW SERVER STATE    TO sqltune;   -- 대시보드 · 세션 · 대기 · 락 · 데드락 · 실행 계획
GRANT VIEW ANY DEFINITION  TO sqltune;   -- Disk: 모든 DB 의 파일 · 볼륨
GRANT ALTER ANY CONNECTION TO sqltune;   -- 선택: Ctrl+K (KILL)

-- Ctrl+B 로 보는 DB 마다 (Top SQL · 인덱스 · 통계 · Health Check)
USE YourDatabase;
CREATE USER sqltune FOR LOGIN sqltune;
GRANT VIEW DATABASE STATE TO sqltune;           -- Query Store · 인덱스 사용량 · 로그 공간
GRANT VIEW DEFINITION     TO sqltune;           -- 인덱스 · 통계 메타데이터
ALTER ROLE db_datareader ADD MEMBER sqltune;    -- 선택: 통계 — 아래 참고

-- Health Check: 백업 · Agent 작업
USE msdb;
CREATE USER sqltune FOR LOGIN sqltune;
GRANT SELECT  ON dbo.backupset      TO sqltune;
GRANT SELECT  ON dbo.sysjobs        TO sqltune;
GRANT SELECT  ON dbo.sysjobhistory  TO sqltune;
GRANT EXECUTE ON dbo.agent_datetime TO sqltune;
```

- `sys.dm_db_stats_properties` 는 그 로그인이 읽을 수 있는 테이블의 통계만 돌려주므로, `db_datareader` 가 없으면
  Statistics 팝업과 Health Check 의 통계 범주가 비어 보입니다. 나머지는 없어도 됩니다.
- `VIEW ANY DEFINITION` 이 없으면 Disk 팝업에 파일 · 볼륨이 나오지 않습니다.
- Agent 점검에는 `SQLAgentReaderRole` 로는 부족합니다 — 그 역할은 Agent 프로시저용이고 작업 테이블을 직접 읽지는 못합니다.
- 권한이 모자라면 빈 목록 대신 어떤 권한이 필요한지 팝업에 적어 줍니다.

## 화면이 안 뜬다면 (WebView2)

창은 뜨는데 내용이 백지라면 WebView2 런타임이 없는 경우입니다.

- **Windows 11**: OS 에 기본 내장이라 항상 있습니다.
- **Windows 10**: 2021년 이후 Windows Update 로 대부분 자동 배포됐지만, 업데이트를 오래 안 한 PC 나 LTSC 같은 특수 에디션에는 없을 수 있습니다.
- **Windows Server (2016/2019/2022)**: 기본 미포함인 경우가 많아 별도 설치가 필요할 수 있습니다.

없으면 Microsoft 의 "에버그린 독립형 설치 프로그램"(`MicrosoftEdgeWebView2RuntimeInstallerX64.exe`)을 설치하시면 됩니다.
→ https://developer.microsoft.com/microsoft-edge/webview2/

## 문제가 생기면

오류가 나면 실행 파일 옆에 `sqltune.log` 가 생깁니다(평소에는 만들지 않습니다).

- **버그 · 질문**: [Issues](https://github.com/Doni-Kim/sqltune-release/issues) 에 남겨 주세요.
  로그 파일은 거기에 올리지 마세요 — 비밀번호는 없지만 서버 주소와 SQL 문장이 들어 있을 수 있습니다.
- **로그 파일**이나 공개하기 어려운 내용은 메일로 보내 주세요: **doniikim@gmail.com**

## 기술 스택

- .NET 11.0 (x64), C# 15, Blazor Hybrid
- 패키지: Microsoft.Data.SqlClient · Microsoft.Data.Sqlite · ClosedXML · Microsoft.Web.WebView2 · Microsoft.AspNetCore.Components.WebView.WindowsForms
- 실행 파일에 묶인 위 구성 요소들의 저작권 고지 · 라이선스 전문: [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) (zip 안에도 들어 있습니다)
- F1 의 시스템 저장 프로시저 한 줄 설명은 Microsoft SQL Server 문서에서 옮겼습니다
  ([MicrosoftDocs/sql-docs](https://github.com/MicrosoftDocs/sql-docs), © Microsoft, CC BY 4.0) — 자세한 내용은 THIRD-PARTY-NOTICES.txt.

## 접속 파일 예시 (`sqltuneDB1.json` …)

zip 에는 기본값(`127.0.0.1:1433` · `sa`)이 든 작은 틀이 `sqltuneDB1.json` · `sqltuneDB2.json` 두 이름으로 들어 있습니다 —
서버마다 파일 하나, 이름은 자유입니다(`prod.json` · `dev.json` …). `sqltune.exe` 옆에 둘 이상이면 시작할 때 고르는 창이 뜨고
(목록에는 `login@server,port/database` 만 — 비밀번호는 보이지 않습니다), 하나뿐이면 바로 붙습니다.
고른 파일이 그 실행의 설정이 되어 암호화된 비밀번호 · 창 위치 · 테마가 그 파일에 저장됩니다.
같은 폴더의 sqltune 은 한 번에 하나만 뜨니, 여러 서버를 동시에 보려면 폴더를 나누세요. 파일을 고쳐서 쓰거나, 접속 창에 맡기면 됩니다:

```json
{
  "databases": [
    {
      "userId": "sa",
      "password": "<password>",
      "server": "127.0.0.1",
      "port": "1433",
      "database": "master",
      "encrypt": "mandatory",
      "trustServerCertificate": true
    }
  ],
  "interval": 5
}
```

- `password` 는 평문으로 적으면 첫 실행 때 자동 암호화되어 `ENC1:...` 로 저장됩니다.
- 이름 있는 인스턴스는 `server` 에 `HOST\\INSTANCE` 로 쓰고 `port` 는 비우거나 `1433` 으로 둡니다(SQL Browser 로 포트를 찾습니다).
- `database` 는 DB 단위 팝업이 처음 볼 DB 이고 `Ctrl+B` 로 바꿉니다. 세션 목록은 늘 서버 전체입니다.
- `encrypt`: `mandatory`(기본) / `optional` / `strict`. `trustServerCertificate` 는 자체 서명 인증서를 받아들일지, `indexes.fragmentationTimeoutSec`(300) · `indexes.bufferPoolTimeoutSec`(30)은 Indexes 의 두 스캔 버튼 제한 시간(5 ~ 3600초)입니다.
- `interval` 은 수집 주기(초)입니다. 3~60, 적지 않으면 5.
- `alerts`, `logRetention`, `logFilter`(`L` 로깅이 남길 세션 — 2.2 까지는 `mssql_monitor_filter.json` 이었고 그 규칙은 옮겨 적습니다) 같은 절은 적지 않아도 됩니다.
  sqltune 이 파일을 쓸 때 기본값으로 채워 넣고, `O` 로 화면에서 고칠 수 있습니다. zip 안의 `sqltune_sample_kr.json` 에 모든 설정의 설명이 있습니다.

## 사용 조건

업무든 개인이든 자유롭게 쓰셔도 됩니다. 실행 파일 재배포와 리버스 엔지니어링은 삼가 주세요.
소스는 공개하지 않습니다.

sqltune 에는 Microsoft 구성 요소(SQL Server 통신 라이브러리 Microsoft.Data.SqlClient.SNI 등)를 비롯한 외부 구성 요소가 들어 있고,
각각은 [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt)(zip 안에도 있음)에 전문이 실린 자기 라이선스 조건을 따릅니다. sqltune 을 쓰시면 그 조건에도 동의하시는 것으로 봅니다.

## 연락처

DBMS Works — **doniikim@gmail.com**

Oracle → PostgreSQL / MySQL 마이그레이션, DB 성능 튜닝 문의도 받습니다.
