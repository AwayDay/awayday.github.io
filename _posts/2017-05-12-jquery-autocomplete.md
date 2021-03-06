---
layout: post
title: "Autocomplete"
description: "검색을 즐겁게"
tags: [javascript, jquery]
---

# 들어가기 전에
* 진실로 간단요약

# JQueru Autocomplete
* Jquery에서 제공하는 __자동 완성__ 기능
    * 기능?
    * 검색 등에서 활용
        * 사실상 검색에서만 쓰는 것이 아닌....?
* HTML `<input>` 엘리먼트에 붙이는 방식으로 추가

```html
<input type="text" id="input_auto"></input>
<script>
    $('#input_auto').autocomplete();
</script>
```

## 주요 데이터
* `.autocomplete`
    * json 객체

```javascript
var data = {};
$('#input_auto').autocomplete(data);
```

* 객체에 필요한 데이터나 함수를 넣어 커스텀

### 자동완성 데이터
* __source__
    * JavaScript Array : `[]`
    * JavaScript String : `""`
    * JavaScript Function : `function(){}`

* Array
    * 데이터가 정적일 경우 사용

```javascript
var data = {
    source: ["사과", "바나나", "포도", ...]
}
```

* 구조화된 데이터가 필요할 경우 : json 형식의 데이터 배열도 가능

```javascript
var data = {
    source: [{ label: "사과", value: "apple" }, { label: "바나나", value: "banana" }, ...]
}
```

    * `label` 은 검색 키워드, `value` 은 실제 입력되는 값

* String
    * String은 json 형식의 데이터를 응답하는 URL이어야 함.

* Function
    * ajax 요청, 필터링, 콜백, 기타 추가적 동작을 필요로 할 경우 사용
    * 거의 이것 때문에 레퍼런스를 찾아보지 않을까?

```javascript
var data = {
    source: function(request, response) {
        // ajax 등
        // request.term === $("#input_auto").val()

        // 최종적으로 결과 데이터를 생산하기 위해 response() 수행
        // 위치 예시 : ajax success 등
        // response에는 JavaScript Array 타입이 인자로 들어감
        response();
        // ..
    } 
};
```

    * ajax 예시

```javascript
var data = {
    source: function(request, response) {
        $.ajax({
            type: "get",
            url: "test.com",
            data: { type: "json", value: request.term },
            dataType: "json",
            success: function(data) {
                // 결과 data를 가공한다
                var filteredArray = data.array.filter(..);
                // 가공한 데이터를 map 돌려서 라벨-밸류 쌍으로 뽑아낸다
                response(
                    $.map(filteredArray, function(item) {
                        return {
                            label: item.key,
                            value: item.value
                        };
                    })
                );
            }
        });
    } 
};
```

* 대충 이런 느낌이다.