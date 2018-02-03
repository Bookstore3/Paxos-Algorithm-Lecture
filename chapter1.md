# 팍소스 알고리즘

---

팍소스 알고리즘은 분산컴퓨팅의 아버지라 불리는 레슬리 램포트\(Leslie Lamport\)에 의해 만들어졌다. 만들어진 이후로 팍소스 알고리즘은 컨센서스와 동의어로 사용되어 왔다. 대학에서 컨센선스를 강의한다고 하면 그 중 알고리즘 파트는 대부분 팍소스 알고리즘을 다룬다고 생각하면 된다. 가장 실용적인 컨센선스라 일컬어지는 것들은 팍소스를 바탕을 두고 있다.

팍소스 알고리즘은 모든 분산 시스템에 존재하는 모든 알고리즘 중 가장 중요한 알고리즘이라고 생각할 수 있다. 팍소스에 대해 첫 번째로 다뤄볼 것은 복제로그의 생성과 이 생성된 복제로그를 통해 복제된 상태 머신이 생성되는 것에 대한 내용이다.

> I'm going to explain Paxos in the context of creating a replicated log and then we'll use the replicated log to create a replicated state machine.





