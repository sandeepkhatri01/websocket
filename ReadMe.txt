This article is about WebSockets with Jetty. 
In this example we will give brief information on WebSockets and show how to implement WebSocket Servers and Clients using Jetty WebSocket APIs. 
In addition to these, an example HTML+JavaScript client interacting with the Server through WebSockets will be provided.

WebSockets
WebSocket is a standard protocol facilitating full-duplex communication over a single TCP socket. The protocol is designed primarily for Web Browsers and Web Servers however it can be applied to other server-client integration cases.
The concept of WebSockets has been emerged due to the limitations of HTTP. HTTP is a pull based (request-response) protocol; which means that server cannot directly push messages to the client. The alternatives aiming to tackle this problem (TCP sockets, Comet, Long-polling etc.) provide workaround solutions causing side effects of their own.
WebSocket protocol is based on existing web application technologies. A WebSocket connection utilizes the same port with the application server(80 or 8080 for instance), hence it is less likely to be blocked by firewalls. WebSocket connection between client and server is established by a handshake through an HTTP upgrade request. After successful handshake, the protocol is switched from HTTP to WebSocket.
WebSocket defines two prefixes for the server URI.

ws: for unsecure connections
Example: ws://example.codegeeks.com/websocketExample

wss: for secure connections
Example: wss://example.codegeeks.com/websocketExample

For the browser based applications, WebSocket protocol support has to be provided by the browser. Currently WebSockets are supported by almost all modern browsers including Chrome, Firefox, Internet Explorer and Safari.
Today WebSockets are widely used in the cases that servers need to push data to the web clients (online gaming, chat).

Jetty WebSocket APIs
WebSocket technology needs to be supported not only on the browser but also on the server side. Different platforms do have their own implementation for WebSockets both for server and client roles. Jetty is one of the platforms providing WebSocket API for both server and clients sides.
Jetty implements two alternative APIs for WebSocket development:

First of them is the JSR-356 compliant one. JSR-356 is the Java API for WebSocket specification which is included in Java EE 7. The spec defines an annotation based API as well as an listener interface based one.
The other alternative is Jetty’s own implementation: Jetty WebSocket API. This API had emerged before JSR-356 was released. The programming model of Jetty WebSocket API is very similar to JSR-356 based one. It also provides similar annotations and listener interfaces.
In the following subsections we expand on both alternatives.

Alternative 1: Jetty JSR-356 Implementation
JSR-356 has two different approaches for WebSocket implementation: One is Annotations based whereas the other is Interface based. In the Annotations approach, you have to decorate your POJOs with relevant annotations of the API. In the Interface approach, you need to implement the WebSocket Endpoint interface.
Both approaches are similar. The annotations in the first approach match to the methods to be implemented in the Interface approach. Here we explain only the annotations.
@ServerEndpoint:

ServerEndpoint is used to annotate a POJO classes as server side WebSockets. The value of the annotation determines the URL path of the WebSocket(similar to the servlet mappings in Java web applications):
An example can be seen below:

@ServerEndpoint(value = "/example")
public class MySocket{
}

@ClientEndpoint:

ClientEndpoint is used to annotate a POJO classes as client side WebSockets.
@ClientEndpoint
public class MySocket{
}


@OnOpen:

OnOpen annotates the method that handles the event when a connection is established. JSR-356 does not mandate anything on naming of annotated methods, so we can name our methods as we like.
@OnOpen
public void onSessionOpened(Session session){
}

Session is the class that encapsulates the Socket connection session.
@OnMessage:

OnMessage is used to annotate the method that handles the incoming messages.
An example is below:
@OnMesssage
public String onMessageReceived(String message, Session session){
}

@OnClose:
We can mark the method that handles the event fired when the socket connection is closed with OnClose annotation. An example usage is below:
@OnClose
public void onClose(Session session, CloseReason closeReason){
}

CloseReason is a class that encapsulates the termination reason along with a code.
@OnError:
OnError annotation defines the method that handles exceptions. An example usage is as follows:
@OnError
public void onErrorReceived(Throwable t) {
}

ServerEndpoint is applicable on server side whereas ClientEndpoint is applicable only on client side. Other annotations are applicable for both sides.
A last remark is that, the signatures (parameter and return types) of the annotated methods must be one of the signatures allowed by JSR-356. The examples presented above are legal according to the spec. JavaDoc presents the allowed method signatures for each annotation.
3.2 Alternative 2: Jetty WebSocket API Implementation
In addition to JSR-356 support, Jetty also provides its own API. The reason that Jetty has two APIs on WebSockets is that JSR-356 was released after Jetty had released its own. JSR-356 API is heavily inspired by Jetty’s.
From the developer’s point of view Jetty API is very similar to the JSR-356’s, with minor differences.
Jetty WebSocket API needs initialization of a Servlet which extends org.eclipse.jetty.websocket.servlet.WebSocketServlet class. In JSR-356 implementation this is not needed. In this servlet, we configure the servlet socket class, which resembles the annotated Server Socket class implementation of JSR-356.
In the Example section, we will show how we can configure this server socket at the Servlet implementation.
Jetty API provides three alternative approaches for WebSocket development:
Annotation Based: Similar to JSR-356 annotations
Listener Based: Similar to JSR-356 listeners
Adapter Based: A convenience approach which eases Listener based implementation.
The easiest way is the Annotation approach. The annotation names (and classes) are different than JSR-356’s however they are almost of the same use.
@WebSocket

This annotation defines that the class is a WebSocket server. It is similar to the @ServletEndpoint of JSR-356 but we do not give the endpoint URL here. In addition to this, this annotation is not specific to the server side. Socket clients are also marked with this annotation.
@WebSocket
public class ExampleWebSocket{
}

@OnWebSocketConnect
This annotation defines that the method to be invoked when a connection is opened. It is similar to @OnOpen of JSR-356.
@OnWebSocketConnect
public void onConnect(Session session){
}

@OnWebSocketMessage
This annotation defines that the method to be invoked when a message is received. It is similar to @OnMessage of JSR-356.
@OnWebSocketMessage
public void onText(Session session, String message){
}

@OnWebSocketClose
This annotation defines that the method to be invoked when a connection is closed. It is similar to @OnClose of JSR-356.
@OnWebSocketClose
public void onClose(Session session, int status, String reason){
}

@OnWebSocketError
This annotation defines that the method to be invoked when a connection related error is thrown. It is similar to @OnError of JSR-356.
@OnWebSocketError
public void onError(Throwable error){
}

Similar to the JSR-356, the method names are not mandated by Jetty in this approach. The full list of allowed parameter types can be viewed in Jetty documentation.

3.3 Which API to Choose?
Both APIs offer similar features and programming approach. However there are subtle differences.
JSR-356 is specification based and standard. You can port easily your server sockets from Jetty to another Servlet container or Application server as long as the server supports JSR-356. In addition to this, programming with this API is a bit simpler. You do not need to configure a servlet with JSR-356 API. The downside of this alternative is, the documentation on Jetty side is missing and it seems less mature than the second alternative (but that might be just an impression).
Jetty WebSocket API is not standard, so you have to change your code when you change your Servlet Container. In addition to this, you have to code the servlet (some piece of boilerplate code) yourself. But Jetty API is more flexible allowing to easier control timeouts, paths, SSL etc. Another advantage is that, Jetty documentation of its own API is better than Jetty documentation on JSR-356.
At this point, I would humbly recommend using JSR-356 API if simpler configuration matters to you or if portability is a major concern. If you need to configure WebSocket parameters in detail and you do not need to port your WebSocket to another container, I would recommend Jetty WebSocket API.
Of course, this portability issue is only about porting WebSocket code from one container to another (from Jetty to Tomcat for example). WebSocket is a standardized protocol, so any kind of WebSocket client connect to any  implementation (JSR compliant or not) server without a problem.
In the following section we will provide examples of both APIs for server and client sides.


How to run ?
run server : mvn jetty:run
run client java : run WebSocket356ClientMain.java
run client html: open ToUpper356ClientSocket.html in browser