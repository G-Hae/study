# 다른 input창 숫자 연속입력

-   요구 상황: 사업자 등록번호와 같은 연속된 숫자 입력을 받는 인풋 필드가 여러 개로 나눠져 있을 때, 사용자가 숫자를 입력하면 자동으로 다음 인풋 필드로 넘어가게 하고 싶음.

-   핵심 해결 방법: useRef와 focus(), select() 메서드를 사용하여 인풋 필드 간에 자연스러운 이동 구현

1. **useRef**로 각 input 요소에 대한 참조 생성 (2번째, 3번째 인풋에만 생성)

2. 사용자가 input에 값을 입력하면 **focus()**를 활용해 자동으로 다음 input으로 포커스를 이동

3. 기존에 텍스트가 있는 경우 지워지고 새로 작성되어야 하므로, **select()** 메서드를 사용하여 사용자가 기존에 입력한 값이 자동으로 선택되게 함

```JS
const sndRef = useRef<HTMLInputElement>(null);
const trdRef = useRef<HTMLInputElement>(null);

const handleInputChange = (key: 'fstBizNum' | 'sndBizNum',|'trdBizNum', maxLengh:number, nextRef?:React.RefObject<HTMLInputElement>)=>{
	(e:React.ChangeEvent<HTMLInputElement>) =>{
		let value = e.target.value.repalce(reg,""); //숫자만 입력
		onChangeBizNumber(key, value);
		if(value.length === maxLength && nextRef?.current){
			nextRef.current.focus();
			nextRef.current.select()
		}
	}

}
```
