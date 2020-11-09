# Health Condition Self-Check - API 분석 시트

## 지역코드 (파라미터코드, 도메인코드)
------------------
<details><summary>지역코드 리스트</summary>
<p>
**주의 :09번 지역코드는 없습니다.**  

서울특별시 = 01, sen</br>
부산광역시 = 02, pen</br> 
대구광역시 = 03, dge</br> 
인천광역시 = 04, ice</br> 
광주광역시 = 05, gen</br> 
대전광역시 = 06, dje</br> 
울산광역시 = 07, use</br> 
세종특별자치시 = 08, sje</br> 
경기도 = 10, goe</br> 
강원도 = 11, gwe</br> 
충청북도 = 12, cbe</br> 
충청남도 = 13, cne</br> 
전라북도 = 14, jbe</br> 
전라남도 = 15, jne</br> 
경상북도 = 16, gbe</br> 
경상남도 = 17, gne</br> 
제주특별자치도 = 18, jje
</p>
</details>

-------

## 학교종류
---------
<details><summary>학교종류 리스트</summary>
<p>
유치원 = 1</br>  
초등학교 = 2</br>  
중학교 = 3</br>  
고등학교 = 4</br>  
특수학교 = 5  
</p>
</details>

----------

# Workflow 워크플로

searchSchool -> findUser -> hasPassword -> validatePassword -> (selectUserGroup -> getUserInfo)

validatePassword에서 비밀번호 검사를 합니다.
실패시 실패 카운트(failCnt)가 증가하는데,  
실패 카운트가 5가 되면 5분 정지입니다. 

(selectUserGroup -> getUserInfo)에서는 자가진단 참여자 목록을 얻고 그 개개인의 정보를 얻습니다. (ex 출석 여부)  

매번 token을 response 하는데, Request Header에 Authorization에 넣어줘야합니다.

# https://hcs.eduro.go.kr/v2/searchSchool - 학교 정보 입수
위 주소는 학교의 상세정보를 얻는데 씁니다. 

GET 형식으로 작동합니다. 

-----------------------
## Request
```
https://hcs.eduro.go.kr/v2/searchSchool?lctnScCode=지역코드&schulCrseScCode=학교급&orgName=학교이름&loginType=school
```
각각의 한글 이름들은 '숫자'로 대응됩니다. (학교 이름 제외)

## Response

학교의 일부 단어만 입력하여도 됩니다.
Response 데이터는 schulList 오브젝트에 차곡차곡 관련 학교데이터가 저장됩니다   
배열 형식으로 학교들이 저장되며, 각각엔  

orgCode : 학교 코드입니다. (request시에는 한글 이름이였습니다.) 
kraOrgNm : 학교 이름입니다.  
addres : 학교 주소입니다.  
atptOfcdcConctUrl : 적합한 도 코드가 들어간 도메인입니다. (= 지역코드hcs.eduro.go.kr)

이러한 프로퍼티들이 있습니다.

그 외에도 많은 정보가 schulList 오브젝트에 담겨있습니다.  
따라서, 사용자의 선택 학교의 orgCode를 가져와 사용할 수 있습니다.

## 학교 이름
-----------
학교이름은 재학중인 학교의 이름을 쓰시면 됩니다. 

# https://지역코드hcs.eduro.go.kr/v2/findUser - 유저 찾기

## Request (Post)

```
{   
  "orgcode":"학교 코드",  
  "name":"암호화된 이름",  
  "birthday":"암호화된 생년월일(6자리 형식)",
  "loginType":"school",
  "stdntPNo":None,
}
```

## Response

```
{
  "orgName":"한글학교이름","admnYn":"N","atptOfcdcConctUrl":"도메인",     
  "mngrClassYn":"N","pInfAgrmYn":"Y","userName":"이름","stdntYn":"Y",    
  "token":"배리어 토큰","mngrDeptYn":"N"
}
```

토큰 매우 중요하니 받아올때마다 새로 저장해야합니다.

# https://지역코드hcs.eduro.go.kr/v2/hasPassword 비밀번호 설정 여부

Request 헤더의 Authorization 에 findUser가 response 한 token을 설정합니다.

## Request (Post)
```
{}
```
## Response
```
boolean
```

# https://지역코드hcs.eduro.go.kr/v2/validatePassword - 비밀번호 로그인

암호화된 비밀번호를 전송합니다.

## Request (Post)

```
{ "password":"암호화된 비밀번호","deviceUuid":""}
```

## Response 

1. 성공
  ```
  true
  ```

2. 실패
  ```
  {"isError":true,"statusCode":252,"errorCode":1001,"data":{"failCnt":실패 카운트,"canInitPassword":false}}
  ```

# https://지역코드hcs.eduro.go.kr/v2/selectUserGroup 유저 목록 얻기

Object Array 형식으로 존재하는 모든 유저가 나옵니다. 원하는 유저의 token을 저장해주세요.  

Request는 Authorization 헤더에 토큰을 넣어서, {}로 없습니다.

Response
```
{ (ex 첫째 유저0)
  많은 값들이 있으나, 
  token과 userPNo 정도만 씁니다. 
},
{ (ex 둘째 유저1)
  많은 값들이 있으나, 
  token과 userPNo 정도만 씁니다. 
}
```


# https://지역코드hcs.eduro.go.kr/v2/getUserInfo 유저 정보

selectuserGroup에서 나온 한 유저의 token과 userPNo를 사용합니다.  
Request 헤더 Authorization 업데이트합니다
## Request
```
{"orgCode":"학교코드","userPNo":userPNo}
```
## Response
```
{ token : 배리어토큰 .. 그 외 여러개 있습니다.}
```


# https://지역코드hcs.eduro.go.kr/registerServey - 설문지 제출

Request Authorization을 업데이트해주세요.

## Request
```
  {"rspns01":"1","rspns02":"1","rspns03":null,"rspns04":null,"rspns05":null,"rspns06":null,"rspns07":"0","rspns08":"0","rspns09":"0","rspns10":null,"rspns11":null,"rspns12":null,"rspns13":null,"rspns14":null,"rspns15":null,"rspns00":"Y","deviceUuid":"","upperToken":token,"upperUserNameEncpt":"이름"}
```
설문지 내용에 따라 저 rspns 값들은 달라지며, 체크박스 2개 이상인 경우, 최솟값(= 긍정)은 1이며, 2개중 한개를 택하는 설문은 긍정값(= 아니요)은 0입니다.

request 토큰은 유저정보에서 얻은 token과 같습니다.

## Response

```
{"registerDtm":"Sep 10, 2020 9:54:58 PM","inveYmd":"20200910"}
```

자가진단 날짜가 나옵니다.

## 결과
결과는 다시 유저 정보 얻기에서 isHealthy가 존재하며 값이 true 이면 확인된 것입니다.

## 위의 이름, 생년월일을 암호화 하는 RSA키
30820122300d06092a864886f70d01010105000382010f003082010a0282010100f357429c22add0d547ee3e4e876f921a0114d1aaa2e6eeac6177a6a2e2565ce9593b78ea0ec1d8335a9f12356f08e99ea0c3455d849774d85f954ee68d63fc8d6526918210f28dc51aa333b0c4cdc6bf9b029d1c50b5aef5e626c9c8c9c16231c41eef530be91143627205bbbf99c2c261791d2df71e69fbc83cdc7e37c1b3df4ae71244a691c6d2a73eab7617c713e9c193484459f45adc6dd0cba1d54f1abef5b2c34dee43fc0c067ce1c140bc4f81b935c94b116cce404c5b438a0395906ff0133f5b1c6e3b2bb423c6c350376eb4939f44461164195acc51ef44a34d4100f6a837e3473e3ce2e16cedbe67ca48da301f64fc4240b878c9cc6b3d30c316b50203010001
