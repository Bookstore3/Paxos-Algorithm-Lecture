# Basic Paxos

## 1. 필요한 요구사항들 \(Requirements for Basic Paxos\)

---

알고리즘의 **안전성\(safety\)과 liveness**에 대한 두 가지 전반적인 요구 사항이 있다. 안전성의 일반적인 의미는 알고리즘이 정상적으로 동작한다는 걸 의미한다. 예상과 다르게 또는 의도치 않게 알고리즘이 동작할 일은 없음을 뜻하며 **basic Paxos 알고리즘은 반드시 하나의 값을 선택완료\(Chosen\)로 처리 한다는 것**을 의미한다 \(선택완료\(Chosen\)는 값이 정해져서 번복할 수 없다는 것을 뜻한다\).

> there are two overall requirements for the algorithm safety and liveness. safety means in general terms that algorithm must never do anything bad and so for basic Paxos that means we must choose at most one value.

첫번째 값을 대체하는 두번째 값을 뽑는 등의 작업은 절대 해선 안된다. 안전성에 대한 또다른 요구 사항은 만약 서버가 값이 **선택완료\(Chosen\)**되어졌다고 믿는다면, 그 값은 정말로 서버가 속한 **클러스터\(cluster\)로부터 선택완료되었다는 것이다.**

반드시 하나의 값을 선택완료로 처리다는 점, 그리고 서버가 해당 값이 선택완료 됐다고 믿는다면 서버 클러스터로부터 정말 선택완료된 것이라는 점, 이 두가지가 **안전\(safety\)**에 대한 특징이다.

![](/images/paxos_0201.PNG)

**liveness\(대상이 살아있음을 확인하는\) 속성**은 시스템이 결국 잘 작동할 것이라는 점을 말해준다. 단순히 안좋은 일이 일어나진 않겠지라고 기대하는 것이 아니라 반드시 잘 작동해야만 한다.

basic Paxos에는 **두 가지의 liveness 속성**이 존재한다. 첫번째로 우리는 **값을 반드시 선택완료 처리가 되게끔 만들어야 한다**는 점이다\(제안된 값들 중 일부는 반드시 선택완료 처리가 될 것이다\). 두번째는 서버들은 **선택완료된 값을 찾을 수 있다**는 점이다.

이러한 liveness 특징들은 **다수의 서버가 정상작동하며 충분히 빠르고 합리적으로  통신이 가능하다는 일반적인 합의 개념하에 **유지되어야 한다. 이러한 조건하에 클러스터는 반드시 살아 있어야 한다. 그래야 값을 결정하고 클러스터 구성원 모두가 그 값을 알게할 수 있기 때문이다.

> now these liveness properties have to hold under the general consensus notion that is as long as a majority of servers are running and as long as they can communicate with each other reasonably quickly.
>
> then under these conditions the cluster should be live that is it will eventually choose a value and everybody will find out about it.

basic Paxos를 구현하기 위해 상호작용하는 두 개의 구성요소가 있다. 바로 **제안자\(proposers\)들과 수용자\(acceptor\)들이다.** 제안자는 활동하는\(깨어있는\) 구성원으로 제안자는 주로 **클라이언트에게 요청을 받고 클라이언트가 요청한 특정 값이 선택완료 되어질 수 있도록 도움**을 준다. 도움을 준다는 건, 클라이언트가 요청한 값을 **자신의 의견으로 내서 클러스터의 구성원들\(서버들\)이 그 값에 동의하도록 시도**한다.

> Proposers are the active elements that is they're actually trying to do something.
>
> They will typically receive requests from clients asking that particular values be chosen and then they will try and put those values forth and get everybody in the cluster to agree to them.

**수용자는 수동적인 구성원**이다. 수용자들의 역할은 단순히 **제안자가 요청한 것에 대한 응답**을 해주는 것이다. 수용자들의 응답 하나 하나는 **일종의 표로 작용되어서 투표에 반영**이 된다. 제안자들은 각각 자신이 제안한 값이 선택완료\(Chosen\)되어지기를 바라기 때문에 클러스터에 존재하는 **다수의 수용자들로부터 표\(수락\)를 얻고 싶을 것**이다.

> acceptors are passive elements. they simply respond to requests that  come from proposers.
>
> you can think of their responses as votes where a proposer is trying to get a majority of votes from the acceptors in the cluster.

**수용자들은 결정 프로세스에 대한 상태 정보 중 여러가지를 저장**한다. 예를 들면 선택완료 또는 선택완료가 되지 못한 값들을 저장하거나 그들이 던진 표\(즉 수용자가 선택한 값\)을 저장한다. 투표로 비유하자면, **어떤 후보들이 나왔고 본인이 어떤 후보에 투표했는지 등을 알고 있다**고 할 수 있다. 또한 **수용자들은 어떤 값이 선택완료 됐는지\(어떤 후보가 당선됐는지\)에 대해 알고 싶어한다.**

![](/images/paxos_0202.PNG)

우리가 처음에 알 수 있듯이, **제안자들만이 값이 선택완료 되었다는 것을 알 수 있지만 우리는 수용자들도 결국 이 값을 찾아내길 원한다.** 수용자\(Acceptor\)는 해당 값이 상태 머신안에서 실행될 수 있도록 처리하기 위해  **수용자는 결국 그 값을 알아낼 수 있어야 한다.**

사실 전통적인 Lamport 방식에서는 **리스너\(Listener\)**라고 불리는 세번째 구성원도 있지만, 이 강의에서는 우린 **리스너와 수용자를 한 번에 묶어낸 개념**으로 설명한다. **별도로 구분하지 않겠다**는 뜻이다. 또한 이 강의에서 **서버는 각각 한 명의 제안자와 한 명의 수용자를 함께 포함하고 있는 것**으로 가정한다. **Paxos**를 구현할 때 각 서버를 한 명의 구성원의 역할만 하게끔 구현할 수도 있지만 이 강의에서는 **하나의 서버가 한 명의 제안자가 될 수도, 수용자가 될 수도 있는 구조로 설명**한다.

## 2. 팍소스를 구현할 때 해결해야 할 문제점

---

### 2.1. 단 하나의 Acceptor를 두었을 때의 문제점

지금부터 보여주는 몇가지 상황은 우리가 합의\(컨센서스\)를 제대로 설계하기 위해 **꼭 해결해야만 하는 문제점들**이다. 아래의 첫번째 템플릿을 살펴보자. 문제점이 발생하는 상황에 대한 템플릿이다.

![](/images/paxos_0203.PNG)

위 템플릿은 우리가** 단 하나의 수용자를 뒀을 때 발생할 수 있는 문제점**을 다룬다. 단 **한 명의 수용자에게** 모든 제안자\(Proposer\)가 보내는 요청들을 처리하도록 시켰다고 가정해보자. \(혹사 시킨다는 생각이 물씬 든다.. 또한 단 명의 근로자에게 일을 맡긴 셈이니 이 근로자가 파업이라도 하면..\)

아마도 수용자는 하나의 값을 선택해서 그 값을 선택완료\(Chosen\) 되었다고 처리할 것이다. \(위 템플릿에서는 jmp 명령어를 선택완료 했다.\) 매우 간단하네! 라고 생각할 수 있지만 **그 단 한 명의 수용자가 만약 다운되거나 제기능을 못할 때를 처리하지 못하는 설계**다.

만약 이 수용자에게 이상이 생긴다면, 우리는 어떤 값이 선택완료 되었는지 알 수 없다. 수용자가 살아날 때까지 기다릴 수밖에 없다.

> if the acceptor crashes right after choosing, we have no way of knowing which value was chosen and so we'd have to wait for that acceptor restart.

우리가 구현하고자 하는 **시스템의 목표는 대다수의 서버\(노드\)만으로 완벽한 서비스를 차질 없이 제공하는 것**이기 때문에 첫번째 문제점에 나타난 것처럼 **하나의 수용자만 두는 설계는 올바르지 않다. **이러한 문제점을 개선하기 위해 우리는 일종의 **정족수\(quorum\)개념을 도입**한다. 합의를 위해 모인 의사구성원들이 **의결을 하는데 필요한 최소한도의 인원수를 지정**하는 것이다.

### 2.1.1 단 하나의 Acceptor를 둠으로써 발생하는 문제의 해결

**정족수\(quorum\)는 보통 3, 5 또는 7과 같이 적은 숫자의 홀수만큼의 수용자들로 구성**한다. 선택완료\(Chosen\) 처리를 하 위해서는 정족수 중 **과반수 이상의 표\(accept\)를 얻어야 해당 값이 선택완료\(chosen\) 처리**될 수 있다. 이렇게 하면 한 명의 수용자가 망가져도, 대다수의 수용자가 제기능을 할 수 있기 때문에 값을 **선택완료 처리하는 매커니즘은 여전히 정상작동**할 수 있다.

만약 값을 **인정\(accept\)** 수용자가 그 즉시 바로 다운되는 경우에는 다른 수용자들이 해당 값이 선택완료 되었는지 확인하고 처리할 수 있기 때문에 이 정족수의 개념은 굉장히 유용하다. 하지만 정족수 개념을 도입해도 아직 해결해야할 과제들이 남아있다. 아래의 템플릿을 살펴보자.

> if an acceptor crashes after accepting there other acceptors around that can indicate that the value was chosen.
>
> Note: 인정\(accept\)과 선택완료\(Chosen\)는 다른 개념이다. 다수의 서버들이 한 값을 인정\(accept\)했을 때 선택완료\(Chosen\)이 되는 매커니즘이다.

![](/images/paxos_0204.PNG)

만약 수용자가 **자신이 첫번째로 받은 값만 인정\(accept\)한다고 가정**해보자. **S1\(1번 서버\)은 클라이언트가 보낸 red라는 정보를 인정\(accept\)했다.** 그리고 자신이 인정\(accept\)한 값이 선택완료\(Chosen\)되는 걸 원하는 S1은 다른 서버에게 전파\(broadcast\)하는 작업을 진행한다. 그와 **거의 동시에 S3와 S5도 클라이언트로부터 요청을 받는다.** 각각 클라이언트로부터 blue와 green이 선택완료되게 도와달라고 부탁을 받은 상태고, 이 값들이 선택완료\(Chosen\)될 수 있게 S3와 S5도 전파작업을 진행한다.

이렇게 동시다발적으로 이루어진 결과, S1은 S1과 S2의 동의\(인정\)를 얻었고 S3는 S3와 S4의 동의\(accepted\)를 얻었다. S5는 자신 빼고는 아무도 없다. 결국 다수결로 의사결정을 진행하지 못하게 되는 상태에 빠졌다. 3명의 찬성\(accept\)을 얻은 안건이 있어야 통과가 되는데 그 어떤 값도 3개의 표를 얻지 못했다. 이 결과가 시사하는 점은 수용자는 때때로 자신이 투표\(accept\)했던 것을 포기하고 다른 값\(후보\)에 투표\(accept\)해야한다.

이러한 점이 시사하는 건 단 한 번의 과정\(round\)으로는 어떤 값이 선택완료\(Chsoen\)되는 일이 굉장히 어렵다는 것이다. 2라운드, 어쩌면 3라운드 정도까지 선출 작업을 반복해야 할지도 모른다. 따라서 우리는 이제 두 개의 말을 구분지으려 한다. 선택완료됐다\(chosen\)와 인정됐다\(accepted\) 이 두 말을 구분해야 한다.

**선택완료\(Chosen\)가 됐다는 건 정말 한 클러스터 안에서 그 값이 선택완료되서 받아들여진다**는 의미이고, 인정되었다는 건 클라이언트가 보낸 값을 수용자가 받아놨긴 한데 아직 이 값이 클러스터에서 선택완료\(선출 혹은 당선\)는 되지 않았다는 의미다. 선택완료\(chosen\)가 되려면 **클러스터\(그룹\) 안의 대다수 서버에게 인정\(accept\) 받아야 한다.**

### 2.2 좀 더 복잡한 문제

이번엔 좀 더 난잡한 문제를 생각해보자.  **수용자가 자신에게 온 값이 뭐든 그냥 다 인정해버리는\(accepted\) 상황**을 생각하자. 이런 상황에서 발생할 수 있는 두 가지 문제점이 있다. 첫번째는 **여러 개의 값이 선택완료\(Chosen\) 되어질 수 있다**는 것이다. 아래의 템플릿을 살펴보자.

> first problem is that we could end up choosing multiple values.

![](/images/paxos_0205.PNG)

S1이 클라이언트가 요청한 red가 선택완료가 될 수 있게 red를 서버들에게 제안한다. S1은 다른 서버들에게 자신의 값을 인정\(accept\)해줄 것을 요청한다. 그 결과 S2, S3가 red를 받아들여 red가 선택완료\(chosen\)되었다.** 5명 중 3명이 동의했으니 다수결의 원칙에 의해 red는 선택완료될 수 있다.**

red가 선택완료\(Chosen\) 되어진 후, **클라이언트는 S5에 blue를 의뢰**했다. S5는 blue를 선택완료\(Chosen\)로 만들기 위해 다른 서버들에게 인정\(accept\)해줄 것을 요청한다. 우리가 가정한 문제상황에 따르면 **수용자는 어떤 값이든 계속해서 인정\(accept\)한다고 했으니** **이미 red를 인정했던 S3는 blue를 인정\(accept\)할 수 있다.** 이렇게 **S3, S4 그리고 S5가 blue를 인정\(accept\)하게 되면서 Blue는 선택완료\(chosen\)되게 된다.**

이렇게 되면 우리가 설계하고자 했던 **시스템의 기본 원칙이 무너지게 되는데**, **단 하나의 값만을 선택완료 처리해야 한다**는 원칙을 위 상황은 위반하게 된다. 따라서 뭔가 해결책이 필요하다. 어떻게 해결할 수 있을까?

템플릿 아래에도 적혀 있지만 위 템플릿에서 S5가 클라이언트로부터 받은 값 blue를 **주장하기 전에! 그 전에 선택완료\(chosen\) 처리가 된 값이 있는지 체크**하는 것이다. 만약 선택완료된 값이 있었다면 S5는 자신이 받은 값 blue를 제안하는 것이 아니라 이미 선택완료된 값 red를 제안하게 하는 것이다.** 자신이 기존에 가지고 있던 값은 포기한다**.

이렇게 되면 위 템플릿에서 S5가 새롭게 주장한 값인 red는 S3, S4에게로 전파되고 또 다시 선택완료로 처리된다. 이것을 우리는 **\(2-phase protocol\)**이라고 칭한다.  하지만 안타깝게도 이 또한 문제를 해결하기엔 충분치 않다. 아래의 템플릿을 살펴보자.

![](/images/paxos_0206.PNG)

S1은 red를 제안하려 한다. 그렇지만** 2-phase protocol**을 적용해서 **그 전에 먼저 선택완료된 값\(chosen value\)이 있는지 확인**한다. 하지만 S1이 red를 제안하던 때에는 \(수직으로 아래까지 선을 이으면 **S1과 동시대에 발생한 이벤트는 없음**\) **어떠한 서버\(S\*\)도 값을 인정하고\(accept\) 있지 않았으므로 S1은 red를 클러스터의 각 서버들에게 전송하기 시작**한다.

하지만 아직까지 어떤 수용자도 S1의 요청에 반응하지 않고 있는 이때!** S5가 클라이언트로부터 요청 받은 blue값을 제안하기 시작**한다. 물론** S5도 제안하기 전에 어떤 값이 수용되어 있는지 확인**을 한다. S1도 이 당시에 아직 자신이 제안한 값을 인\( accept\)하지 않았던 때이기에** S5도 이상없이 각 서버들에게 자신의 제안을 전파**한다.

그리고 **전파된 blue값이 먼저 선택완료\(chosen\)되어지는 상황이 발생**했다. 하지만 이러는 동안에도 S1은 계속 동작하고 있었고 각 서버는 들어오는 값들을 인정\(accept\)할 수 있기에 또 하나의 선택완료\(Chosen\)된 값이 탄생한다. 이렇게 되면 단 하나의 값만 선택완료 돼야 한다는 기본 설계원칙을 또다시 어기게 된다.

이 문제를 해결하기 위해 일단 어떤 값이 선택완료\(Chosen\)되었다면, 그 이후 제안\(propose\)되는 것들의 값은 버려야 한다. 위 템플릿에서 S3를 살펴보면, **이미 blue를 선택완료\(Chosen\)했기 때문에 이후에 제안이 들어온 red를 거부해야 한다.** 이러한 절차를 위해 우리는 **순서 개념을 도입**한다. **선택완료된 값\(Chosen value\)이 있다면 오래된 값을 버리는 규칙을 적용**해보자. S1이 red를 인정\(accept\)할 때는 blue가 선택완료되기 이전의 시간이다.** 즉 선택완료된 값 blue가 있을 때를 기준으로 S1이 제안한 red는 오래된 것이므로 S1은 red값을 버려야한다.**

즉 우리는** 2-phase protocol**에서 공부했던 원칙\(제안하기 전에 선택완료된 값\(Chosen value\)이 있는지 확인!\)과 더불어 각 제안들에 순서를 매겨야 한다는 것을 알게된 것이다.

### 2.2.1 좀 더 복잡한 문제를 해결하기 위해 필요한 개념

제안들에 순서를 매기는 방법에 대해 생각해보자. 단순하게 생각할 수 있는 건 **각 제안에 고유한 번호**를 부여하는 것을 생각해볼 수 있다\(이전의 제안에 있던 값을 다시 쓰는 등의 방법은 허용되지 않는다.\). 그리고** 높은 번호가 낮은 번호보다 우선순위를 갖는다**는 규칙을 함께 적용한다.

이러한 규칙을 적용한다면 각 서버가 프로토콜을 실행할 때 \(규칙을 따라 실행할 때\) **서버가 제안하는 제안의 번호는 고유한 값**이어야 함은 물론이고 예전에 사용되었던 번호 혹은 자신이 발견했던 번호보다 커야한다. 높은 번호가 우선순위가 있다고 했는데 굳이 서버가 **낮은 번호를 쓴다면 자신의 제안이 영영 받아들여지지 않을  것이기에 서버는 높은 번호를 사용할 것이다.**

![](/images/paxos_0207.PNG)

이것을 구현하기 위해 고안된 방법은 **두 값\(value\)을 결합하는 것**이다. 위 템플릿의 **제안 번호\(Proposal Number\)**를 구성하고 있는 **Server ID**를 살펴보자.** 모든 서버에게 고유한 식별자**를 부여하고 **이 고유한 서버 아이디를 제안 번호의 낮은 비트 순서**에 둔다. \(낮은 비트 순서란 어려운 것이 아니다. 1100 이라는 비트가 있다면 00 파트에 해당하는 것이 낮은 비트 순서에 속한다.\)

이렇게 서버 고유 아이디 값을 줌으로써 **다른 서버들이 이 아이디를 사용하지 않을 거라는 점을 보장**할 수 있고 또한 이 값이 생성되기 전 다른 어떤 서버에서도 이전에 생성한 적이 없었다는 점 또한 보장할 수 있다. \(고유함을 보장한다고 보면 된다.\)

> first start with server Id.
>
> give every server a unique identifier and put that in the low-order bits of the proposal number so that guarantees that now no other server will ever generate this proposal or could have generated it before.

그 다음 우리는 **높은 비트 순서에 라운드 번호\(Round Number\)를** 둔다. 이 라운드 번호는 **시간이 지나면 지날수록 게속 증가**하면서** 모든 서버에게 공유**된다. 이 라운드 번호는** 제안을 생성하거나 시도할 때 사용될 것**이고** 이 번호는 계속해서 커지기에 가장 최신의 제안일 수록 가장 큰 값일 것이고 고유한 번호**일 것이다.

라운드 번호가 이렇게 작동하려면 **서버는 자신에게 들어온 메시지\(제안\) 혹은 자신이 낸 제안 속에 있는 라운드 번호를 계속 주시**하고 있어야 한다. 그리고 이렇게 살펴본 후에 서버는 **가장 큰 라운드 번호를 저장**하고 있는데, 이것을 우리는 **최대라운드값\(maxRound\)**라고 한다.

따라서 **새로운 제안 번호를 생성하는건 단지 라운드 번호를 증가**시키고** 서버 아이디와 결합하는 것**이다. 이러한 작업을 위해서 각 제안자는 디스크나 안정적인 저장매체에서 사용된 **가장 최신의 최대라운드\(maxRound\)값을 저장해야만 한다. **그래야 각 서버에 이상\(혹은 충돌\)이 생겼을 때 **정상적인 복구**가 가능해지고 **이전에 사용되었던 제안번호를 사용하지 않는다는 원칙을 지킬 수 있기 때문**이다.

> to generate a new number, it simply increments that value and then concatenates it with its server ID.
>
> in order for this to work, the proposers must make sure they save the latest value of maxRound that they've used on disk or some other stable medium so that it can be recovered after a crash.
>
> this is needed to make sure that we don't accidentally reuse a proposal number if we crash and then restart.

## 3. 팍소스 중간 정리

---

![](/images/paxos_0208.PNG)

앞서 설명했듯 우리는 **2-phase approach**를 사용해야만 한다. **첫번째 phase**에서는 제안자가 자신이 제안한 값이 선택완료가 될 수 있게 서버들에게 값을 전송하려할 때, 모든 서버에게 **RPC\(remote procedure call\)**를 보낸다. RPC를 잘 모른다면 강의에 사용된 영어단어 및 용어 카테고리 참조하자. **Phase1에서 사용하는 RPC를 우리는 prepare**라고 부른다.

위 과정은 **선택완료\(Chosen\)된 값이 있는지 확인**하는데 도움을 주는데, 만약 선택완료된 값이 이미 존재했다면 기존에 제안자 본인이 가지고 있던 값을 포기하고 선택완료된 값을 다시 제안하게 된다.

또한 **선택완료된 값이 있는 시점 이전에 등장한 제안이 있었다면 그 제안을 무시하고 거부한**다. 그럼으로써 **기존에 선택완료된 값과 경쟁하게 되는 상황을 막을 수 있게 된다.**

위의 두 가지 규칙은 우리가 위에서 다뤘던 문제상황으로부터 비롯된 중요한 해결책들이다. **Prepare 과정은 Paxos 사용자들에게 자신의 값이 선택완료가 되게끔 요청하는 과정을 안전하게 진행할 수 있다는 것을 말해준다.** 즉 **제안하는 프로세스를 걱정 없이 자유롭게 해도 된다는 뜻이다.**

**Phase2에서는 우린 Accept\(인정해달라고 요청\)라 불리는 또다른 RPC를 전송**하게 된다. 전송할 때는 선택완료로 처리하고 싶은 값을 전송하게 되고** 다수에 의해 인정\(accept\)된다면 Phase2에서 값이 선택완료 됐음을 확인**할 수 있다.

## 4. 기본 팍소스의 전체 프로토콜 구조 \(the full protocol for basic Paxos\)

---

요청\(request\)의 생명주기\(life cycle\)에 따라 basic Paxos의 프로토콜을 살펴보자. 프로토콜의 첫 시작은 제안자\(Proposer\)가 하게 되는데 제안자 본인이 구성원들로부터 인정받길 원하는 값\(선택완료가 됐으면 하는 값\)으로 시작하게 된다. **2개의 라운드로 구성되어 있으며 각 라운드는 Prepare 메시지를 브로드캐스팅**\(전체 구성원에게 값을 전송하는 행위\)하는 **Prepare phase**와 **Accept 메시지를 브로드캐스팅 하는 Accept Phase**로 구성된다. 아래 템플릿의 2 ~ 4에 해당하는 것이 Prepare phase고 5 ~ 6에 해당하는 것이 Accept phase다.

![](/images/paxos_0209.PNG)

**첫째로 해야할 작업은 새로운 제안 번호를 선택**하는 것이다\(고유한 제안 번호를 선택하는 매커니즘은 위에 설명되어 있다\). 그 후에 Prepare phase에 진입하게 된다. 2번에서 제안자는 **Prepare RPC를 클러스터 안에 존재하는 모든 수용자\(acceptor\)에게 전송**한다\(브로드캐스팅\). **전송한 각 메시지 안에는 제안번호가 존재**한다.

**수령자가 Prepare 요청들 중 하나를 받게 되면 두 가지 작업**을 하게 된다. 첫번째로 **수령자는 지금 받은 메시지 내의 제안번호 보다 더 작은 값을 절대 받아들이지 않을 것이라는 점을 보장**한다. 그러기 위해서 수령자는 내부에 **minProposal라는 변수**를 하나 둔다. **minProposal 변수는 지금까지 수령자가 받았던 제안 중 가장 작은 제안 번호**를 담고 있다. 이 변수는 시간이 지남에 따라 단조롭게 증가하게 되어 있다.

아직까지 이 수령자가 다른 제안자들에게 이 크기\(제안자가 넘긴 제안번호 값 n\)의 제안을 받아들이지 않겠다는 약속을 하지 않았다면, **제안자로부터 온  메시지 안에 담겨 있는 제안 번호n이 수령자가 지금까지 본 값 중 가장 큰 값의 제안번호**일 것이다. 수령자는 자신이 가지고 있던 제안 값보다 더 큰 값의 제안 번호를 가진 새로운 제안을 받았으므로\(n &gt; minProposal\) **minProposal 값을 갱신**한다. **한마디로 n에 담겨 있는 값이 minProposal 보다 크다면 minProposal을 갱신**한다고 생각하면 된다. 이것이 수령자가 해야할 첫번째 작업이다.

> if we haven't already made a promise to somebody else not to accept proposals of this size so this is the highest proposal we've seen so far then we update minProposal to keep track of that.

**수령자\(acceptor\)가 해야할 두번째 작업**은 **이전에 인정\(accept\) 됐을 수도 있는 제안에 대한 모든 정보를 반환**하는 것이다. 만약 이전에 어떤 제안을 **인정\(accept\)했었다면, 수령자는 자신이 수령한 값\(accepted value\)과 수령한 제안번호\(number of proposal\)을 저장**하고 있다. **지금까지 저장한 값들 중 가장 높은 제안번호를 가지고 있는 정보를 반환**한다.

![](/images/paxos_0212.PNG)

**제안자\(proposer\)는 대다수의 수령자\(majority of acceptors\)들로부터 응답이 올 때까지 기다린다.** 수령자들로부터 응답을 받았다면, 제안자는 수령자가 **이전에 인정했던 제안\(accepted proposal\)이 있었는지 확인**한다. 만약 있었다면, **수령자들이 응답한 제안들의 번호를 한 번 더 비교해서 가장 높은 제안번호를 가진 제안을 고른다.**

그리고 **제안자 본인이 처음에 제안했던 제안 값\(value\)을 포기**하고 **수령자들이 보낸 응답으로부터 추출한 가장 큰 제안번호에 담겨 있는 제안 값\(value\)을 자신의 제안 값\(value\)으로 사용**한다\(**제안자는 value의 선택완료를 위해 계속 작업할 것**이다\). 만약 수령자들로부터 기존에 제안이 있었다는 응답이 없었다면 **자신이 원래 가지고 있던 제안번호\(value\)을 가지고 다음 단계로 진행**하게 된다. 이로써 첫번째 phase가 끝나게 된다.

이제** 제안자가 Accept RPC를 브로드캐스팅하는 두번째 phase**로 넘어가보자. **Accept RPC는 두 가지 값을 포함**하고 있는데, 첫번째 값은 제안번호이다. 이 제안번호는 반드시 Prepare message로부터 왔던 제안번호\(n\)와 같아야 한다. **두번째로 포함하고 있는 것**은 **제안자가 시작했던 값\(value\)** 혹은 **수령자로부터 온 응답안에 있던 값\(value\)**이다.

> And the Accept RPC contains two values. first it contains a Proposal Number and this must be the same as the Proposal Number from the Prepare message and in addition it includes the Value either the initial Value the Proposer started with or an Accepted Value that it received back from an Acceptor.

이 Accept 메시지는 클러스터 내의 모든 수령자에게 전송되며 수령자들은 이 메시지를 처리하게 된다. 처리하는 방식은 간단하다. **받은 제안의 제안번호\(n\)가 수령자\(acceptor\)가 기존에 갖고 있던 minProposal 보다 작다면\(n &lt; minProposal\) 지금 온 제안은 거절**당한다. **반대로 minProposal보다 크거나 같다면\(n &gt;= minProposal\) 수령자\(acceptor\)는 제안을 인정\(accept\)**하게 된다. 인정\(accept\)한다는 건 **받아들인 제안의 제안번호와 값을 기억**한다는 것이다\(acceptedProposal = minProposal = n, acceptedValue = value\).

![](/images/paxos_0211.PNG)

수령자\(acceptor\)가 요청\(request\)를 받아들이든 말든, **결국 수령자는 minPropsoal을 반환**하게 된다. 제안자는 클러스터 내 **대다수의 수령자로부터 응답이 올 때까지 기다린다. **대다수의 수령자로부터 응답을 받으면, 제안자는 보냈던 요청이 거절당했는지 확인하는 작업을 진행한다**\(result &gt; n ? goto\(1\) : chosen\)**. 정상적으로 받아들여 졌다면, 제안자 본인이 보냈던 제안번호와 같은 값을 가진 result를 받았을 것이므로\(result == n: true\) 값은 선택완료가 되게 된다.

그러나 만약 **처음 보냈던 제안 번호보다 result가 크다면,** 제안자가 보낸 제안이 **거절당했다는 뜻**이므로 제안자는 1번 과정부터 다시 진행하는 절차를 밟아야 한다\(본인이 보냈던 제안번호 보다 큰 값이 리턴됐다는 건, 중간에 더 높은 제안번호를 가진 다른 제안에게 기회를 뺏겼다는 의미이다\).

하지만 다행히도 제안자 입장에서는 result 값을 확인할 수 있으므로 **다음 번에 제안을 할 때는 좀 더 유리한 위치에서 시작**할 수 있다. 제안 번호가 높으면 높을수록 유리한 매커니즘이기 때문에 거절당했을 때 받은 result 값을 확인함으로써 현재 가장 큰 제안 번호가 어떤 것인지 살펴볼 수 있고, 본인이 새롭게 제안할 때는 해당 번호보다 큰 값을 제안번호로 사용할 수 있다.

지금까지 설명한 이러한 매커니즘이 정상적으로 작동하기 위해서는 **수령자\(acceptor\)가 3가지 변수\(minProposal, acceptedProposal, acceptedValue\)의 안정성\(stability\)을 보장**해줘야 한다. 그러기 위해서 이 변수들은 플래시 메모리 또는 디스크 등 오동작\(crash\)에 대처할 수 있는 하드웨어에 저장되어야 한다.

### 4.1. 발생 가능한 상황\(1\)

팍소스가 과연 **제안들이 경쟁적으로 대칭되는 상황에서 어떻게 작동**하는지 살펴보자. 왜 팍소스가 정상 작동하는지에 대해 알기 위해서는 **2번째 제안의 Prepare Phase를 주목**해야한다. 아래 템플릿을 살펴보기 전에 그림을 해석하는 법을 먼저 언급하고자 한다.

먼저 맨 좌측에 존재하는** X, Y의 경우 클라이언트가 서버에게 넘기는 값\(value\)**이다. 서버는 각 클라이언트로부터 받은 X와 Y가 선택완료\(Chosen\)처리가 될 수 있도록 다른 서버들에게 이 값을 전파\(broadcast\)할 것이다. P 3.1의 경우 **제안번호 3.1을 가진 Proposal**이란 의미고 higher bits order의 3은 라운드 번호를, lower bits order의 1은 서버 번호를 뜻한다. 즉, S1으로부터 온 제안이라는 것을 뜻한다.** 마찬가지로 A3.1X의 경우 서버 S1으로부터 온 X값을 인정\(Accept\)했다는 뜻**이다. 이제 아래 템플릿을 살펴보자.

> Note: Higher Bits Order와 Low Bits Order를 헷갈리면 안된다. 1111\(2\)가 있을 경우 Higher Bits Order에 해당하는 이진수는 맨 좌측 두 개 11\(십진수로는 12\), Low Bits Order에 해당하는 이진수는 우측 11\(십진수로 3\)이다.
>
> 따라서 3.1과 3.2중 더 큰 제안값은 3.2가 되고, 4.1과 3.8중 더 큰 제안값은 4.1이 된다. 높은 비트열에 속하는 4가 훨씬 가중치가 많이 더해지기 때문에 4.1이 더 큰 값이 된다.

![](/images/paxos_0210.PNG)

**새로운 제안\(P4.5\)이 등장하기 전에 먼저 도착한 제안\(P3.1\)이 이미 전체 과정\(whole process\)을 완료했고 값 X가  선택완료\(Chosen\)처리가 된 상황**이다. 그말인 즉슨, **X값이 이미 대다수\(3개의 서버\)로부터 인정\(accept\) 됐다**는 뜻이다.

여기서 한 가지 주의해야 할 점은, **overlap\(겹쳐짐\)이 적어도 하나의 서버에서는 발생한다**는 것이다. **새로운 제안\(P4.5\)는 서버 S5로부터 온 제안이며, S5또한 대다수의 서버로부터 인정\(accept\)을 받기 위한 프로세스를 진행**할 것이다. 따라서 3명의 Server에게 제안을 전파할텐데, **이미 전에 Accept를 했던 Server3가 S5의 새로운 제안을 받게된다**. S3은 이미 이전 제안\(P3.1\)을 인정\(accept\) 했지만 새로운 제안\(P4.5\)을 받을 수 있다는 것을 우린 보장해야 한다.

S5는 새로운 값\(Y\)가 선택완료처리가 될 수 있게끔 **Prepare RPC를 S3, S4, S5에게 각각 전송**한다. 이 때 S3에는 **이미 인정\(Accept\)된 값 X가 있었으므로, S5는 자신의 초기 제안 값\(value\) Y를 포기하고 X로 Accept Phase를 진행하기 시작**한다. S5는 결국 값 Y가 아닌 값 **X를 성공적으로 선택완료\(Chosen\)처리**하게 된다.

### 4.2. 발생 가능한 상황\(2\)

이번에 다룰 상황은 두 번째 제안의 Prepare Phase가 올 때까지 아직 어떠한 값도 선택완료\(Chosen\)처리되지 않은 상황이다.

![](/images/paxos_0213.PNG)**먼저 온 제안\(P3.1\)은 주장하는 값 X가 선택완료 처리될 수 있게끔 각 서버에게 Accept를 요청**했다. **서버 S5는 이 때 새로운 제안\(P4.5\)를 내놓고 S3, S4, S5에게 Prepare Phase를 진행**한다. 이때 **서버 S3에 이미 인정\(accepted\)된 제안과 값\(value\) X이 있었다**는 것을 발견한다. 이러한 경우** 두 번째 제안\(P4.5\)는 이미 존재하는 값 X를 발견했으므로 기존에 제안하던 값 Y를 포기하고 X로 각 서버에게 Accept Phase를 진행**하게 된다.

결국 1번 상황하고 같은 결론에 도달하게 되고 **두 상황 모두 다 값 X를 선택완료 **처리하게 된다. 왜 기존의 값 Y를 포기하고 X로 주장하게 되는걸까? **S5는 S3에서 인정했던 값이 선택완료 처리가 되었는지는 알 수가 없다. **모든 서버를 살펴보는 것도 아니고, 단지 자신이 주장을 퍼트린 서버들에 한해서 정보를 알 수 있기 때문이다.

S5입장에서는 S3로부터 이미 인정된 값 X가 존재했다는 사실로 **"값 X가 어쩌면 이미 S1, S2, S3로부터 인정\(accept\)되어 선택완료처리가 됐거나 그럴 수 있는 가능성이 있겠다" **라고 미뤄 짐작할 수 있을 뿐이다. **두 가정 상황 모두 다 값 X를 주장하는 것이 좀 더 합리적일 것**이기 때문에 **값 Y를 포기하고 새롭게 X로 Accept Phase를 진행하게 되는 것이다.**

### 4.3. 발생 가능한 상황\(3\)

3번째 상황은** 2번째 제안\(P4.5\)이 나타나기 전에 선택완료\(Chosen\)처리된 값이 존재하지 않고, 2번째 제안\(P4.5\)이 이전 값에 대한 어떠한 인정\(Accept\)도 발견하지 못한 상황**을 다룬다. 위에서 설명했던 두 번째 상황 + 어떠한 인정\(accept\)된 값도 발견하지 않은 점이 추가된 것이다.

![](/images/paxos_0214.PNG)

물론 S1에 A3.1X가 있음으로써 값** X가 이미 인정\(accept\)된 것은 틀림없는 사실**이지만, **2번째 제안을 시작한 서버 S5 입장에서는 S3, S4, S5만 살펴봤을 뿐**이므로 **어떠한 값도 인정되지 않았다고 하는 것이다.**

이러한 경우** S5는 자신이 최초에 주장했던 값\(value\) Y를 그대로 주장**하게 되고 **S3, S4, S5로부터 인정\(accept\)받을 수 있다. **결국 **값 Y는 선택완료 **처리가 된다. 유일하게 위 상황에서 문제가 되는 것은 **overlapping된 S3**이다. S3에 추가적인 설명을 좀 더 보태보자.

**S5로부터 진행된 Prepare Phase는 다행히도, S1이 뒤늦게 보낸 Accept요청\(A3.1X\)을 효과적으로 차단**할 수 있는데, 이는** S5의 제안 번호\(4.5\)가 S1이 보낸 Accept 요청 안에 있는 제안 번호\(3.1\)보다 더 크기 때문**이다. 따라서 **S3는 첫 번째 제안의 Accept Phase를 무시**하고 두 번째 제안의 Accept Phase를 안전하게 진행할 수 있게 된다.

![](/images/paxos_0211.PNG)

위에서도 다룬 바 있지만**, 6번 If문을 통해 minProposal \(4.5\)보다 작은 n\(3.1\)은 거절**된다. 헷갈린다면, 전체 흐름도를 다시 살펴보도록 하자.

**S1은 자신의 요청이 거절 되었으므로, 더 높은 제안번호로 제안을 새롭게 다시 시작하게 될 것이다\(Any rejections \(result &gt; n\) ? goto\(1\)\)**. 새롭게 제안할 때 또한 값 Y의 보존이 보장되는데, 이는 **S1이 요청할 때 S1, S2, S3에게 보낸다고 가정해보면 S3가 이미 accept한 값 Y를 발견하게 될 것이므로** 기존에 주장하던 X를 포기하고 Y로 Accept Phase를 진행할 것임이 자명하기 때문이다. 이로써 이미 선택완료된 값은 보존된다는 원칙 또한 지킬 수 있다.

## 5. 무한 루프

---

**파트 4에서 우리는 basic Paxos가 어떠한 경쟁 상황에서도 유일한 하나의 값만을 선택완료\(Chosen\) 처리하게 될 것이라는 걸 확인**했다. 하지만 아직 완전히 해결됐다고 말할 수는 없다. **서로의 제안이 엉키고 얽혀서** 어떠한 값도 선택완료처리 될 수 없는 상황이 존재한다. 지금까지의 결론만으로는 **basic Paxos의 Liveness를 보장할 수 없는 것이다**. 아래의 상황을 살펴보자.

![](/images/paxos_0215.PNG)

**S1이 첫 번째 제안\(P3.1\)**을 내놓았고** Accept Phase\(A3.1X\)를 진행하기 전에 S5로부터 새로운 제안\(P3.5\)가 등장했다고 가정**해보자. 이럴 경우** overlapping된 S3에서의 A3.1X에 포함되어 있는 제안 번호 3.1은 새롭게 등장한 제안 번호 3.5보다 작으므로 S3에서의 A3.1X는 거절\(rejecet\)**된다.

**S1은 S3에서의 AcceptPhase가 실패**했다는 사실을 즉각 알아차리고 새로운 **RoundNumber 4를 보태어 새로운 제안 4.1을 내놓게 된다. **뒤늦게 도착한** S5로부터의 AcceptPhase A3.5Y**를 살펴보자. **역시 이번에도 overlapping되는 S3에서 문제가 발생**한다.

이전에 존재하는 **제안 번호 4.1은 방금 들어온 AcceptPhase의 제안 번호 3.5보다 크므로 A3.5Y는 거절**된다. 이와 같은 상황이 계속 연이어 발생하면서 **어떠한 값도 선택완료 되지 않는 무한루프 상황\(Live Lock\)이 발생**한다.

이런 상황들을 해결하기 위해서는 2가지 정도의 방안이 있다. 첫번째는 서버의 제안이 실패했을 때,** 제안을 재시작하기 전에 잠시 기다리는 것**이다. 다른 제안이 처리를 끝낼 수 있는 기회를 주는 것이다. 물론 이 때 주는 **대기시간도 똑같이 주면 또다시 Lock이 걸려버릴테니, 대기시간을 랜덤하게 주는 것이 좋겠다.**

또는 우리가 다음 Article\(멀티 팍소스\)에서 다루게 될 **Multi-Paxos의 Leader Election\(리더 선출\) 방법**을 사용하는 것이다. 정확히** 한 순간에 하나의 제안자\(Leader\)가 활동**할 수 있게 하는 것이 멀티 팍소스의 리더 선출 방법이다.

## 6. 베이직 팍소의 한계

---

여전히 베이직 팍소스에는 해결해야 할 문제들이 남아 있다.

![](/images/paxos_0216.PNG)아직까지는 값을 제안한 서버만이 해당 값이 선택완료 처리 되었는지를 알 수 있다. 각 서버는 값을 제안하기 전까지는 해당 값의 선택완료 여부를 알 수 없다. 수령자\(acceptor\)는 제안을 받고 응답을 해줄 뿐, 그 값이 선택완료 되었는지는 전혀 알 수 없다.

서버들은 어떤 값이 선택완료 되었는지 궁금할 테지만, 역시 이를 알 수 있는 유일한 방법은 전체 프로토콜을 다시 진행하는 방법 뿐이다. 자신이 제안한 값이 그대로 돌아왔다면, 그 값이 선택완료 됐다고 판단할 수 있는 것이고 만약 다른 값으로 바뀌어져 돌아왔다면 그 값이 선택완료 되었다고 판단할 수밖에 없다.
