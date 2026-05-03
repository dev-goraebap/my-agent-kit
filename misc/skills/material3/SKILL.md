---
name: material3
description: >-
  Material Design 3 (M3) 공식 가이드라인 지식 스킬. m3.material.io 전체(foundations·styles·components·develop)를
  117페이지 로컬 문서로 보관한다. 레이아웃(웹·모바일 차이, Window Size Classes, Canonical Layouts),
  디자인 토큰(Color Roles, Typography Scale, Shape, Elevation, Motion), 컴포넌트 38종 overview를 포함.
  기술스택 독립적 M3 전문가 에이전트를 위한 참조 지식 팩.
  Triggers — "M3 규격", "머테리얼 디자인", "Material3", "m3 컴포넌트", "m3 색상 토큰", "m3 레이아웃", "m3 타이포", "m3 모션".
license: Apache-2.0
metadata:
  author: dev-goraebap
  version: "1.0"
  source: https://m3.material.io
  fetched_at: "2026-04-29"
  pages: 117
---

# material3

Material Design 3 공식 가이드라인 지식 팩.  
실제 내용은 `references/` 하위 파일에 있으며, 질문 유형에 따라 필요한 파일만 로드한다 (Progressive Disclosure).

## 라우팅 테이블

질문 유형 → 먼저 읽어야 할 파일:

| 질문 유형 | 1순위 파일 | 보조 파일 |
|----------|-----------|---------|
| M3 개요·용어·전체 맥락 | `references/index.md` | `references/foundations/glossary.md` |
| 레이아웃 원리 (column, grid, spacing) | `references/foundations/layout/understanding-layout/overview.md` | `references/foundations/layout/understanding-layout/parts-of-layout.md` |
| Window Size Classes (compact/medium/expanded) | `references/foundations/layout/applying-layout/window-size-classes.md` | `references/foundations/layout/applying-layout/pane-layouts.md` |
| Canonical Layouts (list-detail, feed, pane) | `references/foundations/layout/canonical-layouts/overview.md` | `references/foundations/layout/canonical-layouts/list-detail.md` |
| 웹·모바일 반응형 차이, 밀도 | `references/foundations/layout/understanding-layout/density.md` | `references/foundations/layout/understanding-layout/hardware-considerations.md` |
| RTL·국제화 레이아웃 | `references/foundations/layout/understanding-layout/bidirectionality-rtl.md` | — |
| 디자인 토큰 개요·사용법 | `references/foundations/design-tokens/overview.md` | `references/foundations/design-tokens/how-to-use-tokens.md` |
| 색상 시스템·Color Roles | `references/styles/color/system/how-the-system-works.md` | `references/styles/color/roles.md` |
| Dynamic Color | `references/styles/color/dynamic/choosing-a-source.md` | `references/styles/color/dynamic/user-generated-source.md` |
| 색상 커스터마이징·테마 | `references/styles/color/static/custom-brand.md` | `references/styles/color/advanced/overview.md` |
| 색상 접근성·대비 | `references/foundations/designing/color-contrast.md` | — |
| 타이포그래피 스케일 | `references/styles/typography/type-scale-tokens.md` | `references/styles/typography/applying-type.md` |
| 폰트·서체 | `references/styles/typography/fonts.md` | `references/styles/typography/overview.md` |
| Shape (corner radius) | `references/styles/shape/overview-principles.md` | `references/styles/shape/corner-radius-scale.md` |
| Elevation | `references/styles/elevation/overview.md` | `references/styles/elevation/tokens.md` |
| Motion·애니메이션 | `references/styles/motion/overview/how-it-works.md` | `references/styles/motion/easing-and-duration/applying-easing-and-duration.md` |
| 화면 전환·Transitions | `references/styles/motion/transitions/transition-patterns.md` | `references/styles/motion/transitions/applying-transitions.md` |
| 인터랙션 상태 (hover, focus, pressed) | `references/foundations/interaction/states/overview.md` | `references/foundations/interaction/states/applying-states.md` |
| 제스처·입력 | `references/foundations/interaction/gestures.md` | `references/foundations/interaction/inputs.md` |
| 커스터마이징·브랜딩 | `references/foundations/customization.md` | `references/foundations/adaptive-design.md` |
| 특정 컴포넌트 (예: Button) | `references/components/<name>/overview.md` | — |
| 전체 컴포넌트 목록 | `references/components/index.md` | `references/components/all-buttons.md` |
| 아이콘 | `references/styles/icons/overview.md` | `references/styles/icons/applying-icons.md` |
| Android/Flutter/Web 구현 | `references/develop/<platform>.md` | — |

## 파일 구조

```
references/
├── index.md                          ← 전체 목차 (117페이지)
├── foundations/
│   ├── glossary.md                   ← M3 용어 사전
│   ├── adaptive-design.md
│   ├── customization.md
│   ├── design-tokens/                ← overview + how-to-use-tokens
│   ├── designing/                    ← color-contrast, elements, flow, overview, structure
│   ├── interaction/                  ← gestures, inputs, selection, states/*
│   ├── layout/
│   │   ├── understanding-layout/     ← overview, parts-of-layout, spacing, density,
│   │   │                               hardware-considerations, bidirectionality-rtl
│   │   ├── applying-layout/          ← window-size-classes ⭐, pane-layouts ⭐,
│   │   │                               compact, medium, expanded, large-extra-large
│   │   └── canonical-layouts/        ← overview ⭐, list-detail ⭐, feed, supporting-pane
│   ├── overview/                     ← principles, assistive-technology
│   ├── usability/
│   └── writing/                      ← best-practices, text-resizing, text-truncation
├── styles/
│   ├── color/
│   │   ├── system/                   ← overview, how-the-system-works ⭐
│   │   ├── dynamic/                  ← choosing-a-source, user-generated-source, content-based-source
│   │   ├── static/                   ← baseline, custom-brand
│   │   ├── advanced/                 ← overview, adjust-existing-colors, apply-colors, define-new-colors
│   │   ├── roles.md ⭐               ← Color Roles 전체 목록
│   │   ├── choosing-a-scheme.md
│   │   └── resources.md
│   ├── typography/                   ← overview, type-scale-tokens ⭐, fonts, applying-type, editorial-treatments
│   ├── shape/                        ← overview-principles, corner-radius-scale, shape-morph
│   ├── elevation/                    ← overview, tokens, applying-elevation
│   ├── motion/
│   │   ├── overview/                 ← how-it-works ⭐, specs
│   │   ├── easing-and-duration/      ← applying-easing-and-duration, tokens-specs
│   │   └── transitions/              ← transition-patterns ⭐, applying-transitions
│   └── icons/                        ← overview, applying-icons, designing-icons
├── components/
│   ├── index.md                      ← 38개 컴포넌트 목록
│   ├── all-buttons.md                ← 버튼 계열 비교
│   └── <component>/overview.md      ← 각 컴포넌트 (38종)
└── develop/
    ├── android/                      ← jetpack-compose, mdc-android
    ├── flutter.md
    └── web.md
```

⭐ = M3 전문가가 가장 먼저 읽어야 할 핵심 파일

## 수집 정보

- **원본:** https://m3.material.io
- **수집일:** 2026-04-29
- **방법:** Chrome SPA 자동 순회 (Angular 라우터 활용, 118페이지 → 1개 실패 제외 117개)
- **제외:** Blog(112p), 상세 Spec(픽셀 수치), Accessibility 세부, Content Design(UX 라이팅)
