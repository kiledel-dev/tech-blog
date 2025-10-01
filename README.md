# Kiledel Tech Blog

Kiledel의 기술 블로그입니다. Gatsby를 기반으로 구축된 정적 사이트 생성 블로그입니다.

## 🚀 Quick Start

### Prerequisites
- Node.js (v18 이상)
- Yarn 또는 npm

### Installation

```bash
# 저장소 클론
git clone https://github.com/kiledel-dev/tech-blog.git
cd tech-blog

# 의존성 설치
yarn install
# 또는
npm install
```

### Development

```bash
# 개발 서버 실행
yarn develop
# 또는
npm run develop
```

브라우저에서 `http://localhost:8000` 접속

### Build & Deploy

```bash
# 프로덕션 빌드
yarn build

# GitHub Pages에 배포
yarn deploy
```

## 📁 프로젝트 구조

```
├── contents/           # 블로그 포스트 (Markdown)
│   ├── tech/          # 기술 관련 포스트
│   └── diary/         # 일기/일상 포스트
├── src/
│   ├── components/    # React 컴포넌트
│   ├── pages/         # 페이지 컴포넌트
│   ├── templates/     # 블로그 템플릿
│   ├── layout/        # 레이아웃 컴포넌트
│   └── styles/        # 스타일 파일
├── static/            # 정적 파일 (이미지, 파비콘 등)
├── gatsby-config.js   # Gatsby 설정
└── gatsby-node.js     # Gatsby Node API
```

## ✨ 주요 기능

- 📝 마크다운 기반 블로그 포스트
- 🎨 반응형 디자인
- 🔍 SEO 최적화
- 📊 Google Analytics 연동
- 💻 코드 하이라이팅 (PrismJS)
- 📋 코드 복사 기능
- 🏷️ 태그 및 카테고리 시스템
- 🖼️ 이미지 최적화 (WebP 지원)

## 🛠️ 기술 스택

- **Framework**: Gatsby v5
- **Language**: TypeScript
- **Styling**: Emotion (CSS-in-JS)
- **Code Highlighting**: PrismJS
- **Font**: Inter, Pretendard, Fira Code
- **Deployment**: GitHub Pages

## 📝 블로그 포스트 작성

새로운 포스트는 `contents/tech/` 또는 `contents/diary/` 디렉토리에 마크다운 파일로 작성합니다.

### 포스트 Front Matter 예시

```markdown
---
title: "포스트 제목"
date: "2025-01-01"
category: "tech" # 또는 "diary"
tags: ["javascript", "react", "gatsby"]
description: "포스트 설명"
---

# 포스트 내용

여기에 마크다운으로 내용을 작성합니다.
```

## 📞 Contact

- Email: Kiledel.dev@gmail.com
- GitHub: [@kiledel-dev](https://github.com/kiledel-dev)
- Blog: [https://kiledel-dev.github.io/tech-blog](https://kiledel-dev.github.io/tech-blog)

## 📄 License

MIT License