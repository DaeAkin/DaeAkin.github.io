# CPU



## Mode bit

CPU에는 mode bit이 존재한다. 사용자가 프로그램의 **잘못된 수행**(예를 들어 무한 for문)을 실수로 실행해서, 다른 프로그램이 CPU를 사용하지 못하게 될 경우, 다른 프로그램과 운영체제에게 피해를 주기 때문에, 이를 막기위한 보호 장치로 사용한다.

Mode bit 은 두 가지 종류가 있다.

- mode bit이 1이 라면 사용자 프로그램을 실행 중이다.
- mode bit이 0이 라면 OS 코드를 실행 중이다.

mode bit이 1인 걸 **<u>사용자 모드</u>**라고 말하고, mode bit이 0 인 걸 **<u>커널 모드 또는 모니터 모드</u>**라고 한다.

I/O 접근은 보안상으로, 사용자 프로그램이 직접 접근하면 안되고, 운영체제만 접근이 가능해야만 한다. 그래서 I/O 수행은 이 mode bit을 보고 커널 모드가 의뢰한 일이면 수행을 하고, 사용자 모드가 의뢰한 일이면 수행을 하지 않는다.



## 타이머

- 정해진 시간이 흐른 뒤 OS에게 제어권이 넘어가도록 인터럽트를 발생시킨다.
- 타이머는 매 클럭 틱 때마다 1씩 감소된다.
- 타이머 값이 0이 되면 타이머 인터럽트가 발생된다.
- CPU를 특정 프로그램이 독점하는 것으로 보호하기 위해 있다.

타이머는 현재 우리가 사용하고 있는 운영체제인 windows , 리눅스 , mac 등 시분할 운영체제를 구현하기 위해 널리 이용되고 있다.



## 인터럽트

인터럽트 당한 시점의 <u>register</u>와 <u>program counter</u>를 save한 후 CPU의 제어를 인터럽트 처리 루틴에 넘긴다.

인터럽트는 크게 두 가지 종류가 있다.

- **Interrupt(하드웨어 인터럽트)** : 하드웨어가 발생시킨 인터럽트
- **Trap(소프트웨어 인터럽트)** 
  - Exception : 프로그램이 오류를 범한 경우
  - System call : 프로그램이 커널 함수를 호출하는 경우

예를 들어 자바 로직을 수행하고 있는 도중에, Scanner나 IntpuStream을 이용해서 I/O 작업이 필요하게 되어 이 클래스를 호출하게 되면, System call이 일어나게 된다. 그럼, Trap 인터럽트가 걸리게 되어서, OS에게 I/O 작업을 요청하게 된다.

**인터럽트 관련 용어**

- 인터럽트 벡터
  - 해당 인터럽트의 처리 루틴 주소를 가지고 있음
- 인터럽트 처리 루틴( = Interrupt Service Routine, 인터럽트 핸들러)
  - 해당 인터럽트를 처리하는 커널 함수



# 메모리

##  Logical address (=virtual address)

- 프로세스마다 독립적으로 가지는 주소 공간
- 각 프로세스마다 0번지 부터 시작
- CPU가 보는 주소는 **<u>Logical address</u>** 임.



## Physical address

- 메모리에 실제 올라가는 위치



## 주소 바인딩

**메모리의 주소를 결정하는 것이다.**

Symbolic Address -> Logical Address -> Physical Address

- Symbolic Address : 프로그래머가 메모리를 사용하지만, 프로그래머 입장에서는 숫자가아닌(메모리 주소가 아닌) , 변수에 값을 할당하는 작업으로 메모리를 사용한다.



### 주소 바인딩이 일어나는 시점

- **Compile time binding**
  - 물리적 메모리 주소(physical address)가 컴파일 시 알려짐.
  - 시작 위치 변경시 재 컴파일
  - 컴파일러는 절대 코드(absoulte code) 생성
- **Load time binding**
  - Loader의 책임하에 물리적 메모리 주소 부여
  - 컴파일러가 재배치가능코드(relocatable code)를 생성한 경우 가능
- **Execution time binding (=Run time binding)** <- 우리가 쓰고 있음
  - 수행이 시작된 이후에도 프로세스의 메모리 상 위치를 옮길 수 있음
  - CPU가 주소를 참조할 때마다 binding 점검(address mapping table)
    - 프로그램 메모리 주소가 계속 바뀌니까, 어디에 있는지 지속적인 점검이 필요함.
  - 하드웨어적인 지원이 필요하다.(e..g base and limit register , MMU)



### Memory-Management Unit(MMU)

logical 주소를 물리적 주소로 매핑해주는 하드웨어

#### MMU scheme

사용자 프로세스가 CPU에서 수행되며 생성해내는 모든 주소값에 대해 base register(=relocation register)의 값을 더한다

#### user program

logical 주소만 다룬다

실제 물리적 주소를 볼 수 없으며 알 필요가 없다.

### Dynamic Relocation

![]({{ site.url }}/images/os/dynamicRelocation.png)

- **limit registe**r : 프로그램의 최대 크기
  - 본인의 프로그램이 3000인데도 불구하고, 4000번지를 읽어달라고 요청하게 되면 다른 프로그램의 구문이 실행되므로 이를 방지하기 위한 수단.



## Dynamic Loading

**프로세스 전체를 메모리에 미리 다 올리는 것이 아니라 해당 루틴이 불려질 때 메모리에 load 하는 것**

- memory utilization의 향상
- 가끔씩 사용되는 많은 양의 코드의 경우 유용
  - 예 : 오류 처리 루틴
- **운영체제의 특별한 지원 없이 프로그램 자체에서 구현 가능 (<u>OS는 라이브러리를 통해 지원 가능</u>)**

다이나믹 로딩은 개발자가 수작업으로 OS 라이브러리를 통해서 코드로 직접 구현해야 한다. 

**<u>OS가 필요없는 메모리를 Swap으로 보내고, 필요한 부분을 자동으로 올리는건 "페이징"기법이기 때문에, Dynamic Loading하고 연관은 없음.</u>**



## Overlays

**Dynamic Loading** 과 똑같은 방법이지만, **<u>OS 라이브러리를 이용하지 않고, 프로그래머가 오로지 코드로만 작성해서 구현한다.</u>**



## Swapping

프로세스를 일시적으로 메모리에서 backing store로 쫓아내는 것 <- 리눅스 설치할 때 Swap 공간의 크기를 정할 수 있다.

- Backing stroe(=swap area)
  - 디스크
    - 많은 사용자의 프로세스 이미지를 담을 만큼 충분히 빠르고 큰 저장 공간
- Swap in / Swap out
  - 일반적으로 중기 스케줄러(swapper)에 의해 swap out 시킬 프로세스 선정
  - **우선순위 기반 CPU 스케줄링 알고리즘**
    - 우선순위가 낮은 프로세스를 swapped out 시킴
    - 우선순위가 높은 프로세스를 메모리에 올려 놓음
  - **Compile time** 혹은 **load time binding**에서는 원래 메모리 위치로 **swap in** 해야함
  - **Execution time binding**에서는 추후 빈 메모리 영역 아무 곳에나 올릴 수 있음
  - **swap time**은 대부분 **transfer time**(**swap** 되는 양에 비례하는 시간)임



## Dynamic Linking

**Linking을 실행 시간(execution time)까지 미루는 기법**

Linking : 여러군데 존재하는 컴파일 된 파일들을 묶어서 하나의 실행파일을 만드는 과정

### Static linking 

- **라이브러리가 프로그램 실행 파일 코드에 포함됨**
- 실행 파일의 크기가 커짐
- 동일한 라이브러리를 각각의 프로세스가 메모리에 올리므로 메모리 낭비(eg. printf 함수의 라이브러리 코드)

### Dynamic linking

- **라이브러리가 실행시 연결(link) 됨**
- 라이브러리 호출 부분에 라이브러리 루틴의 위치를 찾기 위한 stub이라는 작은 코드를 둠
- **라이브러리가 이미 메모리에 있으면 그 루틴의 주소로 가고 없으면 디스크에서 읽어옴**
- 운영체제의 도움이 필요 
- eg : 윈도우의 dll파일



## 물리적 메모리 할당

메모리는 일반적으로 두 영역으로 나뉘어 사용한다.

- **OS 상주 영역**
  - interrupt vector와 함께 낮은 주소 영역 사용
- **사용자 프로세스 영역**
  - 높은 주소 영역 사용



### 사용자 프로세스 영역의 할당 방법

- **Contiguous allocation (연속적인 할당)**
  - 각각의 프로세스가 메모리의 연속적인 공간에 적재되도록 하는 것
  - Fixed partition allocation
  - Variable partition allocation
- **Noncontiguous allocation**
  - 하나의 프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있음
  - Paging
  - Segmentation
  - Paged Segmentation

#### Contiguous allocation

- **고정분할(Fixed partition) 방식**
  - **물리적 메모리를 몇 개의 영구적 분할로 나눔**
  - 분할의 크기가 모두 동일한 방식과 서로 다른 방식이 존재
  - 분할당 하나의 프로그램 적재
  - 융퉁성이 없음
    - 동시에 메모리에 load되는 프로그램의 수가 고정됨
    - 최대 수행 가능 프로그램 크기 제한
  - **Internal fragmentation 발생 (external fragmentation도 발생)**
- **가변분할(Variable partition) 방식**
  - **프로그램의 크기를 고려해서 할당**
  - 분할의 크기, 개수가 동적으로 변함
  - 기술적 관리 기법 필요
  - **External fragmentation 발생**

![]({{ site.url }}/images/os/contiguous allocation.png)

> External fragmentation (외부 조각)
>
> - 프로그램 크기보다 분할의 크기가 작은 경우 
> - 아무 프로그램에도 배정되지 않은 빈 곳인데도 프로그램이 올라갈 수 없는 작은 분할
>
> Internal fragmentation (내부 조각)
>
> - 프로그램 크기보다 분할의 크기가 큰 경우
> - 하나의 분할 내부에서 발생하는 사용되지 않는 메모리 조각
> - 특정 프로그램에 배정되었지만 사용되지 않는 공간



### Hole

- 가용 메모리 공간
- 다양한 크기의 hole들이 메모리 여러 곳에 흩어져 있음
- 프로세스가 도착하면 수용가능한 hole을 할당
- 운영체제는 다음의 정보를 유지
  - 할당 공간
  - 가용 공간(hole)

![]({{ site.url }}/images/os/hole.png)

### Dynamic Storage-Allocation Problem

**가변 분할 방식에서 size n인 요청을 만족하는 가장 적절한 hole을 찾는 문제**

#### First fit

Size가 n 이상인 것 중 최초로 찾아지는 hole에 할당

#### Best-fit

- Size가 n 이상인 가장 작은 hole을 찾아서 할당
- Hole들의 리스트가 크기순으로 정렬되지 않은 경우 모든 hole의 리스트를 탐색해야함
- 많은 수의 아주 작은 hole들이 생성됨

#### Worst-fit

- 가장 큰 hole에 할당
- 역시 모든 리스트를 탐색해야 함
- 상대적으로 아주 큰 hole들이 생성됨

Fisrt-fit과 best-fit이 worst-fit보다 속도와 공간 이용률 측면에서 효과적인 것으로 알려짐 (실험적인 결과)

#### compaction

- 외부조각 문제를 해결하는 한 가지 방법
- 사용 중인 메모리 영역을 한군데로 몰고 hole들을 다른 한 곳으로 몰아 큰 block을 만드는 것
- 매우 비용이 많이 드는 방법이다.
- 최소한의 메모리 이동으로 compaction 하는 방법은 매우 복잡한 문제임
- compaction은 프로세스의 주소가 실행 시간에 동적으로 재배치 가능한 경우에만 수행될 수 있다