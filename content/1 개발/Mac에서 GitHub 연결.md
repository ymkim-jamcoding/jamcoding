# Mac에서 GitHub 처음 연결하기 (SSH 방식)

## 1. Git 설치 확인

```bash
git --version
```

없으면 Xcode Command Line Tools 설치 안내가 뜹니다. 그대로 설치하면 됩니다.

## 2. 사용자 정보 설정

```bash
git config --global user.name "본인 이름"
git config --global user.email "github가입이메일@example.com"
```

## 3. SSH 키 생성

```bash
ssh-keygen -t ed25519 -C "github가입이메일@example.com"
```

경로·암호는 Enter로 넘겨도 됩니다. → `~/.ssh/id_ed25519`(비밀키), `~/.ssh/id_ed25519.pub`(공개키) 생성

## 4. ssh-agent에 등록

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

## 5. 공개키를 GitHub에 등록

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

GitHub → **Settings → SSH and GPG keys → New SSH key** → 붙여넣기 → Add

## 6. 연결 테스트

```bash
ssh -T git@github.com
```

`Hi 아이디! You've successfully authenticated...` 가 나오면 성공입니다.

## 7. 저장소 연결 후 push

```bash
cd 프로젝트폴더
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:아이디/저장소이름.git
git push -u origin main
```

---

**참고:** HTTPS 방식(`https://github.com/...`)을 쓰면 비밀번호 대신 **Personal Access Token**을 발급받아 입력해야 합니다. 한 번 설정해두고 오래 쓸 거라면 위의 SSH 방식이 더 편합니다.

# Mac에서 GitHub 연결하기 (HTTPS + Personal Access Token)

## 1. Git 설치 및 사용자 정보 설정

```bash
git --version

git config --global user.name "본인 이름"
git config --global user.email "github가입이메일@example.com"
```

## 2. Personal Access Token 발급

GitHub 웹사이트에서:

**Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)**

설정 항목:

|항목|권장 값|
|---|---|
|Note|용도 메모 (예: `macbook-push`)|
|Expiration|90 days (또는 필요에 맞게)|
|Scopes|**`repo`** 체크 (비공개 저장소 포함 push 권한)|

생성 후 나오는 `ghp_...` 문자열은 **그 화면을 벗어나면 다시 볼 수 없습니다.** 반드시 복사해 두세요.

> Fine-grained tokens을 쓰면 저장소별로 권한을 세밀하게 줄 수 있지만, 처음이라면 classic + `repo`가 간단합니다.

## 3. Keychain에 저장되도록 설정

```bash
git config --global credential.helper osxkeychain
```

Mac은 보통 기본으로 켜져 있지만, 명시해두면 토큰을 한 번만 입력하면 됩니다.

## 4. 저장소 연결 후 push

```bash
cd 프로젝트폴더
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/아이디/저장소이름.git
git push -u origin main
```

첫 push 때 인증 창이 뜹니다:

- **Username**: GitHub 아이디
- **Password**: **발급받은 토큰 붙여넣기** (계정 비밀번호 아님)

한 번 성공하면 Keychain에 저장되어 다음부터는 묻지 않습니다.

## 5. 토큰이 만료되거나 바꿀 때

```bash
# 저장된 인증 정보 삭제
git credential-osxkeychain erase
host=github.com
protocol=https
# 빈 줄 하나 입력 후 Enter
```

또는 **키체인 접근(Keychain Access)** 앱에서 `github.com` 검색 → 삭제 후 다시 push하면 재입력 창이 뜹니다.

---

### SSH vs PAT 비교

||SSH|PAT (HTTPS)|
|---|---|---|
|초기 설정|키 생성 + 등록|토큰 발급만|
|만료|없음|있음 (재발급 필요)|
|방화벽 환경|막히는 경우 있음|대부분 통과|
|여러 기기|기기마다 키 등록|토큰 재사용 가능(비권장)|

학원이나 회사처럼 SSH 포트가 막힌 네트워크에서는 PAT 방식이 안전한 선택입니다.