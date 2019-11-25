---
layout: post
title: LogLab 계획
author: wzlog
---

## 소개와 설치

로그랩은 로그를 설계하고 활용하기 위한 프레임웍이다. 크게 다음과 같은 기능을 가지고 있다:

- 로그를 객체지향적으로 설계
- 설계된 로그의 문서 출력
- 설계된 로그의 샘플(가짜) 로그 생성
- 로그가 설계에 맞게 작성되었는지 검증

로그 명세는 로그 지원조직과 로그 코드를 작성하는 서비스 개발자가 협업하여 작성하는 것이 바람직할 것이다.

로그랩은 https://github.com/haje01/loglab 에서 받을 수 있다. [[TODO]]

```
$ git checkout https://github.com/haje01/loglab
$ pip install -e .
```
으로 설치하자.

로그랩은 **메타파일**로 불리는 JSON 파일에 로그 명세를 기술하는 것으로 로그를 설계한다. 메타파일은 로그랩에서 제공하는 JSON 스키마 형식에 맞추어 작성하며, 확장자는 `.meta.json` 을 사용한다. VSCode 등 JSON 스키마를 지원하는 에디터를 이용하면 편집이 용이할 것이다.

## 간단한 로그 명세 만들기
로그랩의 사용법을 빠르게 살펴보기위해, 애크미(Acme)라는 가상의 게임 회사의 foo 라는 모바일 게임을 위한 메타파일을 만들어 보자.

```json
{
	"events": {
		"Login": {
			"desc": "유저 로그인.",
		}
	}
}
```
위와 같이 JSON 문서의 `events` 요소 아래에 사용할 로그 이벤트 요소를 정의하면 된다. 여기서는 `Login` 이벤트를 정의하고 있다. `desc` 요소에 이벤트의 설명을 기입한다.

이 파일을 `foo.meta.json` 이라는 파일명으로 저장하고, 로그랩의 문서 출력 기능을 이용해 보겠다. 문서 출력은 다음과 같은 명령으로 한다.

```
    $ loglab -m foo.meta.json doc
```

이 명령은 -m 인자로 사용할 메타파일을 지정한 후, doc 부명령으로 문서를 생성한다. 다음과 같이 출력된다.

```
Event : Login
Description: 유저 로그인.
+------------+----------+---------------+------------+
| Property   | Type     | Description   | Required   |
|------------+----------+---------------+------------+
| Datetime   | datetime | 이벤트 일시   |  true      |
| Event      | string   | 이벤트 타입   |  true      |
+------------+----------+---------------+------------+
```

Login 에 대한 문서가 출력된다. 각 속성별로 행이 나오고, `속성 이름(Property)`, `타입(Type)`, `설명(Description)`, `필수 여부(Required)` 컬럼으로 속성을 설명하고 있다. Required가 true인 속성은 해당 이벤트에 꼭 필요한 필수 속성이며, false인 경우 없어도 무관한 선택 속성이다. 기본적으로 모든 속성은 필수 속성이다.

첫 행에 만들어 주지 않은 `Datetime` 이라는 속성이 보인다. 모든 로그 이벤트는 출력 시간이 꼭 필요하기에, 로그랩에서 자동으로 만들어 주는 속성이다. 다음으로 `Event`는 이벤트의 종류를 나타낸다. 문서 출력시 원하지 않는 컬럼은 `--exclude-column` 인자로 무시할 수 있다. `Required`를 제외하고 다시 문서를 보면,

```
$ loglab -m foo.meta.json doc --exclude-column required

Event : Login
Description: 유저 로그인.
+------------+----------+---------------+
| Property   | Type     | Description   |
|------------+----------+---------------+
| Datetime   | datetime | 이벤트 일시   |
| Event      | string   | 이벤트 타입   |
+------------+----------+---------------+
```
`Required` 컬럼이 빠진 문서가 출력된다.

### 새로운 속성 추가하기
이제 여기에 로그인한 서버 번호 속성을 추가해 보자. 속성은 이벤트 객체 아래 `props` 요소를 이용한다. 로그랩의 이벤트 및 속성의 이름은 각 단어의 시작을 대문자(Pascal Case)로 한다.

```json
{
	"events": {
		"Login": {
			"desc": "유저 로그인.",
			"props": [
				["ServerNo", "integer", "서버 번호"]
			]
		}
	}
}
```
`props` 는 어레이 값으로, 하나 이상의 속성을 기술할 수 있다. 각 속성은 어레이 형식 또는 객체 형식으로 기술할 수 있다. 어레이 형식은 간단히 속성을 추가할 때 유용하며 `[속성_이름, 속성_타입, 속성_설명, 필수여부(생략가능)]`의 순으로 기술한다. 객체 형식은 이후에 살펴보겠다.

다시 문서를 출력해 보자.

```
$ loglab -m foo.meta.json doc

Event : Login
Description: 유저 로그인.
+------------+----------+---------------+-----------+
| Property   | Type     | Description   | Required  |
|------------+----------+---------------+-----------+
| Datetime   | datetime | 이벤트 일시   |  true     |
| Event      | string   | 이벤트 타입   |  true     |
| ServerNo   | integer  | 서버 번호     |  true     |
+------------+----------+---------------+-----------+
```
`ServerNo`가 추가된 것을 알 수 있다. 이제 `Logout` 이벤트를 새로 추가해 보자.

```json
{
	"events": {
		"Login": {
			"desc": "유저 로그인.",
			"props": [
				["ServerNo", "integer", "서버 번호"]
			]
		},
		"Logout": {
			"desc": "유저 로그아웃.",
			"props": [
				["ServerNo", "integer", "서버 번호"]
			]
		}
	}
}
```

다시 문서를 보면

```
$ loglab -m foo.meta.json doc --skip required --exclude-column required

Event : Login
Description: 유저 로그인.
+------------+----------+---------------+
| Property   | Type     | Description   |
|------------+----------+---------------|
| Datetime   | datetime | 이벤트 일시   |
| Event      | string   | 이벤트 타입   |
| ServerNo   | integer  | 서버 번호     |
+------------+----------+---------------+

Event : Logout
Description: 유저 로그아웃.
+------------+----------+---------------+
| Property   | Type     | Description   |
|------------+----------+---------------|
| Datetime   | datetime | 이벤트 일시   |
| Event      | string   | 이벤트 타입   |
| ServerNo   | integer  | 서버 번호     |
+------------+----------+---------------+
```
로그아웃 이벤트가 추가된 것을 알 수 있다.

### 공통 속성 정리하기
위에서 같은 `ServerNo` 속성이 로그인과 로그아웃에서 중복되고 있는데, 객체 참조 기능을 이용하여 리팩토링할 수 있다.

```json
{
	"bases": {
		"Common": {
			"desc": "공통 요소."
			"props": [
				["ServerNo", "integer", "서버 번호"]
			]
		}
	},
	"events": {
		"Login": {
			"desc": "유저 로그인.",
			"mixins": ["bases.Common"]
		},
		"Logout": {
			"desc": "유저 로그아웃.",
			"mixins": ["bases.Common"]
		}
	}
}
```

참조될  공통 속성을 `bases` 요소 아래 베이스 요소 형식으로 정의하면 된다. 베이스 요소는 이벤트와 비슷하나, 직접 출력되지 않고, 이벤트나 다른 Base 요소에서 참조되기 위한 용도이다.

Login/Logout 이벤트에서는 `mixins` 요소를 통해 참조할 하나 이상의 베이스나 이벤트 요소를 기술할 수 있다. `bases.Common`에서 `bases`는 루트 요소의 이름이고, `Common`은 참조할 베이스 요소의 이름이다. 이벤트 요소를 참조하려면 `events.이벤트_이름` 식이 된다.

만약 `mixins`에 하나 이상의 요소가 있고 그들간 겹치는 속성이 있으면, 나중에 등장하는 것의 속성이 우선하게 된다.

참조되는 측과 참조하는 측에 같은 요소가 겹치면, 참조하는 측의 것을 사용하게 된다. 예를 들어 위의 `bases.Common` 과 `events.Login` 모두에 `desc` 요소가 있는데, 이 경우 `Login` 의 `desc` 요소가 우선하게 된다.

수정된 메타파일로 다시 문서를 출력하면, 리팩토링 전과 같은 결과를 확인할 수 있을 것이다.


## 샘플 로그와 속성값 제약하기
실제 로그가 어떻게 생겼는지 미리 살펴볼 수 있다면, 로그를 만들거나 처리하는 입장에서 도움이 될 것이다. 로그랩에서는 다음과 같이 샘플 로그를 생성할 수 있다.

```
$ loglab -m foo.meta.json sample

{
	"Logout": {
		"Datetime": "2019-11-13T20:20:39+09:00",
		"ServerNo": -1932
	}
}
{
	"Login": {
		"Datetime": "2019-11-13T20:20:40+09:00",
		"ServerNo": 94840191
	}
}
{
	"Login": {
		"Datetime": "2019-11-13T20:20:41+09:00",
		"ServerNo": -3948
	}
}
{
	"Logout": {
		"Datetime": "2019-11-13T20:20:42+09:00",
		"ServerNo": 114938
	}
}
```
임의의 이벤트 로그가 4개 생성이 되었다. Datetime 속성은 현재 시간에서 증가하는 식으로 채워진다. 로그랩의 날자 및 시간 형식은 RFC3339를 따른다. (https://json-schema.org/latest/json-schema-validation.html#RFC3339)

### 최대/최소값 지정
`ServerNo` 속성의 값은 좀 특이한데, 지나치게 큰 값이나 음의 수가 섞여서 나오고 있다. 이는 속성의 타입인 `integer`형에 맞춰 임의의 값이 채워지기 때문이다. 좀 더 그럴듯한 서버 번호를 위해 속성 값에 제약을 추가할 수 있다. 그런데, 제약을 지정하기 위해서는 `props` 아래 속성값을 리스트형이 아닌 객체형으로 기술해야 한다.

```json
{
	"bases": {
		"Common": {
			"desc": "공통 요소."
			"props": [
				{
					"name": "ServerNo",
					"desc": "서버 번호",
					"type": "integer",
					"minimum": 1,
					"maximum": 10
				}
			]
		}
	},
	"events": {
		"Login": {
			"desc": "유저 로그인.",
			"mixins": ["bases.Common"]
		},
		"Logout": {
			"desc": "유저 로그아웃.",
			"mixins": ["bases.Common"]
		}
	}
}
```
객체형은 좀 더 장황하나, 제약 등 세세한 내용을 명시할 수 있다. 속성을 객체형으로 기술할 때는 `name`, `type`, `desc` 등이 필수 요소이며, 필요에 따라 제약 등 다양한 추가 요소를 기술할 수 있다.

다시 샘플 로그를 보면

```
$ loglab -m foo.meta.json sample

{
	"Logout": {
		"Datetime": "2019-11-13T20:20:39+09:00",
		"ServerNo": 3
	}
}
{
	"Login": {
		"Datetime": "2019-11-13T20:20:40+09:00",
		"ServerNo": 9
	}
}
{
	"Login": {
		"Datetime": "2019-11-13T20:20:41+09:00",
		"ServerNo": 1
	}
}
{
	"Logout": {
		"Datetime": "2019-11-13T20:20:42+09:00",
		"ServerNo": 4
	}
}
```

여전히 임의의 속성값이나, 제약 덕분에 `ServerNo` 값이 좀 더 그럴듯해 보인다. 문서에서도 속성의 제약을 확인할 수 있다.

```
$ loglab -m foo.meta.json doc --exclude-column required

Event : Login
Description: 유저 로그인.
+------------+----------+-----------------+------------------+
| Property   | Type     | Description     | Constraint       |
|------------+----------+-----------------+------------------+
| Datetime   | datetime | 이벤트 일시     |                  |
| Event      | string   | 이벤트 타입     |                  |
| ServerNo   | integer  | 서버 번호       | 1이상 10이하     |
+------------+----------+-----------------+------------------+

Event : Logout
Description: 유저 로그아웃.
+-------------+----------+----------------+----------------+
| Property    | Type     | Description    | Constraint     |
|-------------+----------+----------------+----------------+
| Datetime    | datetime | 이벤트 일시    |                |
| Event       | string   | 이벤트 타입    |                |
| ServerNo    | integer  | 서버 번호      | 1이상 10이하   |
+-------------+----------+----------------+----------------+
```
이러한 속성값 제약은 나중에 설명할 로그 검증시에도 유용하게 사용될 수 있다. 위와 같이 로그랩 요소의 속성에 제약을 가할 때는 JSON 스키마의 방식을 따른다. 다양한 제약 방법이 있으니 https://json-schema.org/understanding-json-schema/reference/index.html 을 참고하자.


### 이벤트별 속성 추가
이제 공통 요소와 별도로 이벤트별 속성을 사용해 보자. 예로 `Login`에는 `Platform` 속성을, `Logout`에는 `PlayTime` 속성을 추가하겠다.

```json
{
	"bases": {
		"Common": {
			"desc": "공통 요소."
			"props": [
				{
					"name": "ServerNo",
					"desc": "서버 번호",
					"type": "integer",
					"minimum": 1,
					"maximum": 10
				}
			]
		}
	},
	"events": {
		"Login": {
			"desc": "유저 로그인.",
			"mixins": ["bases.Common"],
			"props": [
				{
					"name": "Platform",
					"desc": "디바이스 플랫폼",
					"type": "string":,
					"enum": ["ios", "aos"]
				}
			]
		},
		"Logout": {
			"desc": "유저 로그아웃.",
			"mixins": ["bases.Common"],
			"props": [
				{
					"name": "PlayTime",
					"desc": "플레이 시간(분)."
					"type": "integer",
					"minimum": 0
				}
			]
		}
	}
}
```

`Login`에 추가된 `Platform` 속성은 유저가 로그인시 이용한 모바일 디바이스의 플랫폼을 가정했다. `string` 타입이되 `enum`으로 나열형 제약이 있다. `enum`은 리스트 값을 가지며 여기에 등록된 값만 허용된다. `Logout`에 추가된 `PlayTime`은 유저가 로그인 후 로그아웃까지 플레이한 시간을 의미하며 분을 단위로 한다. 0이상 값만 허용하는 제약이 있다.

이제 다시 로그 문서를 살펴보자.

```
$ loglab -m foo.meta.json doc --exclude-column required

Event : Login
Description: 유저 로그인.
+------------+----------+-----------------+------------------+
| Property   | Type     | Description     | Constraint       |
|------------+----------+-----------------+------------------+
| Datetime   | datetime | 이벤트 일시     |                  |
| Event      | string   | 이벤트 타입     |                  |
| ServerNo   | integer  | 서버 번호       | 1이상 10이하     |
| Platform   | string   | 디바이스 플랫폼 | ios, aos 중 선택 |
+------------+----------+-----------------+------------------+

Event : Logout
Description: 유저 로그아웃.
+-------------+----------+-----------------+----------------+
| Property    | Type     | Description     | Constraint     |
|-------------+----------+-----------------+----------------+
| Datetime    | datetime | 이벤트 일시     |                |
| Event       | string   | 이벤트 타입     |                |
| ServerNo    | integer  | 서버 번호       | 1이상 10이하   |
| PlayTime    | integer  | 플레이 시간(분) | 0 이상         |
+-------------+----------+-----------------+----------------+
```
`Login` 이벤트에는 `Platform` 속성이, `Logout` 이벤트에는 `PlayTime` 속성에 대한 설명이 추가된 것을 알 수 있다.


## 로그의 검증
잘 정의된 문서와 샘플을 참고해 로그 코드를 만들었다 하더라도, 개발자는 여전히 생성된 로그가 표준에 적합한지 불안할 수 있다. 로그랩의 로그 검증 기능을 이용하면 생성된 로그의 표준 준수 여부를 확인하고, 만약 문제가 되는 부분이 있다면 어디에서 어떤 문제가 있는지 알 수 있다.

예를 들어 다음과 같은 내용의 JSON 로그파일 `my_log.json`이 있다고 하자.

```json
{
	"Login": {
		"Datetime": "2019-11-13T20:20:41+09:00",
        "ServerNo": 1,
        "Platform": ios
	}
}
{
	"Logout": {
		"Datetime": "2019-11-13T20:21:42+09:00",
		"ServerNo": "1"
	}
}
```

아래의 명령으로 이것을 검증할 수 있다.

```
$ loglab -m foo.meta.json verify my_log.json

Error: Not a valid JSON file.
  json.decoder.JSONDecodeError: Expecting value: line 5 column 21 (char 100)
```
먼저 발생한 문제는 이 로그 파일이 유효한 JSON 파일이 아니라는 것이다. 위의 Platform 속성의 값인 ios 에 문자열을 위한 쿼테이션 마크가 없기 때문. 이를 `ios`로 수정하여 다시 돌려보자.

```
$ loglab -m foo.meta.json verify my_log.json

Error: Value type mismatch.
  'ServerNo' is not of type 'integer'
```
메타파일에서 `integer` 형으로 명시한 `ServerNo`의 값에 정수가 아닌 문자열 `1`이 온 것이 문제이다. 이것도 1로 수정하여 다시 돌려보자.

```
$ loglab -m foo.meta.json verify my_log.json

Error: Required property 'PlayTime' does not exist
```

`Logout` 이벤트에  (기본적으로)필수로 선언된 `PlayTime` 속성이 존재하지 않아 발생한 문제이다. `PlayTime`을 추가해 다시 검증하면, 이제 모든 문제가 해결될 것이다.

```
$ loglab -m foo.meta.json verify my_log.json

`my_log.json` is a valid log file!

```
## 공용 메타파일의 이용
지금까지는 단순히 하나의 서비스를 위한 메타파일을 작성해 보았다. 만약 조직에 하나 이상의 서비스가 존재하고, 그것의 로그들이 표준적인 형식을 유지하도록 하려면 어떻게 해야 할까?

로그랩에서는 메타파일들간 공통점을 리팩토링해 공용 메타파일을 만들고, 다른 메타파일이 그것을 참조하여 일관성을 지키게 할수 있다.

앞에서 살펴본 애크미 게임 회사에서, 모바일 게임 foo 외에 새로운 온라인 게임 boo 를 만들고 있다고 하자. 이 회사는 두 서비스가 공통 표준을 따르기를 원하고, 이에 애크미를 위한 공용 메타파일을 작성하기로 한다.

`foo`의 내용을 차용해 아래처럼 `acme.meta.json` 을 작성한다.

```json
{
    "info": {
		"name": "애크미 로그 표준",
		"desc": "애크미의 로그 표준 명세. 자세한 것은 https://github.com/acme/loglab 을 참고하세요."""
    },
	"bases": {
		"Common": {
			"props": [
				{
					"name": "ServerNo",
					"type": "integer",
					"minimum": 1,
					"maximum": 10
				}
			]
		}
	},
	"events": {
		"Login": {
			"desc": "로그인",
			"mixins": ["bases.Common"],
			"props": [
				{
					"name": "Platform",
					"desc": "디바이스 플랫폼",
					"type": "string",
				}
			]
		},
		"Logout": {
			"desc": "로그아웃",
			"mixins": ["bases.Common"],
			"props": [
				{
					"name": "PlayTime",
					"desc": "플레이 시간(분)."
					"type": "integer",
					"minimum": 0
				}
			]
		}
	}
}
```

먼저 새로운 것은, 가장 먼저 나오는 `info` 요소이다. 여기에 메타파일에 관한 여러 정보 요소를 기술할 수 있다. `name`은 메타파일의 이름이고, `desc` 는 메타파일에 관한 추가 설명이다. 다양한 메타파일을 사용할 때 도움이 될 것이다.

나머지는 기존에 `foo` 메타파일의 내용 중, 모바일 특화된 내용인 `Platform`의 `enum` 을 제외한 대부분 요소를 가져왔다. 문서를 출력해보면, 다음과 같이 나온다.

```
$ loglab -m acme.meta.json doc --exclude-column required

Meta : 애크미 로그 표준
Meta Description: 애크미의 로그 표준 명세. 자세한 것은 https://github.com/acme/loglab 을 참고하세요.

Event : Login
Description: 유저 로그인.
+------------+----------+-----------------+------------------+
| Property   | Type     | Description     | Constraint       |
|------------+----------+-----------------+------------------+
| Datetime   | datetime | 이벤트 일시     |                  |
| Event      | string   | 이벤트 타입     |                  |
| ServerNo   | integer  | 서버 번호       | 1이상 10이하     |
| Platform   | string   | 디바이스 플랫폼 |                  |
+------------+----------+-----------------+------------------+

Event : Logout
Description: 유저 로그아웃.
+-------------+----------+----------------+----------------+
| Property    | Type     | Description    | Constraint     |
|-------------+----------+----------------+----------------+
| Datetime    | datetime | 이벤트 일시    |                |
| Event       | string   | 이벤트 타입    |                |
| ServerNo    | integer  | 서버 번호      | 1이상 10이하   |
| PlayTime    | integer  | 세션 시간(분)  | 0 이상         |
+-------------+----------+----------------+----------------+
```
### 공용 메타파일 참조하기
이제 기존의 `foo` 메타파일이 이것을 참조하도록 `foo.meta.json`을 아래와 같이 수정한다.

```json
{
	"info": {
		"name": "foo 로그 명세",
		"desc": "모바일 게임 foo의 로그 명세. 자세한 것은 https://github.com/acme/loglab/foo 를 참고하세요
	},
	"metas": [
		["https://github.com/acme/loglab/aceme.meta.json", "acme"]
	],
	"events": {
		"Login": {
			"mixins": ["acme.events.Login"],
			"props": [
				{
					"name": "Platform",
					"enum": ["ios", "aos"]
				}
			]
		}
	}
}
```
새로운 `metas` 요소가 보인다.  여기에 참조하는 공용 메타파일을 `[메타파일_URL, 메타파일_별칭]` 형식으로 하나 이상 등록할 수 있다.

이런 식으로 외부 메타파일을 참조하면, 다음과 같은 효과가 있다.

- `mixins` 에서 별칭을 통해 외부 메타파일의 베이스나 이벤트 요소를 참조 할 수 있다.
- 샘플 로그나 로그 스키마 생성시, 참조된 메타파일에 있는 이벤트 요소도 자동으로 생성된다.
- 참조되는 측과 참조하는 측에 같은 요소가 겹치면, 참조하는 측의 것이 우선한다.

위 파일에서는 `Login` 이벤트 `Platform` 속성의 `enum` 만 지정하고 있지만, 문서를 출력해보면,

```
$ loglab -m foo.meta.json doc --exclude-column required

Meta : foo 로그 명세
Meta Description: 모바일 게임 foo의 로그 명세. 자세한 것은 https://github.com/acme/loglab/foo 를 참고하세요.

Event : Login
Description: 유저 로그인.
+------------+----------+-----------------+------------------+
| Property   | Type     | Description     | Constraint       |
|------------+----------+-----------------+------------------+
| Datetime   | datetime | 이벤트 일시     |                  |
| Event      | string   | 이벤트 타입     |                  |
| ServerNo   | integer  | 서버 번호       | 1이상 10이하     |
| Platform   | string   | 디바이스 플랫폼 | ios, aos 중 선택 |
+------------+----------+-----------------+------------------+

Event : Logout
Description: 유저 로그아웃.
+-------------+----------+----------------+----------------+
| Property    | Type     | Description    | Constraint     |
|-------------+----------+----------------+----------------+
| Datetime    | datetime | 이벤트 일시    |                |
| Event       | string   | 이벤트 타입    |                |
| ServerNo    | integer  | 서버 번호      | 1이상 10이하   |
| PlayTime    | integer  | 세션 시간(분)  | 0 이상         |
+-------------+----------+----------------+----------------+
```

메타파일 참조를 이용하기 전과같이 `Login`과 `Logout` 두 이벤트의 모든 속성이 잘 출력되는 것을 확인할 수 있다.

마찬가지로 `boo`의 메타파일 `boo.meta.json` 도 아래와 공용 메타파일을 참조하도록 작성하고,

```
{
	"info": {
		"name": "boo 로그 명세",
		"desc": "온라인 게임 boo의 로그 명세. 자세한 것은 https://github.com/acme/loglab/boo 를 참고하세요
	},
	"metas": [
		["https://github.com/acme/loglab/aceme.meta.json", "acme"]
	],
	"events": {
		"Login": {
			"mixins": ["acme.events.Login"],
			"props": [
				{
					"name": "Platform",
					"enum": ["pc", "mac", "linux"]
				}
			]
		}
	}
}
```
문서를 출력해보자.

```
$ loglab -m foo.meta.json doc --exclude-column required

Meta : boo 로그 명세
Meta Description: 온라인 게임 boo의 로그 명세. 자세한 것은 https://github.com/acme/loglab/boo 를 참고하세요.

Event : Login
Description: 유저 로그인.
+------------+----------+-----------------+------------------------+
| Property   | Type     | Description     | Constraint             |
|------------+----------+-----------------+------------------------+
| Datetime   | datetime | 이벤트 일시     |                        |
| Event      | string   | 이벤트 타입     |                        |
| ServerNo   | integer  | 서버 번호       | 1이상 10이하           |
| Platform   | string   | 디바이스 플랫폼 | pc, mac, linux 중 선택 |
+------------+----------+-----------------+------------------------+

Event : Logout
Description: 유저 로그아웃.
+-------------+----------+----------------+----------------+
| Property    | Type     | Description    | Constraint     |
|-------------+----------+----------------+----------------+
| Datetime    | datetime | 이벤트 일시    |                |
| Event       | string   | 이벤트 타입    |                |
| ServerNo    | integer  | 서버 번호      | 1이상 10이하   |
| PlayTime    | integer  | 세션 시간(분)  | 0 이상         |
+-------------+----------+----------------+----------------+
```
공용 메타파일의 이벤트를 유지하되, 온라인 게임에 맞게 `Platform`만 변경된 것을 확인할 수 있다. 이런 방식으로 기본 로그 구조는 공유하면서, 서비스별로 특화된 내용을 추가/갱신할 수 있게 된다.

[[TODO]]
