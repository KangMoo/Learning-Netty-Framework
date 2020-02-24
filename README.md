# Learning-Netty-Framework

채팅 어플리케이션 개발을 위해 공부한 Netty Framework의 관한 내용 정리

# 네티소개

## 네티란

- 네티는 프로토콜 서버 및 클라이언트 같은 네트워크 어플리케이션의 빠르고 쉬운 개발을 도와주는 프레임워크 입니다. TCP 또는 UDP 소켓 서버와 같은 네트워크 프로그래밍을 간단하게 할 수 있습니다.
- 비동기면서 이벤트 기반입니다. ( tomcat < 10,000명 vs netty < 100,000 ~ 1,000,000 ) < connection >
- 접속을 맺고, 데이터를 보내고 받고, 접속을 끊을 때 어떤 이벤트가 발생할지 Netty가 제공한다.
- 자바 네트워크 어플리케이션 프레임워크이다.
- 이벤트 기반 ( Event-Driven) 이며 비동기 방식을 지원한다.
- OIO ( Old I/O), New I/O 모두 지원한다.

## 네티의 주요 특징

### 동기와 비동기

- 여기서 살펴보고자 하는 동기와 비동기는 함수 또는 서비스의 호출 방식에 관한 내용이다.
    - 동기식 호출
    로그인시 호출자가 인증 요청을 하고 컨트롤러/서비스/데이터베이스에서 인증 처리 로직이 완료 된 후 인증 결과를 돌려주는 로직의 서비스가 있을때 호출자는 호출 시작 이후 부터 종료 까지는 호출의 결과를 확인 할 수 없다. 즉 호출이 종료 되고 난 이후 처리 결과를 확인 할 수 있다. 
    이처럼 서비스 처리가 완료된 이후 처리 결과를 알 수 있는 방식을 동기식 호출 이라고 한다. 
    특정 메소드나 서비스가 이와 같은 호출 방식을 지원한다면 그것을 동기식 호출을 지원한다고 표현한다.
    - 비동기식 호출
    비동기식  호출에서는 호출자의 인증 요청을 서비스가 수신하면 서비스는 스레드의 인증 요청을 위한 함수를 다른 스레드에 등록한다. 실제 인증 처리가 완료 되지 않아도 서비스 호출 종료가 이루어 지고 그 응답으로 티켓을 호출자에게 전달한다. 
    호출자의 관점에서는 인증 요청에 대한 결과를 수신하지는 못했지만 인증 요청 자체는 완료되었기 때문에 인증 요청 결과를 기다리는 시간에 다른 작업을 수행할 수 있고 필요한 시기에 서비스로부터 받은 티켓을 사용하여 요청한 인증 처리가 완료되었는지 확인 할 수 있다.

### 블로킹과 논블로킹

- 소켓의 동작 방식은 블로킹 모드와 논블로킹 모드로 나뉜다. 블로킹은 요청한 작업이 성공하거나 에러가 발생하기 전까지는 응답을 돌려주지 않는 것을 말하며 논블로킹은 요청한 작업의 성공 여부와 상관없이 바로 결과를 돌려주는 것을 말한다. 이때 요청의 응답값에 의해서 에러나 성공 여부를 판단한다.
- 블로킹 소켓은 ServerSocket, Socket클래스. 논블로킹 소켓은 ServerSocketChannel, SocketChannel 클래스를 사용한다.
    - 블로킹 소켓 
    클라이언트가 서버로 연결 요청을 보내면 서버는 연결 수락을 하고 클라이언트와 연결된 소켓을 새로 생성하는데 이때 해당 메소드의 처리가 완료되기 전까지 스레드의 블로킹이 발생하게 된다. 클라이언트가 연결된 소켓을 통해서 서버로 데이터를 전송하면 서버는 클라이언트가 전송한 데이터를 읽기 위하여 read 메소드를 호출하고 이 메소드의 처리가 완료되기 전까지 스레드가 블로킹 된다. 클라이언트 또 한 마찬가지로 서버에서 클라이언트로 전송한 데이터를 읽기 위한 메소드에서 블로킹이 발생한다.
    * 데이터 입출력에서 스레드의 블로킹이 발생하기 때문에 동시에 여러 클라이언트에 대한 처리가 불가능
    이러한 문제점을 해결하는 방법은 연결된 클라이언트별로 각각 스레드를 할당하는 방법이 있다. 하지만 동시에 접속 요청을 하는 상황에서 대기시간이 길어진다는 점과 서버의 스레드 수가 증가해서 자바의 힙 메모리 부족으로 인한 out of memory 오류가 발생 할 수 있다. 
    이러한 오류를 피하기 위해 스레드 풀을 사용하기도 한다. 클라이언트가 서버에 접속하면 서버 소켓으로 부터 클라이언트 소켓을 얻어온다. 다음으로 스레드 풀에서 가용 스레드를 하나 가져오고 해당 스레드에 클라이언트 소켓을 할당한다. 이와 같은 구조에서는 동시에 접속 가능한 사용자 수가 스레드 풀에 지정된 스레드 수에 의존하는 현상이 발생한다. 스레드 풀의 크기를 자바 힙이 허용하는 최대 한도에 도달할 때까지 늘리는 것이 합당한지 두가지 관점에서 고민해 봐야한다. 
    1. 힙에 할당된 메모리가 크면 클수록 가비지 컬렉션이 수행되는 횟수는 줄어들지만 수행시간은 상대적으로 길어진다. 
    2. 컨텍스트 스위칭 시 수많은 스레드가 CPU 자원을 획듣하기 위하여 경쟁하면서 CPU자원을 소모하고 실제로 사용할 CPU 자원이 적어지게 된다.
    - 논블로킹 소켓
    앞써 말한 블로킹 모드의 소켓은 read, write, accept 메소드 등과 같은 입출력 메소드가 호출되면 처리가 완료될 떄까지 스레드가 몸추게 되어 다른 처리를 할 수 없었다. 이러한 단점을 해결하는 방식이 논블로킹 소켓이다. 논블로킹 소켓은 구조적으로 소켓으로 부터 읽은 데이터를 바로 소켓에 쓸 수가 없다. 이를 위해서 각 이벤트가 공유하는 데이터 객체를 생성하고 그 객체를 통해서 각 소켓 채널로 데이터를 전송한다.
    블로킹 소켓과 다르게 소켓 채널을 먼저 생성하고 사용할 포트를 바인딩한다. 
    여러 클라이언트는 하나의 스레드에 등록된다. 그리고 각가은 Selector 라는 객체에 채널을 등록하고 Selector는 자신에게 등록된 채널에 변경 사항이 발생했는지 검사를 하고 이에 응답한다.

### 동기와 비동기 / 블로킹과 논블로킹의 대한 간략 정리

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png)

### 이벤트 기반 프로그래밍

- 전통적으로 사용자 인터페이스가 포함된 프로그램에 많이 사용된다. ( 예를 들어 마우스 클릭에 반응하는 코드) 이와 같이 각 이벤트를 먼저 정의해두고 발생한 이벤트에 따라서 코드가 실행되도록 프로그램을 작성하는것이 이벤트 기반 프로그래밍이다. * 앞서 말한 논블로킹 소켓의 Selector를 사용한 I/O이벤트 감지 역시 이벤트 기반 프로그램의 한 종류이다.
- 추상화 수준 : 이벤트를 나눌 떄 작은 단위로 나누었는지 큰 단위로 나누었는지의 정도를 말하는 것.
- 네트워크 프로그램에서 이벤트가 발생하는 주체는 소켓이다
- 발생하는 이벤트는 크게 소켓 연결 / 데이터 송수신으로 나눌 수 있다.
- 서버는 클라이언트 연결을 수락하기 위해 서버 소켓을 생성하고 포트를 서버에 바인딩 한다.
- 이로서 클라이언트의 연결을 수락하고 데이터 송수신 할 소켓을 생성할 수 있고 이때부터 송수신이 가능해진다.
- 일반적으로 네트워크 프로그램에서 소켓은 IP주소와 포트를 가지고 있고 양방향 네트워크 통신이 가능한 객체이다.
- 만약 소켓이나 연결된 스트림에 문제가 발생한다면 새로운 소켓을 생성하고 데이터를 전송하는 예외 처리 코드가 작동되게 해야한다.
- 네티는 데이터의 읽기 쓰기를 위한 이벤트 핸들러를 지원한다. (ChannelInboundHandlerAdapter 등 )
- 네티는 데이터를 소켓으로 전송하기 위해 채널에 직접 기록하는 것이 아니라 데이터 핸들러를 통해서 기록한다. 이 와같은 방법은 서버 애플리케이션의 코드를 클라이언트 애플리케이션에서 재사용 할 수 있는 장점이 있다.
- 데이터 핸들러는 에러 이벤트도 같이 정의하기 때문에 에러처리에 용이하다.

# 네티 구성요소

## Bootstrap

- 부트스트랩은 네티로 작성한 네트워크 애플리케이션이 시작할때 가장 처음으로 수행되는 부분이다. 부트스트랩 설정은 크게 이벤트 루프, 채널의 전송 모드, 채널 파이프라인으로 나뉜다.
    - 이벤트 루프는 소켓 채널에서 발생한 이벤트를 처리하는 스레드 모델에 대한 구현이 담겨있다.
    - 부트스트랩에서 설정한 소켓의 모드에 따라 사용되는 이벤트 루프의 구현체가 달라지기도 한다. 사용할 소켓 채널의 모드에 따라서 블로킹, 논블로킹, epoll로 지정가능하다. epoll은 가장 빠르지만 linux에서만 동작한다.
    - 채널 파이프라인은 연결된 채널에서 사용할 데이터 핸들러에 대한 내용을 포함한다 ( 코덱도 포함)

### Bootstrap 정의

- 네트워크 애플리케이션의 동작 방식과 환경을 설정을 도와주는 클래스이다.
- 예를 들어 네트워크에서 수신한 데이터를 단일 스레드로 데이터베이스에 저장하는 네트워크 애플리케이션을 구현하고자 한다면 이 애플리케이션은 8088번 포트에서 데이터를 수신하고 소켓모드는 NIO를 사용한다. 이와 같은 요구사항을 부트스트랩으로 설정 가능하다. NIO소켓 모드를 지원하는 서버 소켓 채널, 데이터를 변환하여 데이터베이스에 저장하는 데이터 처리 이벤트 핸들러, 애플리케이션이 사용할 8088번 포트 바인딩, 단일 스레드를 지원하는 이벤트 루프에 대한 설정이 필요하다.

### Bootstrap 구조

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c59a962d-d964-4783-ae4f-2b31703a0ffc/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c59a962d-d964-4783-ae4f-2b31703a0ffc/Untitled.png)

- 위의 표는 부트스트랩이 지원하는 설정 목록이다.
- 통상적으로 네트워크 애플리케이션은 서비스를 제공할 네트워크 포트, 네트워크 전송에 사용할 소켓 모드와 소켓 옵션, 소켓의 데이터를 처리하는 스레드, 그리고 애필르케이션에서 사용하는 프로토콜로 구성된다.
- 서버 애플리케이션과 클라이언트 애플리케이션의 구분은 소켓 연결을 요청하느냐 아니면 대기하느냐에 따른 구분이다.
- ServerBootstrap 클래스는 클라이언트 접속을 대기할 포트를 설정하는 메소드가 추가되있다.
- 네티의 부트스트랩 설정을 통해서 데이터 처리 코드를 변경하지 않고 BlockingServer 또는 NonBlockingServer와 동일한 동작을 하는 애플리케이션을 작성할 수 있다. 사용자는 단순히 사용할 입출력 모드에 해당하는 소켓 채널 클래스를 설정하기만 하면 변경이 완료 된다.

### ServerBootstrap 과 Bootstrap

- 부트스트랩 클래스가 빌더패턴 ( 인수없는 생성자 생성 후 객체 추가) 으로 작성되어 있으므로 인수 없는 생성자로 객체를 생성하고 group, channel 과 같은 메소드로 객체를 초기화 한다. ( 이와 같은 객체 초기화 방법은 생성자를 사용한 객체 생성 방법에 비해 코드가 더 간결하고 설정하려는 내용이 명확해진다.
- ServerBootstrap 객체에 서버 애플리케이션이 사용할 두 스레드 그룹을 설정하는데 첫 번째 스레드 그룹은 클라이언트의 연결을 수락하는 부모 스레드 그룹이고 두번째 스레드 그룹은 연결된 클라이언트의 소켓으로 부터 데이터 입출력 및 이벤트 처리를 담당하는 자식 스레드 그룹니다. 그룹은 NioEventLoopGroup 클래스의 객체로 설정했다. 클래스 생성자의 인수 사용되는 숫자는 스레드  그룹 내에서 생성할 최대 스레드 수를 의미한다. 1 = 단일 스레드로 동작.
- 인수 없는 생성자는 사용 할 스레드 수를 서버 애플리케이션이 동작하는 하드웨어 코어 수를 기준으로 결정한다
- 서버 애플리케이션의 소켓 모드를 블로킹 모드로 변경하여면 OioEventLoopGroup으로 변경하고 소켓 채널의 클래스를 OioServerSocketChannel로 지정하면 된다. ( 매우 간단함 )
- 이와같이 부트스트랩은 추상화 모델을 제공하기 때문에 네트워크 애플리케이션 개발자가 확장성과 유연성을 동시에 만족하는 코드를 작성할 수 있다.
- ServerBootstrap API
    - 부트스트랩의 주요 기능인 이벤트 루프, 채널의 전송 모드, 채널 파이프라인 등은 ServerBootstrap의 API로 매칭된다.
    - ServerBootstrap → .group 메소드 인수 두개 가능
    - Bootstrap → .group 메소드 인수 한개 가능
    - Channel 은 소켓의 입출력 모드를 설정하는 메소드이다. 이 Channel 메소드는 abstractBootstrap 추상 클래스의 구현체인 ServerBootstrap 과 Bootstrap 클래스에 모두 존재하는 API고 channel 메소드에 등록된 소켓 채널 생성 클래스가 소켓 채널을 생성한다.
    - channelFactory
        - channel 매소드와 동일하게 입출력 모드를 설정하는 API이다. 네티가 제공하는 ChannelFactory 인터페이스의 구현체로는 NioUdtProvider가 있다.
    - handler
        - 서버 소켓 채널의 이벤트를 처리할 핸들러 설정 API이다. ServerBootstrap의 handler 메소드로 등록된 이벤트 핸들러는 서버 소켓 채널에서 발생한 이벤트만을 처리한다. ( 비슷하지만 다른 메소드로는 childHandler 가 있음 )
    - childHandler
        - 클라이언트 소켓 채널로 송수신 되는 데이터를 가공하는 데이터 핸들러 설정 API 이다. handler 메소드와 childHandler메소드는 ChannelHandler 인터페이스를 구현한 클래스를 인수로 입력 한다. 이를 통해 등록되는 이벤트 핸들러는 서버에 연결된 클라이언트 소켓 채널에서 발생하는 이벤트를 수신하여 처리한다.
    - option
        - 서버 소켓 채널의 소켓 옵션을 설정하는 API 이다. 소켓 옵션이란 소켓의 동작 방식을 지정하는 것을 말한다. ( 커널( 송신 버퍼, 수신 버퍼) 에서 사용되는 값을 변경)
        - 네티는 자바 가상머신을 기반으로 동작하기 때문에 자바에서 설정할 수 있는 소켓 옵션을 모두 설정 할 수 있다. 예: SO_KEEPALIVE (운영체제에서 지정된 시간에 한번씩  keepalive 패킷을 상대방에게 전송한다. SO_LINGER (소켓을 닫을 때 커널의 송신 버퍼에 전송되지 않은 데이터의 전송 대기시간을 지정한다) 등등이 있다.
    - childOption
        - 앞서 말한 옵션은 소켓 채널에 소켓 옵션을 설정하지만 childOption 메소드는 서버에 접속한 클라이언트 소켓 채널에 대한 옵션을 설정한다. 예 : SO_LINGER ( close 메소드를 호출한 이후 제어권은 애플리케이션에서 운영체제로 넘어가게 된다. 이때 커널 버퍼에 아직 전송되지 않은 데이터가 남아 있으면 이 옵션의 사용 여부와 타임 아웃 값의 설정에 따라 처리할 수 있다. 서버 애플리케이션이 동작 중인 운영체제에서 TIME_WAIT이 많이 생기는 것을 방지하기 위해 사용하거나 Proxy 서버가 동작중인 운영체제에서 TIME_WAIT가 발생하는것을 방지하기 위해 사용함.
- Bootstrap API
    - option 과 childOption 같이 두 가지 설정을 제공하지 않는다. 즉 option 메소드만 존재한다 (child가 없기 때문에)
    - group
        - 위와 같이 그룹 메소드는 소켓 채널의 이벤트 처리를 위한 이벤트 루프를 설정한다. Bootstrap의 group 메소드는 단 하나의 이벤트 루프만 설정 가능하다. ( 클라이언트는 서버에 연결할 소켓채널 하나만 가지고 있기 때문)
    - channel
        - channel 메소드는 소캣 채널의 입출력 모드를 설정한다. 클라이언트 소켓만 서정 가능하다. 예: NioSocketChannel.class
    - channelFactory
        - ServerBootstrap 의 channelFactory 와 동일한 동작을 수행한다.
    - handler
        - 클라이언트  소켓에서 발생하는 이벤트를 수신하여 처리한다. 보통 ChannelInitializer 클래스의 handler 메소드를 통해 등록된다.

## Channel Pipeline

채널 파이프라인은 채널에서 발생한 이벤트가 이동하는 통로다. 이 통로를 통해 이동하는 이벤트를 처리하는 클래스를 이벤트 핸들러하고 하고 이벤트 핸들러를 상속받아서 구현한 구현체들을 코덱이락고 한다. 

### 네티의 이벤트 실행

- 네티 이벤트 메소드는 데이터가 수신되면 네티가 자동으로 호출된다.
- 만약 네티 없이 일반 서버 네트워크 프로그램을 작성한다고 하면 서버는 클라이언트 소켓으로 접속되어 있고 데이터가 들어오기를 기다리고 있다.
    1. 소켓에 데이터가 있는지 확인한다 
    2. 데이터가 존재하면 데이터를 읽어 들이는 메소드를 호출한다. 
    3. 읽어들일 데이터가 존재하지 않는다면 데이터가 도착할 때 까지 기다린다.
    4. 데이터를 기다리는 중에 네트워크가 끊어지면 에러 처리를 위한 메소드를 호출한다
- 이와 같은 관점에서 네티는 이벤트 채널 파이프라인과 이벤트 핸들러로 추상화 한다.
- 따라서 네티는데이터가 수신되었는지 소켓의 연결이 끊어졌는지와 같은 상태에서 메소드 호출에 관여할 필요가 없다.
    1. 부트스트랩으로 네트워크 애플리케이션에 필요한 설정을 지정한다. 
    2. 부트스트랩에 이벤트 핸들러를 사용하여 채널 파이프라인을 구성한다. 
    3. 이벤트 핸들러의 데이터 수신 이벤트 메소드에서 데이터를 읽어 들인다. 
    4. 이벤트 핸들러의 네트워크 끊김 이벤트 메소드에서 에러 처리를 한다. 
- 위와 같이 구현하면 네티의 이벤트 루프가 소켓 채널에서 발생한 이벤트에 해당하는 이벤트 메소드를 자동으로 실행한다.
- 소켓채널에 데이터가 수신 되었을 때 네티가 이벤트 메소드를 실행하는 방법
    1. 네티의 이벤트 루프가 채널 파이프라인에 등록된 첫 번째 이벤트 핸들러를 가져온다 
    2. 이벤트 핸들러에 데이터 수신 이벤트 메소드가 구현되어 있으면 실행한다 
    3. 데이터 수신 이벤트 메소드가 구현되어 있지 않으면 다음 이벤트 핸들러를 가져온다
    4. 2를 수행한다. 
    5. 채널 파이프라인에 등록된 마지막 이벤트 핸들러에 도달할때까지 1을 반복한다. 

데이터를 처리하는 입출력은 네티가 이벤트로 관리한다. 때문에 해당 이벤트에 해당하는 코드만 구현하면 된다. 

### 채널 파이프라인

채널 파이프라인은 네티의 채널과 이벤트 핸들러 사이에서 연결 통로 역할을 수행한다. 

- 채널은 일반적인 소켓 프로그래밍에서 말하는 소켓과 같다고 보면 된다.
- 네티는 이벤트 처리를 위한 추상화 모델로서 채널 파이프라인을 사용하고 소켓 채널에서 발생한 이벤트가 이를 통로로 하여 이동한다.
- 통상 childHandler 메소드를 통해서 연결된 클리이언트 소켓 채널이 사용할 채널 파이프라인을 설정한다.
- ChannelInitializer 인터페이스를 구현한 클래스를 작성하여 childHandler의 인자로 입력한다.
- initChannel 메소드는 클라이언트 소켓 채널이 생성될 때 자동으로 호출되고 파이프 라인의 설정을 수행한다.
- initChannel 메소드의 인자로 입력된 소켓 채널 (즉 연결된 클라이언트 소켓 채널) 에 설정된 채널 파이프 라인을 가져오게 되는데, 네티의 내부에서는 클라이언트 소켓 채널을 생성할 때 빈 채널 파이프라인 객체를 생성하여 할당한다.
- ChannelInitializer 클래스의  initChannel 메소드 본체는 부트스트랩이 초기화 될 때 수행되며 이때 서버 소켓 채널과 채널 파이프라인이 연결된다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b268944a-6d4c-4467-a5d0-6da116c10708/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b268944a-6d4c-4467-a5d0-6da116c10708/Untitled.png)

위의 구성도를 설명하자면 

1. 클라이언트 연결에 대응하는 소켓 채널 객체를 생성하고 빈 채널 파이프라인 객체를 생성하여 소켓 채널에 할당한다. 
2. 소켓 채널에 등록된 ChannelInitializer 인터페이스의 구현체를 가져와서  initChannel 메소드를 호출한다. 
3. 소켓 채널 참조로 부터 1에서 등록한 파이프라인 객체를 가져오고 채널 파이프라인에 입력된 이벤트 핸들러의 객체를 등록한다. 

위와 같은 3 단계가 완료되면 채널이 등록됐다는 이벤트가 발행하고 이때부터 클라이언트와 서버 간의 데이터 송수신을 위한 이벤트 처리가 시작된다. 

## Event Handler

네티는 비동기 호출을 지원하는 두 가지 패턴을 제공한다. 

1. 퓨처 패턴 
2. 리액터 패턴 ( 의 구현체인 이벤트 핸들러) 

이벤트 핸들러는 네티의 소켓 채널에서 발생한 이벤트를 처리하는 인터페이스이다. 

더 자세히는 소켓 채널의 이벤트를 인터페이스로 정의하고 이 인터페이스를 상속 받은 이벤트 핸들러를 작성하여 채널 파이프라인에 등록한다.

### 채널 인바운드 이벤트

네티는 소켓 채널에서 발생하는 이벤트를 인바운드 이벤트와 아웃바운드 이벤트로 추상화 한다. 

이바운드 이벤트는 소켓 채널에서 발생한 이벤트 중에서 연결 상대방이 어떤 동작을 취했을 때 발생한다. ( 예를 들면 채널 활성화, 데이터 수신 등의 이벤트) 

- 클라이언트가 서버에 잡속하면 서버로 데이터를 전송하는데 서버에 관점에서는 데이터가 수신된다. 이때 네티는 소켓 채널에서 읽을 데이터가 있다는 이벤트를 채널 파이프라인으로 흘려보내고 채널 파이프라인에 등록된 이벤트 핸들러 중에서 인바운드 이벤트 핸들러가 해당 이벤트에 해당하는 메소드를 수행한다.  네티는 인바운드 이벤트를 ChannelInboundHandler 인터페이스로 제공한다.
    - channelRegistered 이벤트
        - 이 이벤트는 채널이 이벤트 루프에 등록되었을 떄 발생한다. ( 이벤트 루프는 네티가 이벤트를 실행하는 스레드( 부트스트랩에 설정한 )
    - channelActive 이벤트
        - 채널이 생성되고 이벤트 루프에 등록된 이후에 네티 API를 사용하여 채널 입출력을 수행 할 상태가 되었음을 알려주는 이벤트다.
            1. 서버 애플리케이션에 연결된 클라이언트의 연결 개수를 셀 때 
            2. 서버 애플리케이션에 연결된 클라이언트에게 최초 연결에 대한 메시지를 전송할 때 
            3. 클리언트 애플리케이션이 연괼된 서버에 최초 메시지를 전달할 때 
            4. 클라이언트 애플리케이션에서 서버에 연결된 상태에 대한 작업이 필요할 때 
    - channelRead 이벤트
        - 수신된 데이터는 네티의  ByteBuf 객체에 저장되있고 이벤트 메소드의 두 번째 인자인 msg 를 통해서 접근 할 수 있다.
        - 네티 내부에서는 모든 데이터가 ByteBuf로 관리된다.
    - channelReadComplete 이벤트
        - 채널의 데이터를 다 읽어서 더 이상 데이터가 없을 때 channelReadComplete 이벤트가 발생한다. * flush 메소드는 네티의 채널 버퍼에 저정된 데이터를 상대방으로 즉시 전송한다.
    - channelInactive 이벤트
        - channelActive 이벤트와는 반대로 채널이 비활성화 되있을 때 발생한다.
    - channelUnregistered 이벤트
        - 채널이 이벤트 루프에서 제거 되었을 때 발생한다.

### 채널 아웃바운드 이벤트

아웃바운드 이벤트는 소켓 채널에서 발생한 이벤트 중에서 네티 사용자가  요청한 동작에 해당하는 이벤트를 말하며 연결 요청, 데이터 전송, 소켓 닫기 등이 이에 해당된다. ChannelOutboundHandler 인터페이스로 제공하고 이의 모든 이벤트는 `ChannelHandlerContext` 객체를 인수로 받는다. 

### bind 이벤트

- bind 이벤트는 서버 소켓 채널이 클라이언트의 연결을 대기하는 IP 포트가 설정 되었을 때 발생한다. bind 이벤트에서는 서버 소켓 채널이 사용중인 socketAddress 객체가 인수로 입력되고 서버 소켓 채널이 사용하는 IP와 포트 정보를 확인 할 수 있다.

### Connect 이벤트

- 클라이언트 소켓 채널이 서버에 연결되었을 때 발생한다. 원격지의 SocketAddress 정보와 로컬 SocketAddress 정보가 인수로 입력된다.

### Disconnect 이벤트

- 클라이언트 소켓 채널의 연결이 끊어 졌을 때 발생한다. 별도의 인수는 없다.

### Close 이벤트

클라이언트 소켓 채널의 연결이 닫혔을 때 발생한다. 별도의 인수는 없다. 

### Wrtie 이벤트

소켓 채널에 데이터가 기록되었을 때 발생한다. write 이벤트에는 소켓 채널에 기록된 데이터 버퍼가 인수로 입력된다. 

### Flush 이벤트

소켓 채널에 대한 flush 메소드가 호출 되었을 때 발생한다. 별도의 인수가 없다. 

### 이벤트 이동 경로와 이벤트 메소드 실행

핸들러가 여러개 등록이 되어 있고 겹치는 이벤트가 있다면 이벤트는 하나의 이벤트 메소드만 수행한다. 만약 두번째 이벤트 핸들러인 channelRead 메소드도 소행하고 싶다면 첫 번째 이벤트 핸들러의 코드를 다음과 같이 수정해야한다.

    ctx.write(msg);
    crx.fireChannelRead(msg);

다음 이벤트 핸들러로 이벤트를 넘겨누는 것이다. ChannelHandlerContext 인터페이스를 사용하여 채널 파이프라인에 이벤트를 발생시키는 것이다. 

ChannelHandlerContext 인터페이스의 fireChannelRead 메소드를 호출하면 네티는 채널 파이프라인에 channelRead 이벤트를 발생시킨다. 

## Codec

### 코덱의 구조

- 네티 애플리케이션을 기준으로 수신은 인바운드 송신은 아웃바운드로 볼 수 있다. 인바운드 아웃바운드에 해당하는 이벤트는 ChannelOutboundHandler와 ChannelInboundHandler 인터페이스로 각각 인코더와 디코더라 부른다. 데이터를 전송할 때는 인코더를 사용하여 패킷으로 변환하고 데이터를 수신할 때는 디코더를 사용하여 패킷을 데이터로 변환한다.

### 템플릿 메소드 패턴이란

- 네티의 코덱은 템플릿 메소드 패턴으로 구현되어있다.
- 템플릿 메소드 패턴이은 상위 구현체에서 메소드의 실행 순서만을 지정하고 수행될 메소드의 구현은 하위 구현체로 위임한다.

### 기본 제공 코덱

- bytes 코덱
    - 바이트 배열 데이터에 대한 송수신을 지원하는 코덱이다
- compression 코덱
    - 송수신 데이터의 압축을 지원하는 코덱이다. ( zlib, gzip, Snappy, LZF 의 압축 알고리즘 등)
- http 코덱
    - HTTP 프로토콜을 지원하는 코덱으로서 하위 패키지에서 다양한 데이터 송수신 방법을 지원한다. ( cors 코덱, multipart 코덱, 웹 소켓 프로토콜의 데이터 송수신을 지원하는 websocket 코덱이 있다. )
- marshalling 코덱
    - 객체를 네트워크를 통해서 송신 가능한 형태로 변환하는 과정이다. 바낻로 수신한 데이터를 객체로 변환하는 과정을 언마샬링이라 한다. ( JBoss 에서 개발한 마샬링 라이브러리 지원)
- protobuf 코덱
    - 구글의 프로토콜 버퍼를 사용한 데이터 송수신을 지원하는 코덱이다.
- rtsp 코덱
    - Real Time Streaming Protocol은 오디오와 비디오 같은 실시간 데이터의 전달을 위해서 특수하게 만들어진 애플리케이션 레벨의 프로토콜이다. 보통 실시간 동영상 스트리밍을 위한 제어 프로토콜로 사용된다.
- string 코덱
    - 문자열의 송수신을 지원하는 코덱이다. 주로 텔넷 프로토콜이나 채팅 서버의 프로토콜에 이용된다.
- serialization 코덱
    - 자바 객체를 네트워크로 전송 할 수 있도록 직렬화와 역직렬화를 지원하는 코덱이다. 네티의 serialization 코덱은 JDK의 ObjectOutputStream 및 ObjectInputStream 과 호환되지 않는다.

## Event Loop

네티는 단일 스레드 이벤트 루프와 다중 스레드 이벤트 루프를 모두 사용할 수 있다. 다중 스레드 이벤트 루프는 이벤트의 발생 순서와 실행 순서가 일치하지 않는다. *** 하지만 네티에서는 이벤트 루프의 종류에 상관없이 이벤트 발생 순서에 따른 실행 순서를 보장한다. 

네티가 다중 스레드 이벤트 루프를 사용함에도 불구하고 이벤트 발생 순서와 실행 순서를 일치시킬 수 있는 이유는 아래의 세가지 특징에 기반한다. 

1. 네티의 이벤트는 채널에서 발생한다.
2. 이벤트 루프 객체는 이벤트 큐를 가지고 있다 
3. 네티의 채널은 하나의 이벤트 루프에 등록된다. 

채널에서 발생한 이벤트는 항상 동일한 이벤트 루프 스레드에서 처리하여 이벤트 발생 순서와 처리 순서가 일치된다. 네티는 이벤트 큐를 이벤트 루프 스레드의 내부에 둠으로써 수행 불일치의 원인을 제거했다. 

*** 처리를 위한 이벤트 루프 스레드가 하나이므로 이벤트 처리 순서는 이벤트 발생 순서와 같다. 

*** 네티는 이벤트 처리를 위하여 SingleThreadEventExecutor를 사용한다. 

### 네티의 비동기 I/O 처리

네티는 비동기 호출을 위한 두가지 패턴을 제공한다. 

1. "리액터 패턴" 의 구현체인 이벤트 핸들러 
2. 퓨처 패턴 

### 퓨처 (Future) 패턴

퓨처 패턴은 미래에 완료될 작업을 등록하고 처리 결과를 확인하는 객체를 통해서 작업의 완료를 확인하는 패턴이다. 퓨처 패턴은 메소드를 호출 하는 즉시 퓨처 객체를 돌려준다 (example : 주문서) 메소드의 처리 결과는 나중에 Future 객체를 통해서 확인한다.

## ByteBuf

### 자바 NIO 바이트 버퍼

자바 NIO  바이트 버퍼는 바이트 데이터를 저장하고 읽는 저장소다. 배열을 멤버 변수로 가지고 있으며 배열에 대한 읽고 쓰기를 추상화 한 메소드를 제공한다. 자바에서 제공하는 버퍼로는 ByteBuffer, CharBuffer, IntBuffer, ShortBuffer, LongBuffer, FloatBuffer, DoubleBuffer가 있으며 바이트 버퍼는 저장되는 데이터형에 따라서 적당한 클래스를 선택하여 사용한다. 바이트 버퍼 클래스는 배열 상태를 관리하는 세가지 속성을 가지고 있다. 

1. capacity 
- 버퍼에 저장 할 수 있는 데이터의 최대 크기로 한번 정하면 변경이 불가능 하다. 이 값은 버퍼를 생성할 때 생성자의 인수로 입력한 값이다.

2. position 

- 읽기 또는 쓰기가 작업중인 위치를 나타낸다. 버퍼 객체가 생성될 때 0으로 초기화 되고 데이터를 입력하는 put 메소드나 데이터를 읽는 get 메소드가 호출되면 자동으로 증가하여 limit 과 capacity 값보다 작거나 같다.

3. limit 

- 읽고 쓸 수 있는 버퍼공간의 최대치를 나타낸다. limit 메소드로 값을 조절 할 수 있다. 이 값을 capacity 값보다 크게 설정 할 수 없다.

자바 NIO 바이트 버퍼를 생성하는 방법 

1. allocate
    - JVM 힙 영역에 바이트 버퍼를 생성한다. 메소드의 인수는 생성 할 바이트 버퍼 크기이며 앞 절에서 설명한 capacity 속성 값이다. 또 한 생성되는 바이트 버퍼 값은 모드 0으로 초기화된다.
2. allocateDirect
    - JVM의 힙 영역이 아닌 운영체제의 커널 영역에 바이트 버퍼를 생성한다. ByteBuffer 추상 클래스만 사용할 수 있다. 즉 다이렉트 버퍼는 ByteByffer로만 생성할 수 있다. 메소드의 인수는 생성할 바이트 버퍼의 크기며 앞 절에서 설명한 capacity의 속성 값이다. 생성되는 바이트 버퍼 값은 모두 0으로 초기화 된다.
3. wrap
    - 입력된 바이트 버퍼 배열을 사용하여 바이트 버퍼를 생성한다. 입력에 사용된 바이트 배열이 변경되면 wrap 메소드를 사용해서 생성한 바이트 버퍼도 변경된다.

일반적으로 allocate 메소드를 사용해서 생성한 버퍼를 힙 버퍼라고 부르고 allocateDirect 메소드를 통해서 생성한 버퍼를 다이렉트 버퍼라고 한다.

자바 NIO ByteBuffer의 .put 과 .get 실행 시 바이트 버퍼의 위치 정보인 position 속성이 값을 읽어 들인 길이 만큼 증가한다. 

자바 바이트 버퍼는 이전에 수행한 put 또는 get 메소드가 호출된 이후의 position 정보를 저장하기 위하여 flip 메소드를 제공한다. 

- flip 메소드를 호출하면 limit 속성값이 마지막에 기록한 데이터의 position 으로 변경된다.

하나의 바이트 버퍼에 대하여 읽기 작업 또는 쓰기 작업의 완료를 의미하는 flip 메소드를 호출하지 않으면 반대 작벙을 수행 할 수 없다. 

자바의 바이트 버퍼를 사용할 때는 읽기와 쓰기를 분리하여 생각해야 하며 특히 다중 스레드 환경에서 바이트 버퍼를 공유하지 않아야한다. 

### 네티 바이트 버퍼

네트 바이트 버퍼의 특징 

1. 별도의 읽기 인덱스와 쓰기 인덱스 
2. flip 메소드 없이 읽기 쓰기 가능 
3. 가변 바이트 버퍼 
4. 바이트 버퍼 풀 
5. 복합 버퍼 
6. 자바의 바이트 버퍼와 네티의 바이트 버퍼 상호 호환 

네티 바이트 버퍼는 저장되는 데이터형에 따른 별도의 바이트 버퍼를 제공하지 않는 대신 각 데이터형에 대한 읽기 쓰기 메소드를 제공한다. ( 예 readFloat, writeFloat ) 

또 한 읽기 쓰기 메소드가 실행될 때 각각 읽기 인덱스와 쓰기 인덱스를 증가시킨다. 읽기 인덱스와 쓰기 인덱스가 분리 되어 있어 별도의 메소드 호출 없이 읽기와 쓰기를 수행할 수 있다. 

- 네티의 바이트버퍼는 하나의 바이트 버퍼에 대하여 쓰기 작업과 읽기 작업을 병행 할 수 있다.

네티 바이트 버퍼 생성 방법 

1. 힙 버퍼 + 풀링 함 : ByteBufAllocator.DEFAULT.heapBuffer() 
2. 힙버퍼 + 풀링 안함 : Unpooled.buffer()
3. 다이렉트 버퍼 + 풀링 함 : ByteBufAllocator.DEFALT.directBuffer()
4. 다이렉트 버퍼 + 풀링 안함 : Unpooled.directBuffer()

### 네티 바이트 버퍼 읽기 쓰기

네티 바이트 버퍼는 자바 바이트 버퍼와 달리 읽기 쓰기 전환에 flip 메소드를 호풀하지 않는다. 자바의 바이트 버프는 쓰기 작업이 끝나고 읽기 작업을 시작하기 전 또는 그 반대 상황에서 항상 flip 메소드를 호출해야 했다. 원인은 바이트 버퍼에서 사용하는 인덱스가 하나이기 때문이다. 

- 가변 크기 변경
    - 네티 바이트 버퍼는 생성된 버퍼의 크기를 동적으로 변경 할 수 있다. 바이트 버퍼의 크기를 경해도 저장된 데이터는 보존된다.

### 네티 바이트 버퍼 풀링

네티 바이트 버퍼 풀링을 사용함으로써 얻을 수 있는 가장 큰 장점은 버퍼를 빈번히 할당하고 해제할 때 일어나는 가비지 컬렉션 횟수의 감소다. 두 문자열을 합쳐서 하나로 만드는 연산을 수행했을 때 가상머신에서는 2개의 객체가 생성되었다가 가바지 컬렉터의 대상이 된다. 가비지 컬렉터의 대상이 되는 객체들은 가비지 컬렉션이 수행되기 전까지 메모리를 점유하고 있게 된다. 

- 네티 바이트 버퍼 풀링은 byteBufAllocator를 사용하여 바이트 버퍼를 생성할 때 자동으로 수행된다.
- 네티는 바이트 버퍼의 참조 수를 관리하기 위하여 ReferenceCountUtil 클래스에 정의된 retain 메소드와 release 메소드를 사용한다.

### 엔디안 변환

네티의 바이트 버퍼 기본 엔디안은 자바와 동일하게 빅엔디안이다. 특별한 상황에서 리틀엔디안의 바이트 버퍼가 필요한데 이때 바이트 버퍼의 order 메소드를 사용하여 엔디안을 변환한다. 

### 바이트 버퍼 상호 변환

네티 바이트 버퍼는 nioBuffer 메소드를 이용하여 자바 NIO 버퍼로 변환이 가능하다. 변환 된 NIO 바이트 버퍼는 네티 바이트 버퍼의 내부 바이트 배열을 공유한다. 

    final String source = "Hello world";
    
    ByteBuf buf = Unpooled.buffer(11);
    buf.writeBytes(source.getBytes());
    
    ByteBuffer nioByteBuffer = buf.nioBuffer();
    new String(nioByteBuffer.array(), nioByteBuffer.arrayOffset(), nioBuffer.remainning()));

### 채널과 바이트 버퍼 풀

네티 내부에서 데이터를 처리할 때 자바의 바이트 버퍼가 아닌 네티 바이트 버퍼를 사용한다. 

채널 파이프라인에 등록한 핸들러 중 이벤트 메소드인 channelRead에선 인수로 사용되는 바이트 버퍼는 네티 바이트 버퍼이다. 즉 channelRead 메소드가 실행된 이후의 네티 바이트 버퍼는 바이트 버퍼 풀로 돌아간다. 네티의 바이트 버퍼 풀은 네티 애플리케이션의 서버 소켓 채널이 초기화 될 때 같이 초기화 되며 ChannelHandlerContext 인터페이스의 alloc 메소드로 생성된 바이트 버퍼 풀을 참조 할 수 있다. 

### ByteBuffer 보다 ChannelBuffer의 성능이 좋은 이유

Nio의 ByteBuffer는 여러가지 바운더리 체크가 항상 일어나지만 Netty의 ByteBuf는 바운더리 체크 제약이 보다 느슨하여 이로인한 복잡한 경비 체크 비용이 줄었으며, ByteBuffer과는 다른 인덱스의 장점과 JVM이 Buffer에 Access 하기 쉬운 장점이 있다.

# 네티 응용

# 참고 개념

### OIO/NIO

- 자바에서는  JDK 1.3까지 블로킹방식의 I/O 만을 지원했다. 이후 JDK 1.4 부터는 NIO 라는 논블로킹 I/O API가 추가 되었다. 입출력과 관련된 기능을 제공하고 소켓도 입출력 채널의 하나로써 NIO API를 사용할 수 있으며 NIO API  를 통해서 블로킹과 논블로킹 모드의 소켓을 사용할 수 있다.

### SCTP

- SCTP는 TCP의 연결지향 및 전송 보장 특성과 UDP의 메시지 지향 특성을 모두 갖추고 있다.  TCP는 세 방향 핸드셰이크를 통해서 연결을 수립하지만 SCTP는 네 방향 핸드셰이크(INT-ACK, COOKIE-ECHO) 를 통해서 연결을 수립하고, 이때 연결 정보에 쿠키를 삽입하여 DOS와 같은 네트워크 공격으로 부터 보호한다.

### ChannelHandlerContext 객체

- 두 가지 네티 객체에 대한 상호작용을 도화주는 인터페이스이다.
    1. 채널에 대한 입출력 처리 ( writeAndFlush 메소드로 채널에 데이터를 기록한다.) 
    2. 채널 파이프라인에 대한 상호작용이다. 채널 파이프라인에 대한 상호작용으로는 사용에 의한 이벤트 발생과 채널 파이프라인에 등록된 이벤트 핸들러의 동적 변경이 있다.

        1.  채널 파이프라인에 등록된 이벤트 핸들러의 동적변경 : ChannelHandlerContext는 채널이 초기화 될 때 설정된 채널 파이프라인을 가져오는 메소드를 제공한다. 그러므로 ChannelHandlerContext를 통해서 설정된 채널파이프라인을 수정할 수 있다. 

        2. 사용자에 의한 이벤트 발생 : 데이터 수신 이벤트 메소드에서 논리적인 오류가 발생했다 하면 이 이 오류를 처리하는 공통 로직이 이미 exceptionCaught 이벤트 메소드에 작성되어 있다고 하면 ChannelHandlerContext의 fireExceptionCaught 메소드를 호출 하면 된다. fireExceptionCaught 메소드를 호출하면 채널 파이프라인으로  exceptionCaught 이벤트가 전달되고 해당 이벤트 메소드가 수행된다. 

### flip() 메소드

### Executor

### SO_LINGER

- 소켓은 close 한다고 해서 바로 소켓이 닫히지 않는다. 일정 시간 동안 대기 상태가 되는데 그 대기시간이 소켓의 TIME_OUT 상태이다. 이 TIME_OUT 상태의 시간을 SO_LINGER 옵션을 통해서 바로 소켓을 닫게 할 수가 있다.
- TIME_OUT 이라는 상태를 두는 이유는 TCP는 신뢰성을 보장하는 프로토콜이기 때문에 A와 B가 TCP로 연결되어 있을때 A가 B에게 접속을 끊는다 라고 신호를 보내고 나서, 서로 3번에 걸쳐 이 신호가 오고 가서 접속을 끊기 때문이다.

### Packet

- 데이터를 주고 받을 때 데이터를 적절한 크게의 묶음으로 만들어 놓은 것이다. 네트워크를 통해 전송하기 쉽도록 자른 데이터의 전송단위이다.

### org.jboss.netty.handler.timeout 과 handler 예제

- org.jboss.netty.handler.timeout은 Timer를 사용하여 Read와 Write의 Timeout과 idle connection의 알림을 위해서 추가되었다.
    - IdleSateHandler - 한 채널에서 read 나 write, 혹은 둘다 특정기간동안 수행되지 않을 떄 IdleStateEvent를 발생시킨다. 인자값으로 ReadIdleTime, WriteIdelTime, AllIdelTime 을 순서대로 입력 받고, 단위는 Second 로 만약 0 을 설정했을 경우 Idle 동작은 Disable 된다.
    - ReadTimeoutHandler - 특정하게 정해진 기간내에 Data를 읽을 수 없을 때 ReadTimeoutException을 발생시킨다.
    - writeTimeoutHandler - 특정하게 정해진 기간내에 Data를 읽을 수 없을때 WriteTimeoutException을 발생시킨다.
- Idle state와 TimeException
    - IdleState - 채널의 Idle 상태를 표시해주는 Enum 이다. 
    ALL_IDLE, READER_IDLE, WRITER_IDLE 3개자의 상태값이 존재한다.
    - TimeoutException 특정하게 정해진 기간내에 읽기나 쓰기 각각 발생하지 않았을 경우 발생한다.
    - ReadTimeoutException - 특정하게 정해진 기간내에 Data의 read를 할 수 없을 경우 ReadTimeoutHandler에 의해서 발생된다.
    - WriteTimeoutException 특정하게 정해진 기간내에 Data를 write 할 수 없을 경우 WriteTimeoutHandler 에 의해서 발생된다.
    

# 소켓 옵션 설정

setsockopt

소켓의 송수신 동작을 setsockopt() 함수를 이용해서 다양한 옵션으로 제어할 수 있습니다. 

    int setsockopt (SOCKET socket, int level, int opt name, const void* optval, int optlen);

- socket : 소켓의 번호
- level : 옵션의 종류, 보통 SOL_SOCKET 과 IPPROTO_TCP 중 하나를 사용
- optname : 설정을 위한 소켓 옵션의 번호
- optval : 설정 값이 저장된 주소값
- oplen : optval 버퍼의 크기

## **SOL_SOCKET 레벨**

`setsockopt()` 함수의 `level`에 들어갈 값입니다.

[SOL_SOCKET LEVEL](https://www.notion.so/cc5f49c57f1e4647ac96810e2f4e375e)

# 주요 객체

**Selector** : 자신에게 등록된 채널에 변경 사항이 발생했는지 검사하고 변경 사항이 발생한 채널에 대한 접근을 가능하게 해준다. 

**ServerSocketChannel** : non-blocking 소켓의 서버 소켓 채널 (소켓을 먼저 생성하고 사용할 포트를 바인딩 한다.

**serverSocketChannel.configureBlocking(false**) : 소켓 채널의 블로킹-모드를 논블로킹-모드로 설정하는 방법.

**serverSocketChannel.bind()** : 클라이언트의 연결을 대기 할 포트를 지정하고 생성된 serverSocketChannel에 연결을 생성 한다.

**serverSocketChannel.register(selector,SelectionKey.OP_ACCEPT) :** serverSocketChannel 객체를 Selector 객체에 등록한다. Selector 가 감지 할 이벤트 연결 요청에 해당하는 SelectionKey.OP_ACCEPT 이다.

**selector.select() :**  selector에 등록된 채널에서 변경 사항이 발생했는지 검사한다. 

**Iterator<SelectionKey> keys = selector.selectedKeys().iterator() : Selector**에 등록된 채널 중에서 I/O이벤트가 발생한 채널들의 목록을 조회한다.

**Bootrap** : 소켓 연결을 요청한다. 각 Bootstrap 클래스는 AbstractBootstrap 클래스와  Channel  인터페이스를 상속받고 있다.

**ServerBootstrap** : 소켓 연결을 대기한다. 클라이언트 접속을 대기할 포트를 설정하는 메소드가 추가되어 있다. 나머지 구조는 Bootstrap과 같음.

**NioEventLoopGroup** : 생성자의 인수로 사용되는 숫자는 그룹 내에서 생성할 최대 스레드의 수를 의미, 빈 칸으로 설정 시 스레드 수를 하드웨어 코어 수를 기준으로 결정. ( 하드웨어가 가지고 있는 CPU 코어 수의 2배를 사용)  

**channel**  : AbstractBootstrap 추상 클래스의 구현체인  ServerBootstrap 과  Bootstrap 클래스에 모두 존재하는 API 이다.|

**LoggingHandler** :  네티에서 기본으로 제공하는 코덱. 채널에서 발생하는 모든 이벤트를 로그로 출력해준다.

**option** : 소켓 옵션을 설정하는 API. SO_SNDBUF → 소켓이 사용할 송신 버퍼의 크기를 지정. (커널에서 사용되는 값을 변경 한다는 의미), SO_RCVBUF → 커널의 수신 버퍼 크기를 지정하는데 사용.

**Channel pipeline ****: 채널에서 발생한 이벤트가 이동하는 통로 

**Event Handler ****: 위의 통로에서 이동하는 이벤트를 처리하는 클래스 

**Codec ****: 이벤트 핸들러를 상속받아서 구현한 구현체들이 코덱이다.

**io.netty.handler.codec** : 이벤트 핸들러를 미리 구현해둔 코덱 묶음 패키지

**ChannelInboundHandler** :  네티는 소켓 채널에서 읽을 데이터가 있다는 이벤트를 채널 파이프라인으로 흘려보내고 채널 파이프라인에 등록된 이벤트 핸들러 중에서 인바운드 이벤트 핸들러가 해당 이벤트에 해당하는 메소드를 수행함. 네티는 인바운드 이벤트를 ChannelInboundHandler 인터페이스로 제공한다. 

**channelRead** : 수신된 데이터는 네티의  ByteBuf 객체에 저장되어 있으며 이벤트 메소드의 두 번째 인자인 msg를 통해서 접근 할 수 있다. 

**channelReadComplete** : 채널의 데이터를 다 읽어서 더 이상 데이터가 없을 때 이 이벤트가 발생한다.

**flush** : 채널 버퍼에 저장된 데이터를 상대방으로 즉시 전송하는 메소드.

**channelInactive**  : 채널에 대한 입출력 작업을 수행할 수 없음 

**channelUnregistered** : 채널이 이벤트 루프에서 제거됬을 때 발생. 채널에서 발생한 이벤트를 처리 할 수 없게됨.

**channelOutboundHandler** : 네티 사용자가 요청한 동작에 해당하는 이벤트를 말함. 연결 요청, 데이터 전송, 소켓 닫기 등이 있다. 채널인바운드핸들러와 마찬가지로 인터페이스로 제공한다. 또 한 모든 채널아웃바운드핸들러는 ChannelHandlerContext 객체를 인수로 받는다. 

**ChannelHandlerContext** 객체 : 두 가지 네티 객체에 대한 상호작용을 도화주는 인터페이스. 
**1.** 채널에 대한 입출력 처리. ( writeAndFlush 메소드로 채널에 데이터를 기록한다. 또 는 ChannelHandlerContext의 close메소드로 채널의 연결을 종료 할 수 있다. ) 
**2.** 채널 파이프라인에 대한 상호작용. ( 사용자에 의한 이벤트 발생과 채널 파이프라인에 등록된 이벤트 핸들러의 동적 변경) 채널이 초기화 될 때 채널 파이프라인을 가져오는 메소드를 제공. 그러므로 ChannelHandlerContext를 통해서 설정된 채널 파이프라인을 수정 할 수 있다. 데이터 수신 이벤트 메소드에서 오류가 발생했을때 이 오류를 처리하는 공통 로직이 이미 exceptionCaught 이벤트 메소드에 작성이 되어있다면  ChannelHandlerContext의 fireExceptionCaught 메소드를 호출하면 된다.

**fireChannelRead** : ChannelHandlerContext 인터페이스의  fireChannelRead 메소드를 호출하면 네티는 채널 파이프라인에 channelRead 이벤트를 발생시킨다.

**네티의 코덱**은 템플릿 메소드 패턴으로 구현되어 있음. 여기서 템플릿 메소드 패턴이란 상위 구현체에서 메소드의 실행 순서만을 지정하고 수행될 메소드의 구현은 하위 구현체로 위임하는 것이다.   

네티의 모든 **이벤트**는 채널 파이프라인을 통해서 이동되며 이벤트 핸들러가 특정 이벤트를 수신하여 처리한다.

**StringDecoder, StringEncoder** : 네티에서 제공하는 문자열 디코더/인코더

**DelimiterBasedFrameDecoder** : 네티가 제공하는 기본 디코더로써 구분자 기반의 패킷을 처리한다.

**@Sharable** : 이 어노테이션은 네티가 제공하는 공유가능 상태표시 어노테이션이다. Sharable로 지정된 클래스를 채널 파이프라인에서 공유할 수 있다는 의미이다. 다중 스레드에서 스레드 경합없이 참조가 가능하다.

**Sharable** : 어노테이션을 표시하기 위한 다른 한가지 제약 조건은 스레드 안전성을 지원해야 한다는 것이다.  

**channelActive**  : 메소드는 채널이 생성된 다음 바로 호출되는 이벤트이다. 통상적으로 채널이 연결된 직후에 수행할 작업을 처리할 때 사용하는 이벤트이다.

**childHandler(ChannelHandler)** : Channel의 요청을 수행하기 위한 메소드 

**ChannelFuture** : 비동기 Channel I/O 수행의 결과를 ChannelFuture 인스턴스로 반환한다.

**channel()** : 연결된 인스턴스의 channel을 반환한다.

**closeFuture()** : ChannelFuture 을 반환하고 채널이 닫힌것을 알려준다.

**EmbeddedChanne**l : 강제로 네티 이벤트를 발생시키기 위해 ChannelHandlerContext 객체를 생성해야하며 이에 따른 이벤트 루프를 설정 해주는 복잡한 과정을거칠 필요 없이 핸들러를 테스트 할 수 있는 클래스이다. 

**EmbeddedChannel.writeInbound()** :  EmbeddedChannel 객체의 인바운드에 기록한다. 클라이언트로 부터 데이터를 수신한 것과 같은 상태가 된다.
