# Multi Paxos

## 1. 멀티 팍소스의 구현

팍소스 알고리즘의 목적은 **복제로그의 생성**과 이 로그를 통해 **복제된 상태머신을 생성**하는 것 이라는 것을 Basic Paxos를 통해 배웠다. **멀티 팍소스**는 단 하나의 값을 선출하는** Basic Paxos의 여러 독립적인 사례를 묶어** 로그를 형성하는 연속된 값을 선출해 내는 방식이다. 이러한 멀티 팍소스를 구현하기 위해서는 각각의 Prepare과 Accpet 요청에 변수\(Argument\)를 추가 하면 된다. 그리고 이러한 요청은 특정 로그 항목을 선택하며 모든 서버는 모든 로그의 항목들에 대해 **서버 각자의 독립된 상태**를 갖추고 있다.

> One way to do that, is to use a collection of basic Paxos instances. One independent instance for each of the entries in the log.
>
> So to do this, we just add extra argument to each of the prepare and accept request that selects particular log entry and all of the servers keep separate state for every entry into the log.

![](/assets/Multi Paxos 1.PNG)



