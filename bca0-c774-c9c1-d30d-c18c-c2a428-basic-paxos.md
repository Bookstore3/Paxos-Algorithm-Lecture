# Basic Paxos

## 필요한 요구사항들 \(Requirements for Basic Paxos\)

알고리즘의 안전성\(safety\)과 liveness에 대한 두 가지 전반적인 요구 사항이 있다. 안전성의 일반적인 의미는 알고리즘이 정상적으로 동작한다는 걸 의미한다. 예상과 다르게 또는 의도치 않게 알고리즘이 동작할 일은 없다는 의미이며 basic Paxos 알고리즘은 반드시 하나의 값을 선택해야 한다는 것을 의미한다.

> there are two overall requirements for the algorithm safety and liveness. safety means in general terms that algorithm must never do anything bad and so for basic Paxos that means we must choose at most one value.

첫번째 값을 대체하는 두번째 값을 뽑는 등의 작업은 절대 해선 안된다. 안전성에 대한 또다른 요구 사항은 만약 서버가 값이 채택되어졌다고 믿는다면, 그 값은 정말로 서버가 속한 클러스터\(cluster\)로부터 선택되었다는 것이다.

반드시 하나의 값을 선택한다는 점, 그리고 서버가 해당 값이 채택되어졌다고 믿는다면 서버 클러스터로부터 정말 채택되어진 것이라는 점, 이 두가지가 안전\(safety\)에 대한 특징이다.

![](/assets/2.PNG)

liveness\(대상이 살아있음을 확인하는\) 속성은 우리가 시스템이 결국 잘 작동할 것을 원한다는 걸 말해준다. 단순히 안좋은 일이 일어나진 않겠지라고 기대하는 것이 아니라 반드시 잘 작동해야한다. 

basic Paxos에는 두 가지의 liveness 속성이 존재한다. 첫번째로 우리는 값을 반드시 선택해야 한다는 점이다. 두번째는 서버들은 선택된 값을 찾을 수 있다는 점이다.

이러한 liveness 특징들은 다수의 서버가 정상작동하는 한 충분히 빠르고 합리적으로  통신이 가능하다는 일반적인 합의 개념하에 유지되어야 한다. 이러한 조건하에 클러스터는 살아 있어야 한다. 그래야 값을 결정하고 클러스터 구성원 모두가 그 값을 알게할 수 있기 때문이다.

> now these liveness properties have to hold under the general consensus notion that is as long as a majority of servers are running and as long as they can communicate with each other reasonably quickly.
>
> then under these conditions the cluster should be live that is it will eventually choose a value and everybody will find out about it.

basic Paxos를 구현하기 위해 상호작용하는 두 개의 구성요소가 있다. 바로 제안자\(proposers\)들과 수용자\(acceptor\)들이다. 제안자는 활동하는\(깨어있는\) 구성원으로 제안자는 주로 클라이언트에게 요청을 받고 클라이언트가 요청한 특정 값이 선택되어질 수 있도록 도움을 준다. 도움을 준다는 건, 클라이언트가 요청한 값을 자신의 의견으로 내서 클러스터의 구성원들\(서버들\)이 그 값에 동의하도록 시도한다.

> Proposers are the active elements that is they're actually trying to do something. 
>
> They will typically receive requests from clients asking that particular values be chosen and then they will try and put those values forth and get everybody in the cluster to agree to them.

수용자는 수동적인 구성원이다. 수용자들의 역할은 단순히 제안자가 요청한 것에 대한 응답을 해주는 것이다. 수용자들의 응답 하나 하나는 일종의 표로 작용되어서 투표에 반영이 된다. 제안자들은 각각 자신이 제안한 값이 채택되어지기를 바라기 때문에 클러스터에 존재하는 다수의 수용자들로부터 표\(수락\)를 얻고 싶을 것이다.

> acceptors are passive elements. they simply respond to requests that  come from proposers.
>
> you can think of their responses as votes where a proposer is trying to get a majority of votes from the acceptors in the cluster.

수용자들은 결정 프로세스에 대한 상태 정보 중 여러가지를 저장한다. 예를 들면 채택되거나 되지 않은 값들을 저장하거나 그들이 던진 표\(즉 수용자가 선택한 값\)을 저장한다. 투표로 비유하자면, 어떤 후보들이 나왔고 본인이 어떤 후보에 투표했는지 등을 알고 있다고 할 수 있다. 또한 수용자들은 어떤 값이 채택됐는지\(어떤 후보가 당선됐는지\)에 대해 알고 싶어한다.

![](/assets/3.PNG)

우리가 처음에 알 수 있듯이, 제안자들만이 값이 선택되었다는 것을 알 수 있지만 우리는 수용자들도 결국 이 값을 찾아내길 원한다. 그 값을 수용자가 상태 머신에 실행을 위해 반영하게 하기 위해서 수용자는 결국 그 값을 알아낼 수 있어야 한다.

사실 전통적인 Lamport 방식에서는 리스너\(Listener\)라고 불리는 세번째 구성원도 있지만, 이 강의에서는 우린 리스너와 수용자를 한 번에 묶어낸 개념으로 설명한다. 별도로 구분하지 않겠다는 뜻이다. 또한 이 강의에서 우리는 서버는 각각 한 명의 제안자와 한 명의 수용자를 함께 포함하고 있는 것으로 가정한다. Paxos를 구현할 때 각 서버를 한 명의 구성원의 역할만 하게끔 구현할 수도 있지만 이 강의에서는 하나의 서버가 한 명의 제안자가 될 수도, 수용자가 될 수도 있는 구조로 설명한다.

지금부터 보여주는 몇가지 상황은 우리가 합의\(컨센서스\)를 제대로 설계하기 위해 꼭 해결해야만 하는 문제점들이다. 아래의 첫번째 템플릿을 살펴보자. 문제점이 발생하는 상황에 대한 템플릿이다.

![](/assets/4.PNG)

위 템플릿은 우리가 단 하나의 수용자를 뒀을 때 발생할 수 있는 문제점을 다룬다. 단 한 명의 수용자에게 우리는 모든 제안자가 자신의 값을 택해달라고 보내는 요청들을 처리하라고 시켰다고 가정해보자. \(혹사 시킨다는 생각이 물씬 든다.. 이러다가 파업이라도 하면 어떻게 될런지..\)

아마도 수용자는 하나의 값을 선택해서 그 값을 채택되었다고 처리할 것이다. \(위 템플릿에서는 jmp 명령어를 채택했다.\) 매우 간단하네! 라고 생각할 수 있지만 그 단 한 명의 수용자가 만약 다운되거나 제기능을 못할 때를 처리하지 못하는 설계다.

만약 이 수용자에게 이상이 생긴다면, 우리는 어떤 값이 선택되었는지 알 수 없다. 수용자가 살아날 때까지 기다릴 수밖에 없다.

> if the acceptor crashes right after choosing, we have no way of knowing which value was chosen and so we'd have to wait for that acceptor restart.

우리가 구현하고자 하는 시스템의 목표는 대다수의 서버\(노드\)만으로 완벽한 서비스를 차질 없이 제공하는 것이기 때문에 첫번째 문제점에 나타난 것처럼 하나의 수용자만 두는 설계는 올바르지 않다. 이러한 문제점을 개선하기 위해 우리는 일종의 정족수\(quorum\)개념을 도입한다. 합의를 위해 모인 의사구성원들이 의결을 하는데 필요한 최소한도의 인원수를 지정하는 것이다. 

정족수는 보통 3, 5 또는 7과 같이 적은 숫자의 홀수만큼의 수용자들로 구성한다. 이렇게 하면 한 명의 수용자가 망가져도, 대다수의 수용자가 제기능을 할 수 있기 때문에 값을 채택하는 매커니즘은 여전히 정상작동할 수 있다.

만약 값을 수용한 수용자가 그 즉시 바로 다운되는 경우에는 다른 수용자들이 그 수용자가 어떤 값을 택했는지 확인하고 처리할 수 있기 때문에 이 정족수의 개념은 굉장히 유용하다. 하지만 정족수 개념을 도입해도 아직 해결해야할 과제들이 남아있다. 아래의 템플릿을 살펴보자.

![](/assets/5.PNG)

만약 수용자가 자신이 첫번째로 받은 값만 수용한다고 가정해보자. S1\(1번 서버\)은 클라이언트가 보낸 red라는 정보를 수용했다. 그리고 자신이 수용한 값이 채택되는 걸 원하기에 다른 서버에게 전파하는 작업을 진행한다. 그와 거의 동시에 S3와 S5도 클라이언트로부터 요청을 받는다. 각각 blue와 green이 채택되게 도와달라고 부탁을 받은 상태고, 이 값들이 채택될 수 있게 S3와 S5도 전파작업을 진행한다.

이렇게 동시다발적으로 이루어진 결과, S1은 S1과 S2의 동의를 얻었고 S3는 S3와 S4의 동의를 얻었다. S5는 자신 빼고는 아무도 없다. 결국 다수결로 의사결정을 진행하지 못하게 되는 상태에 빠졌다. 3명의 찬성을 얻은 안건이 있어야 통과가 되는데 어느 누구도 3개의 표를 얻지 못했다. 이 결과가 시사하는 점은 수용자는 때때로 자신이 투표했던 것을 다시 번

