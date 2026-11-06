---
layout: page
title: 블로그 글 작성 및 게시 가이드
permalink: /blog-publishing-guide/
---

# 블로그 글 작성 및 게시 가이드

이 문서는 `robotics-engineering-portfolio` 저장소에 새 글을 직접 작성하고 공개 블로그에 게시하는 실전 매뉴얼입니다.

- 저장소: https://github.com/itschoz/robotics-engineering-portfolio
- 공개 블로그: https://itschoz.github.io/robotics-engineering-portfolio/
- 기본 브랜치: `main`
- 게시 방식: Jekyll + GitHub Actions + GitHub Pages

## 1. 게시 흐름 이해하기

글을 게시하는 전체 흐름은 다음과 같습니다.

1. `_posts/` 폴더에 Markdown 파일(`.md`)을 만듭니다.
2. 파일 맨 위에 Jekyll Front Matter를 작성합니다.
3. 본문을 Markdown으로 작성하고 필요한 이미지를 `assets/images/`에 추가합니다.
4. 변경 사항을 `main` 브랜치에 Commit/Push 합니다.
5. GitHub Actions가 사이트를 빌드하고 GitHub Pages에 배포합니다.
6. 공개 블로그에서 글과 이미지가 정상적으로 보이는지 확인합니다.

**중요:** GitHub 저장소에 파일이 보인다는 것과 공개 블로그에 배포되었다는 것은 다릅니다. Actions 실행이 성공했는지 확인해야 합니다.

## 2. 새 글 파일 만들기

새 글은 반드시 저장소의 `_posts/` 폴더 안에 만듭니다.

파일명 형식은 다음과 같습니다.

```text
YYYY-MM-DD-english-or-kebab-case-title.md
```

예시:

```text
_posts/2026-11-07-ros2-topic-service-action.md
```

규칙:
- 날짜는 `YYYY-MM-DD` 형식으로 씁니다.
- 제목 부분은 영문 소문자와 하이픈(`-`)을 권장합니다.
- 파일명에 공백이나 한글을 넣지 않는 편이 URL 관리에 유리합니다.
- 같은 날짜에 글을 여러 개 쓰면 제목 부분을 다르게 지정합니다.

## 3. Front Matter 작성하기

Markdown 파일의 가장 위에는 아래 형식의 메타데이터가 필요합니다. 첫 줄과 마지막 줄의 `---`도 반드시 포함해야 합니다.

```yaml
---
title: "ROS 2 Topic, Service, Action 이해하기"
date: 2026-11-07 20:00:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Topic, Service, Action]
---
```

각 항목의 의미:
- `title`: 블로그에 표시할 글 제목
- `date`: 게시 날짜와 시간. 한국 시간은 `+0900`을 사용합니다.
- `categories`: URL 분류에 사용됩니다. 영문 소문자 권장.
- `tags`: 검색과 주제 분류를 위한 키워드 목록

이 저장소는 Jekyll permalink를 `/:categories/:year/:month/:day/:title/` 형식으로 설정했습니다. 따라서 카테고리를 바꾸면 글의 공개 URL도 달라질 수 있습니다.

## 4. 본문 Markdown 작성법

### 제목과 문단

```markdown
# 가장 큰 제목
## 주요 섹션
### 세부 항목

일반 문단은 빈 줄로 구분합니다.
```

글의 제목은 Front Matter의 `title`을 사용하고, 본문은 보통 `##`부터 시작하면 깔끔합니다.

### 강조와 목록

```markdown
**중요한 개념**
*강조할 표현*

- 첫 번째 항목
- 두 번째 항목

1. 첫 번째 단계
2. 두 번째 단계
```

### 코드와 터미널 명령어

코드 블록에는 언어를 표시합니다.

```markdown
\`\`\`bash
source /opt/ros/humble/setup.bash
ros2 topic list
\`\`\`
```

예를 들어 터미널 명령은 `bash`, Python은 `python`, C++는 `cpp`, YAML은 `yaml`을 사용합니다. 실제 명령어와 예상 출력, 직접 확인한 결과를 구분해서 기록하세요.

### 링크

저장소 안의 문서 링크는 상대 경로를 사용할 수 있습니다.

```markdown
[6주 학습 로드맵](../../curriculum/6-week-roadmap.md)
[시스템 아키텍처](../../architecture/system-architecture.md)
```

사이트 페이지로 연결할 때는 Jekyll의 baseurl을 고려하세요.

```markdown
[About]({{ site.baseurl }}/about/)
```

## 5. 이미지와 다이어그램 추가하기

1. 저장소의 `assets/images/` 또는 `assets/diagrams/` 폴더에 이미지 파일을 업로드합니다.
2. 파일명은 영문 소문자와 하이픈으로 정리합니다. 예: `ros2-node-graph.png`
3. 본문에서 다음처럼 불러옵니다.

```markdown
![ROS 2 node graph]({{ site.baseurl }}/assets/images/ros2-node-graph.png)
```

SVG 다이어그램도 같은 방식으로 사용할 수 있습니다.

```markdown
![System architecture]({{ site.baseurl }}/assets/diagrams/robot-system-architecture.svg)
```

이미지 경로와 대소문자가 실제 저장소의 파일명과 정확히 일치해야 합니다. 이미지가 표시되지 않으면 파일이 존재하는지, 경로가 맞는지, baseurl을 빠뜨리지 않았는지 확인합니다.

## 6. GitHub 웹사이트에서 직접 게시하기

컴퓨터에 Git을 설정하지 않아도 브라우저만으로 글을 게시할 수 있습니다.

1. 저장소의 [_posts 폴더](https://github.com/itschoz/robotics-engineering-portfolio/tree/main/_posts)를 엽니다.
2. **Add file → Create new file**을 선택합니다.
3. 파일 이름 칸에 `_posts/2026-11-07-example-post.md`처럼 전체 경로를 입력합니다.
4. 편집 영역에 Front Matter와 Markdown 본문을 작성합니다.
5. 이미지가 필요하면 먼저 `assets/images/` 폴더에서 **Add file → Upload files**로 이미지를 올립니다.
6. 화면 아래 **Commit changes**에서 변경 내용을 간단히 적습니다.
   - 예: `Add ROS 2 topic service action post`
7. `Commit directly to the main branch`를 선택하고 Commit합니다.

Commit이 완료되면 GitHub Actions가 자동 실행됩니다. 저장소의 **Actions** 탭에서 가장 최근 `Build and deploy Jekyll site` 실행을 확인하세요.

## 7. 내 컴퓨터에서 Git으로 게시하기

로컬에서 편집하는 방법입니다. Git과 저장소 clone 설정이 되어 있어야 합니다.

처음 한 번만 저장소를 내려받습니다.

```bash
git clone git@github.com:itschoz/robotics-engineering-portfolio.git
cd robotics-engineering-portfolio
```

새 글 파일을 만들고 편집합니다.

```bash
touch _posts/2026-11-07-example-post.md
```

VS Code 등 편집기로 내용을 작성한 다음 변경 사항을 확인합니다.

```bash
git status
git diff
```

문제가 없으면 Commit하고 Push합니다.

```bash
git add _posts/2026-11-07-example-post.md
git commit -m "Add example robotics learning post"
git push origin main
```

이미지와 문서를 여러 개 수정했다면 `git add` 뒤에 파일 경로를 모두 지정하거나, 변경 사항을 확인한 뒤 `git add .`를 사용할 수 있습니다. 무조건 `git add .`부터 실행하기보다는 `git status`로 불필요한 파일이나 비밀정보가 포함되지 않았는지 먼저 확인하세요.

## 8. 게시 확인

1. 저장소의 [Actions 탭](https://github.com/itschoz/robotics-engineering-portfolio/actions)을 엽니다.
2. 가장 최근 배포 작업을 선택합니다.
3. 모든 단계가 초록색 체크 표시로 끝났는지 확인합니다.
4. 공개 블로그를 열고 새 글이 목록에 표시되는지 확인합니다.
5. 글을 열어 코드 블록, 링크, 이미지가 정상적으로 보이는지 확인합니다.

공개 사이트: https://itschoz.github.io/robotics-engineering-portfolio/

배포 직후에는 반영까지 잠시 시간이 걸릴 수 있습니다. 새로고침 후에도 보이지 않으면 Actions 실행 상태부터 확인하세요.

## 9. 자주 발생하는 문제

### 글이 블로그에 나타나지 않음
- 파일이 `_posts/` 안에 있는지 확인합니다.
- 파일명이 `YYYY-MM-DD-title.md` 형식인지 확인합니다.
- Front Matter의 `---`가 열리고 닫혔는지 확인합니다.
- 날짜가 미래로 설정되어 있지 않은지 확인합니다.
- GitHub Actions가 성공했는지 확인합니다.

### 이미지가 깨져 보임
- 이미지가 실제 저장소에 업로드되었는지 확인합니다.
- 경로와 파일명의 대소문자를 확인합니다.
- 사이트 경로에 `{{ site.baseurl }}`을 포함했는지 확인합니다.

### GitHub Actions에서 Pages site를 찾을 수 없다는 오류
오류에 `Get Pages site failed` 또는 Pages site가 활성화되지 않았다는 내용이 나오면 저장소의 [Settings → Pages](https://github.com/itschoz/robotics-engineering-portfolio/settings/pages)로 이동합니다. **Build and deployment → Source**가 **GitHub Actions**로 설정되어 있는지 확인한 뒤 Actions에서 실패한 작업을 다시 실행합니다.

### Markdown이나 Jekyll 빌드 오류
- YAML Front Matter의 콜론, 따옴표, 들여쓰기를 확인합니다.
- 코드 블록의 백틱 개수가 맞는지 확인합니다.
- 링크나 이미지 경로에 오타가 없는지 확인합니다.
- Actions 로그에서 처음 발생한 실제 오류 메시지를 확인합니다. 마지막의 요약 오류만 보는 것보다 처음 실패한 단계가 원인을 찾는 데 도움이 됩니다.

## 10. 권장 블로그 글 구성 템플릿

아래 템플릿을 복사해 새 글의 시작점으로 사용하세요.

```markdown
---
title: "글 제목"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [ros2, fundamentals]
tags: [ROS2, 학습주제]
---

## 1. 학습 목표

이번 글에서 확인하려는 내용을 작성합니다.

## 2. 핵심 개념

개념을 본인의 말로 설명하고 필요한 경우 다이어그램을 추가합니다.

## 3. 실습 환경

- OS:
- ROS 2 distribution:
- Hardware / Simulator:

## 4. 실습 과정

실행한 명령어와 코드를 기록합니다.

## 5. 결과 및 검증

실제 실행 결과, 로그, 스크린샷을 첨부합니다.
실행하지 않은 내용은 검증 완료라고 쓰지 않습니다.

## 6. 문제와 해결

발생한 문제, 원인 분석, 해결 방법을 기록합니다.

## 7. 배운 점과 다음 단계

이번 실습에서 얻은 결론과 다음에 진행할 작업을 작성합니다.
```

## 11. 포트폴리오 품질 체크리스트

게시하기 전에 아래 항목을 확인하세요.

- [ ] 파일명과 날짜 형식이 올바른가?
- [ ] Front Matter가 정상적으로 작성되었는가?
- [ ] 개념 설명을 본인의 언어로 작성했는가?
- [ ] 실행한 것과 계획 중인 것을 구분했는가?
- [ ] 명령어와 코드 블록을 복사해 재현할 수 있는가?
- [ ] 결과를 로그, 이미지, 수치 등으로 검증했는가?
- [ ] 이미지와 내부 문서 링크가 실제로 존재하는가?
- [ ] 개인 정보, 토큰, 비밀번호, 회사 기밀이 포함되지 않았는가?
- [ ] GitHub Actions 배포가 성공했는가?
- [ ] 공개 사이트에서 최종 결과를 확인했는가?

---

**핵심 원칙:** 글을 저장소에 Commit하는 것으로 작업이 끝나는 것이 아닙니다. 재현 가능한 기록을 남기고, 자동 배포 성공과 실제 공개 페이지까지 확인해야 게시가 완료된 것입니다.
