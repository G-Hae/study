# slice

### 배열 생성

slice() 메서드는 어떤 배열의 begin 부터 end 까지(end 미포함)에 대한 얕은 복사본을 새로운 배열 객체로 반환합니다. 원본 배열은 바뀌지 않는다. 따라서 배열을 복사하기 위해 slice()를 사용하기도 한다.

#### 사용법

```js
arr.slice([begin[, end]])
```

-   begin(optional): 0을 시작으로 하는 추출 시점에 대한 인덱스이다. 음수의 값은 배열의 뒤부터 추출한다. begin이 undefined인 경우 0번 인덱스부터 slice한다. begin이 배열의 전체 길이보다 긴 경우에는 빈 배열을 반환한다.

-   end(optional): 추출을 종료 할 0기준 인덱스이다. slice는 end를 제외하고 추출한다. 음수 인덱스는 배열의 끝에서부터의 길이를 나타낸다. end가 생략되는 경우 배열의 끝까지 추출하고, end가 배열의 전체 길이보다 긴 경우에는 배열의 끝까지 추출한다.

#### 실제 활용

```js
const isInclude = Number(param.depth) > 1;

const mapTableData = (list, isInclude) =>
    (Array.isArray(list) ? list : []).map((item) => ({
        ...(isInclude && {
            groupLabel: item.groupLabel ?? '',
            groupIdx: item.groupIdx ?? '',
        }),
        label: item.label ?? '',
        idx: item.idx ?? '',
    }));

const rows = mapTableData(total, isInclude);

const header = ['그룹', '그룹 Idx.', '라벨', 'Idx.'];

makExcelFile({
    header: isInclude ? header : header.slice(2),
    rows,
    title: '라벨과 인덱스',
});
```

-   상기 코드는 엑셀의 첫행과 그에 대한 값들을 만드는 함수이다.
-   isInclude에 따라서 엑셀의 컬럼 갯수가 다르다.
-   mapTableData에서는 isInclude인경우만 그룹에 대한 컬럼을 생성한다.
-   makExcelFile에서는 isInclude인 경우에만 그룹에 대한 헤더값(엑셀 첫 행)을 생성한다.
-   header.slice(2)를 사용하였다. 따라서, 2번째 인덱스 '라벨'부터 끝까지 얕은 복사를 진행하여 새로운 배열이 생성된다.
