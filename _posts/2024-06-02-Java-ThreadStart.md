---
title: "[Java] Thread의 start 메서드"
date: 2024-06-02 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
## Thread의 run 메서드
---
thread를 실행시키려면 run 메서드를 실행시키는 것으론 안된다.

run만 실행시켰을 땐 새로운 **call stack이 생겨나지 않는다.**   
다른 객체들과 마찬가지로 메서드가 **호출되기만 할 뿐**이다.

run을 호출 했을 때의 Call Stack의 모습은 아래와 같다.

![](images/2024-06-02-Java-ThreadStart.png)

run 메서드가 끝나기 전까지는 main 메서드는 실행되지 못한다. (싱글쓰레드)

즉, 독립적인 실행흐름을 가지지 못한다.

따라서, start 메서드는 어떻게 Call Stack을 만들어 주는 걸까?

## Thread의 start 메서드
---
start 메서드의 코드를 뜯어보면 아래와 같다.

```java
public synchronized void start() {
   /**
   * This method is not invoked for the main method thread or "system"
   * group threads created/set up by the VM. Any new functionality added
   * to this method in the future may have to also be added to the VM.
   *
   * A zero status value corresponds to state "NEW".
   */
   if (threadStatus != 0)
      throw new IllegalThreadStateException();

  /* Notify the group that this thread is about to be started
   * so that it can be added to the group's list of threads
   * and the group's unstarted count can be decremented. */
   group.add(this);

  boolean started = false;
   try {
      start0();
      started = true;
   } finally {
      try {
            if (!started) {
               group.threadStartFailed(this);
            }
      } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
            it will be passed up the call stack */
      }
   }
}
```

보다시피 start 메서드를 호출하면 **start0 메서드를 호출**한다.

start0 메서드는 뭘까??

![](images/2024-06-02-Java-ThreadStart-1.png)

start0 메서드는 **native 메서드**이다.    
start0 메서드를 호출하면 **JVM_StartThread라는 cpp함수**를 호출한다.

[공신문서](https://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/9b0ca45cd756/src/share/vm/prims/jvm.cpp)를 가보면 JVM_StartThread 함수의 구현코드를 확인할 수 있다.

요약해보면 다음처럼 정리할 수 있다.
- 입력받은 **Thread 인스턴스가 null이 아니면**(이미 실행했던 Thread 일 경우) 예외를 발생시킨다.
- **Call Stack 사이즈를 계산**하여 Thread를 생성하고 start한다.

이 때, 이미 실행했던 Thread라서 발생시키는 예외는 **IllegalThreadStateException**이다. 


#### Reference
---
[블로그](https://kim-jong-hyun.tistory.com/101)
책 : 자바의 정석