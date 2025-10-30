---
title: 'WebFlux와 WebSocket을 이용한 채팅 구현'
description: 지난주에 공부했던 WebFlux와 WebSocket을 이용하여 간단하게 채팅을 구현해보려고 한다. 메세지를 비동기 처리하며 클라이언트와 서버 간에 지속적으로 연결을 유지하는데 있어 mvc가 아닌 WebFlux를 통해 구현해보려 한다. 
categories:
 - Java
tags:
 - Java
 - Spring
---

## 개요
지난주에 공부했던 WebFlux를 어디에 응용해보면 좋을지 고민하던중,
**데브나인 팀장 웨이드**가 팀 소개 페이지에 간단하게 채팅같은걸 붙여보는건 어떠냐고 제안하며 시작되었다.

![Desktop Preview](/assets/images/post/websocket/dev9_wade.jpeg)

실시간 채팅같은 경우는 한이음 공모전에서 **웹 소켓**을 이용해서 구현해본 경험이 있고, 사용자간 빠르게 메세지를 주고받는데 있어 비동기 처리 방식인 **WebFlux**의 처리 방식에도 적합할거 같아 진행해보려고 한다.

## 웹 소켓(WebSocket)?
웹 소켓(WebSocket)은 클라이언트와 서버 간의 양방향 통신을 가능하게 하는 프로토콜인데 이는 실시간 데이터를 주고받는 데 매우 유용하다.  
왜냐하면 웹 소켓은 HTTP와 달리 지속적인 연결을 유지하며, 클라이언트와 서버 간의 실시간 데이터 전송을 가능하게 하기 때문이다.  
  
**HTTP 통신**은 한번 요청 후 응답하면 연결이 끝나기에, 여러 번 요청하면 여러 번 연결이 맺어지는데, **웹 소켓**은 한번 요청 후 연결을 끊기 전까지 계속 연결을 유지하기 때문에, 매번 요청마다 연결을 시도할 비용을 절약할 수 있어 실시간 채팅을 구현하는데 있어 적합하다.

## 구현
### gradle websocket 의존성 추가
웹 소켓 통신에 필요한 라이브러리와 설정을 위한 의존성을 추가해준다.
```gradle
implementation 'org.springframework.boot:spring-boot-starter-websocket' // WebSocket 지원
```

### 웹 소켓 handler 파일 작성
```java
public class ChatWebSocketHandler extends TextWebSocketHandler {

    // 현재 연결된 세션 관리 (간단한 메모리 기반)
    private final Map<String, List<WebSocketSession>> activeSessions = new HashMap<>();
    private Map<String, String> roomSessions = new HashMap<>(); // roomId -> sender

    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        String message = "{\"type\": \"INFO\", \"content\": \"Connected to chat\"}"; // 최초 연결시 세션에 전달할 메세지!

        session.sendMessage(new TextMessage(message));
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        // 클라이언트로부터 받은 메시지 처리
        String payload = message.getPayload();
        Map<String, Object> messageData = new ObjectMapper().readValue(payload, Map.class);

        // 메시지 타입 확인
        String type = (String) messageData.get("type");

        if ("JOIN".equals(type)) {
            // JOIN 메시지 처리
            String memberId = (String) messageData.get("memberId");  // 클라이언트에서 보낸 사용자ID
            String roomId = (String) messageData.get("roomId");  // 클라이언트에서 보낸 채팅방ID

            // 세션에 sender와 roomId를 저장
            session.getAttributes().put("memberId", memberId);
            session.getAttributes().put("roomId", roomId);

            // 입장시!(최초 연결시) 채팅방 세션에 사용자의 세션 추가
            activeSessions.computeIfAbsent(roomId, k -> new ArrayList<>()).add(session);

            // 채팅방에 입장한 것을 알리는 메시지 전송
            sendToAll(new ChatDto(memberId, ChatDto.MessageType.JOIN, memberId + " 님이 입장했습니다.", roomId));

            roomSessions.put(roomId, memberId); // 채팅방과 유저 매핑 저장
        } else {
            // 다른 타입의 메시지 처리 (MESSAGE: 채팅 내용 또는 LEAVE : 채팅방을 나갔을떄)
            String sender = (String) session.getAttributes().get("sender");
            String roomId = (String) session.getAttributes().get("roomId");

            if ("MESSAGE".equals(type)) {
                // MESSAGE 처리 로직
                sendToAll(new ChatDto(sender, ChatDto.MessageType.MESSAGE, messageData.get("content").toString(), roomId));
            } else if ("LEAVE".equals(type)) {
                // LEAVE 처리 로직
                sendToAll(new ChatDto(sender, ChatDto.MessageType.LEAVE, messageData.get("content").toString(), roomId));
            }
        }
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        // 클라이언트가 연결을 종료했을 때 처리
        String roomId = (String) session.getAttributes().get("roomId");

        // 채팅방에서 해당 사용자의 세션 제거
        List<WebSocketSession> roomSessions = activeSessions.get(roomId);
        if (roomSessions != null) {
            roomSessions.remove(session);
        }
    }

    private void sendToAll(ChatDto message) throws IOException {
        // 세션에 있는 모든 사용자에게 메시지를 전송
        ObjectMapper objectMapper = new ObjectMapper();
        String messageJson = objectMapper.writeValueAsString(message);  // Chat 객체를 JSON 문자열로 변환

        List<WebSocketSession> roomSessions = activeSessions.get(message.getRoomId());
        if (roomSessions != null) {
            for (WebSocketSession session : roomSessions) {
                session.sendMessage(new TextMessage(messageJson));
            }
        }
    }
}
```

`session`: 현재 연결된 웹소켓 세션들을 담는 Set

메모리 상에 현재 연결된 웹소켓을 담고, 세션이 추가(afterConnectionEstablished())되거나 종료(afterConnectionClosed())되면 반영한다.
`activeSessions`: 채팅방 당 연결된 세션을 담은 Map 형태로 세션을 저장한다.

채팅 메세지를 보낼 채팅방을 찾고, 해당 채팅방에 속한 세션(사용자들)에게 메세지를 전송한다.
`handleTextMessage()`: 웹소켓 통신 시 메세지 전송을 다루는 함수

TextWebSocketHandler 클래스의 handleTextMessage() 메소드를 Override하여 구현

웹소켓 통신 메세지를 TextMessage로 받고, 이를 mapper로 파싱하여 ChatDto 클래스로 변환하여 메세지 전송

### DTO 클래스 작성
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ChatDto {
    private String sender;
    private MessageType type;
    private String content;
    private String roomId;

    public enum MessageType {
        MESSAGE,  // 채팅 메시지
        JOIN,  // 채팅방 참여
        LEAVE  // 채팅방 퇴장
    }
}
```

**MessageType**에 따라 채팅방에 표시할 내용을 구분한다.  
`MESSAGE` : 채팅 메세지  
`JOIN` : 채팅방 참여시  
`LEAVE` : 채팅방 퇴장시  

채팅방 번호를 가져오고, 채팅방에 대한 세션이 없으면 만들어준다.  
ChatDto의 타입이 JOIN이면 채팅 세션에 웹소켓 클라이언트의 세션을 넣어준다.  
마지막에 sendToAll(new ChatDto(memberId, ChatDto.MessageType.JOIN, memberId + " 님이 입장했습니다.", roomId)) 호출하여 해당 채팅방에 있는 모든 사용자에게 메세지 전송


### 웹 소켓 config 파일 작성
```java
@Configuration
@EnableWebSocket
public class ChatConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatWebSocketHandler(), "/chat")
                .setAllowedOrigins("*"); // CORS 허용 (중요)
    }

    @Bean
    public WebSocketHandler chatWebSocketHandler() {
        return new ChatWebSocketHandler();
    }
}
```

다른 도메인에서 WebSocket 연결을 시도하면 브라우저에서 CORS 오류가 발생하기 때문에 `setAllowedOrigins("*")` CORS를 반드시 허용해주도록 하자 (한이음 공모전에서 구현할 당시 별거 아닌걸로 엄청 해맸던 기억이...)

### 실행
![Desktop Preview](/assets/images/post/websocket/websocket_1.png)

작성된 서버 코들를 실행시켜서 간단히 확인해보면

![Desktop Preview](/assets/images/post/websocket/websocket_2.png)

웹 소켓에 연결되어 채팅에 참여했다는 메세지로 응답이 오는것을 확인할 수 있다.

다음 포스팅에서 간단하게 뷰단을 작성해서 실제 채팅과 비슷하게 구현하고 데이터베이스와 연결하여 채팅방과 채팅 메세지 기록을 저장 및 불러오는 기능을 추가하도록 하겠다.