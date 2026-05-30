# macOS 개발 환경 셋팅 가이드

> **AI 에이전트용 자동화 가이드** — 이 문서를 순서대로 읽고 각 단계를 실행하여 macOS 개발 환경을 구성합니다.
> 모든 명령어는 복사하여 터미널에 붙여넣을 수 있습니다. 각 단계마다 성공 여부를 검증하세요.

> **읽는 방법** — `(선택)`·`(대안)` 표시가 붙은 섹션은 건너뛰어도 기본 환경 구성에는 지장이 없습니다.
> 특히 **버전 매니저는 택1**입니다. 기본 구성은 fnm(Node) + pyenv(Python)이며, 6.5의 `mise`로 통합하려면
> fnm·pyenv 단계를 건너뛰세요. 셋을 모두 활성화하면 `~/.zshrc`에서 PATH가 충돌합니다.

---

## 목차

1. [사전 확인](#1-사전-확인)
2. [Xcode Command Line Tools](#2-xcode-command-line-tools)
3. [Homebrew (패키지 매니저)](#3-homebrew-패키지-매니저)
4. [Git 설정](#4-git-설정)
5. [Shell 환경 (Zsh + Oh My Zsh + Starship)](#5-shell-환경-zsh--oh-my-zsh--starship)
6. [Node.js 환경](#6-nodejs-환경)
7. [Python 환경](#7-python-환경)
8. [Docker 환경](#8-docker-환경)
9. [VS Code (에디터)](#9-vs-code-에디터)
10. [AI/LLM 개발 도구](#10-aillm-개발-도구)
11. [기타 유틸리티](#11-기타-유틸리티)
12. [언어 런타임 & 데이터베이스](#12-언어-런타임--데이터베이스)
13. [클라우드 & 인프라 CLI](#13-클라우드--인프라-cli)
14. [폰트](#14-폰트)
15. [최종 검증](#15-최종-검증)

---

## 1. 사전 확인

### 1.1 macOS 버전 확인

```bash
sw_vers
```

> **요구사항**: macOS 13 (Ventura) 이상
> **검증**: `ProductVersion` 필드가 `13.x` 이상이면 통과
> **참고**: 이 가이드는 macOS 26.5 (Apple Silicon)에서 검증되었습니다.

### 1.2 CPU 아키텍처 확인

```bash
uname -m
```

> - `arm64` → Apple Silicon (M1/M2/M3) → Homebrew 경로: `/opt/homebrew`
> - `x86_64` → Intel → Homebrew 경로: `/usr/local`
>
> 이후 명령어는 아키텍처에 따라 Homebrew 경로를 자동으로 선택합니다.

### 1.3 디스크 여유 공간 확인

```bash
df -h / | tail -1 | awk '{print $4}'
```

> **요구사항**: 최소 20GB 이상 여유 공간 권장

---

## 2. Xcode Command Line Tools

macOS에서 컴파일 도구(git, make, clang 등)를 사용하기 위해 필요합니다.

```bash
xcode-select --install
```

> 이미 설치된 경우 `xcode-select: error: command line tools are already installed` 메시지가 표시됩니다.

**검증:**

```bash
xcode-select -p
# 출력 예시: /Library/Developer/CommandLineTools
git --version
# 출력 예시: git version 2.39.x
```

---

## 3. Homebrew (패키지 매니저)

### 3.1 설치

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> 설치 중 비밀번호 입력이 필요할 수 있습니다.

### 3.2 PATH 등록 (Apple Silicon 사용자인 경우)

```bash
if [ "$(uname -m)" = "arm64" ]; then
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)"
fi
```

**검증:**

```bash
brew doctor
# 출력: Your system is ready to brew.
brew --version
# 출력 예시: Homebrew 4.x 이상
```

### 3.3 필수 패키지 설치

```bash
brew install \
  git \
  wget \
  curl \
  jq \
  ripgrep \
  fd \
  tree \
  htop \
  tmux
```

**검증:**

```bash
for pkg in git wget curl jq rg fd tree htop tmux; do
  echo -n "$pkg: "
  command -v "$pkg" && echo "✓" || echo "✗ MISSING"
done
```

### 3.4 모던 CLI 도구 (선택)

기본 명령어를 더 빠르고 보기 좋게 대체하는 도구들입니다.

```bash
brew install \
  bat \
  eza \
  btop \
  fzf \
  mosh \
  pipx \
  git-lfs
```

> - `bat`(cat 대체, 문법 강조) · `eza`(ls 대체) · `btop`(htop 대체, 시각화)
> - `fzf`(퍼지 파인더) · `mosh`(끊김 없는 SSH) · `pipx`(Python CLI 격리 설치) · `git-lfs`(대용량 파일)
> - 프롬프트 `starship`, 통합 버전 매니저 `mise`, App Store CLI `mas`는 각 전용 섹션에서 설치합니다.

**검증:**

```bash
for pkg in bat eza btop fzf mosh pipx git-lfs; do
  echo -n "$pkg: "
  command -v "$pkg" && echo "✓" || echo "✗ MISSING"
done
```

---

## 4. Git 설정

### 4.1 사용자 정보 설정

```bash
# 아래 값을 본인 정보로 변경하세요
GIT_NAME="Jeongseok Oh"
GIT_EMAIL="jeongsk@example.com"

git config --global user.name "$GIT_NAME"
git config --global user.email "$GIT_EMAIL"
```

### 4.2 기본 설정

```bash
git config --global init.defaultBranch main
git config --global core.editor "code --wait"      # VS Code를 기본 에디터로
git config --global pull.rebase false
git config --global core.autocrlf input
git config --global core.precomposeunicode true     # macOS 한글 파일명 정규화
```

### 4.3 SSH 키 생성 (GitHub용)

```bash
ssh-keygen -t ed25519 -C "$GIT_EMAIL" -f ~/.ssh/id_ed25519 -N ""
```

**SSH 키를 GitHub에 등록하려면:**

```bash
cat ~/.ssh/id_ed25519.pub
```

> 출력된 공개키를 https://github.com/settings/keys 에서 등록하세요.

**검증:**

```bash
ssh -T git@github.com
# 출력: Hi jeongsk! You've successfully authenticated...
```

---

## 5. Shell 환경 (Zsh + Oh My Zsh + Starship)

### 5.1 Oh My Zsh 설치

```bash
# --unattended: 설치 후 새 zsh 세션으로 전환하지 않음(자동화 시 멈춤 방지)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
```

### 5.2 유용한 플러그인 설치

```bash
# zsh-autosuggestions (명령어 자동 완성)
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting (명령어 구문 강조)
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### 5.3 플러그인 활성화

```bash
# ~/.zshrc 의 plugins 라인을 찾아 다음으로 교체
sed -i '' 's/^plugins=(.*/plugins=(git zsh-autosuggestions zsh-syntax-highlighting docker docker-compose node npm python)/' ~/.zshrc
```

### 5.4 설정 반영

```bash
source ~/.zshrc
```

**검증:**

```bash
echo $SHELL
# 출력: /bin/zsh
omz version
# 출력 예시: 1.x.x
```

### 5.5 Starship 프롬프트 (선택)

> 빠르고 미니멀한 크로스 셸 프롬프트입니다. oh-my-zsh 테마 대신 사용할 수 있습니다.

```bash
brew install starship
echo 'eval "$(starship init zsh)"' >> ~/.zshrc
source ~/.zshrc
```

**검증:**

```bash
starship --version
# 출력 예시: starship 1.x.x
```

---

## 6. Node.js 환경

### 6.1 fnm 설치 (Fast Node Manager — nvm보다 빠르고 .node-version 자동 인식)

```bash
brew install fnm
```

### 6.2 fnm 초기화 설정

```bash
echo 'eval "$(fnm env --use-on-cd --shell zsh)"' >> ~/.zshrc
source ~/.zshrc
```

> **(선택) nvm 호환 alias** — `nvm` 명령이 손에 익었다면 alias를 추가할 수 있습니다.
> fnm은 `install`·`use`·`ls`·`current` 등 자주 쓰는 명령은 nvm과 호환되지만,
> `nvm alias default` 같은 일부 서브커맨드는 다릅니다(fnm은 `fnm default`).
> `.nvmrc`/`.node-version` 자동 인식은 위 `--use-on-cd` 옵션으로 이미 지원됩니다.
>
> ```bash
> echo 'alias nvm="fnm"' >> ~/.zshrc
> ```

### 6.3 Node.js LTS 버전 설치

```bash
fnm install --lts
fnm default lts-latest
fnm use lts-latest
```

**검증:**

```bash
node --version
# 출력 예시: v22.x.x (LTS)
npm --version
# 출력 예시: 10.x.x
fnm --version
# 출력 예시: 1.x.x
```

### 6.4 글로벌 패키지 설치

> `pnpm`은 Node 버전에 묶이지 않도록 Homebrew로 설치하는 것을 권장합니다.

```bash
brew install pnpm

npm install -g \
  typescript \
  ts-node \
  yarn \
  vercel \
  prettier \
  eslint
```

**검증:**

```bash
for cmd in tsc pnpm yarn vercel prettier eslint; do
  echo -n "$cmd: "
  command -v "$cmd" && echo "✓" || echo "✗ MISSING"
done
```

### 6.5 (대안) mise — 여러 언어를 한 도구로 관리

> [mise](https://mise.jdx.dev)는 Node·Python·Go·Elixir 등 여러 런타임 버전을 단일 도구로
> 관리합니다. fnm·pyenv를 따로 쓰는 대신 mise 하나로 통합할 수 있습니다.
> (PATH가 충돌할 수 있으니, 통합 시 기존 fnm/pyenv 초기화 라인을 정리하세요.)

```bash
brew install mise
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
source ~/.zshrc

# 예시: 전역 기본 버전 지정
mise use -g node@lts python@3.12
```

**검증:**

```bash
mise --version
mise ls
```

---

## 7. Python 환경

### 7.1 pyenv 설치 (Python 버전 관리)

```bash
brew install pyenv
```

### 7.2 pyenv 초기화 설정

> 가상환경까지 pyenv로 관리하려면 `pyenv-virtualenv`를 함께 설치합니다.

```bash
brew install pyenv-virtualenv

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
source ~/.zshrc
```

### 7.3 Python 설치

```bash
pyenv install 3.12
pyenv global 3.12
```

**검증:**

```bash
python --version
# 출력 예시: Python 3.12.x
pip --version
# 출력 예시: pip 24.x
```

### 7.4 Python 개발 도구 설치

```bash
pip install --upgrade pip setuptools wheel

# pipx PATH 등록 (최초 1회)
pipx ensurepath

# 독립 실행 CLI 도구는 pipx로 격리 설치 (전역 pip 오염·의존성 충돌 방지)
for tool in ipython black ruff mypy poetry pipenv pre-commit; do
  pipx install "$tool"
done
```

> `pytest`/`pytest-cov`는 보통 프로젝트별 가상환경에 설치합니다.
> 전역으로도 쓰려면: `pipx install pytest && pipx inject pytest pytest-cov`

**검증:**

```bash
for cmd in ipython black ruff mypy poetry pre-commit; do
  echo -n "$cmd: "
  command -v "$cmd" && echo "✓" || echo "✗ MISSING"
done
```

### 7.5 uv (초고속 패키지·프로젝트 매니저)

> uv는 Astral에서 만든 Rust 기반 도구로, pip·virtualenv·poetry를 대체할 수 있습니다.
> pyenv와 함께 사용하거나 단독으로 Python 버전까지 관리할 수 있습니다.

```bash
# Homebrew로 설치
brew install uv
```

**검증:**

```bash
uv --version
# 출력 예시: uv 0.x.x
```

**기본 사용 예시:**

```bash
# 새 프로젝트 초기화 (pyproject.toml 생성)
uv init myproject && cd myproject

# 패키지 추가 (가상환경 자동 생성·관리)
uv add requests

# Python 버전 설치·고정
uv python install 3.12

# 스크립트 실행
uv run python main.py
```

---

## 8. Docker 환경

### 8.1 Docker Desktop 설치

> Docker Desktop은 Homebrew로도 설치 가능하지만, 공식 다운로드를 권장합니다.

```bash
# Homebrew로 설치 (cask 정식 이름은 docker-desktop)
brew install --cask docker-desktop
```

### 8.2 Docker Desktop 실행

```bash
open -a Docker
```

> Docker Desktop이 처음 실행되는 동안 라이선스 동의와 권한 설정이 필요할 수 있습니다.
> Docker 엔진이 완전히 시작될 때까지 30~60초 기다립니다.

### 8.3 Docker Compose 확인

```bash
# Docker Desktop에 내장된 Compose V2 확인
docker compose version
```

**검증:**

```bash
docker run --rm hello-world
# 출력: Hello from Docker! ...
```

---

## 9. VS Code (에디터)

### 9.1 설치

```bash
brew install --cask visual-studio-code
```

### 9.2 CLI 등록

```bash
# code 명령어를 PATH에 등록 (VS Code 내에서 Cmd+Shift+P → "Shell Command: Install 'code' command in PATH")
# 또는 직접 심링크:
if [ "$(uname -m)" = "arm64" ]; then
  ln -sf "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" /opt/homebrew/bin/code
else
  ln -sf "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" /usr/local/bin/code
fi
```

### 9.3 필수 확장 프로그램 설치

```bash
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.black-formatter
code --install-extension charliermarsh.ruff
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint
code --install-extension bradlc.vscode-tailwindcss
code --install-extension ms-azuretools.vscode-docker
code --install-extension github.copilot
code --install-extension github.copilot-chat
code --install-extension eamodio.gitlens
code --install-extension yzhang.markdown-all-in-one
code --install-extension ms-vscode-remote.remote-containers
code --install-extension tldraw-org.tldraw-vscode
```

**검증:**

```bash
code --list-extensions --show-versions
```

---

## 10. AI/LLM 개발 도구

### 10.1 LangChain + 관련 패키지 (Python)

```bash
pip install \
  langchain \
  langchain-community \
  langchain-openai \
  langchain-anthropic \
  langgraph \
  chromadb \
  tiktoken \
  unstructured \
  python-dotenv
```

### 10.2 MCP (Model Context Protocol) 관련 도구

```bash
# Python MCP SDK
pip install mcp
```

> JS/TS SDK `@modelcontextprotocol/sdk`는 전역(-g)이 아니라 **프로젝트별 의존성**으로 설치합니다:
> `npm install @modelcontextprotocol/sdk`

### 10.3 Obsidian CLI (플러그인 개발용)

```bash
npm install -g obsidian-cli
```

### 10.4 n8n (워크플로우 자동화)

```bash
npm install -g n8n
```

**검증:**

```bash
# Python 패키지 import 검증
for env in langchain langgraph chromadb tiktoken mcp; do
  echo -n "$env: "
  if python -c "import ${env//-/_}" 2>/dev/null; then echo "✓"; else echo "✗"; fi
done

# n8n (전역 CLI) 검증
echo -n "n8n: "
if command -v n8n &>/dev/null; then echo "✓"; else echo "✗"; fi
```

---

## 11. 기타 유틸리티

### 11.1 GitHub CLI

```bash
brew install gh
gh auth login
```

> 웹 기반 인증 또는 토큰 인증을 선택하고 안내를 따르세요.

### 11.2 추가 개발 도구 (cask)

```bash
# 터미널
brew install --cask ghostty warp

# 생산성 유틸리티
brew install --cask raycast rectangle karabiner-elements shottr keka itsycal switchhosts

# 브라우저
brew install --cask brave-browser firefox google-chrome

# 협업·노트
brew install --cask slack discord obsidian notion

# AI 개발 앱
brew install --cask cursor zed chatgpt cherry-studio ollama-app lm-studio

# 미디어·DB·기타
brew install --cask iina vlc db-browser-for-sqlite bruno fork utm
```

**검증:**

```bash
for app in "Ghostty" "Raycast" "Rectangle" "Brave Browser" "Cursor" "Obsidian"; do
  echo -n "$app: "
  ls "/Applications/$app.app" &>/dev/null && echo "✓" || echo "✗ NOT INSTALLED"
done
```

### 11.3 Mac App Store 앱 (mas)

> [mas](https://github.com/mas-cli/mas)는 Mac App Store 앱을 CLI로 설치합니다.
> **먼저 App Store 앱에서 Apple 계정으로 로그인**해야 `mas install`이 동작합니다.

```bash
brew install mas

# 로그인 계정 확인 (미로그인 시 App Store 앱에서 먼저 로그인)
mas account

# 앱 검색 → 출력된 ID 확인 → 설치
mas search Amphetamine
# 937984704  Amphetamine  (5.x)   ← 맨 앞 숫자가 앱 ID
mas install 937984704   # 검색 결과에서 확인한 ID로 설치

# 메뉴바 CPU 모니터 RunCat 설치
mas install 1429033973   # RunCat

# 여러 앱을 ID로 한 번에 설치할 수도 있습니다
# mas install 497799835 425424353   # 예: Xcode, The Unarchiver
```

> 설치된 MAS 앱 목록은 `mas list`로, 업데이트는 `mas upgrade`로 관리합니다.
> 자주 쓰는 앱 ID를 모아두면 재설치 시 편리합니다.

**검증:**

```bash
mas version
# 출력 예시: 1.8.x
```

---

## 12. 언어 런타임 & 데이터베이스

### 12.1 언어 런타임

```bash
brew install go rust openjdk@17
```

> Apple Silicon에서 `openjdk@17`은 keg-only이므로, 시스템에 JDK로 등록하려면:
>
> ```bash
> sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk \
>   /Library/Java/JavaVirtualMachines/openjdk-17.jdk
> ```
>
> `javac` 등을 PATH에서 바로 쓰려면 다음도 추가합니다:
>
> ```bash
> echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
> ```

**검증:**

```bash
go version
rustc --version
java -version
```

### 12.2 데이터베이스 (선택)

```bash
brew install postgresql@16 mysql
```

> 서비스 실행/중지:
>
> ```bash
> brew services start postgresql@16
> brew services start mysql
> ```

**검증:**

```bash
brew services list
```

---

## 13. 클라우드 & 인프라 CLI

```bash
# 클라우드 CLI
brew install awscli azure-cli
brew install --cask gcloud-cli

# 네트워크·인프라
brew install tailscale cloudflared cloudflare-wrangler
```

**검증:**

```bash
aws --version
az version
gcloud --version
```

---

## 14. 폰트

개발용 고정폭 폰트와 한글 폰트입니다. (Nerd Font는 아이콘 글리프 포함)

```bash
brew install --cask \
  font-jetbrains-mono-nerd-font \
  font-d2coding-nerd-font \
  font-maple-mono-nf \
  font-cascadia-code \
  font-pretendard \
  font-noto-sans-kr
```

> 설치된 폰트는 `Font Book` 앱 또는 `시스템 설정 → 글꼴`에서 확인할 수 있습니다.

---

## 15. 최종 검증

모든 설치가 완료되었는지 확인하는 종합 검증 스크립트입니다.

```bash
echo "=========================================="
echo "  macOS 개발 환경 셋팅 검증"
echo "=========================================="

check() {
  if command -v "$1" &>/dev/null; then
    echo "✓ $1"
  else
    echo "✗ $1 — 누락됨"
  fi
}

echo ""
echo "[ 시스템 도구 ]"
check brew
check git
check wget
check jq
check rg
check fd
check tree
check htop
check tmux

echo ""
echo "[ Shell ]"
check zsh
check starship
echo "  SHELL: $SHELL"

echo ""
echo "[ 모던 CLI ]"
check bat
check eza
check btop
check fzf
check mise
check mas

echo ""
echo "[ Node.js ]"
check node
check npm
check fnm
check tsc
check pnpm
check yarn

echo ""
echo "[ Python ]"
check python
check pip
check pyenv
check uv
check ipython
check black
check ruff
check poetry

echo ""
echo "[ Docker ]"
check docker

echo ""
echo "[ VS Code ]"
check code

echo ""
echo "[ GitHub CLI ]"
check gh

echo ""
echo "[ 클라우드 / 런타임 ]"
check aws
check az
check gcloud
check go
check rustc

echo ""
echo "=========================================="
echo "  macOS 버전: $(sw_vers -productVersion)"
echo "  아키텍처: $(uname -m)"
echo "  디스크 여유: $(df -h / | tail -1 | awk '{print $4}')"
echo "=========================================="
```

---

## 유지보수

### 업데이트 명령어 모음

```bash
# Homebrew로 설치된 모든 패키지 업데이트
brew update && brew upgrade

# Node.js 글로벌 패키지 업데이트
npm update -g

# Python 패키지 업데이트
pip install --upgrade $(pip list --outdated | awk 'NR>2{print $1}')

# fnm 최신 LTS 확인
fnm ls-remote --lts

# Mac App Store 앱 업데이트
mas upgrade

# mise 관리 도구 업데이트
mise upgrade
```

### 백업 및 복원

```bash
# Homebrew 패키지 목록 백업
brew bundle dump --file=~/.Brewfile --force

# Homebrew 패키지 복원
brew bundle install --file=~/.Brewfile
```

---

> **문서 버전**: 1.2.0
> **마지막 업데이트**: 2026-05-30
> **대상 환경**: macOS 13+ (Apple Silicon / Intel) — macOS 26.5에서 검증
