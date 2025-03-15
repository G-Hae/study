# 리팩토링: filter, sort, map, reduce

### 이전 코드

```js
useEffect(() => {
    const sortedTableData = tableData?.list
        ?.filter((item) => item.chnnelTy !== CHANNEL.INCHAT && item.chnnelTy !== CHANNEL.AGENT)
        .sort((a, b) => a.regDt - b.regDt);
    const indexTableData = sortedTableData?.map((table, index) => ({ ...table, index }));

    setFilterServiceList(indexTableData);
}, [tableData?.list]);
```

이전 코드 내용

-   tableData?.llist가 변경될 때마다 useEffect에서 filterServiceList를 업데이트한다.
-   filterServiceList는 다음의 과정을 거친다

    -   filter: tableData에서 chnnelTy이 INCHAT인것과 AGENT을 제외한다
    -   sort: 등록일이 높은 순으로 저장한다.(즉, 가장 오래된 등록일이 index0으로 된다)
    -   map: filter와 sort가 된 데이터에 map을 포함하여 인덱스 값을 추가한다.

이전 코드의 문제점

-   중복된 루프: filter, sort, map을 각각 실행하며 총 3번의 루프가 도는데 이렇게 3번 반복되는 작업은 코드의 성능에 영향을 미칠 수 있고, 특히 데이터가 많을 경우 비효율적일 수 있음.

-   원본 데이터 변경 가능성: sort() 메서드는 원본 배열을 변경하기 때문에 원본 데이터가 의도치 않게 변할 수 있음

---

다음은 리팩토링한 코드이다.

```js
useEffect(() => {
    const sortedData = tableData?.list.slice().sort((a, b) => a.regDt - b.regDt);
    const filterData = sortedData?.reduce((acc, item, idx) => {
        if (item.chnnelTy !== CHANNEL.INCHAT && item.chnnelTy !== CHANNEL.AGENT) {
            acc.push({ ...item, index: idx });
        }
        return acc;
    }, []);
    setFilterServiceList(filterData);
}, [tableData?.list]);
```

-   slice(): sort()가 원본 배열을 변경하기 때문에 원본 배열을 보호하기 위해 slice()를 사용하였다.

-   sort() 순서 변경: filter와 map을 하나로 처리하기 위해 우선 sort()를 먼저 적용하였다. 본 코드가 적용되는 배열은 전체 배열 길이가 10개정도 밖에 되지 않는 것이라 filter를 먼저 하는 것의 효과가 크게 없었다.

-   reduce()를 통한 통합: filter와 map을 reduce() 하나로 통합하였다. reduce는 배열의 각 요소에 주어진 함수를 실행하여 하나의 결과값을 반환하는 메서드이다. 본 reduce()는 배열을 순회하면서 조건에 맞는 항목을 필터링하고, 인덱스를 추가하는 작업을 동시에 처리했다.
