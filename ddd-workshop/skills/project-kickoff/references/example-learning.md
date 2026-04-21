---
skill: project-kickoff
version: 0.1.0
context: 학습
confidence: high
next_suggested: domain-research
created_at: 2026-04-21
---

# 프로젝트 정체성: 간단한 블로그 엔진

## 개요
- **한 줄 정의**: Markdown으로 글을 쓰고, 정적 HTML로 렌더링하는 1인용 블로그 엔진.
- **맥락**: 학습/취미
- **해결하려는 문제 (Why)**: SSG(Static Site Generator) 내부 동작을 직접 구현하며 이해한다.
- **핵심 가치**: 학습 + 실제로 내 블로그에 쓸 수 있는 결과물.

## 사용자·액터
- 주요 액터: 본인 (글쓴이), 블로그 방문자

## 범위
- 반드시 포함:
  - Markdown 파싱 (CommonMark 수준)
  - 파일 → HTML 변환 커맨드
  - 태그·카테고리 분류
- 명시적 제외 (영영 안 함):
  - 회원가입·로그인 (본인만 씀)
  - 댓글 시스템 (외부 서비스 임베드도 안 함)
- 나중으로 미룸:
  - RSS 피드 (v0.2)
  - 검색 인덱싱 (v0.3)

## 제약 조건
- 기술: Rust (학습 목적). 외부 SSG 라이브러리 사용 금지 (파서는 직접 구현).
- 일정·인력: 본인 주말 프로젝트. 완성 데드라인 없음.

## 학습 목표
- CommonMark 파서 구현
- 정적 파일 빌드 파이프라인 이해
- Front Matter YAML 처리

## 미검증 가정 / 리스크
- ⚠️ 파서 직접 구현 난이도가 어느 정도인지 아직 감 없음 → 1주 타임박스 후 외부 라이브러리 허용 여부 재검토

## 다음 단계
학습 맥락이라 `domain-research`는 가볍게 건너뛰고 바로 `requirement-clarifier`로 진입해도 무방하다. 단, Markdown 명세(CommonMark vs GFM)는 확인할 가치가 있다.
