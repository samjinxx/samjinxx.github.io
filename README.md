
## 🐸 Samslow's Github Blog
- [https://samslow.github.io](https://samslow.github.io)
- 이 블로그는 [박민](https://github.com/isme2n/isme2n.github.io)님 테마를 기반으로 제작되어 [변성윤](https://github.com/zzsza)님의 검색기능이 추가 된 것입니다.
- 테마에 대한 추가 문의는  [제 메일](shsdf302@gmail.com)로 연락 주세요.

### Structure
- 다른 분들이 이 테마를 Fork할 경우, 사용할 수 있도록 블로그 구조에 대해 설명합니다

```
├── README.md
├── _config.yml : 기본 설정이 저장된 파일
├── _data : 유저 데이터가 저장된 폴더, author.yml만 수정하면 됨
├── _draft : 초안 작성 폴더, 커밋해도 반영되지 않음
├── _featured_categories : 카테고리(메뉴판의 큰 제목)
├── _featured_tags : 카테고리의 태그(메뉴판의 소제목)
├── _includes : 기본 홈페이지 포맷
├── _ipynbs : ipynb 저장 폴더
├── _js : 자바스크립트 소스 저장 폴더
├── _layouts : 타입별 레이아웃 폴더
├── _plugins : 플러그인 저장 폴더. 그러나 Github에서 빌드시 플러그인 사용 불가능
├── _posts : 글 저장 폴더
├── _sass
├── _site : 빌드시 생기는 폴더, 신경쓸 필요 없음
├── about.md : about에서 나타날 내용
├── assets : css, js, img 등 저장하는 폴더
├── favicon.ico : favicon 아이콘
├── feed.xml
├── index.html
├── robots.xml
├── search.html
├── sitemap.xml
├── tile-wide.png
└── tile.png
```

- ```_config.yml```, ```_data```, ```_featured_categories```, ```_featured_tags```, ```about.md``` 내용 수정
- ```favicon.ico```, ```tile-wide.png```, ```tile.png``` 원하는 이미지로 설정

### 로컬 빌드
- Ruby가 설치되어 있어야 합니다
- Ruby 설치는 [공식 문서](https://www.ruby-lang.org/ko/documentation/installation/) 참고

```
bundle exec jekyll serve
```

### 원격 빌드
- Github 저장소를 **{YOURNAME}.github.io** 로 만들어 Push후 **{YOURNAME}.github.io**로 접속
  - 변경은 최대 5분정도 소요 될 수 있습니다.

### 글 작성
- ```_featured_categories```, ```_featured_tags``` 설정한 후, ```_posts```에 글을 작성합니다
- 글 제목 형태는 ```2018-11-22-블로그시작.md``` 이런 방식처럼 작성해야하며, 날짜를 빼고 쓰면 반영되지 않습니다
