## Github Discussion에서 제공하는 댓글 시스템인 Giscus를 Docusaurus에서 생성된 정적 웹 사이트에 추가하기

1. 댓글이 저장될 토론 섹션을 활성화한 Github 저장소 생성
2. 생성된 저장소의 메인 페이지에서 설정으로 이동
3. Features 섹션에서 Set up discussion
4. Start a new discussion 템플릿 선택 후 Start discussion 클릭

---

### Giscus 활성화

1. [Github 계정에서 giscus를 구성](https://github.com/apps/giscus)
2. Repository access 섹션에서 giscus에서 액세스할 저장소(이전 단계에서 댓글을 저장하기 위해 생성한 저장소) 추가하고 저장 클릭

### API key 발급

- [GraphQL API Explorer에 깃헙 계정으로 로그인](https://docs.github.com/en/graphql/overview/explorer)
- 아래 쿼리를 사용하여 생성된 저장소의 ID, 세부 정보(ID 및 이름)가 있는 토론 범주 가져오고, owner과 name란에 각각 Github 계정 이름과 생성한 저장소 명을 기입합니다.

쿼리 실행

```bash
query {
  repository(owner: "ghdwlsgur", name:"comment"){
    id
    discussionCategories(first:1) {
      edges {
        node {
          id
          name
        }
      }
    }
  }
}
```

쿼리 결과

```bash
{
  "data": {
    "repository": {
      "id": "abc",
      "discussionCategories": {
        "edges": [
          {
            "node": {
              "id": "abc",
              "name": "Announcements"
            }
          }
        ]
      }
    }
  }
}
```

---

### Giscus 컴포넌트 생성하기

#### @giscus/react 패키지 설치

```bash
npm i @giscus/react # npm
yarn add @giscus/react # yarn
```

#### /src/components/GiscusComponent 아래 생성된 파일 수정

- **repoId**에 GraphQL API Explorer 쿼리 실행 결과로 얻은 `data["repository"]["id"]` 입력
- **categoryId**에 GraphQL API Explorer 쿼리 실행 결과로 얻은 `data["repository]["discussionCategories"]["edges"][0]["node"]["id
]` 입력

```react
import React from 'react';
import Giscus from "@giscus/react";
import { useColorMode } from '@docusaurus/theme-common';
import useDocusaurusContext from "@docusaurus/useDocusaurusContext";
import styled from "styled-components";

const S = {
  Wrapper: styled.section`
    padding-top: 0.5rem;
  `,
}

export default function GiscusComponent() {
  const { colorMode } = useColorMode();
  const { siteConfig } = useDocusaurusContext();

  return (
    <S.Wrapper>
      <Giscus
        repo="ghdwlsgur/comment"
        repoId={siteConfig.customFields.GISCUS_REPO_ID}
        category="General"
        categoryId={siteConfig.customFields.GISCUS_CATEGORY_ID}
        mapping="https://ghdwlsgur.github.io/comment"
        term="Giscus Comment"
        strict="0"
        reactionsEnabled="1"
        emitMetadata="1"
        inputPosition="top"
        theme={colorMode === 'dark' ? 'dark' : 'light'}
        lang="ko"
        loading="lazy"
        crossorigin="anonymous"
        async
      />
    </S.Wrapper>
  )
}
```

---

### BlogPostItem 컴포넌트 생성

```bash
npm run swizzle [theme name] [component name] -- --wrap

# Example:
npm run swizzle @docusaurus/theme-classic BlogPostItem -- --wrap
yarn run swizzle @docusaurus/theme-classic BlogPostItem -- --wrap
```

#### src/theme/BlogPostItem/index.js 파일 생성 및 수정

```react
import React from "react";
import { useBlogPost } from "@docusaurus/theme-common/internal";
import BlogPostItem from "@theme-original/BlogPostItem";
import GiscusComponent from "../../components/Giscus/index.tsx";
import useIsBrowser from "@docusaurus/useIsBrowser";

export default function BlogPostItemWrapper(props) {
  const { metadata, isBlogPostPage } = useBlogPost();

  const { frontMatter, slug, title } = metadata;
  const { enableComments } = frontMatter;

  return (
    <>
      <BlogPostItem {...props} />
      {enableComments && isBlogPostPage && <GiscusComponent />}
    </>
  );
}
```

여기까지 설정을 완료하였다면 docusaurus로 실행되는 정적 페이지에서 /blog 경로로 생성되는 게시글에 댓글 달기를 활성화할 수 있으며 댓글 달기 활성화를 위해서 enableComments: true를 마크다운 파일 상단에 추가해야 합니다.

```bash
---
title: Title of blog post
authors: author
tags: [keywordOne, keywordTwo]
enableComments: true # for Gisqus
---
```

---

## Giscus 텔레그램으로 알람 받기

### 텔렘그램 봇 생성

- BotFather 대화 시작
- /newbot
- 텔레그램 HTTP API 발급

### 생성된 봇 채널로 이동 후 대화 2번 전송

<span style="color: red;">1번만 전송할 경우 result 배열 빈 값으로 전달</span>

### HTTP GET 요청 전달

- https://api.telegram.org/bot토큰/getUpdates
- result 배열의 chatId 확인

### 텔레그램 API를 사용하여 메시지 전달

```bash
curl -X POST "https://api.telegram.org/[BOT토큰]/sendMessage?parse_mode=MarkdownV2" \
     -H "Content-Type: application/json" \
     -d '{"chat_id": "[chatID]", "disable_web_page_preview": true, "text": "`const a = api()` 메시지 확인"}'
```

---

### 환경 설정

- Settings -> Secrets and variables -> Actions
- `TELEGRAM_CHAT_ID`, `TELEGRAM_TOKEN` 환경 변수 생성
- GITHUB ACTION 생성
- `.github/workflows/telegram.yml` 참고

### 참고 자료

- https://m19v.github.io/blog/how-to-add-giscus-to-docusaurus
- https://jojoldu.tistory.com/704
- https://jojoldu.tistory.com/705?category=777282
