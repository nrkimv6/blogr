---
cssclasses: 
obsidianUIMode: preview
dg-publish: true
dg-permalink: obsidian/dataview-inline-tbl
updated: 2024-05-28 02:33:20
title: 옵시디언 데이터뷰(dataview) - 인라인 테이블 구성하기
published: true
---

저는 obsidian의 dataview를 사용하는 것을 좋아합니다.
Notion-like로 문서를 관리하는 것에는 분명 장점을 느끼긴 하지만, 필요 이상으로 파일량을 증가시키는 게 산만하게 느껴지고
또 테이블은 옵시디언에서 사용하기에는 솔직히 불편합니다. 개인적으로 데이터베이스 연동을 고민하다가, 그것도 귀찮아서 색다른 방법을 찾아보게 되었습니다.

이 방법은 하나의 '단일 md파일' 내에 
1) 데이터를 heading과 list 형태로 구성하고, 
2) 그 결과를 dataview형태로도 출력하는 것이 목적입니다.

## 사전 준비

### 샘플 데이터
```
## part 1
- Time:: 14:00
- Scene:: DT-01.01.01
- Summary:: Summary 1

## part 2
- Time:: 16:00
- Scene:: DT-01.01.02
- Summary:: Summary 2

## part 3
- Time:: 22:00
- Scene:: DT-01.01.03
- Summary:: Summary 3
```

heading을 가진 part 로 시작하는 제목 아래에 Time, Scene, Summary가 공통으로 존재하는 데이터를 표 형태로 뽑아보겠습니다.

### 표시방법
 ![dataview|100](https://i.imgur.com/RwAABS6.png)  내부에 적으세요!

```
TABLE WITHOUT ID  rows.L.Time[0] as Time, rows.L.Scene[1] as Scene, rows.L.Summary[2] as Summary
WHERE file.path=this.file.path
FLATTEN file.lists AS L
WHERE contains(meta(L.section).subpath, "part")
GROUP BY file.name + meta(L.section).subpath
SORT rows.file.name ASC
```



### 설명

- dataview TABLE 스크립트를 사용합니다
- WITHOUT ID : 구분 컬럼을 표시하지 않습니다.
- 원 데이터 항목의 컬럼순서는 : Title, Scene, Summary입니다. 
	- 따라서 쿼리에 적을때에는 0번째부터 rows.L.Time[0], rows.L.Scene[1], rows.L.Summary[2]로 적어야 합니다.
- 현재 페이지만을 기준으로 가지고 옵니다.
- file.lists를 기준으로 가지고 옵니다.
- heading 중 part 라는 문자열이 포함된 것들만을 집계하여 가지고 옵니다.


### 결과
![img|700](https://i.imgur.com/XV2kUoZ.png)


## 예시2
바로가기 링크가 있는 '유틸리티 관리' 페이지를 만들어 보았습니다.


### 원본 데이터

```
### 파일 관리 도구
#### Unlocker
- url::https://unlocker.softonic.kr/
- description::다른 프로세스에서 사용 중인 파일을 잠금 해제하고 삭제하는 유틸리티
- category::📁 파일 관리
- necessary::false

#### 7-Zip
- url::https://www.7-zip.org/download.html
- description::높은 압축 비율을 자랑하는 오픈소스 파일 아카이버
- category::📁 파일 관리
- necessary::true

### 개발 도구
#### WinMerge
- url::https://winmerge.org/downloads/?lang=ko
- description::파일 및 디렉토리에 대한 시각적 차이 표시 및 병합을 위한 도구
- category::🔧 개발
- necessary::true

#### Visual Studio Code
- url::https://code.visualstudio.com/docs/?dv=win64user
- description::다양한 언어 및 프레임워크를 지원하는 강력하고 다재다능한 코드 편집기
- category::🔧 개발
- necessary::true

#### PyCharm
- url::https://www.jetbrains.com/ko-kr/pycharm/download/?section=windows
- description::파이썬 프로그래밍을 위한 통합 개발 환경(IDE)
- category::🔧 개발
- necessary::true

#### IntelliJ IDEA
- url::https://www.jetbrains.com/ko-kr/idea/download/?section=windows
- description::자바 개발 및 기타 언어를 위한 종합적인 통합 개발 환경(IDE)
- category::🔧 개발
- necessary::true
```


일단 예제를 설명하기 위한 일부만 가져 왔습니다.
* h3(###로 시작) - 실제 데이터 아님
* 실제로 가져올 데이터는 h4(####로 시작)헤더와 그 아래에 존재하는 리스트(- 로 시작) 들입니다.
* ![img|500](https://i.imgur.com/O8eWA8m.png)


### dataview script

 ![dataview|100](https://i.imgur.com/RwAABS6.png)  내부에 적으세요!
<div>
```
TABLE WITHOUT ID rows.L.category[2] as Category, 
choice(rows.L.necessary[3] , "✅", "✖ ") as "Necessary",
elink(rows.L.url[0], title) as "Name and URL", 
rows.L.description[1] as Description, 
link("####"+title, "⚙️") as Link
WHERE file.path=this.file.path
WHERE file.lists
FLATTEN file.lists AS L
GROUP BY meta(L.section).subpath as title
SORT rows.L.necessary[3] DESC
SORT rows.L.category[2] DESC
```
</div>


### 설명

* dataview table을 사용합니다.
* 데이터는 아래와 같은 순서로 배치되어 있습니다.
	* `url`, `description`, `category`, `necessary`
* column의 각각 순서는 아래와 같습니다.
	* `WITHOUT ID` : 기준열 없음
	* `Category` : rows.L.category[2] //병합된 열의 3번 째 입니다.
	* `Necessary` : 병합된 열의 4번째 데이터가 true이면 ✅, 아닌 모든 경우 ✖를 표시합니다.
	* `Name and URL` : 병합된 열의 첫번째 열(rows.L.url[0])을 링크로, title(아래에서 정의)을 표시명칭으로 표현한 외부 링크(elink)입니다.
	* `Description` :병합된 열의 두번째 항목입니다..
	* `Link` : title(아래에서 정의)을 h4헤더로 한 옵시디언 링크(link) 입니다. 내부 문서의 항목을 가르킵니다.
* 기타 컬럼 설명
	* `WHERE file.path=this.file.path` : 현재 파일 내에서 수행합니다.
	* `WHERE file.lists` : file내의 모든 List bullet을 가지고 옵니다.
		* 이 스크립트에서는 heading에 대한 조건을 걸지 않았습니다.
	* `FLATTEN file.lists AS L` : file.lists의 각 항목 기준으로 행을 분할합니다.
	* `GROUP BY meta(L.section).subpath as title` : 각 섹션의 path를 기준으로 그룹핑하고 그것의 이름을 title로 정의합니다.
	* 정렬은 맨 아래에 오는 게 우선적으로 정렬됩니다.
		* `SORT rows.L.necessary[3] DESC` : necessary 항목의 명칭을 가나다역순으로 정렬합니다.
		* `SORT rows.L.category[2]` DESC : category 항목의 명칭을 가나다역순으로 정렬합니다.

### 결과

![](https://i.imgur.com/rIl0tjy.png)


link : 결과 페이지 온라인으로 보기


## 한계


다만 이 방법대로 한다면, 로딩에 시간이 꽤 걸린다는 분명한 한계가 존재하니 선택적으로 적용하시기 바랍니다.
사실 속도 때문에 그냥 Advance Table로 만족해야 하나 고민도 됩니다.

옵시디언 공홈 에도 데이터베이스에 대한 개발 로드맵이 있던데, 얼른 적용됐으면 좋겠네요.

그리고 더 좋은 방법이 있으시거나 의견 있으시다면 공유 부탁드립니다:)


### 사족

#2024-04-01 처음썼을때보다 속도개선이 꽤 됐긴 한것같은데, 그래도 최근해 해보았을때
 데이터 량이 많으면(속성 10개이상에 각 속성의 최대길이 1000자 이상) 못쓸정도로 심각하게 느려지는 문제가 있었습니다.
데이터뷰로도 해보고 dataviewjs로도 해보고 여러가지 방법으로 시도해 봤지만 
가장 빠른 방법은 csv로 만들어서 임포트하는거였었네요(flatten 사용하지 않음)

어쨌건 양이 많은 데이터는 옵시디언으로 관리하기에 적합하진 않은 것 같습니다.


## 참고

* https://github.com/TfTHacker/obsidian42-brat
* https://forum.obsidian.md/t/dataview-make-a-table-with-a-separate-row-for-different-values-of-a-key/43780


---
## 연관포스트
옵시디언 데이터뷰(dataview) - 조건에 따른 행(row) 색상 변경하기 1
옵시디언 데이터뷰(dataview) - 조건에 따른 행(row) 색상 변경하기 2 - 쉬운 버전

#obsidian #dataview
