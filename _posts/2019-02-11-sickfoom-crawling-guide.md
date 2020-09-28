---
layout: post
title:  "식품안전나라 크롤링 가이드"
subtitle:   "식품안전나라 크롤링 가이드"
categories: diary
tags: diary
comments: true
---
API를 개떡같이 제공해줘서 `Ruby on Rails` `Nokogiri`로 직접 크롤링하는 삽질을 쓰는 글

# 식품안전나라 크롤링

우리나라에서 식품정보를 얻을 수 있는 방법을 시도 해 본 개발자라면, [공공데이터포털](https://www.data.go.kr/)을 한번 쯤 들어 봤을 것이다.

사용 용도에 따라 음식의 이름만 사용하기도 하고, 레시피를 쓰기도 하지만 필자는 영양소 정보가 필요했다.

공공 API에서 제공해주는 정보는 `'탄수화물', '단백질', '지방'` 외의 기본 영양소 9종을 제공해준다. 

따라서 처음에 필자는 이 정보만을 사용했지만 팀원으로부터 `추가 영양소` 가 있는 곳을 찾았다는 제보를 받았다.

하지만, 놀랍게도 그곳은 같은 DB를 사용하는 `식품안전나라` 였고, 아직도 왜 이곳만 영양소 정보가 더 있는지는 모른다.

현재까지는 영양소정보를 34종까지 제공 해주는것을 봤다. (다른 음식엔 더 많은 영양소가 있을 수도 있다)

<img src="/assets/post_img/image-20190211083747629.png" />

필자의 서비스는 이런 영양소 정보가 많을수록 자세한 통계정보를 제공 해 줄 수 있기 때문에 해당 DB를 가져오려고 했으나

어째서인지 API형태로는 기본 영양소 9종만 제공해주었고 위와같은 추가영양소는 상세페이지에서만 제공되었기 때문에

해당 페이지들을 모두 크롤링 하는것을 시도했다.

그 첫번째 시도로 크롤링 해야 할 URL을 분석했다.

### URL분석

- `간편검색 페이지` URL 간소화
  - 기존버전
    - `https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/simpleSearch.do?menu_no=&menu_grp=&code4=1&code2=&search_name=&page=245`
    - 주소에 불필요한 키값이 너무 많고, 있다해도 실제 검색과는 독립적으로 존재하는 부분의 대한 정보이기에 아래와같이 간소화 할 수 있다.
  - 간소화 버전
    - `https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/simpleSearch.do?&page=245&code4=1`
    - 추가적으로 3개의 탭을 이동하는 변수는`code4=[1,2,3]` 이다.
    - 전체 변수를 6개에서 2개로 줄일 수 있다.
- `음식 세부정보 페이지` URL 간소화 
  - 기존 버전
    - `https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/searchDetail.do?menu_no=&menu_grp=MENU_NEW03&code4=1&code2=&search_name=&food_cd=100101000100000001&labor_cd=201&year2=2001&page=1`
    - 세부정보 페이지는 변수가 하나만 잘못 전달해주어도 아예 결과가 뜨지 않기 때문에 더 세세하게 봐야한다.
    - 위의 `간편검색 페이지` 와 마찬가지로 불필요한 변수를 제거한다.
  - 간소화 버전
    - `https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/searchDetail.do?food_cd=100101000100000001&labor_cd=201&year2=2001`
    - 필요없는 키를 제거하고 실제 필요한 키값은 `food_cd`,`labor_cd`,`year2` 3가지
    - 전체 변수를 9에서 3개로 줄일 수 있다.
    - 이를 지금부터 `baseform` 이라고 한다.



### 문제 정의

1.  `간편검색 페이지`에서도 음식 별 최대 10가지 정보만을 제공 해주기 때문에  `음식 세부정보 페이지` 를 크롤링해야한다.

2. 각 페이지의 데이터를 `간편검색 페이지` 에서 가져와서 모든 페이지를 한번씩 들어가서 영양소를 크롤링한다.

3. 각 페이지의 `<a>` 태그는 아래와같이 되어있다.

   <img src="/assets/post_img/image-20190211085007813.png" />

   



### `간편검색 페이지`의 `<a>` 태그들의 `href` 속성 크롤링

#### 문제인식

* `<a href="javascript:goDetail('100106002100100002', '201', '2017');">`
* `<a>` 태그의 `href` 속성이 일반적인 링크가 아니라 js function으로 인자를 넘긴다.
* `baseForm`을 모두 활용하지 않고 3가지 정보만을 js function으로 주고있다.    



#### 시도 할 것

1. 해당 attr받아서 인자를 추출할 수 있다면 `음식 세부정보 페이지 URL`  간소화폼에 넣을 수 있을 것 같다.
2. 인자추출은 `('')` 을 기준으로 `,` 로 나누면 되지 않을까?
3. 시도!



#### 시도과정

1. 아래는 Ruby on Rails 를 Nokogiri 를 사용하여 구현한 Controller 코드이다.

   * ```ruby
     def pull_cptfood
         require 'open-uri'
         (1..3).each do |tab|
           (1..126).each do |page|
             @doc = Nokogiri::HTML(open("https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/simpleSearch.do?&page=#{page}&code4=#{tab}"))
             @foods = @doc.css('#tab1 th > a')
             @foods.each do |x|
               tit = x.text.strip
               url = x['href']
               @factors = url.split('(\'')[1].split(',').map {|x| x[/\d+/]}
               
               @food_cd = @factors[0]
               @labor_cd = @factors[1]
               @year = @factors[2]
               
               Cptfood.create(tab: tab, page: page, name: tit, food_cd: @food_cd, labor_cd: @labor_cd, year: @year)
             end
           end
         end
       end
     ```

2. 모든탭 > 모든 페이지 > 모든 음식 정보( 키의 3요소 ) 저장 성공!



#### 시도 결과

* 3개의 탭마다 최대 페이지수를 알아내 탭별로 크롤링 하였다. (위의 경우 1번째 탭은 126가지)

* 나머지 2개의 탭도 위의 코드와 동일하고 최대 페이지수만 달라짐

  크롤링 결과는 아래와 같다.

  <img src="/assets/post_img/image-20190211090850674.png"/>



#### (참고)goDetail함수 구성

```javascript
function goDetail(food_cd, labor_cd, year2) {

		fn_addRequiredClassForId('menu_no');
		fn_addRequiredClassForId('menu_grp');

		fn_addRequiredClassForId('code4');
		fn_addRequiredClassForId('code2');
		fn_addRequiredClassForId('search_name');
		fn_addRequiredClassForId('food_cd');
		fn_addRequiredClassForId('labor_cd');
		fn_addRequiredClassForId('year2');
		fn_addRequiredClassForId('page');

		fn_removeFormInputElementsForId('baseForm');

		$('#food_cd').val(food_cd);
		$('#labor_cd').val(labor_cd);
		$('#year2').val(year2);

		//$('#search_name').val(encodeURI($('#search_name').val()));

		document.baseForm.target = "_self";
		document.baseForm.method = "get";
		document.baseForm.action = "searchDetail.do?menu_grp=MENU_NEW03&menu_no=2805";
		document.baseForm.submit();
	}
```



### `음식 세부정보 페이지` 의 각 영양소 크롤링

<img src="/assets/post_img/image-20190211091323857.png" />

페이지 구성은 위와 같다.

`식품 안전 나라` 의 정보는 어쩔땐 모든 영양소가 `N/A`로 표기되어 있음에도 행이 존재하기도 하고 

어쩔때는 정보가 없으면 행 자체가 없기도 하여 일관성이 없다. ~~역시 이럴땐 개발자가 고생해야지!~~

위의 이유로 여기서는 저장된 주소값 3요소를 가지고 세부 페이지에 들어가 이터레이터로 모든 행을 담아 DB에 저장하면 된다.

```ruby
def cptfoodInfo
    require 'open-uri'

    Cptfood.all.each do |f|
        @doc = Nokogiri::HTML(open("https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/foodnutrient/searchDetail.do?food_cd=#{f.food_cd}&labor_cd=#{f.labor_cd}&year2=#{f.year}"))

        @rows = @doc.css('article > table > tbody > tr')
        @rows.each do |r|
            @cols = r.search('td')
            InfoCptfood.create(
                name: f.name,
                year: f.year,
                num: @cols[0].text,
                fact_id: @cols[1].text,
                short: @cols[2].text,
                fact_name: @cols[3].text,
                content: @cols[4].text.strip().to_f,
                unit: @cols[5].text
                )
        end
    end
end
```



위의 코드로 세부정보 페이지의 모든 영양소까지 크롤링이 가능하고 [CSV로 추출](https://samslow.me/development/2018/12/09/Dialog-Flow%EC%97%90-csv-%EB%84%A3%EA%B8%B0/)하여 필요한 곳에 전달해주면 된다.

다만, 여기서 사소한 문제가 있는데 가끔씩 `음식 세부정보 페이지`  `음식이름` 속에 `\n` 심볼이 끼어들어가 있는 곳이 있어 

CSV로 추출하게되면 말썽을 일으키는 데이터가 발견되는데 잊지말고 잘 처리하여 전달받는 사람이 정상적으로 데이터를 볼 수 있도록 하자.



### 후기

이런식의 이중 구조 크롤링은 처음이라 잘 모르기도 하고, 사실 더 간단하게 할 수 있는 방법이 있을텐데

필자는 테이블을 2개를 만들어 따로따로 작업을 해주었다.

그래도 다른 누군가가 보기 편하도록 기능이 원자단위로 되니 유익한 글이 될것으로 본다.

















