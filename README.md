# 밥대생 OPEN API #

## API Setting ##

* request_url = "https://bds.bablabs.com/openapi/v1" + url

* API를 사용하시기 위해서는 key를 발급받으셔야 합니다. key값을 발급받기 위해서는 openapi@bablabs.com으로 메일 주십시오. "DEBUG" 값을 넣어, DEBUG 모드로 테스트하실 수 있습니다.

* key값을 이용해 AuthListApi를 호출하시면, accesstoken을 발급받으실 수 있습니다. accesstoken은 유효기간이 존재합니다. accesstoken이 만료되면, AuthListApi를 다시 호출해 accesstoken을 갱신해주세요. AuthListApi는 반드시 지정된 서버에서 호출해주세요. 지정된 서버는 BABLABS측에 key값을 발급요청하실 때 알려주시면 됩니다.

* AuthListApi 이외의 Api에서는 다음과 같은 parameters를 헤더에 추가해 주셔야 합니다.

* AuthListApi 를 제외한 Api들은 Cross domain available하게 설정가능합니다. 허용을 원하시면, key값을 요청하실 때, 같이 알려주세요!


```
header {
	accesstoken: < str > ,
	babsession: < str > 		# random string, 0자 초과 512자 이하
								# 각 유저를 구분하는 데 사용됩니다.
}
```


## Object ##

* result

```
result {
    status: < str | "ok", "fail" > ,
    message: null or < str >
}
```

* auth

```
auth {
    accesstoken: < str > ,
    expired_date: < str | "%Y-%m-%d", "2016-04-02" >
}
```

* campus

```
campus {
    id: < str > ,        
    name: < str > ,     # 밥대생 서비스에 등록된 캠퍼스 이름입니다.
    alias: < str > , 	# 캠퍼스의 별칭입니다. KAIST의 경우, 한국과학기술원, 카이스트가 별칭으로 등록되어있습니다.
    location: null or < location >
}
```

* location

```
location {
    latitude: < float > ,        
    longitude: < float >
}
```


* store

```
store {
	id: < str > ,
    name: < str > ,
    description: null or < str > ,      # 식당에 대한 소개메세지입니다.
                                        # detail api에서만 보내는 것을 원칙으로 합니다.
    open: null or < bool > ,			# 영업시간을 알 수 없는 경우, null을 보냅니다.
    phones: [ < str > ] ,               # 전화번호 리스트를 보냅니다.
    menus: null or [ < menu > ] ,       # menu object를 리스트 형식으로 보내드립니다.
                                        # detail api에서만 보내는 것을 원칙으로 합니다.
    menu_summary: null or < str > ,     # 어떤 메뉴를 파는 곳인지에 대한 요약입니다.
    menu_description: null or < str >   # menus의 길이가 0인 경우에, 왜 0인지 보내드립니다.
    delivery: {
        description: < str >            # 배달 상세정보를 표시합니다.
    }
    open_hours: null or {               # 영업시간과 관련된 정보입니다. detail api에서만 보내는 것을 원칙으로 합니다.
        mon: < open_hour > ,            
        tue: < open_hour > ,
        wed: < open_hour > ,
        thu: < open_hour > ,
        fri: < open_hour > ,
        sat: < open_hour > ,
        sun: < open_hour > ,
        description: < str >
    }
}
```

* menu

```
menu {
    type: < str | "text", "image" > ,
    date: null or < str | "%Y-%m-%d" ex. "2016-10-31" > , # date가 null인 경우 고정메뉴라는 뜻입니다.
    time: < int | 0:아침, 1:점심, 2:저녁, 3:종일, 4:기타 > ,  
    name: null or < str > ,
    description: < str > ,              # type이 text인 경우 식단 정보를 보내드립니다.
                                          type이 image인 경우, 이미지 url을 보내드립니다.
    price: null or < int >              # 가격정보가 입력되지 않은 경우, null을 보냅니다.
}
```

## APIs ##


### AuthListApi  


* 다른 API들을 사용하기 위한 access_token을 조회합니다.
* method - `GET`
* url - `/auths`
* request

```
query {
    key: < str >
}
```

* response

```
json {
    result: < result > ,
    auths: [ < auth > ]
}
```


### CampusListApi ###

* 학식 메뉴 조회가 가능한 캠퍼스 리스트를 보여줍니다.
* method - `GET`
* url - `/campuses`
* request

```
query {
    keyword: null or < str | max 30 > 		# 검색할 키워드를 30자 이내로 요청가능합니다.
}
```

* response

```
json {
    result: < result > ,
    campuses: [ < campus > ]
}


```

### CampusDetailApi ###

* 캠퍼스 상세 정보를 조회할 수 있는 api 입니다.
* method - `GET`
* url - `/campuses/<str: campus_id>`
* request

```
query {
}
```

* response

```
json {
    result: < result > ,
    campus: < campus >
}
```


### StoreListApi ###

* 각 캠퍼스의 식당 리스트를 조회할 수 있는 api 입니다.
* method - `GET`
* url - `/campuses/<str: campus_id>/stores`
* request

```
query {
    type: null or < str | "cafeteria"(default), "campus", "local", "delivery" > ,
    # 기본이 "cafeteria" 요청이라고 가정합니다.
    # cafeteria는 급식제 식당을 의미합니다.
    # campus는 급식제 식당이 아닌 교내 식당을 의미합니다.
    # local은 campus 외부 식당을 의미합니다.
    # delivery는 배달 식당을 의미합니다.
    date: < str | "%Y-%m-%d", "2016-04-02" >
    # 반드시 요청값에 포함해야 합니다. null로 보내시면 안됩니다.
}
```

* response

```
json {
    result: < result > ,
    stores: [ < store > ]
}
```

### StoreDetailApi ###

* 식당 상세 정보를 조회하실 수 있습니다. menus 정보가 포함되어 있습니다. 현재 시각을 기준으로 30일 이후의 식단이나, 15일 이전의 식단은 조회할 수 없습니다.
* method - `GET`
* url - `/campuses/<str: campus_id>/stores/<str: store_id>`
* request

```
query {
    date: < str | "%Y-%m-%d", "2016-04-02" >
    # 반드시 요청값에 포함해야 합니다. null로 보내시면 안됩니다.
}
```

* response

```
json {
    result: < result > ,
    store: < store >
}
```

## Lisence ##

* 절대 API를 DB나 메모리 등 기타 방법을 사용해 캐싱하시면 안됩니다. 저희가 실 사용 후, 저희 서버에 api가 요청되지 않으면 불시에 차단할 수 있습니다.
* 제공해드리는 menu db의 편집저작권은 (주)비에이비랩스에 있습니다.
* 기타 자사의 영업을 방해하는 일체의 행위는 모두 금지됩니다.
* 현재 API는 V1 버전으로 추후 자사의 영업 상황에 따라, 일정 유예기간을 두고 V2로 이전될 수 있습니다. 이전 시에 저희에게 연락주신 방법으로 저희가 개별연락도 드리겠습니다.

## Plan ##

* Free PLAN : 팀 단위가 아닌 개인에게는 월 1만회까지 API가 무료로 제공됩니다. 이 횟수를 초과해서 사용 시, 다른 플랜을 이용하셔야 합니다.
* Basic PLAN : 월 1만회까지 API가 무료로 제공되며, 이후 초과 api 1회 콜당 1원이 과금됩니다.
* Premium PLAN : 월 100만원으로 매월 API 콜 수와 관계 없이, API를 무제한 이용하실 수 있습니다.
* Non-profit PLAN : 비영리 단체의 경우 저희에게 신청해주시면, 내부심사 후 월 10만회 이상도 무료로 API를 제공해드립니다. 
* Special PLAN : 기타 저희와의 협상을 통해 Plan을 수정하고 싶으시면, help@bablabs.com 으로 메일 부탁드립니다.

## Deep Link ##

* Store List UI를 직접 작성하기 힘든 분들을 위해 딥링크도 오픈했습니다.
* 현재 안드로이드에서만 지원합니다.(밥대생 안드로이드 버전 1.10.1.4 이상의 버전에서만 정상동작합니다.)
* access_token을 API를 통해 받아오시면, Deep Link도 사용하실 수 있습니다.
* 사용자가 밥대생 앱을 설치했는지 확인하는 코드가 있어야 앱이 강제 종료되지 않습니다. 주의 바랍니다.

### StoreListActivity ###

* 단체 급식업체 리스트를 보여주는 화면을 띄우실 수 있습니다.
* uri - `bab://campuses/<str: campus_id>/stores`
