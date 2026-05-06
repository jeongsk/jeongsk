# macOS 개발 환경 셋팅 가이드

> **AI 에이전트용 자동화 가이드** — 이 문서를 순서대로 읽고 각 단계를 실행하여 macOS 개발 환경을 구성합니다.
> 모든 명령어는 복사하여 터미널에 붙여넣을 수 있습니다. 각 단계마다 성공 여부를 검증하세요.

---

## 목차

1. [사전 확인](#1-사전-확인)
2. [Xcode Command Line Tools](#2-xcode-command-line-tools)
3. [Homebrew (패키지 매니저)](#3-homebrew-패키지-매니저)
4. [Git 설정](#4-git-설정)
5. [Shell 환경 (Zsh + Oh My Zsh)](#5-shell-환경-zsh--oh-my-zsh)
6. [Node.js 환경](#6-nodejs-환경)
7. [Python 환경](#7-python-환경)
8. [Docker 환경](#8-docker-환경)
9. [VS Code (에디터)](#9-vs-code-에디터)
10. [AI/LLM 개발 도구](#10-aillm-개발-도구)
11. [기타 유틸리티](#11-기타-유틸리티)
12. [최종 검증](#12-최종-검증)

---

## 1. 사전 확인

### 1.1 macOS 버전 확인

```bash
sw_vers
```

> **요구사항**: macOS 13 (Ventura) 이상
> **검증**: `ProductVersion` 필드가 `13.x` 이상이면 통과

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
# 출력 예시: Homebrew 4.x.x
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

## 5. Shell 환경 (Zsh + Oh My Zsh)

### 5.1 Oh My Zsh 설치

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
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

### 6.3 Node.js LTS 버전 설치

```bash
fnm install --lts
fnm default lts-latest
fnm use lts-latest
```

**검증:**

```bash
node --version
# 출력 예시: v20.x.x (LTS)
npm --version
# 출력 예시: 10.x.x
fnm --version
# 출력 예시: 1.x.x
```

### 6.4 글로벌 패키지 설치

```bash
npm install -g \
  typescript \
  ts-node \
  pnpm \
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

---

## 7. Python 환경

### 7.1 pyenv 설치 (Python 버전 관리)

```bash
brew install pyenv
```

### 7.2 pyenv 초기화 설정

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
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
pip install \
  ipython \
  black \
  ruff \
  mypy \
  pytest \
  pytest-cov \
  pre-commit \
  poetry \
  pipenv
```

**검증:**

```bash
for cmd in ipython black ruff mypy pytest poetry; do
  echo -n "$cmd: "
  command -v "$cmd" && echo "✓" || echo "✗ MISSING"
done
```

---

## 8. Docker 환경

### 8.1 Docker Desktop 설치

> Docker Desktop은 Homebrew로도 설치 가능하지만, 공식 다운로드를 권장합니다.

```bash
# Homebrew로 설치
brew install --cask docker
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
pip install mcp
npm install -g @modelcontextprotocol/sdk
```

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
for env in langchain langgraph chromadb tiktoken mcp n8n; do
  echo -n "$env: "
  python -c "import ${env//-/_}" 2>/dev/null && echo "✓" || npx -q "$env" --version 2>/dev/null && echo "✓" || echo "✗"
done
```

---

## 11. 기타 유틸리티

### 11.1 GitHub CLI

```bash
brew install gh
gh auth login
```

> 웹 기반 인증 또는 토큰 인증을 선택하고 안내를 따르세요.

### 11.2 추가 개발 도구

```bash
brew install --cask \
  iterm2 \
  rectangle \
  obsidian \
  postman \
  figma \
  slack \
  discord
```

**검증:**

```bash
for app in "iTerm" "Rectangle" "Obsidian" "Postman" "Figma" "Slack" "Discord"; do
  echo -n "$app: "
  ls "/Applications/$app.app" &>/dev/null && echo "✓" || echo "✗ NOT INSTALLED"
done
```

---

## 12. 최종 검증

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
echo "  SHELL: $SHELL"

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
```

### 백업 및 복원

```bash
# Homebrew 패키지 목록 백업
brew bundle dump --file=~/.Brewfile --force

# Homebrew 패키지 복원
brew bundle install --file=~/.Brewfile
```

---

> **문서 버전**: 1.0.0
> **마지막 업데이트**: 2026-05-06
> **대상 환경**: macOS 13+ (Apple Silicon / Intel)
