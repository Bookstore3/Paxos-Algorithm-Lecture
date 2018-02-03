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



