# 팍소스 알고리즘

---

팍소스 알고리즘은 분산컴퓨팅의 아버지라 불리는 레슬리 램포트\(Leslie Lamport\)에 의해 만들어졌다. 만들어진 이후로 팍소스 알고리즘은 컨센서스와 동의어로 사용되어 왔다. 대학에서 컨센선스를 강의한다고 하면 그 중 알고리즘 파트는 대부분 팍소스 알고리즘을 다룬다고 생각하면 된다. 가장 실용적인 컨센선스라 일컬어지는 것들은 팍소스를 바탕을 두고 있다.

팍소스 알고리즘은 모든 분산 시스템에 존재하는 모든 알고리즘 중 가장 중요한 알고리즘이라고 생각할 수 있다. 팍소스에 대해 첫 번째로 다뤄볼 것은 복제로그의 생성과 이 생성된 복제로그를 통해 복제된 상태 머신이 생성되는 것에 대한 내용이다.

> I'm going to explain Paxos in the context of creating a replicated log and then we'll use the replicated log to create a replicated state machine.

강의 도중 언급되는 상태 머신\(혹은 유한 상태 기계\)은 단순히 프로그램 혹은 어플리케이션이라고 생각하면 된다. 단, 입력을 필요로 하고 이 입력으로부터 출력이 생성되며 기계 내 상태 정보를 저장할 수 있어야 한다. 단순히 프로그램 또는 어플리케이션 이라고 생각해도 무방하다.

> when I say state machine, I just mean a program or application that takes inputs and produces outputs and hold some internal state. so you can think of almost any program or application as a state machine.

이제 본격적인 아이디어에 대해 소개하고자 한다. 만약 우리가 신뢰성이 높은 상태 머신을 만든다고 해보자. 신뢰성이 높은 상태 머신을 만들기 위해 우리가 사용하고 싶은 방법은** 같은 상태정보를 가진 머신**들이 **각기 다른 서버에서 동시다발적으로 실행**되게끔 만드는 것이다.

또한 각 상태 머신에게 우리가 기계에게 내리고 싶은 **명령들을 같은 내용과 같은 순서로** 각 상태머신에게 줬을 때, 각 상태 머신은 우리가 내린 명령들을 순서대로 틀리지 않고 그대로 잘 수행해줄 것이며 똑같은 동작과 같은 출력을 만들어내게끔 우리는 상태머신을 만드려 한다.

> The idea is that we would like to make a state machine highly reliable and the way we would like to do that is by having the same state machine run concurrently on several different servers and if each of those state machines receives the same set of commands in the same order, then they should all behave identically and produce the same results.

만약 위와 같이 설계만 된다면 몇몇 기계들에 결함이 생겨도 다른 기계들이 내가 원하는 작업을 똑같이 수행할 수 있으니 별 문제가 발생하지 않는다. 매우 바람직하고 꿈만 같은 일이다.

**분산화되어 있는 각 상태머신**들이 **동일한 내용의 작업을 같은 순서로 실행**하게 만드는 것이 바로 **복제 로그의 최종 목표**다. 각 상태 머신들이 각자 받은 명령을 같은 순서로 정확하게 수행하는 것.

![](/assets/0.PNG)

지금부터 그 과정을 한 번 살펴보도록 하자. 먼저 로그에 명령들을 저장하고 모든 로그가 동일한 순서로 동일한 명령을 가지고 있는지 확인한다. 그 후  로그에 있는 명령을 처리하고 작업을 진행한다. \(단, 각 상태 머신이 모두 동일한 작업을 수행하는지는 우리가 보장하지 못한다.\)

클라이언트가 상태 머신에서 명령을 실행하려고 할 때 시스템이 작동하는 방법은 다음과 같다. \(위 템플릿의 그림 부분을 보면서 Consensus Module을 살펴보자\) 먼저 실행하고자 하는 명령을 서버 중 하나에 전송한다. 전송된 명령은 위의 템플릿 중 Log의 가장 우측에 있는 'shl' 명령어다. 전송을 받은 서버는 해당 명령을 자신의 로컬 로그에 저장하고 다른 서버들에게 해당 내용을 전파한다. 마찬가지로 다른 서버들 또한 받은 정보를 자신만의 로컬 로그에 저장해 둔다.

명령어가 안전하게 모든 로그들에 저장된 것이 확인된다면 그 다음으로 명령어의 실행을 위해 상태 머신에게 명령어 정보를 넘긴다. 

> Let's suppose the command is 'shl' that server records the command in its own local log and then passes the command to other servers and each of them records the command in its log.
>
> once the command has been safely replicated in all of the logs then it can be passed to the state machines for execution.

받은 정보를 토대로 상태 머신이 해당 명령어를 실행하면, 그 결과값이 비로소 클라이언트에게 돌아갈 수 있게 된다. 모든 머신이 갖고 있는 로그들이 동일하고 상태 머신들이 해당 명령들을 순서대로 잘 처리하기만 한다면 각 상태 머신들은 동일하게 작동한다는 걸 확인할 수 있다.



