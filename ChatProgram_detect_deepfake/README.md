# 캡스톤디자인 및 AI 해커톤 - 채팅 애플리케이션

이 프로젝트는 **카카오톡 스타일 UI**를 갖춘 채팅 애플리케이션입니다. 사용자는 실시간으로 메시지와 이미지를 주고받을 수 있으며, 서버는 **FastAPI**로 구현되었습니다. WebSocket을 활용하여 실시간 통신 기능을 제공합니다.

|채팅 애플리케이션 - 메인 화면|**캡스톤디자인 및 AI 해커톤** - 상장|
|:---:|:---:|
|<img src="https://github.com/user-attachments/assets/04de8912-bce7-4d54-b2fc-f4fbdfbc361b" width="900"/>|<img src="https://github.com/user-attachments/assets/63114c0e-230a-487e-99ee-4e235207ee66" width="500"/>|
|메인 채팅 화면.|최우수상|

---

## 주요 기능
1. **카카오톡 스타일 UI**
   - 직관적이고 깔끔한 사용자 인터페이스 제공.
   - 메시지와 이미지를 구분하여 표현.

2. **WebSocket 실시간 통신**
   - 사용자 간 실시간 메시지 및 이미지 전송.

3. **이미지 필터링**
   - 부적절하거나 딥페이크 이미지를 탐지하여 경고 표시.

4. **FastAPI 기반 서버**
   - 클라이언트와의 데이터 송수신 처리.

---

## 사용 기술
### 프론트엔드
- **HTML/CSS/JavaScript**: UI와 동작 로직 구현.
- **FontAwesome**: 아이콘 사용.
- **SweetAlert2**: 사용자 알림 창 표시.

### 백엔드
- **FastAPI**: 경량화된 서버 프레임워크.
- **WebSocket**: 실시간 데이터 통신.

---

## 프로젝트 구조

```
project/
├── css/                         # UI 스타일 정의 폴더
├── images/                      # 이미지 리소스 폴더
├── test_data/                   # 테스트 데이터 폴더
└── index.html                   # 메인 HTML 파일
```

## UI 설명

### 1. **메인 화면**
HTML과 CSS를 이용해 기본적인 카카오톡 채팅방과 유사한 화면을 설계하였습니다.

```html
<header class="top-header chat-header">
  <div class="header__top">
    <div class="header__column">
      <i class="fa fa-plane"></i>
      <i class="fa fa-wifi"></i>
    </div>
    <div class="header__column">
      <span class="header__time">18:38</span>
    </div>
    <div class="header__column">
      <i class="fa fa-moon-o"></i>
      <i class="fa fa-bluetooth-b"></i>
      <span class="header__battery">66% <i class="fa fa-battery-full"></i></span>
    </div>
  </div>
  <div class="header__bottom">
    <div class="header__column">
      <a href="chats.html">
        <i class="fa fa-chevron-left fa-lg"></i>
      </a>
    </div>
    <div class="header__column">
      <span class="header__text">Chat Room</span>
    </div>
    <div class="header__column">
      <i class="fa fa-search"></i>
      <i class="fa fa-bars"></i>
    </div>
  </div>
</header>
```

### 2. **실시간 채팅과 서버 통신**
WebSocket을 사용해 서버와 통신하며, 메시지를 송수신하거나 이미지를 업로드하는 기능을 구현하였습니다.

#### 사용자 이름 요청
사용자의 이름을 입력받아 WebSocket 연결을 초기화합니다.

```javascript
async function requestUserName() {
  const { value: name } = await Swal.fire({
    icon: 'question',
    title: '이름을 입력하세요',
    input: 'text',
    allowOutsideClick: false,
    allowEscapeKey: false,
    showCancelButton: false,
    confirmButtonText: '확인',
    inputValidator: (value) => {
      if (!value) {
        return '이름을 입력해야 합니다!';
      }
    }
  });

  myName = name;
  initializeChat(myName);
}
```

#### WebSocket 연결 초기화
사용자 이름을 포함하여 서버와 WebSocket 연결을 생성합니다.

```javascript
function initializeChat(user_id) {
  ws = new WebSocket('ws://192.168.216.47/api/chat?user_id=' + user_id);

  ws.onopen = async () => {
    console.log("WebSocket 연결 성공!");
    await Swal.fire({
      icon: 'success',
      title: '환영합니다, ' + myName + '님!',
      text: '채팅에 입장하셨습니다.',
    });
    document.getElementById('message-input').focus();
  };
}
```

---

## 핵심 기능 설명

### 1. **딥페이크 탐지 및 이미지 검증**
서버에서 받은 이미지가 딥페이크로 판단될 경우, 사용자에게 경고 메시지를 표시하고 삭제를 확인받습니다.

```javascript
if (type === 'deepfake') {
  const confirmation = await Swal.fire({
    icon: 'warning',
    title: '딥페이크 이미지 삭제',
    html: '딥페이크 이미지가 탐지되었습니다.<br>해당 이미지가 불법 딥페이크물이 아님을<br>확신하며 이에 대한 법적 책임을 진다는 것을<br>확인하겠습니까?',
    showCancelButton: true,
    confirmButtonText: '확인',
    cancelButtonText: '취소',
  });
}
```

### 2. **메시지와 이미지 렌더링**
텍스트 메시지와 이미지를 분리하여 렌더링하는 코드입니다.

```javascript
if (/\.(jpeg|jpg|gif|png|webp|bmp)$/.test(message)) {
  messageElement.innerHTML = `
    <span class="chat__message-time">${time}</span>
    <img src="${message}" alt="Image" class="chat__image" style="width: 40vw;">
  `;
} else {
  messageElement.innerHTML = `
    <span class="chat__message-time">${time}</span>
    <span class="chat__message-body">${message}</span>
  `;
}
```

### 3. **이미지 업로드**
이미지를 선택하여 서버에 업로드하고, 성공적으로 업로드된 이미지를 채팅창에 추가합니다.

```javascript
async function sendImages() {
  const imageInput = document.getElementById('image-input');
  const files = imageInput.files;

  if (files.length > 0) {
    for (let i = 0; i < files.length; i++) {
      const file = files[i];
      const formData = new FormData();
      formData.append("files", file);

      try {
        const response = await fetch("http://192.168.216.47:8000/api/upload-image/?user_id=" + encodeURIComponent(myName), {
          method: "POST",
          body: formData
        });

        if (!response.ok) {
          Swal.fire({
            icon: 'warning',
            title: '이미지 업로드 실패!',
            text: '이미지 업로드에 실패했습니다. 다시 시도해주세요.',
          });
        }
      } catch (error) {
        Swal.fire({
          icon: 'warning',
          title: '에러 발생!',
          text: '이미지 업로드 중 에러가 발생했습니다. 다시 시도해주세요.',
        });
      }
    }
  }
}
```

---

## 사용한 기술 스택

- **프론트엔드**: HTML, CSS, JavaScript
- **라이브러리**: Font Awesome, SweetAlert2
- **백엔드**: FastAPI
- **통신 프로토콜**: WebSocket, RESTful API

---

## 결과 및 기대 효과

- 사용자 친화적인 UI와 실시간 채팅을 통해 자연스러운 사용자 경험을 제공.
- 이미지 검증 기능으로 사용자 보안과 데이터 무결성을 확보.
- 해커톤에서 팀 협업을 통해 성공적인 프로토타입 완성.
- 해커톤에서 **최우수상** 수상.

---
