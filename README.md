
## OS_Study_2022
---

### Thread pool 
---

> Thread Per request model
<img width="314" alt="스크린샷 2022-12-11 오후 6 30 23" src="https://user-images.githubusercontent.com/51349774/206896139-244ffa83-7a4d-4254-839e-a944aa8b0de8.png">

- 하나의 요청이 하나의 스레드에 맵핑되어 처리된다.
- 스레드를 생성하는 것은 커널에서 이뤄지기 떄문에 스레드 생성은 단순 연산 작업보다는 시간이 더 소요된다
- 때문에 요청마다 새로운 스레드를 생성하고 작업이 끝난 스레드는 버리는 형식은 시간이 많이 소요될 수 있다.
- 또 서버의 처리속도보다 더 빠르게 요청이 늘어난다면 컨텍스트 스위칭이 더 자주 발생되고 CPU 오버헤드 증가 CPU time이 낭비된다다.
- 스레드는 각각 메모리를 점유하기 때문에 메모리가 고갈 된다.
- 이러한 점을 해결하기위해 thread pool이 적용된다.

> Thread pool 동작방식

<img width="431" alt="스크린샷 2022-12-11 오후 6 41 07" src="https://user-images.githubusercontent.com/51349774/206896600-9b3cd0af-e9fd-4f6a-8d59-1b23f8b539fd.png">

- 각각의 요청은 스레드 풀의 queue에 쌓인다. 
- 스레드풀에는 미리 스레드들을 준비해두어 요청에 준비해둔 스레드를 할당한다.
- 작업을 마친 스레드는 스레드 풀로 돌아와 재사용된다.
- 제한된 개수의 스레드를 운용하기때문에 스레드가 무제한으로 생성되는 것을 방지한다.

> Thread pool이 사용되는 사례

- thread per request 모델
- task를 subtask로 나누어서 동시에 처리하는 경우
- 순서 상관없이 동시 실행이 가능한 task

> Thread pool 사용 주의사항

1. 스레드 풀에 몇개의 스레드를 만들어 두는 것이 좋은가
- cpu의 코어 개수와 task의 성향에 따라 다르다.
- cpu-bound task 라면 코어 개수 만큼 혹은 그 보다 몇개 더 많은 정도의 스레드가 필요
- IO-bound task 라면 경험적으로 얼마나 IO-bound인지에 따라 스레드 수를 맞추는 것이 좋다.

2. 스레드 풀에서 실행될 task 개수에 제한이 없다면?
- 큐에 요청이 무한정 쌓인다면 그만큼 메모리를 점유하고 고갈될 수 있다.
- 반드시 제한을 둬서 큐가 다차면 요청을 버리도록하여 전체 시스템에 장애를 일으키는 것을 막는 것이 좋다.
- 예를 들어 자바에서 스레드를 관리할 수 있게해주는 클래스인 Executors의 newFixedThreadPool 메서드는
- capacity가 default로 Integer.MAX_VALUE(20억이 넘는다.)로 되어있기 때문에 확인하고 큐사이즈를 지정해 주는 것이 좋다

### block I/O 과 non-block I/O 
---

> I/O의 종류

- network(socket) : 네트워크 통신에서 사용됨
- file : 파일 읽기 쓰기에 사용됨
- pipe : process 간의 통신에 사용되는 형식
- device : 모니터 키보드와 같은 장치와의 통신에 사용됨

> socket이란

- 프로세스들끼리 네트워크 통신을 할때 사용되는 파일 형태의 인터페이스이다.
- 클라이언트의 요청은 서버에서 각각에 대응하는 소켓을 통해 데이터를 주고 받게된다.

### block I/O 

- I/O 작업을 요청한 프로세스 / 스레드는 요청이 완료될 때까지 블락된다.

<img width="334" alt="스크린샷 2022-12-11 오후 4 57 44" src="https://user-images.githubusercontent.com/51349774/206892639-ddce02b6-04e4-4d23-ae92-dbe13598a10a.png">

1. 스레드가 실행되고 read라는 system call이 호출 되면 kernel모드로 전환된다.
2. block I/O일 경우 이때 thread가 block 된다.
3. kernel 에서는 read I/O를 실행하고 관련된 디바이스에 데이터를 요청한다.
4. 요청을 받은 디바이스는 데이터 전송 준비를 마치면 커널로 응답을 준다.
5. 4번이 일어난후 데이터가 커널로 옮겨지고 커널은 다시 user space로 데이터를 옮긴다.
6. 데이터가 옮겨진 후(I/O 작업 종료) thread의 block이 해제되고 
7. I/O 작업을 통해 준비된 데이터를 읽어 다음 작업을 이어간다.


> Socket에서의 block I/O

<img width="358" alt="스크린샷 2022-12-11 오후 5 14 11" src="https://user-images.githubusercontent.com/51349774/206893337-9e705e7d-03da-45e6-a8a9-573d82006fcc.png">

- 소켓은 send buffer(데이터를 보낼때), receive buffer(데이터를 받을때) 두가지 버퍼를 갖는다.
- 위 그림에서처럼 socket b가 socket a에게 데이터를 보낸다고 할때 
- socket A receive bufferd에 데이터가 올때까지 
- read()를 호출한 스레드는 block이 걸린다.
- socket B는 반대로 데이터를 보낼때 send buffer에 데이터를 옮겨서 보내는데 
- 버퍼가 가득찰 경우 write를 호출한 스레드는 block이 걸린다.  

---

### non-block I/O 

- 프로세스 / 스레드를 블락시키지 않고 요청에 대한 현재 상태를 즉시 리턴하여 스레드가 다른 작업을 수행할 수 있게 한다.


> non-block I/O의 진행 흐름


<img width="535" alt="스크린샷 2022-12-11 오후 5 53 22" src="https://user-images.githubusercontent.com/51349774/206894748-0c6bcac5-cacc-4934-852e-ca6ad837d877.png">

 
 1. 스레드가 read()를 non-block으로 호출 
 2. 커널모드로 컨텍스트가 스위칭되고 커널은 read I/O 작업을 수행한다.
 3. non-block I/O에서는 이때 응답이 리턴된다.(리눅스 기준으로 -1이라는 값과 ,EAGAIN or EWOULDBLOCK의 에러코드가 리턴됨)
 4. 응답을 받고 스레드가 다시 run한다. 
 5. I/O작업이 끝나 응답이 넘어오면
 6. 다시 read()를 호출하여 , 준비된 데이터를 user space로 전송한다.
 7. user space로 넘어온 데이터를 읽어 쓰레드는 작업을 이어간다.


> non-block I/O의 단점

- non block I/O 에서는 I/O 작업 완료를 어떻게 확인할 것인지에 대한 해결이 필요하다.

1. 완료가 되었는지 반복적으로 확인한다.
- 위의 그림에서와 같이 실제 IO 작업이 종료되는 시점과 I/O 작업의 종료를 확인하는 시점에 사이에 시간 지연이 일어난다
- 반복적으로 완료여부를 확인하며 cpu 낭비가 일어난다.

2. I/O multiplexing(다중 입출력)
- 네트워크 통신에서 많이 사용된다.
- 여러개의 관심있는 I/O 작업들을 동시에 모니터링 하고 완료된 I/O 작업을 한번에 notify 해준다.
- select(성능 나쁨) , poll(성능 나쁨) , epoll(리눅스에서 사용) , kqueue(mac OS에서 사용), IOCP(window , solalis 계열에서 사용) 네가지 I/O multiplexing이 있다.

3. callback 또는 signal 형태로 응답
- 많이 사용되진 않는다.
- posix AIO, linux AIO 두가지가 있다.

---

### 스레드의 종류

> hardware thread

- 연산처리를 하는 중에 메모리에서 데이터를 읽어올때 코어는 연산을 하지않고 데이터가 올때까지 기다린다(코어 낭비).
- 이를 해결하기위해 한 스레드에서 메모리를 불러오는 작업을 할때 또 다른 스레드가 코어를 점유하는 방식으로 서로 번갈아가며 코어를 점유하는 방식이 사용된다.
- 이때 각각의 스레드를 하드웨어 스레드라 부른다.
- 인텔에서 hyper - threading이라고 부르며 물리적인 코어마다 하드웨어 스레드가 두개씩 있다.
- 만약 싱글 코어 CPU에 하드웨어 스레드가 두개라면 OS는 이 CPU를 듀얼 코어로 인식하고 듀얼 코어에 맞춰서 OS레벨의 스레드들을 스케줄링 한다.

> OS thread

- OS 커널 레벨에서 생성되고 관리되는 스레드를 말한다.
- CPU에서 실제로 실행되는 단위이다(CPU 스케쥴링의 단위)
- OS 스레드의 컨텍스트 스위칭은 커널이 개입하고 비용이 발생한다.
- 사용자 코드와 커널 코드 모두 OS스레드에서 실행된다.(유저모드 코드가 진행되다가 system call이 호출되면 커널모드로 전환)
- 네이티브 스레드 , 커널 스레드 , 커널 - 레벨 스레드 ,os-레벨 스레드 라고도 불리며
- 네이티브가 OS를 의미하기 떄문에 보통 네이티브 스레드라고 부른다.

> user thread (user level thread)

- 스레드 개념을 프로그래밍 레벨에서 추상화한 것을 말한다.
- 자바에서 new Thread()를 통해 생성하는 스레드가 유저 스레드에 해당한다
- thread.strat() 함수는 start0라는 메서드를 호출한다 start0는 JNI를 통해 OS에 System call 을 호출한다.
- linux의 경우 이때 clone이 호출되고 OS레벨의 스레드가 생성된다. 생성된 OS 레벨의 스레드는 user level의 thread 객체와 연결된다.
- 유저스레드는 반드시 OS스레드와 연결되어야 CPU에 의해 실행된다. 


--- 

### user thread와 OS thread의 연결방식

> One-to-One model

<img width="339" alt="스크린샷 2022-12-16 오후 6 31 57" src="https://user-images.githubusercontent.com/51349774/208067960-a1647f38-73e8-4e55-b667-253a98083e42.png">

- 자바에서 사용되는 스레드 연결방식이다.
- 유저스레드와 OS 스레드가 1:1로 연결된다.
- 스레드 관리(생성 , 삭제 , 스케쥴 관리등..)를  OS(Kernel)에 위임한다.
- OS에서 스케쥴링을 관리하기 떄문에 멀티코어 환경에서도 잘 동작한다.
- 1:1로 스레드가 연결되기 때문에 한 스레드가 block-IO로 수행된다해도 다른 스레드는 잘 동작한다.
- 경우에 따라 race condition이 발생할 수 있다.



> Many-to-One Model
> 
<img width="339" alt="스크린샷 2022-12-16 오후 7 08 09" src="https://user-images.githubusercontent.com/51349774/208074958-18531ee6-2638-48bd-96a7-d137d7359843.png">

- 유저 스레드가 여러개가 하나의 OS 레벨 스레드에 연결된 형태
- 컨텍스 스위칭이 user 레벨에서만 일어나고 kernel이 개입하지 않기떄문에 훨씬 빠르게 스위칭된다.
- OS 스레드에서 race condition이 일어날 가능성이 적다 
- OS 스레드가 하나이기 떄문에 멀티코어를  활용할 수 없다.
- 여러 user thread가 하나의 os thread에 연결되어 있기 떄문에 하나의 user thread가 block IO를 수행한다면
- 다른 스레드가 block에 걸리기 때문에 주로 non-block IO를 사용한다 

> Many-to-Many Model
<img width="342" alt="스크린샷 2022-12-16 오후 7 08 03" src="https://user-images.githubusercontent.com/51349774/208074937-0d744cc5-0e06-4a1a-a869-6589bd65e926.png">

- 여러개의 user thread와 거기에 맞게 적당한 os thread가 사용된다.
- 여러개의 user thread 들이 kernel에 의해 OS thread에 분배된다.
- GO lang에서 사용되는 모델이다.
- Many to One 과 one to one model의 장점이 합쳐져있다.
- 구현이 복잡하다

> green thread

- 자바의 초창기 버전은 Many-to-One 모델이 사용되었는데
- 이때의 유저 스레드들을 그린 스레드라고 불렀다.
- 요즘은 개념이 확장되어 OS와는 독립적으로 유저 레벨에서 스케줄링되는 스레드를 의미한다.
- many-to-one , many-to-many model의 user thread들을 의미하기도 한다.

--- 

### user mode 와  kernel mode

> kernel이란?

- 운영체제의 핵심이다 시스템의 전반을 관리/감독하고 하드웨어 관련된 작업을 직접 수행한다.
- 시스템을 보호하기 위해서 kernel mode가 만들어 졌다.

> user mode ->kernel mode 로 전환되는 과정

1. user mode 프로그램 실행 중에 인터럽트가 발생 하거나 시스템 콜을 호출하게 되면 커널모드가 실행된다.
2.  kernel mode로 전환되면 맨 처음으로 실행되던 프로그램의 cpu상태를 저장한다.
3.  다음으로 kernel이 인터럽트 혹은 시스템콜을 직접 처리한다.(cpu에서 커널 코드가 실행된다.)
4.  처리가 완료되면 중단됐던 프로그램의 cpu 상태를 복원하고 다시 통제권을 프로그램에게 반환한다.
5.  kernel mode에서 user mode로 돌아오고 프로그램이 이어서 실행된다.

> Interrupt
- 시스템에서 발생한 다양한 종류의 이벤트 혹은 이벤트를 알리는 메커니즘을 말한다.
- interrupt가 발생하면 cpu에서는 즉각적으로 커널 코드로 인터럽트를 처리하기 위해 kernel mode로 전환된다.

> 대표적으로 interrupt가 발생하는 경우

- 전원에 문제가 생겼을 때
- I/O 작업이 완료됐을 때
- 시간이 다 됐을 때(timer 관련)
- 0으로 나눴을 때 (program 레벨에서 발생하기 때문에 trap이라고도 부른다)
- 잘못된 메모리 공간에 접근을 시도할 때(program 레벨에서 발생하기 때문에 trap이라고도 부른다)

> System call

- 프로그램이 OS커널이 제공하는 서비스를 이용하고 싶을때 시스템 콜을 통해 실행한다.
- 시스템 콜이 발생하면  해당 콜에 대응하는 각각의 커널 코드가 커널모드에서 실행된다.
- 자바 , 파이썬등 하이 레벨 언어에서는 간접적으로 시스템 콜을 사용할 수 있도록 기능들이 추상화되어 있기 때문에 
- 쉽게 시스템 콜을 사용할 수 있다. 
- 자바의 경우 native라는 예약어가 붙은 메서드는 JNI(java native interface)를 통해 시스템 콜을 호출하는 메서드이다 

> 시스템 콜의 종류

- 프로세스 / 스레드 생성 kill 관련
- 파일  I/O 관련
- 소켓 관련(네트워크)
- 장치 관련(device)
- 프로세스 통신 관련

> 시스템 콜 , 인터럽트 호출 예시 (파일 read)

-  두개의 스레드가 시스템 콜과 인터럽트가 호출 되는 예시이다 
-  t1은 파일을 불러오는 system call을 호출하고 있고 running 상태이다. 
-  싱글 코어 멀티테스킹 방식으로 동작하고 있다.

<img width="788" alt="스크린샷 2022-12-16 오후 7 53 45" src="https://user-images.githubusercontent.com/51349774/208083096-0ec27d46-0c86-4b55-a5b8-660e38413193.png">


### context switching

- CPU/코어에서 실행 중이던 프로세스 / 스레드가 다른 프로세스/스레드로 교체되는 것을 말한다.
- 여러 프로세스/스레드를 동시에 실행시키기 위해 필요하다.
- 주어진 timeslice(quantum)을 다 사용했거나 IO 작업을 해야하거나 다른 리소스를 기다리는등의 작업에서 발생된다.
- OS kernel에 의해 실행되며 cpu의 레지스터 상태를 교체한다.
- 다른 프로세스간의 컨텍스트 스위칭(process context switching), 같은 프로세스 안에서의 컨텍스트 스위칭(thread context switching) 두가지로 나뉜다.


> process context switching과 thread context switching

- process context switching의 경우 프로세스간의 가상 메모리 주소가 다르기 떄문에 메모리 관련 처리를 추가로 해줘야한다.

--- 

### CPU sceduler와 dispatcher

> CPU scheduler 란 ?

- cpu가 지속적으로 작동할 수 있도록 실행할 프로세스를 선택하는 역할을 한다
- cpu의 ready queue(실행을 기다리는 ready 상태의 프로세스들이 queue 형태로 모여 있는 곳)

> dispatcher 란 ? 

- 선택된 프로세스에게 CPU를 할당하는 역할
- 선택된 프로세스를 cpu에서 실행될 수 있는 형태로 만들어주는 역할을 한다.
- context switching을 담당.
- kernel mode -> user mode로의 전환
- 선택된 프로세스가 작업을 시작할 적절한 위치로 이동시킨다

> scheduling 선점 방식

 1. Nonpreemptive (비선점) scheduling
 - 실행중인 프로세스가 종료되거나 , IO 작업을 위해 waiting 상태로 가는 것 , 자발적으로 ready상태로 넘어가는 것이 비선점 방식에 해당한다.
 - 신사적, 협력적(cooperative)  , 느린 응답성인 특성을 갖는다(한 프로세스가 자발적으로 끝날때까지 다른 프로세스는 기다려야 하기 떄문에)

 2. Preemptive(선점) scheduling
 - 비선점 방식을 기본적으로 포함하고 추가적으로 
 - 실행중인 프로세스를 강제적으로 ready상태로 변경시킨다.
 - 특정 프로세스에 우선순위를 주거나 , time slice가 모두 소진 되었을때 발생할 수 있다.
 - 적극적 ,강제적 , 빠른 응답성 , 데이터 일관성 문제의 특징을 갖는다 . 


 

 