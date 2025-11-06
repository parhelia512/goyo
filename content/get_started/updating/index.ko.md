+++
title = "Goyo 테마 업데이트"
description = "Goyo 테마를 최신 버전으로 업데이트하는 방법을 알아보세요."
weight = 2
sort_by = "weight"

[extra]
+++

Goyo 테마를 최신 상태로 유지하면 최신 기능, 개선 사항 및 버그 수정을 사용할 수 있습니다. 이 가이드는 테마를 업데이트하는 다양한 방법을 다룹니다.

## 수동 업데이트 방법

### clone으로 설치한 경우

저장소를 직접 clone하여 Goyo를 설치한 경우, 다음 명령으로 업데이트할 수 있습니다:

```bash
cd themes/goyo
git pull origin main
```

이 명령은 main 브랜치에서 최신 변경 사항을 가져와 병합합니다.

### submodule로 추가한 경우

Goyo를 git submodule로 설치한 경우, 다음 명령으로 업데이트할 수 있습니다:

```bash
git submodule update --remote themes/goyo
```

또는 모든 submodule을 포괄적으로 업데이트하려면:

```bash
git submodule sync
git submodule update --remote
```

submodule 업데이트 후, 변경 사항을 저장소에 커밋합니다:

```bash
git add themes/goyo
git commit -m "Update Goyo theme to latest version"
git push
```

## GitHub Actions를 통한 자동 업데이트

GitHub에 호스팅된 프로젝트의 경우, GitHub Actions를 사용하여 테마 업데이트를 자동화할 수 있습니다. 이를 통해 새로운 버전의 Goyo가 출시될 때마다 주기적으로 Pull Request가 생성됩니다.

### 1단계: 워크플로우 파일 생성

문서 저장소에 `.github/workflows/update-goyo-theme.yml` 파일을 생성합니다:

```yaml
name: Update Goyo Theme

on:
  schedule:
    # 매주 월요일 오전 9시(UTC)에 실행
    - cron: '0 9 * * 1'
  workflow_dispatch: # 수동 실행 허용

jobs:
  update-theme:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: true
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Update Goyo submodule
        id: update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          
          # 현재 커밋 해시 가져오기
          cd themes/goyo
          OLD_COMMIT=$(git rev-parse HEAD)
          cd ../..
          
          # submodule을 최신 버전으로 업데이트
          git submodule update --remote themes/goyo
          
          # 새 커밋 해시 가져오기
          cd themes/goyo
          NEW_COMMIT=$(git rev-parse HEAD)
          cd ../..
          
          # 변경 사항 확인
          if [ "$OLD_COMMIT" != "$NEW_COMMIT" ]; then
            echo "updated=true" >> $GITHUB_OUTPUT
            echo "old_commit=$OLD_COMMIT" >> $GITHUB_OUTPUT
            echo "new_commit=$NEW_COMMIT" >> $GITHUB_OUTPUT
          else
            echo "updated=false" >> $GITHUB_OUTPUT
          fi
      
      - name: Create Pull Request
        if: steps.update.outputs.updated == 'true'
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "Update Goyo theme to latest version"
          title: "Goyo 테마 업데이트"
          body: |
            이 PR은 Goyo 테마를 최신 버전으로 업데이트합니다.
            
            **변경 사항:** ${{ steps.update.outputs.old_commit }} → ${{ steps.update.outputs.new_commit }}
            
            새로운 기능에 대한 자세한 내용은 [Goyo 변경 로그](https://github.com/hahwul/goyo/releases)를 참조하세요.
            
            ---
            *이 PR은 Update Goyo Theme 워크플로우에 의해 자동으로 생성되었습니다.*
          branch: update-goyo-theme
          delete-branch: true
          labels: dependencies, documentation
```

### 2단계: 스케줄 커스터마이징

워크플로우는 기본적으로 매주 월요일 오전 9시(UTC)에 실행되도록 설정되어 있습니다. `cron` 표현식을 수정하여 스케줄을 커스터마이징할 수 있습니다:

- **매일**: `'0 9 * * *'` - 매일 오전 9시(UTC)에 실행
- **매주 (월요일)**: `'0 9 * * 1'` - 매주 월요일 오전 9시(UTC)에 실행
- **매월**: `'0 9 1 * *'` - 매월 1일 오전 9시(UTC)에 실행

저장소의 Actions 탭에서 언제든지 워크플로우를 수동으로 실행할 수도 있습니다.

### 3단계: Actions 활성화

1. GitHub에서 저장소로 이동합니다
2. **Actions** 탭으로 이동합니다
3. Actions가 비활성화되어 있다면 "I understand my workflows, go ahead and enable them"을 클릭합니다

### 4단계: 업데이트 모니터링

설정이 완료되면 워크플로우는:
- 예약된 시간에 자동으로 Goyo 테마 업데이트를 확인합니다
- 업데이트가 있으면 Pull Request를 생성합니다
- PR 설명에 커밋 해시 정보를 포함합니다

Pull Request에서 변경 사항을 검토하고 준비가 되면 병합할 수 있습니다.

## 수동으로 업데이트 확인하기

업데이트를 가져오지 않고 사용 가능한 업데이트가 있는지 확인하려면:

```bash
# clone한 저장소의 경우
cd themes/goyo
git fetch origin
git log HEAD..origin/main --oneline

# submodule의 경우
git submodule update --remote --dry-run themes/goyo
```

## 문제 해결

### 병합 충돌

테마를 로컬에서 수정했고 업데이트 중 충돌이 발생하는 경우:

1. **clone한 저장소의 경우**: 충돌을 수동으로 해결하거나 최신 버전으로 재설정합니다:
   ```bash
   cd themes/goyo
   git stash  # 로컬 변경 사항 저장
   git pull origin main
   git stash pop  # 로컬 변경 사항 재적용
   ```

2. **submodule의 경우**: 직접 수정 대신 fork 사용을 고려하세요

### 권한 문제

GitHub Actions 워크플로우가 권한 오류로 실패하는 경우:
- 워크플로우에 `contents: write` 및 `pull-requests: write` 권한이 있는지 확인하세요
- 저장소 설정에서 Actions가 Pull Request를 생성할 수 있도록 허용되어 있는지 확인하세요 (Settings → Actions → General → Workflow permissions)

## 모범 사례

- **프로덕션에 배포하기 전에 로컬에서 업데이트 테스트**하기
- **변경 로그를 검토**하여 무엇이 변경되었는지 이해하기
- **더 쉬운 업데이트를 위해 테마를 submodule로 유지**하기
- **테마 파일을 직접 수정하지 않기** - 대신 구성 옵션이나 사용자 정의 CSS 사용하기
- **워크플로우를 모니터링**하여 업데이트 실패를 조기에 발견하기
