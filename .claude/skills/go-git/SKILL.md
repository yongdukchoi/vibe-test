---
name: go-git
description: 원격 저장소 업데이트 → 새/변경 파일 add → 날짜가 붙은 메시지로 commit → push 까지 한 번에 실행한다. 사용자가 /go-git 을 입력하거나 "go-git", "깃 올려줘", "커밋하고 푸시해줘" 등을 요청할 때 사용.
argument-hint: "[커밋 메시지(선택)]"
allowed-tools: Bash(git *), PowerShell(git *)
---

# go-git

현재 저장소의 변경 사항을 원격 저장소까지 한 번에 반영한다.

## 커밋 메시지 규칙

- 날짜 형식: `YYYY-MM-DD HH:mm` (커밋 실행 시점의 로컬 시간)
- 인자(`$ARGUMENTS`)가 **없으면**: `commit YYYY-MM-DD HH:mm`
- 인자가 **있으면**: `<입력한 텍스트> YYYY-MM-DD HH:mm`

예)
- `/go-git` → `commit 2026-09-28 21:10`
- `/go-git 로그인 화면 추가` → `로그인 화면 추가 2026-09-28 21:10`

전달된 인자: `$ARGUMENTS`

## 실행 순서

아래 단계를 순서대로 실행하고, 한 단계라도 실패하면 즉시 멈추고 오류 내용을 사용자에게 알린다.

1. **원격 업데이트**
   ```
   git remote update
   ```
   이후 `git status -sb` 로 원격 대비 상태를 확인한다. 로컬 브랜치가 원격보다 뒤처져 있으면(behind) `git pull --rebase` 로 먼저 동기화한다. 충돌이 나면 멈추고 사용자에게 알린다.

2. **파일 추가** (새 파일, 수정, 삭제 모두 포함)
   ```
   git add -A
   ```
   `git status --short` 로 스테이징된 내용이 있는지 확인한다. 커밋할 변경 사항이 없으면 commit 은 건너뛰고, 아직 push 되지 않은 커밋이 있을 때만 4단계(push)를 진행한다. 둘 다 없으면 "변경 사항 없음"을 알리고 종료한다.

3. **커밋**
   - 날짜는 `date "+%Y-%m-%d %H:%M"` (Bash) 또는 `Get-Date -Format "yyyy-MM-dd HH:mm"` (PowerShell) 로 구한다.
   - 위 규칙대로 메시지를 만들어 커밋한다. 메시지는 규칙 그대로만 사용하고 다른 문구(Co-Authored-By 등)는 덧붙이지 않는다.
   ```
   git commit -m "<메시지>"
   ```

4. **푸시**
   ```
   git push
   ```
   upstream 이 설정되지 않은 브랜치면 `git push -u origin <현재 브랜치>` 로 푸시한다.

## 완료 보고

끝나면 다음을 짧게 알려준다.
- 사용한 커밋 메시지
- 커밋된 파일 개수 (또는 목록)
- push 한 브랜치와 결과
