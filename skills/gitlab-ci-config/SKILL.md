---
name: gitlab-ci-config
description: Создание и редактирование .gitlab-ci.yml для мобильных проектов. Android сборка, iOS сборка, тесты, detekt, ktlint.
globs: [".gitlab-ci.yml", "**/.gitlab-ci.yml"]
autoContext: true
---

# GitLab CI Config Skill

Помогает создавать и редактировать `.gitlab-ci.yml` для мобильных проектов.

## Стандартные stages для мобильного проекта

```yaml
stages:
  - lint
  - test
  - build

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false -Dorg.gradle.parallel=true"
```

## Lint stage

```yaml
detekt:
  stage: lint
  script:
    - ./gradlew detekt
  artifacts:
    reports:
      codequality: "**/build/reports/detekt/*.xml"
  rules:
    - if: $CI_MERGE_REQUEST_IID

ktlint:
  stage: lint
  script:
    - ./gradlew ktlintCheck
  rules:
    - if: $CI_MERGE_REQUEST_IID
```

## Test stage

```yaml
unit-tests:
  stage: test
  script:
    - ./gradlew :shared:allTests
  artifacts:
    reports:
      junit: "**/build/test-results/**/*.xml"
  rules:
    - if: $CI_MERGE_REQUEST_IID

android-tests:
  stage: test
  script:
    - ./gradlew :app:testDebugUnitTest
  artifacts:
    reports:
      junit: "**/build/test-results/**/*.xml"
  rules:
    - if: $CI_MERGE_REQUEST_IID
```

## Build stage

```yaml
android-debug:
  stage: build
  script:
    - ./gradlew :app:assembleDebug
  artifacts:
    paths:
      - app/build/outputs/apk/debug/
  rules:
    - if: $CI_MERGE_REQUEST_IID

android-release:
  stage: build
  script:
    - ./gradlew :app:assembleRelease
  artifacts:
    paths:
      - app/build/outputs/apk/release/
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

## Правила

- Всегда проверяй существующий `.gitlab-ci.yml` перед изменениями
- Не ломай существующие jobs
- Используй `rules` вместо `only/except`
- Кэшируй Gradle: `~/.gradle/caches` и `~/.gradle/wrapper`
- Для KMP проектов: тестируй shared модуль отдельно
