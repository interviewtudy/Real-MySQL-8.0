# Real MySQL 8.0

<img width = "50%" src="https://contents.kyobobook.co.kr/sih/fit-in/400x0/pdt/9791158392703.jpg"/>

도서 <Real MySQL 8.0> 스터디 저장소입니다.

- Real MySQL 8.0의 내용을 챕터별로 정리하고 있습니다.

<br>

## 스터디 방식

- 스터디 진행은 독서 요일과 질문 요일로 나누고 차수는 질문 요일을 기준으로 합니다.
- 독서 요일에는 차수에 할당한 챕터를 읽고 정리합니다.
- 질문 요일에는 차수에 할당한 챕터의 질의 응답을 진행하고, 돌아가며 질의 응답 내용을 정리합니다.
- 매차수 내용을 최종 검토한 뒤 main에 merge합니다.

<br>

## 커밋 컨벤션

- `[차수] 챕터 - 챕터제목 (커밋타입)` 의 형태로 커밋은 진행일 및 타입 별로 진행합니다.
- 매 독서 요일마다 자신이 읽은 분량만큼을 정리해서 커밋합니다.
- 루트 디렉토리의 README 업데이트, Merge 등은 별도의 수정없이 진행합니다.

### 커밋타입

- `정리` 챕터에 대한 내용 정리. 한 차수에 대한 최초의 커밋
- `수정` 이미 업로드한 내용에 대한 수정. 한 차수에 대한 최초의 커밋을 제외한 나머지 커밋
- `질문` 챕터에 대한 질의 응답 정리

### 예시

```
[1차수] 1장 - 디자인 패턴과 프로그래밍 패러다임 (정리)
[1차수] 1장 - 디자인 패턴과 프로그래밍 패러다임 (질문)
```

## 기타 규칙

- 브랜치는 자신의 이름을 딴 브랜치로 만들고, 차수마다 각각 챕터를 정리해서 PR을 만듭니다.
- PR에 대한 머지는 `merge commit` 로 진행합니다.
- PR 제목은 진행한 챕터를 콤마(,)로 구분하여 `[차수] 진행챕터` 로 진행합니다. (ex.[1차수] 3장 - 함수, 4장 - 주석)

<br/>

## 👨‍👩‍👧‍👦 스티디원

<table>
  <tr align="center">
    <td>김민지</td>
    <td>김혜연</td>
  </tr>
  <tr>
     <td align="center">
        <a href="https://github.com/mouse0429"><img src="https://avatars.githubusercontent.com/u/68915238?v=4" width="150px" alt="김민지"/><br /></a>
     </td>
    <td align="center">
        <a href="https://github.com/clscls2530"><img src="https://avatars.githubusercontent.com/u/65168017?v=4" width="150px" alt="김혜연"/><br /></a>
     </td>
  <tr>
</table>
