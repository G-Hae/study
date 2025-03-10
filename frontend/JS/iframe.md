# iframe설정

-   문제 상황: 현재 iframe을 통해 채팅화면을 그려주고 있음, 그런데 사파리 브라우저에서 다시 채팅을 연결하는 "재연결"을 클릭할 때 재연결이 되지 않음

-   원인: 현재 로컬스토리지에 채팅 데이터를 저장하고 있었는데, 사파리 브라우저의 경우 로컬 스토리지의 데이터가 초기화가 됨. 이는 HOST URL과 실제 DOCUMENT상 보내주는 HOST URL의 로컬 스토리지 저장 경로가 다르기 때문

-   요청 Host URL 과 실행 Host URL 이 다르다는건 Cross-domain 관련 이슈로 제약 및 제재 가능성을 배제할 수 없음

-   해결방안: 접속 요청한 Host URL 로 로컬스토리지 전환

**작업 내용**

-   위젯 사이트 접속시, BE에서 전달해주는 JS파일 통해 위젯 및 iframe 창 생성

-   **기존**
-   BE에서 요청시 응답받은 url을 iframe.src로 넣어 로컬 스토리지가 별도로 생성되었음

-   **변경**
-   BE에서 받은 url을 호출 할 때 받는 스크립트를 추가 (위젯 배포시 해시 값 없이 정적 주소로 변경)
-   즉, iframe을 사용할 때 src를 사용하여 주소를 넣지 않고 innerHtml을 사용하여 js파일을 불러오고 그 안에서 로직이 처리되도록 수정하였다.

```js
const frameJs = `${url}/static/js/main.static.js`;
const frameCss = `${url}/static/css/main.static.css`;
const frameFont = `${url}/fontawesome.js`;

function hasIframe() {
    let iframe = document.getElementById('widgetFrame');
    return iframe !== null;
}

function createIframe() {
    if (hasIframe()) return;
    let iframe = doc.createElement('iframe');
    iframe.id = 'widgetFrame';
    iframe.classList.add('add_iframe');
    iframe.setAttribute('style', 'border: none');

    iframe.onload = () => {
        const iframeDoc = iframe.contentWindow.document;
        iframeDoc.open();
        const newHtml = `
			<!DOCTYPE html>
			<html lang="ko">
			<head>
			<meta charset="utf-8" />
			<meta name="viewport" content="width=device-width, initial-scale=1" />
			<meta name="theme-color" content="#000000" />
			<meta name="description" content="Web site created using create-react-app" />
			 <title>Chat Widget</title>
			 </head>
			 <body>
			 <noscript>You need to enable JavaScript to run this app.</noscript>
			 <div id="root" data-version="2022_09_08"></div>
			</body>
			</html>
			`;
        iframeDoc.close();
        iframeDoc.documentElement.innerHTML = newHtml;
        const script = iframeDoc.createElement('script');
        script.defer = true;
        script.src = frameJs;
        iframeDoc.head.appendChild(script);

        const link = iframeDoc.createElement('link');
        link.rel = 'stylesheet';
        link.href = frameCss;
        iframeDoc.head.appendChild(link);

        const scriptFont = iframeDoc.createElement('script');
        scriptFont.async = true;
        scriptFont.src = frameFont;
        scriptFont.setAttribute('data-auto-replace-svg', 'nest');
        iframeDoc.head.appendChild(scriptFont);
    };
    chat_widget.appendChild(iframe);
}

function removeIframe() {
    if (!hasIframe()) return;
    let iframe = document.getElementById('widgetFrame');
    iframe.remove();
}

async function reloadIframe() {
    await removeIframe();
    createIframe();
}

if (url) {
    createIframe();
}
```

-   주의점
-   처음에는 write() 메서드를 사용하여 html을 가져오려고 하였다. 그러나 write메서드는 Deprecated되었기 때문에 사용에 주의가 필요했다. 따라서, innerHTML을 사용하여 html을 수정하는 방향으로 처리하였다.
    [MDN write메서드](https://developer.mozilla.org/en-US/docs/Web/API/Document/write)
