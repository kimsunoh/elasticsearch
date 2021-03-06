# 5. 검색

예제 5.1 5_1_books.json 파일 내용을 벌크 API로 입력
```
curl -XPOST -H 'Content-Type: application/json' localhost:9200/_bulk --data-binary @5_1_books.json

curl -XPOST -H 'Content-Type: application/json' localhost:9200/_bulk --data-binary @5_2_magazines.json
```


## 5.1 검색(_search) API

예제 5.2 books 인덱스, book 타입에서 hamlet 검색
```
GET /books/book/_search?q=hamlet
```


예제 5.3 books 인덱스에서 hamlet 검색
```
GET /books/_search?q=hamlet
```


예제 5.4 books, magazines 인덱스에서 time 검색
```
GET /books,magazines/_search?q=time
```


예제 5.5 _all을 사용해 전체 인덱스에서 time 검색
```
GET /_all/_search?q=time
```


예제 5.6 인덱스 지정을 생략해 전체 인덱스에서 time 검색
```
GET /_search?q=time
```


## 5.2 URI 검색

### 5.2.1 q(query)

예제 5.7 전체 인덱스의 title 필드에서 time 검색
```
GET /_search?q=title:time
```


예제 5.8 title 필드에 검색어 time과 machine을 AND 조건으로 검색
```
GET /_search?q=title:time AND machine
```


### 5.2.2 df(default field)

예제 5.9 df 매개변수를 사용해서 title 필드에서 time 검색
```
GET /_search?q=time&df=title
```


### 5.2.3 default_operator

예제 5.10 default_operator 매개변수를 사용해서 기본 조건 명령어를 AND로 지정
```
GET /_search?q=title:time machine&default_operator=AND
```


### 5.2.4 explain

예제 5.11 explain 매개변수를 사용해서 검색 처리 결과 표시
```
GET /_search?q=title:time&explain
```


### 5.2.5 _source

예제 5.12 _source 매개변수를 false로 설정해 도큐먼트 내용을 배제하고 검색
```
GET /_search?q=title:time&_source=false
```


### 5.2.6 \_source_include

예제 5.13 _source_include 매개변수를 사용해 title, author, category 필드만 출력
```
GET /_search?q=title:time&_source_includes=title,author,category
```


### 5.2.7 sort

예제 5.14 author 필드가 jules인 도큐먼트를 pages 필드를 기준으로 오름차순 정렬
```
GET /books/_search?q=author:jules&sort=pages
```


예제 5.15 author 필드가 jules인 도큐먼트를 pages 필드를 기준으로 내림차순 정렬
```
GET /books/_search?q=author:jules&sort=pages:desc
```


예제 5.16 author 필드가 jules인 도큐먼트를 _score 를 기준으로 오름차순 정렬
```
GET /books/_search?q=author:jules&_source_includes=title&sort=_score:asc
```


### 5.2.8 timeout

### 5.2.9 from

예제 5.18 from 매개변수를 사용해서 2번째 결과부터 표시
```
GET /books/_search?q=author:jules&_source_includes=title&from=1
```

### 5.2.10 size

### 5.2.11 search_type

예제 5.19 search_type=query_then_fetch로 검색
```
GET /books/_search?size=1&q=author:William&search_type=query_then_fetch&_source_includes=title,author
```


예제 5.21 search_type=dfs_query_then_fetch으로 검색
```
GET /books/_search?q=author:william&_source_includes=title,author&search_type=dfs_query_then_fetch&scroll=10m
```

## 5.3 리퀘스트 바디 검색

예제 5.23 리퀘스트 바디로 author 값이 william인 값 검색
```
GET /books/_search
{
  "query" : {
    "term" : { "author" : "william" }
  }
}
```

### 5.3.1 size, from, fields

예제 5.24 from:1, size:2, _source:[“title”,”category”] 조건으로 전체 필드에서 time 검색
```
GET /_search
{
  "from" : 1,
  "size" : 2,
  "_source": ["title","category"],
  "query" : {
    "term" : {  "author" : "william"  }
  }
}
```

### 5.3.2 sort
*실습 필요. 오류로 실습하지 못함*

예제 5.25 category - 내림차순, pages, title - 오름차순 순서로 검색 결과 정렬
```
GET /books/_search
{
  "_source": ["title","author","category","pages"],
  "sort" : [{"pages":"desc"},"sell"],
  "query" : {
    "term" : { "title" : "the" }
  }
}
```


예제 5.26 pages mode: min으로 검색
```
GET /books/_search
{
  "_source": ["title","author","category","pages"],
  "sort": [{"pages":{"order":"desc","mode":"min"}},"sell"],
  "query": {
    "term": { "category" : "science" }
  }
}
```


예제 5.27 pages mode: max로 검색
```
GET /books/_search
{
  "_source": ["title","author","category","pages"],
  "sort": [{"pages":{"order":"desc","mode":"max"}},"sell"],
  "query": {
    "term": { "category" : "science" }
  }
}
```


예제 5.28 title, author 필드로 정렬. author 필드가 없어서 검색 실패
```
GET /_search
{
  "_source" : ["title","author","category"],
  "sort" : ["title","author"],
  "query" : {
    "term" : { "title" : "time" }
  }
}
```


예제 5.29 ignore_unmapped를 true로 설정. author 필드가 없이도 검색 성공
```
curl 'localhost:9200/_search?pretty' -d '
{
  "fields" : ["title","author","category"],
  "sort" : ["title",{"author":{"ignore_unmapped" : true}}],
  "query" : {
    "term" : { "title" : "time" }
  }
}'
```


예제 5.30 track_scores를 true로 설정. 점수 정보 표시
```
curl 'localhost:9200/_search?pretty' -d '
{
  "fields" : ["title","author","category"],
  "sort" : ["title",{"author":{"ignore_unmapped" : true}}],
  "track_scores": true,
  "query" : {
    "term" : { "title" : "time" }
  }
}'
```

### 5.3.3 _source

예제 5.31 _source: false로 설정해 도큐먼트 내용은 보이지 않게 검색
```
GET /books/_search
{
  "_source": false,
  "query": {
    "term": { "author": "william" }
  }
}
```


예제 5.32 _source를 이용해서 title과 c로 시작하는 필드명의 값 표시
```
GET /magazines/_search
{
  "_source": ["title","c*"]
}
```


### 5.3.4 partial_fields, fielddata_fields
*실습 필요. 오류로 실습하지 못함*

예제 5.34 title, category 필드 출력
```
curl 'localhost:9200/magazines/_search?pretty' -d '
{
  "fields":["title","category"]
}'
```


예제 5.35 c로 시작하면서 ry로 끝나지 않는 필드 표시
```
curl 'localhost:9200/magazines/_search?pretty' -d '
{
  "partial_fields" : {
    "partial_1" : {
      "include" : "c*",
      "exclude" : "*ry"
    }
  }
}'
```

### 5.3.5 highlight

예제 5.37 author 필드의 검색어 william 강조
```
GET /books/_search?
{
  "query" : {
    "term" : { "author" : "william" }
  },
  "highlight" : {
    "fields" : { "author" : {} }
  }
}
```


예제 5.38 strong 태그를 이용해 author 필드의 검색어 william 강조
```
GET /books/_search?
{
  "query" : {
    "term" : { "author" : "william" }
  },
  "highlight" : {
    "pre_tags" : ["<strong>"],
    "post_tags" : ["</strong>"],
    "fields" : { "author" : { } }
  }
}
```
