# Typing Effect 

## 1. 프로젝트 개요
- **목적**: 코드 타이핑 효과와 커서 애니메이션을 구현한 웹 컴포넌트
- **기술 스택**: HTML, CSS, JavaScript (Vanilla)
- **주요 파일**: index.html

## 2. 코드 섹션별 설명

### 2.1 HTML 구조
- 각 언어별 카드(`.card`)와 코드 표시 영역(`.content`)으로 구성
- 접근성 및 커서 제어를 위해 `tabindex`, `role` 속성 사용

```html
<div class="card" id="html">
  <div class="label">🔧 HTML</div>
  <div class="content" tabindex="-1" role="presentation"></div>
</div>
<!-- scss, js 카드도 동일 구조 -->
```

### 2.2 CSS 스타일
- 커서 애니메이션, 기본 커서 숨김, 선택 방지 등 시각 효과 담당

```css
.content {
  user-select: none;
  caret-color: transparent;
  outline: none;
}
.cursor {
  display: inline-block;
  width: 2px;
  height: 1em;
  background: #d4d4d4;
  animation: blink 1s step-end infinite;
  vertical-align: text-bottom;
  pointer-events: none;
}
@keyframes blink {
  50% { opacity: 0; }
}
```

### 2.3 코드 데이터 구조
- 각 언어별로 타이핑될 코드 라인을 배열로 저장

```javascript
const codeData = {
  html: [ ... ],
  scss: [ ... ],
  js: [ ... ]
};
```

### 2.4 타이핑 효과 및 커서 제어 JS
- 각 `.content` 영역에 한 글자씩 타이핑하며, 커서를 항상 마지막에 위치시킴
- 기본 커서는 숨기고, CSS로 만든 커서만 표시

#### 주요 코드(전체 흐름 및 주석)

```javascript
function typeWithCursor(containerId, lines, lineDelay = 800, charDelay = 80) {
  // 1. 타겟 컨테이너(코드 표시 영역) 선택 및 초기화
  const target = document.querySelector(`#${containerId} .content`);
  target.innerHTML = '';

  // 2. 커서(span) 생성 및 코드 영역에 추가
  const cursor = document.createElement('span');
  cursor.className = 'cursor';
  target.appendChild(cursor);

  let lineIndex = 0; // 현재 타이핑 중인 라인 인덱스

  // 실제 타이핑 효과를 담당하는 함수
  function typeLine() {
    if (lineIndex >= lines.length) return; // 모든 라인 타이핑 완료 시 종료

    const htmlLine = lines[lineIndex];
    const tempDiv = document.createElement("div");
    tempDiv.innerHTML = htmlLine;
    const childNodes = Array.from(tempDiv.childNodes);
    let childIndex = 0;

    // [타이핑 효과의 핵심] 한 라인의 각 노드(코드 조각)를 순차적으로 처리
    function typeChild() {
      if (childIndex >= childNodes.length) {
        lineIndex++;
        setTimeout(typeLine, lineDelay); // 다음 라인 타이핑
        return;
      }
      const node = childNodes[childIndex];
      const span = document.createElement("span");
      span.className = node.className || '';
      const text = node.textContent;
      let charIndex = 0;

      // [타이핑 효과의 실제 동작] 한 글자씩 추가
      function typeChar() {
        if (charIndex < text.length) {
          span.textContent += text[charIndex]; // 한 글자 추가
          charIndex++;
          target.insertBefore(span, cursor); // 커서 앞에 현재까지 입력된 span 추가
          setTimeout(typeChar, charDelay); // 다음 글자 타이핑
        } else {
          target.insertBefore(span, cursor); // 해당 코드 조각 완성 후 커서 앞에 위치
          childIndex++;
          setTimeout(typeChild, charDelay); // 다음 코드 조각 타이핑
        }
      }
      typeChar();
    }
    typeChild();
  }
  typeLine();
}
```

#### 상세 설명
- **typeWithCursor 함수**: 타이핑 효과의 진입점. 각 카드(`html`, `scss`, `js`)에 대해 별도로 실행됨
- **typeLine 함수**: 한 줄씩 순차적으로 타이핑 시작. 줄이 끝나면 다음 줄로 넘어감
- **typeChild 함수**: 한 줄 내에서 여러 코드 조각(span)을 분리하여 처리
- **typeChar 함수**: 한 코드 조각의 텍스트를 한 글자씩 타이핑하며 추가. 커서가 항상 마지막에 위치하도록 `insertBefore(span, cursor)` 사용
- **커서**: 실제 입력 커서는 숨기고, CSS로 만든 `.cursor`만 보이게 하여 애니메이션 효과를 줌

> **타이핑 효과의 실제 동작**은 `typeChar` 함수 내부에서 이루어집니다. 한 글자씩 span에 추가하고, 매번 커서 바로 앞에 위치시켜 마치 실제 타이핑처럼 보이게 만듭니다.

  let lineIndex = 0;

  function typeLine() {
    if (lineIndex >= lines.length) return;
    const htmlLine = lines[lineIndex];
    const tempDiv = document.createElement("div");
    tempDiv.innerHTML = htmlLine;
    const childNodes = Array.from(tempDiv.childNodes);
    let childIndex = 0;

    function typeChild() {
      if (childIndex >= childNodes.length) {
        lineIndex++;
        setTimeout(typeLine, lineDelay);
        return;
      }
      const node = childNodes[childIndex];
      const span = document.createElement("span");
      span.className = node.className || '';
      const text = node.textContent;
      let charIndex = 0;
      function typeChar() {
        if (charIndex < text.length) {
          span.textContent += text[charIndex];
          charIndex++;
          target.insertBefore(span, cursor);
          setTimeout(typeChar, charDelay);
        } else {
          target.insertBefore(span, cursor);
          childIndex++;
          setTimeout(typeChild, charDelay);
        }
      }
      typeChar();
    }
    typeChild();
  }
  typeLine();
}
// 각 카드에 타이핑 효과 적용
 typeWithCursor('html', codeData.html, 800, 80);
 typeWithCursor('scss', codeData.scss, 800, 80);
 typeWithCursor('js', codeData.js, 800, 80);
```

## 3. 유지보수 및 확장 가이드
- **새 코드 추가**: codeData 객체에 원하는 언어 key와 라인 배열을 추가하면 됨
- **커서 스타일 변경**: .cursor CSS 수정
- **애니메이션 속도 조정**: typeWithCursor 함수의 delay 파라미터 조정

## 4. 전체 동작 요약
1. 각 카드의 `.content`에 커서와 함께 한 글자씩 코드가 표시됨
2. 기본 텍스트 커서는 완전히 숨겨지고, CSS 커서만 보임
3. 코드와 커서의 동기화가 자연스럽게 이루어짐

---

담당자: (인수인계자 이름)
작성일: 2025-05-31
