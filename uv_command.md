# uv 명령어 레퍼런스

[uv](https://docs.astral.sh/uv/)는 Rust로 작성된 빠른 Python 패키지·프로젝트 관리 도구입니다.
`pip`, `venv`, `pyenv`, `pipx`, `poetry`의 역할을 하나로 대체합니다.

> 작성 기준 버전: uv 0.13.0

---

## 1. 설치 및 업데이트

```powershell
# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# pip로 설치
pip install uv
```

| 명령어 | 설명 |
|---|---|
| `uv --version` | 설치된 버전 확인 |
| `uv self update` | uv 자체를 최신 버전으로 업데이트 |
| `uv help <명령어>` | 명령어 도움말 보기 |

---

## 2. 프로젝트 관리

### 프로젝트 생성

| 명령어 | 설명 |
|---|---|
| `uv init` | 현재 폴더를 uv 프로젝트로 초기화 |
| `uv init <이름>` | 새 폴더를 만들고 프로젝트 생성 |
| `uv init --lib <이름>` | 라이브러리(패키지) 형태로 생성 (`src/` 구조) |
| `uv init --app <이름>` | 애플리케이션 형태로 생성 (기본값) |
| `uv init --python 3.12` | 사용할 Python 버전을 지정해 생성 |

생성되는 주요 파일:

```
my-project/
├── .python-version   # 사용할 Python 버전
├── pyproject.toml    # 프로젝트 설정과 의존성 목록
├── README.md
└── main.py
```

`uv sync`나 `uv run`을 처음 실행하면 `.venv/`(가상환경)와 `uv.lock`(잠금 파일)이 추가로 생성됩니다.

### 의존성 추가와 제거

| 명령어 | 설명 |
|---|---|
| `uv add <패키지>` | 패키지 추가 (`pyproject.toml`과 `uv.lock`에 반영하고 설치) |
| `uv add "requests>=2.31"` | 버전 조건을 지정해 추가 |
| `uv add <패키지> --dev` | 개발용 의존성으로 추가 (pytest, ruff 등) |
| `uv add <패키지> --group <그룹>` | 특정 의존성 그룹에 추가 |
| `uv add <패키지> --optional <이름>` | 선택적 의존성(extra)으로 추가 |
| `uv add -r requirements.txt` | requirements.txt의 패키지를 한꺼번에 추가 |
| `uv add git+https://github.com/<user>/<repo>` | Git 저장소에서 추가 |
| `uv add --editable ./경로` | 로컬 패키지를 편집 가능 모드로 추가 |
| `uv remove <패키지>` | 패키지 제거 |
| `uv remove <패키지> --dev` | 개발용 의존성에서 제거 |

### 동기화와 잠금

| 명령어 | 설명 |
|---|---|
| `uv sync` | `uv.lock` 기준으로 `.venv`를 맞춤 (설치와 불필요 패키지 제거) |
| `uv sync --no-dev` | 개발용 의존성을 빼고 동기화 |
| `uv sync --all-extras` | 모든 선택적 의존성을 포함해 동기화 |
| `uv sync --frozen` | lock 파일을 갱신하지 않고 그대로 설치 (CI용) |
| `uv sync --locked` | lock 파일이 최신이 아니면 오류 발생 (CI 검증용) |
| `uv lock` | `uv.lock` 파일 생성 또는 갱신 |
| `uv lock --upgrade` | 모든 패키지를 허용 범위 내 최신 버전으로 갱신 |
| `uv lock --upgrade-package <패키지>` | 특정 패키지만 갱신 |

### 실행

| 명령어 | 설명 |
|---|---|
| `uv run <파일>.py` | 프로젝트 가상환경에서 스크립트 실행 (필요하면 자동으로 sync) |
| `uv run python` | 가상환경의 Python 인터프리터 실행 |
| `uv run pytest` | 가상환경에 설치된 도구 실행 |
| `uv run --with <패키지> <파일>.py` | 일회성으로 패키지를 추가해 실행 |
| `uv run --python 3.11 <파일>.py` | 특정 Python 버전으로 실행 |

> `uv run`을 쓰면 가상환경을 직접 활성화(`activate`)할 필요가 없습니다.

### 확인과 내보내기

| 명령어 | 설명 |
|---|---|
| `uv tree` | 의존성 트리 출력 |
| `uv tree --outdated` | 업데이트 가능한 패키지 표시 |
| `uv export --format requirements.txt -o requirements.txt` | lock 파일을 requirements.txt로 내보내기 |
| `uv version` | 프로젝트 버전 확인 |
| `uv version --bump minor` | 프로젝트 버전 올리기 (`major` / `minor` / `patch`) |

---

## 3. 단일 스크립트 실행

프로젝트 없이 스크립트 하나에 의존성을 붙여 실행할 수 있습니다.

| 명령어 | 설명 |
|---|---|
| `uv run script.py` | 스크립트 실행 |
| `uv run --with requests script.py` | 일회성 의존성과 함께 실행 |
| `uv add --script script.py requests` | 스크립트 파일 안에 의존성 메타데이터 기록 |
| `uv init --script script.py` | 메타데이터가 포함된 새 스크립트 생성 |

`uv add --script`를 실행하면 스크립트 상단에 다음 내용이 추가됩니다.

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "requests",
# ]
# ///
```

---

## 4. Python 버전 관리

| 명령어 | 설명 |
|---|---|
| `uv python list` | 설치 가능하거나 설치된 Python 버전 목록 |
| `uv python install 3.12` | 특정 Python 버전 설치 |
| `uv python install 3.11 3.12 3.13` | 여러 버전 한 번에 설치 |
| `uv python uninstall 3.11` | Python 버전 삭제 |
| `uv python pin 3.12` | 현재 프로젝트의 Python 버전 고정 (`.python-version` 작성) |
| `uv python find` | 현재 사용될 Python 경로 확인 |
| `uv python dir` | uv가 관리하는 Python 설치 경로 확인 |

---

## 5. 가상환경

| 명령어 | 설명 |
|---|---|
| `uv venv` | 현재 폴더에 `.venv` 생성 |
| `uv venv <이름>` | 지정한 이름으로 가상환경 생성 |
| `uv venv --python 3.12` | 특정 Python 버전으로 생성 |

가상환경 직접 활성화 (필요한 경우):

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1

# Windows cmd
.venv\Scripts\activate.bat

# macOS / Linux
source .venv/bin/activate

# 비활성화
deactivate
```

---

## 6. 도구(CLI) 실행과 설치

`pipx`를 대체하는 기능입니다. ruff, black 같은 CLI 도구를 프로젝트와 분리해 사용합니다.

| 명령어 | 설명 |
|---|---|
| `uvx <도구>` | 도구를 임시 환경에서 바로 실행 (`uv tool run`의 축약형) |
| `uvx ruff check .` | 예: ruff로 코드 검사 |
| `uvx <도구>@<버전>` | 특정 버전으로 실행 |
| `uvx --from <패키지> <명령>` | 패키지명과 명령어 이름이 다를 때 |
| `uv tool install <도구>` | 도구를 전역으로 설치 |
| `uv tool list` | 설치된 도구 목록 |
| `uv tool upgrade <도구>` | 도구 업그레이드 |
| `uv tool upgrade --all` | 모든 도구 업그레이드 |
| `uv tool uninstall <도구>` | 도구 삭제 |

---

## 7. pip 호환 인터페이스

기존 `pip` 방식에 익숙하거나 `pyproject.toml` 없이 쓸 때 사용합니다.
`pip` 명령 앞에 `uv`만 붙이면 됩니다.

| 명령어 | 설명 |
|---|---|
| `uv pip install <패키지>` | 패키지 설치 |
| `uv pip install -r requirements.txt` | requirements.txt로 설치 |
| `uv pip install -e .` | 현재 프로젝트를 편집 가능 모드로 설치 |
| `uv pip uninstall <패키지>` | 패키지 삭제 |
| `uv pip list` | 설치된 패키지 목록 |
| `uv pip show <패키지>` | 패키지 정보 확인 |
| `uv pip freeze > requirements.txt` | 설치된 패키지 목록 저장 |
| `uv pip compile requirements.in -o requirements.txt` | 버전이 고정된 requirements 생성 |
| `uv pip sync requirements.txt` | requirements.txt와 정확히 일치하도록 환경 맞춤 |
| `uv pip tree` | 의존성 트리 출력 |

> `uv add`는 `pyproject.toml`에 기록하지만, `uv pip install`은 기록하지 않습니다.
> uv 프로젝트에서는 `uv add`를 쓰는 것을 권장합니다.

---

## 8. 빌드와 배포

| 명령어 | 설명 |
|---|---|
| `uv build` | 패키지 빌드 (`dist/`에 wheel과 sdist 생성) |
| `uv publish` | PyPI에 배포 |
| `uv publish --token <토큰>` | 토큰을 지정해 배포 |

---

## 9. 캐시 관리

| 명령어 | 설명 |
|---|---|
| `uv cache dir` | 캐시 위치 확인 |
| `uv cache clean` | 캐시 전체 삭제 |
| `uv cache clean <패키지>` | 특정 패키지 캐시만 삭제 |
| `uv cache prune` | 사용하지 않는 캐시 정리 |

---

## 10. 자주 쓰는 작업 흐름

### 새 프로젝트 시작

```powershell
uv init my-project
cd my-project
uv add requests pandas
uv add --dev pytest ruff
uv run main.py
```

### 기존 프로젝트 내려받아 실행

```powershell
git clone <저장소 주소>
cd <프로젝트>
uv sync
uv run main.py
```

### requirements.txt 프로젝트를 uv로 옮기기

```powershell
uv init
uv add -r requirements.txt
```

---

## 11. 기존 도구와 비교

| 작업 | 기존 방식 | uv |
|---|---|---|
| 가상환경 생성 | `python -m venv .venv` | `uv venv` |
| 패키지 설치 | `pip install requests` | `uv add requests` |
| 의존성 설치 | `pip install -r requirements.txt` | `uv sync` |
| 스크립트 실행 | `activate` 후 `python main.py` | `uv run main.py` |
| Python 설치 | pyenv install 3.12 | `uv python install 3.12` |
| CLI 도구 실행 | `pipx run ruff` | `uvx ruff` |

---

## 참고 링크

- 공식 문서: https://docs.astral.sh/uv/
- 명령어 전체 레퍼런스: https://docs.astral.sh/uv/reference/cli/
- GitHub: https://github.com/astral-sh/uv
