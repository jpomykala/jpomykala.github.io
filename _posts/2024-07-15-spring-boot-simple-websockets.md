---
layout: post
title: "🔌 Spring Boot and WebSockets"
tags: [ 'java', 'spring boot', 'frontend', 'websocket' ]
---

[Recently, I asked Spring Boot developers on Reddit](https://www.reddit.com/r/SpringBoot/comments/1dzpfa7/comment/lcheh4w/)
if anyone uses plain WebSockets in their applications without [STOMP](https://stomp-js.github.io)
and/or [SockJS](https://github.com/sockjs/sockjs-client). I was curious about the answers
because in all the resources I found on the internet, people use STOMP and SockJS to implement WebSockets in Spring Boot
apps, but those technologies are not so popular on the frontend side. I started wondering, do I really need them? And
why is there no simple example of using WebSockets in Spring Boot without STOMP and SockJS? I guess the reason is that
Spring Boot is mostly used in enterprise applications where STOMP and SockJS are more suitable or documentation is just
outdated in this matter or configuration of WebSockets without these technologies is that easy that it doesn't require
any explanation. However, I decided to write this post to show you how to configure WebSockets in Spring Boot without
it.

> I won't cover how to send messages from the client to the server, as it didn't fit into the context of the project
> presented in this post.

## Prerequisites

To follow this tutorial, it's good to know the basics of Java, Spring Boot, and Spring Security, and have a general
understanding of WebSockets and ReactJS, as we will use ReactJS to communicate with the server using WebSockets.

## Why WebSockets and not Server-Sent Events

A few months back, I started working on a workflow automation engine at [SimpleLocalize](https://simplelocalize.io). The
idea was very simple: I
wanted to create a feature where you can create workflows for translations. For example, when you modify a source
translation, it would auto-translate other translations for all other languages.

I knew that building an automation engine handling all the rules and logic would be a challenge, but I also knew that
some automations created by users wouldn't run fast enough to return results in a few seconds. I needed a way to notify
users browsing the content that the translations were being updated.

![Auto-translation workflow in SimpleLocalize](/assets/2024-07-15/translations-automation.mp4)

After a few hours of research, I decided to
use [server-side events (SSE)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) to send live updates
to the client. SSE works
similarly to a regular HTTP request but allows the server to send updates to the client without the client having to
send a request. I didn't need duplex communication; it was enough to send updates from the server to the client.

Unfortunately, the solution didn't work well with the reverse proxy we used, which struggled with many long-running
connections. At some point, when specific conditions were met, the frontend received malformed JSON responses,
like `{ message: 'OK' } { mes` (yiiikes!). The short-term solution was to set an SSE timeout to 15 seconds and reconnect
after that time, but it wasn't a good solution.

Shortly after that, I started looking into WebSockets. WebSockets were a buzzword in the web development world a few
years back, and they are still very popular and widely used. They are well-supported by all modern browsers and
infrastructure middlewares. I hoped that WebSockets would solve my problem with live updates, and they did!

Currently, we have over 100 live users simultaneously connected to the server without any hiccups, and we are gradually
increasing
this number by switching from SSE to WebSockets.

## Why I don't want to use STOMP and SockJS

[Most of the tutorials (including the Spring Framework docs)](https://spring.io/guides/gs/messaging-stomp-websocket) on
the internet use STOMP and SockJS to implement WebSockets in Spring Boot applications. It's worth explaining what STOMP
and SockJS are:

- **STOMP** provides a higher-level API to work with WebSockets, such as subscribing to topics, sending messages, etc.
  However, it adds complexity to the application, which I don't need, at least for now.

- **SockJS** provides fallback support for browsers that don't support WebSockets. Since all modern browsers support
  WebSockets, fallback support is generally unnecessary. While SockJS can be useful if WebSockets are blocked in a
  user's network, such cases are rare these days, and I haven't encountered any so far.

I've decided to use just the Spring Boot WebSocket dependency and the native WebSocket API provided by the browser to
avoid dependence on any external libraries and Spring Boot magic.

## Authentication in WebSockets

In a real-world application, you would need to authenticate and authorize users before they can connect to the
WebSocket. You can use Spring Security to handle this process. Nearly seven years ago, I wrote a post on how to use
Spring Security with WebSockets; you can check it out [here](/2017/12/19/spring-react-websockets-with-auth). However,
after gaining more experience, I'd prefer to use less Spring magic and keep things simple.

I've decided to use a straightforward ticket-based authentication mechanism. The server generates a ticket for the user
when they connect to the WebSocket, and the client uses this ticket to authenticate with the server.

## How to configure WebSockets in Spring Boot

Let's dive into the code and see how we can configure WebSockets in Spring Boot 3.
I've started by creating a service class called `WssConnectorService` that extends `TextWebSocketHandler` class provided
by Spring Boot. To keep it easy to read for the purpose of this tutorial, the `WssConnectorService` realizes a few
functionalities:

- It keeps track of all the connected users and their sessions.
- It creates a ticket for a user when they connect to the WebSocket.
- It removes unused tickets every 5 minutes.
- It sends a message to a user.
- It pings all the connected users every 10 seconds to check if the connection is still alive.

All tickets and sessions are stored in-memory, but in a real-world application, you could store them in a database or a
cache like Redis.

```java

@Component
public class WssConnectorService extends TextWebSocketHandler
{
  private final ObjectMapper objectMapper;
  private final List<WssUser> wssUsers = new CopyOnWriteArrayList<>();
  private final Map<String, WssTicketDetails> tickets = new ConcurrentHashMap<>();

  // Connection closed by client
  @Override
  public void afterConnectionClosed(@NotNull WebSocketSession session, @NotNull CloseStatus status)
  {
    wssUsers.removeIf(wssUser -> wssUser.session().equals(session));
  }

  // Connection established by client
  @Override
  public void afterConnectionEstablished(@NotNull WebSocketSession session)
  {
    // Example uri: /wss?ticket=123
    String ticket = Optional.ofNullable(session.getUri())
            .map(URI::toString)
            .map(UriComponentsBuilder::fromUriString)
            .map(builder -> builder.build().getQueryParams().getFirst("ticket"))
            .filter(StringUtils::hasText)
            .orElseThrow(() -> new BadRequestException("Ticket not found in query"));

    WssTicketDetails ticketDetails = tickets.get(ticket);
    WssUser wssUser = WssUser.builder()
            .userId(ticketDetails.userId())
            .projectToken(ticketDetails.project().projectToken())
            .session(session)
            .build();
    tickets.remove(ticket);
    wssUsers.add(wssUser);
  }

  public void sendEvent(WssUser wssUser, LiveEvent event)
  {
    try
    {
      String json = objectMapper.writeValueAsString(event);
      WebSocketSession session = wssUser.session();
      session.sendMessage(new TextMessage(json));
    } catch (
            Exception e) // you can catch more specific exception here and handle it in a different ways, e.g.: when the session is closed unexpectedly
    {
      wssUsers.remove(wssUser);
    }
  }

  public String createTicketForUser()
  {
    User user = // e.g.: get current user
            Project project = // e.g.: get project for the user
          WssTicketDetails ticketDetails = WssTicketDetails.builder()
          .project(project)
          .userId(userId)
          .createdAt(Instant.now())
          .build();
    String ticket = // create a unique ticket, e.g. UUID, SecureRandom, or JWT token with expiration time
            tickets.put(ticket, ticketDetails);
    return ticket;
  }

  @Scheduled(fixedDelay = 5, initialDelay = 5, timeUnit = TimeUnit.MINUTES)
  public void removeUnusedTickets()
  {
    tickets.entrySet().removeIf(this::isTicketExpired);
  }

  private boolean isTicketExpired(Map.Entry<String, WssTicketDetails> entry)
  {
    return entry.getValue().createdAt().toEpochMilli() < System.currentTimeMillis() - TimeUnit.MINUTES.toMillis(5);
  }

  @Scheduled(fixedDelay = 10, initialDelay = 10, timeUnit = TimeUnit.SECONDS)
  public void pingSessions()
  {
    for (WssUser wssUser : wssUsers)
    {
      sendEvent(wssUser, LivePingEvent.ping());
    }
  }
}
```

`LiveEvent` class is a simple interface that represents an event that can be sent to the client. We have two types of
events: `PING` and `CONTENT_CHANGE`. Here is the implementation of the `LivePingEvent` class:

```java
public interface LiveEvent
{
  LiveEventType type();
}

public enum LiveEventType
{
  PING,
  CONTENT_CHANGE
}

@Builder
public record LivePingEvent(LiveEventType type, String message) implements LiveEvent
{
  public static LivePingEvent ping()
  {
    return LivePingEvent.builder().type(LiveEventType.PING).message("ping").build();
  }

  @Override
  public LiveEventType getType()
  {
    return type;
  }
}
```

Isn't it simple? We have a service class that extends `TextWebSocketHandler` and keeps track of all the connected users
and their sessions. Here is the controller class that we will use to create a ticket for a user when they connect to the
WebSocket, and send the ticket to the client:

```java

@RestController
public class WssConnectorController
{
  private final WssConnectorService wssConnectorService;

  @GetMapping("/wss/ticket")
  public WssTicketResponse getTicket()
  {
    String ticket = wssConnectorService.createTicket();
    return WssTicketResponse.builder().ticket(ticket).build();
  }
}
```

Solid and dead simple, not very scalable as we used in-memory storage to keep everything, but it's a good starting
point.

Here is the configuration class that we will use to configure WebSockets in our Spring Boot 3 application. I've
created a class called `WssConfig` that implements the `WebSocketConfigurer` interface, and added `@EnableWebSocket`
annotation to enable WebSockets in our application.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WssConfig implements WebSocketConfigurer
{

  public WssConfig(WssConnectorService wssConnectorService)
  {
    this.wssConnectorService = wssConnectorService;
  }

  @Override
  public void registerWebSocketHandlers(WebSocketHandlerRegistry registry)
  {
    registry.addHandler(wssConnectorService, "/wss").setAllowedOrigins("*");
  }
}
```

> Please also remember that you have to adjust your Spring Security configuration to allow access to the `/wss`
> endpoint.

## How to connect to the WebSocket from the client

Now, that we have configured WebSockets in our Spring Boot 3 application, let's see how we can connect to the WebSocket
from the client. This part is a most satisfying one, as we will use a native APIs provided by the browser to connect to
the WebSocket.

```typescript
useEffect(() => {
  let wss: WebSocket;

  const connectToServer = async () => {
    const openWebSocket = (ticket: string) => {
      wss = new WebSocket("ws://localhost:8080/wss?ticket=" + ticket);
      wss.onopen = () => console.log("Connection opened");
      wss.onmessage = (message: any) => {
        const response = JSON.parse(message?.data);
        if (response.type === SseEventType.CONTENT_CHANGE) {
          // update the UI with the new content
        }
      };
      wss.onclose = () => console.log("Connection closed"); // you can try to reconnect here
    };
    fetch("/wss/ticket")
      .then((response) => response.json())
      .then((data) => data.ticket)
      .then((ticket) => openWebSocket(ticket))
      .catch((error) => console.error("Error getting ticket", error));
  };

  connectToServer().catch((error) => console.error("Error while initial connection", error));
  return () => {
    console.log("Gracefully closing the connection");
    wss.close();
  };
}, []);
```

In this code snippet, we are using the `WebSocket` and `fetch` APIs provided by the browser to connect to the WebSocket
from the client. We are first fetching a ticket from the server using the `/wss/ticket` endpoint, and then we are
connecting to the WebSocket using the ticket.

We are also handling the `onopen` and `onmessage` events of the WebSocket to log the connection status and process the
messages received from the server.

Once you connect to the WebSocket, you can also periodically check if the connection is closed and reconnect if
necessary, e.g.:

```typescript
setInterval(() => {
  if (wss?.readyState === WebSocket.CLOSED) {
    console.log("Connection closed, reconnecting...");
    connectToServer().catch((error) => console.error("Error while reconnecting", error));
  }
}, 2000);

// cleanup the interval when the component is unmounted
return () => clearInterval(intervalId);
```

## Conclusion

That's all! In this post, we have set up the most basic implementation of WebSockets with Spring Boot 3 to send live
updates from the server to the client.

The biggest advantage of this approach (and disadvantage at the same time) is that it's very simple and doesn't require
any additional libraries like STOMP or SockJS. This simplicity makes the setup lightweight and easy to understand,
reducing the overhead of learning and maintaining extra dependencies. However, it also means that you miss out on the
additional features and abstractions provided by these libraries, such as automatic fallback mechanisms and higher-level
messaging protocols.

If your application requirements are straightforward and you prefer to minimize dependencies, this approach is ideal.
However, if you need more advanced features like message brokering, topic subscriptions, or support for older browsers
and network environments, you might want to consider using mentioned technologies.

Below is an example of how the auto-translation feature works using a Context Menu in [SimpleLocalize](https://simplelocalize.io):

![Auto-translation via Context Menu in SimpleLocalize](/assets/2024-07-15/context-menu-auto-translation.mp4)
