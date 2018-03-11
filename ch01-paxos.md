# 팍소스\(Paxos\) 알고리즘

---

**팍소스 알고리즘**은 분산컴퓨팅의 아버지라 불리는 레슬리 램포트\(Leslie Lamport\)에 의해 만들어졌다. 만들어진 이후로 팍소스 알고리즘은 **Consensus\(합의\)**로 여겨져 왔다. 대학에서 Consensus\(합의\)를 강의한다고 하면 그 중 알고리즘 파트는 대부분 팍소스 알고리즘을 다룬다고 생각하면 된다. 가장 실용적인 Consensus\(합의\)라고 일컬어지는 것들은 팍소스를 바탕을 두고 있다.

팍소스 알고리즘은 모든 분산 시스템에 존재하는 모든 알고리즘 중 가장 중요한 알고리즘이라고 생각할 수 있다. 팍소스에 대해 첫 번째로 다뤄볼 것은 **복제로그의 생성**과 이 생성된 복제로그를 통해 **복제된 상태 머신이 생성되는 것**에 대한 내용이다.

> I'm going to explain Paxos in the context of creating a replicated log and then we'll use the replicated log to create a replicated state machine.

강의 도중 언급되는 상태 머신\(혹은 유한 상태 기계\)은 단순히 프로그램 혹은 어플리케이션이라고 생각하면 된다. 단, 입력을 필요로 하고 이 입력으로부터 출력이 생성되며 기계 자체의 상태 정보를 저장할 수 있어야 한다. 단순히 프로그램 또는 어플리케이션 이라고 생각해도 무방하다.

> when I say state machine, I just mean a program or application that takes inputs and produces outputs and hold some internal state. so you can think of almost any program or application as a state machine.

이제 본격적인 아이디어에 대해 소개하고자 한다. 만약 우리가 신뢰성이 높은 상태 머신을 만든다고 해보자. 신뢰성이 높은 상태 머신을 만들기 위해 우리가 사용하고 싶은 방법은** 같은 상태정보를 가진 머신**들이 **각기 다른 서버에서 동시다발적으로 실행**되게끔 만드는 것이다.

또한 각 상태 머신에게 우리가 기계에게 내리고 싶은 **명령들을 같은 내용과 같은 순서로** 각 상태머신에게 줬을 때, 각 상태 머신은 우리가 내린 명령들을 순서대로 틀리지 않고 그대로 잘 수행해줄 것이며 똑같은 동작과 같은 출력을 만들어내게끔 우리는 상태머신을 만드려 한다.

> The idea is that we would like to make a state machine highly reliable and the way we would like to do that is by having the same state machine run concurrently on several different servers and if each of those state machines receives the same set of commands in the same order, then they should all behave identically and produce the same results.

만약 위와 같이 설계만 된다면 몇몇 기계들에 결함이 생겨도 다른 기계들이 내가 원하는 작업을 똑같이 수행할 수 있으니 별 문제가 발생하지 않는다. 매우 바람직하고 꿈만 같은 일이다.

**분산화되어 있는 각 상태머신**들이 **동일한 내용의 작업을 같은 순서로 실행**하게 만드는 것이 바로 **복제 로그의 최종 목표**다. 각 상태 머신들이 각자 받은 명령을 같은 순서로 정확하게 수행하는 것.

![](/assets/0.PNG)

지금부터 그 과정을 한 번 살펴보도록 하자. 먼저 **로그에 명령들을 저장하고 모든 로그가 동일한 순서로 동일한 명령을 가지고 있는지 확인**한다. 그 후  로그에 있는 **명령을 처리하고 작업을 진행**한다. \(단, 각 상태 머신이 모두 동일한 작업을 수행하는지는 우리가 보장하지 못한다.\)

클라이언트가 상태 머신에서 명령을 실행하려고 할 때 시스템이 작동하는 방법은 다음과 같다. \(위 템플릿의 그림 부분을 보면서 Consensus Module을 살펴보자\) 먼저 클라이언트가 자신이 실행하고자 하는 명령을 서버 중 하나에 전송한다\(위 템플릿에서는 가장 우측에 있는 서버에게 전송이 되었음\). 전송된 명령은 Log의 가장 우측에 있는 'shl' 명령어다. 전송을 받은 서버는 해당 명령을 자신의 로컬 로그에 저장하고 다른 서버들에게 해당 내용을 전파한다. 마찬가지로 다른 서버들 또한 받은 정보를 자신만의 로컬 로그에 저장해 둔다.

명령어가 안전하게 모든 로그들에 저장된 것이 확인된다면 명령의 실행을 위해 상태 머신에게 명령어 정보를 넘긴다.

> Let's suppose the command is 'shl' that server records the command in its own local log and then passes the command to other servers and each of them records the command in its log.
>
> once the command has been safely replicated in all of the logs then it can be passed to the state machines for execution.

받은 정보를 토대로 상태 머신이 해당 명령어를 실행하면, 그 결과값이 비로소 클라이언트에게 돌아갈 수 있게 된다. 모든 머신이 갖고 있는 로그들이 동일하고 상태 머신들이 해당 명령들을 순서대로 잘 처리하기만 한다면 각 상태 머신들은 동일하게 작동한다는 걸 확인할 수 있다.

로그가 제대로 복제되었는지를 확인하는 것이 **컨센서스 모듈\(합의 모듈\)의 역할이**다. 우리가 팍소스를 공부하는 이유기도 하다.

컨센서스를 기반으로 한 접근 방식의 가장 중요한 핵심은 시스템이 대부분의 서버가 정상 작동하는 한 모든 서비스를 제공할 수 있다는 것이다. 우리가 5개의 서버로 이루어진  **클러스터\(군집, 집단\)**를 갖고 있다면, 클러스터 중 3개의 서버가 정상 작동하는 한 서비스 제공에는 아무 문제가 없다. 즉, 클러스터 중 2개의 서버가 다운되는 상황을 견딜 수 있는 것이다. 일반적으로 클러스터\(cluster\)의 사이즈는 3, 5 또는 7과 같은 작은 홀수이다.

## 팍소스 알고리즘의 failure model

이 파트는 실패모델에 대해 다룬다. 서버들이 망가지거나 멈추거나 재시작할 수도 있으니 말이다. 이것이 첫 번째로 생각할 수 있는 실패 가능성이다.

단, 서버가 일단 정상적으로 작동하기만 하면 서버는 자신의 일을 올바르게 처리한다. 흔히 알려진 비잔틴 장군 문제와는 다르게 서버들은 악의적인 방법으로 동작하지 않는다\(어디까지나 이 모델만의 가정이다\).

> This is a fail stop model which means that servers may crash or they may stop and restart. But when they're running, they always behave correctly.
>
> They do not behave in malicious fashions \(so-called Byzantine failures\).

두 번째 실패 가능성은 네트워크 상에 전송된 메시지가 실종되거나 지연될 수 있는 것이다. 즉, 메시지가 처음 보낸 것과는 다른 순서로 도착지에 도착하거나 통신이 끊어진 채로 네트워크가 잠시 분할될 수도 있으며 분할된 네트워크가 다시 복구되어 정상 통신이 가능해지는 상황이 반복되는 등 여러가지 상황 발생할 수 있다.

> we also assume that any message of the network can be lost or they can be delayed which means they can potentially arrive in different orders from what they were sent or the network could partition for a while with communication chopped off and then eventually the partition could be repaired to allow communication again.

하지만 일단 메시지가 네트워크 망을 통과하면 중간에 깨지는 일 없이 안전하게 통과하고, 서버의 동작은 앞서 언급했듯 항상 올바르게 동작할 것이라는 가정은 한다.

> when message get through they get through safely.
>
> they are not corrupted and when servers are operation they're operating correctly.

복제 로그를 구현하는 문제를 분해하는 몇 가지 방법이 있다. 팍소스 알고리즘은 우선 가장 단순하면서 쉽게 생각해볼 수 있는 컨센서스 문제로 시작한다. basic paxos 혹은 single degree paxos라고 불린다.

![](/assets/1.PNG)

서버들이 모여 있는 집합이 있고 그 중 몇몇 서버는 특정 값을 각 서버들에게 제안할 수가 있다. basic paxos의 목표는 **제시된 값들 중 정확히 하나**의 값을 고르는 것이고, 그 골라진 값을 **'선택완료\(Chosen\)' **됐다고 한다.

> in this problem, there's a collection of servers and some of them may propose particular values.
>
> the goal of **basic Paxos** is pic exactly one of those values and if that values picked it's called **chosen.**

**단 하나의 값을 한 순간**에\(컴퓨터의 성능이나 알고리즘의 합의 방식에 따라 이 시간은 달라질 수 있음\) **결정하는 것이 팍소스가 하는 일**이다. 두 번째 값을 선택하는 일도 없고, 선택을 취소한다거나 번복하는 일은 없다. 이것이 우리가 떠올릴 수 있는 가장 단순한 합의 알고리즘이다. 사람들이 보통 합의 알고리즘\(consensus algorithm\)에 대해 말할 때는 이 알고리즘을 말하곤 한다. 만약 누군가가 팍소스라는 단어를  언급한다면, 아마도 우리가 다루게 될 basic Paxos에 대한 내용일 것이다.

팍소스는 **basic Paxos**와** multi-Paxos**로 나눌 수 있고 첫번째로 basic을 먼저 다루고, 그 후에 multi-Paxos에 대해 다룰 예정이다.

> once  we have this very simple form choose one value, then we can create a log by putting together several instances of this one. instance for each of the entries in the log and that's called **multi-Paxos.**



