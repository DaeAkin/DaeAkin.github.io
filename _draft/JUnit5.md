## JUnit5

이전 JUnit 버전과 다르게, JUnt5는 세개의 서브 프로젝트로 이루어져 있다.

JUnit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

JUnit Platform은 JVM에서 테스팅 프레임워크를 실행하는데 기초를 제공한다. 또한 TestEngine API를 제공해 테스팅 프레임워크를 개발할 수 있다.

JUnit Jupiter는 JUnit 5에서 테스트를 작성하고 확장을 하기 위한 새로은 프로그래밍 모델과 확장 모델의 조합이다.

JUnit Vintage는 하위 호완성을 위해 JUnit3과 JUnt4를 기반으로 돌아가는 플랫폼에 TestEngine을 제공해준다.

## 요구사항

JUnit5은 java 8부터 지원하며, 이전 버전으로 작성된 테스트 코드여도 컴파일이 정상적으로 지원된다.



