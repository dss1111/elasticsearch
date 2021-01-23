# elasticsearch
elasticsearch 공부한거 까먹기전에 조금씩 올리기

## 사용이유
1. 학교에 설치된 IoT센서는 종류도 매우 많고 필드형식도 다양해서 SQL로 저장하긴 힘들었다.
2. 사용자에 관심사에 따라서 관심사에 매칭된 IoT데이터를 가져와야 했기 때문에 elasticsearch의 검색기능을 사용한다면 저장된 데이터들중 인덱스에 매칭된 데이터를 빠르게 가져올 수 있을것이라 생각했다.

## elasticsearch 특징
* 클러스터 - 여러개의 노드를 클러스터로 구성할 수 있다.
* 인덱싱(색인) - 데이터를 저장할 때 인덱싱 후 저장한다. 따라서 저장할때 별도의 인덱싱 시간 소요후 검색가능하다.
* 샤드,리플리카 - 인덱싱을 샤드라고 불리는 조각으로 분할하여 분할,확장이 가능해지고 병렬화를 통해 성능이 좋아진다.

## Text Query
#### match_all 
모든 도큐먼트 검색
```
GET this_index/_search
{
    "query":{
        "match_all":{ ~~~~ }
    }
}
```
#### match
조건에 알맞은 도큐먼트 검색
* size 지정하면 설정한 사이즈만큼의 데이터만 가져온다. 디폴트가 20이였던가?..(까먹음)
```
GET this_index/_search
{
    "size": 1000, //1000개만큼의 검색결과를 가져온다.
    "query":{
        "match":{
            "name":"yoon"
        }
    }
}
```
### bool
bool은 여러조건을 넣을때 사용했다.
#### must
must의 모든 조건이 만족해야 검색된다. and랑 비슷하게 사용했었다. 이름이 yoon이고 나이가 25인 결과를 검색한다.
```
GET this_index/_search
{
    "query":{
        "bool":{
            "must":[
                {"match":{"name": "yoon"}},
                {"match":{"age": "25"}}
            ]
        }
    }
}
```

#### should
or처럼 동작한다. 이름이 yoon이거나 age가 25인 검색결과를 모두 가져온다.
```
GET this_index/_search
{
    "query":{
        "bool":{
            "should":[
                {"match":{"name": "yoon"}},
                {"match":{"age": "25"}}
            ]
        }
    }
}
```
#### must_not
아닌 경우를 검색한다. 아래와 같은 경우 이름이 yoon이고 id가 hihi가 아닌경우가 검색된다.
```
GET this_index/_search
{
    "query":{
        "bool":{
            "must":[
                {"match":{"name": "yoon"}}
            ],
            "must_not":[
                {"match":{"id": "hihi"}}
            ]
        }
    }
}
```
## 필터
필터링, range등으로 결과 값의 범위를 정한다거나 할때 사용한다. 날짜 범위를 줘서 센서데이터를 검색하는 식으로 사용했다.(이후 날짜데이터는 날짜별 index로 따로 분리하는 방법으로 변경)  
#### range
gt,gte,lt,lte로 데이터 범위를 지정해서 검색할때 사용한다. range로 나이가 15<=age<30인 범위 검색  
gt : greater than     
gte: greater than or equal  
lt : less than        
lte: less than or equal    
```
GET this_index/_search
{
    "query":{
        "bool":{
            "must":[
                {"range":
                    {"age": 
                        {"gte":"15","lt":"30"}
                    }
                }
            ]
        }
    }
}
```
## 통계기능
aggs를 사용한다. aggs에 aggregation을 주고 원하는 결과값을 만들기 위해 쿼리를 수정해주면 된다. 나같은 경우 특정 강의실의 하루간 온도의 평균을 구하는 경우와 같을때 사용했었다.   

모든 학생들의 성적필드의 합을 구하는 경우
```
GET this_index/_search
{
    "aggs": {
        "all_students":{
            "sum":{
                "field":"score"
            }
        }
    }
}
```
min,max,sum,avg 등을 사용할 수 있다. 만약 4개모두와 count를 다가져와야 한다면 stats를 사용하면 된다. 
더 자세히 알고싶다면 [https://esbook.kimjmin.net/08-aggregations/8.1-metrics-aggregations][agg] 설명이 아주 잘되어있다.

## 사용 팁
1. 기간별로 인덱싱하는 것이 관리하기 편하다.     
-> 2020_01,  2020_02 이런식으로 매달마다 생성된 데이터를 인덱스 하나에 저장한다고 생각해보자, 현재 조건으로는 메모리에 3달간의 데이터만 저장할 수 있다면?    
1월,2월,3월 3달간의 데이터를 저장하고 4월이 되었을때 1월의 index를 따로 떼어 백업을 한다거나 삭제한다면 효과적으로 관리할 수 있다!  
2. 날짜는 UTC기준으로 저장하고 불러올때+9:00을 계산하자. Elasticsearch가 UTC 기준으로 되어있어서 우리나라 시간에 맞춰서 저장하는 것은 권장하지 않는다고 한다.
3. RDBMS 와 비교해보면 Database - index ,  Table - type, Column - field, Row - document에 대응하여 구조변경에 참고할 수 있다.

[agg]:<https://esbook.kimjmin.net/08-aggregations/8.1-metrics-aggregations>
