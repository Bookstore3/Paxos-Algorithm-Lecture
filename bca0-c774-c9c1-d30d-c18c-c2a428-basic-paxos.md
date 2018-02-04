# Basic Paxos

## 필요한 요구사항들 \(Requirements for Basic Paxos\)

알고리즘의 안전성\(safety\)과 liveness에 대한 두 가지 전반적인 요구 사항이 있다. 안전성의 일반적인 의미는 알고리즘이 정상적으로 동작한다는 걸 의미한다. 예상과 다르게 또는 의도치 않게 알고리즘이 동작하진 않는다는 의미이며 basic Paxos 알고리즘 관점에서 해석해보면 basic Paxos 알고리즘은 반드시 하나의 값을 선택해야 한다는 것을 의미한다.

> there are two overall requirements for the algorithm safety and liveness. safety means in general terms that algorithm must never do anything bad and so for basic Paxos that means we must choose at most one value.

첫번째 값을 대체하는 두번째 값을 뽑는 등의 작업은 절대 해선 안된다. 안전성에 대한 또다른 요구 사항은 만약 서버가 값이 채택되어졌다고 믿는다면, 그 값은 정말로 서버가 속한 클러스터\(cluster\)로부터 선택되었다는 것이다.

반드시 하나의 값을 선택하는 것과 서버가 해당 값이 채택되어졌다고 믿는다면 서버 클러스터로부터 정말 채택되어진 것이라는 점, 이 두가지가 안전\(safety\)에 대한 특징이다.

liveness 속성은 우리가 시스템이 결국 잘 작동할 것을 원한다는 걸 말해준다. 단순히 안좋은 일이 일어나진 않겠지 등의 기대가 아니라 반드시 잘 작동해야한다.

