## 15장 엔터티와 인코딩

- HTTP는 메시지가 올바르게 수송, 식별, 추출, 처리되는 것을 보장하기위해 콘텐츠를 나르기 위한 잘 라벨링된 엔터티를 사용한다.

## 1. 메시지는 컨테이너, 엔터티는 화물

- HTTP 메시지를 인터넷 운송 시스템의 컨테이너라고 생각한다면, HTTP 엔터티는 메시지의 실질적 화물이다.
- Content-Type, Content-Length, Content-Language, Content-Encoding, Content-Location, Content-Range, Content-MD5, Last-Modified, Expires, Allow, Etag, Cache-Control

### 1.1 엔터티 본문

- 엔터티 본문은 가공되지 않은 데이터만을 담고 있다. 가공되지 않은 데이터이기 때문에 엔터티 헤더는 데이터의 의미에 대해 설명할 필요가 있다.
- 엔터티 본문은 헤더 필드의 끝을 의미하는 빈 CRLF 줄 바로 다음부터 시작한다.

## 2. Content-Length: 엔터티의 길이

- Content-Length 헤더는 메시지의 엔터티 본문의 크기를 바이트 단위로 나타낸다. 메시지를 청크 인코딩으로 전송하지 않는 이상, 엔터티 본문을 포함한 메시지에서는 필수적으로 있어야 한다.
- 서버 충돌로 인해 메시지가 잘렸는지 감시하고자 할 때와 지속 커넥션을 공유하는 메시지를 올바르게 분할하고자 할 때 필요

### 2.1 잘림 검출

- Content-Length가 없으면 클라이언트는 커넥션이 정상적으로 닫힌 것인지 서버에 충돌이 발생한 것인지 구분하지 못한다. 따라서 메시지 잘림을 검출하기 위해 Content-Length를 필요로 한다.
- 메시지 잘림은 캐싱 프락시 서버에서 특히 취약하다. 캐시가 잘린 메시지를 수신했으나 잘렸다는 것을 인식하지 못한 경우, 캐시는 결함이 있는 콘텐츠를 저장하고 계속해서 제공하게 되므로 이를 방지하기 위해 캐싱 프락시 서버는 Content-Length 헤더를 갖고 있지 않은 HTTP 본문은 보통 캐시하지 않는다.

### 2.2 잘못된 Content-Length

- Content-Length가 잘못된 값을 담지 않도록 HTTP/1.1 사용자 에이전트는 잘못된 길이를 인지했을 때 사용자에게 알려주게 되어 있다.

### 2.3 Content-Length와 지속 커넥션(Persistent Connection)

- 지속된 커넥션을 위하여 Content-Length는 필수이고 지속된 커넥션 하에 커넥션이 닫힌 위치를 근거로 메시지의 끝을 인식하는 것은 불가능하다. 또한 HTTP 애플리케이션은 엔터니 본문과 다음 메시지의 구분을 하지 못한다.
- 청크 인코딩을 사용한 경우 엔터티 전체 크기를 알 수 없더라도 전송할 수 있다.

### 2.4 콘텐츠 인코딩

- HTTP 보안을 강화하거나 압축을 통해 공간을 절약 할 수 있도록, 엔터티 본문을 인코딩할 수 있게 한다. 콘텐츠가 인코딩이 된 경우, 원본 길이가 아닌 인코딩 후의 길이로 본문의 길이를 바이트 단위로 정의한다.

### 2.5 엔터티 본문 길이 판별을 위한 규칙

1. 본문을 갖는 것이 허용되지 않는 특정 타입의 HTTP메시지에서는 본문계산을 위한 Content-Length 헤더가 무시된다. 예를들어 HEAD 응답의 경우 본문을 갖지 않기에 반드시 헤더 이후의 첫번째 빈 줄에서 끝나야 한다.
2. Transfer-Encoding 헤더를 포함고 있으면 커넥션이 닫혀서 먼저 끝나지 않는 이상 엔터티는 '0 바이트 청크'라 부르는 특별한 패턴으로 끝나야 한다.
3. Transfer-Encoding 헤더 필드를 갖고 있는 메시지를 받았다면 반드시 Content-Length 헤더를 무시해야 하는데, 전송 인코딩이 엔터티 본문을 표현하고 전송하는 방식을 바꿀 것이기 때문이다.
4. 'multipart/byteranges' 미디어 타입을 사용하는 멀티파트 유형은 스스로의 크기를 결정할 수 있는 유일한 엔터티 본문 유형이다.
5. 위 규칙에 해당되지 않으면 엔터티는 커넥션이 닫힐 때 끝나며 서버만이 커넥션을 닫을 수 있다.
6. HTTP/1.0 애플리케이션과 호환을 위해 엔터티 본문을 갖고 있는 HTTP/1.1 요청은 반드시 유효한 Content-Length 헤더도 갖고 있어야 한다.

## 3. 엔터티 요약

- 엔터티 본문 데이터에 대한 의도하지 않은 변경을 감지하기 위해, 최초 엔터티 생성시 송신자는 데이터에 대한 체크섬을 생성할 수 있고 수신자는 그 체크섬으로 기본적인 검사를 할 수 있다.
- 메시지의 무결성을 검증하려는 클라이언트는 먼저 전송 인코딩을 디코딩 한 뒤 그 디코딩 된 엔터티 본문에 대한 MD5 계산을 하여 검증을 한다.
- MD5는 문서의 위치를 알아내고 콘텐츠의 중복 저장을 방지하기 위한 해시 테이블의 키로 이용될 수 있다.

## 4. 미디어 타입과 차셋(Charset)

- Content-Type 헤더 필드는 엔터티 본문의 MIME 타입을 기술한다. 주 미디어 타입과 빗금, 구체적인 미디어 타입을 서술하는 부 타입으로 구성된다.
- 엔터티가 콘텐츠 인코딩을 거친 경우에도 Content-Type 헤더는 여전히 인코딩 전의 엔터티 본문 유형을 명시할 것이다.

### 4.1 텍스트 매체를 위한 문자 인코딩

- 엔터티의 비트 집합을 텍스트 파일의 글자들로 변환하기 위한 'charset' 매개변수는 Content-Type 헤더가 내용 유형을 더 자세히 지정하기 위한 대표적인 예이다.

### 4.2 멀티파트 미디어 타입

- MIME 멀티파트 이메일 메시지는 서로 붙어있는 여러 개의 메시지를 포함하며, 하나의 복합 메시지로 보내진다. 각 구성요소는 자신을 서술하는 헤더를 포함하고 문자열 하나로 서로의 경계가 식별된다.

### 4.3 멀티파트 폼 제출

- HTTP 폼 제출 시, 텍스트 필드와 업로드 될 객체는 각각 멀티파트 본문을 구성하는 하나의 파트가 되어 보내진다.
- 멀티파트 본문은 여러 다른 종류/길이의 값으로 채워진 폼을 허용하는데, 이러한 경우 `Content-type: multipart/mixed` 또는 `Content-type: multipart/form-data` 헤더에 멀티파트 본문을 함께 보낸다.
  `Content-type: multipart/form-data; boundary=[abcdefghijk]`
- boundary는 본문의 서로 다른 부분을 구분하기 위한 구분자로 쓰인다.

### 4.4 멀티파트 범위 응답

- 범위 요청에 대한 HTTP 응답 또한 멀티파트가 될 수도 있다. 그런 응답은 Content-type: multipart/byteranges 헤더 및 각각 다른 범위를 담고 있는 멀티파트 본문이 함께 온다.
