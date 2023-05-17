# Spring Request DTO

## Issue
Spring Boot로 REST API를 테스트 하는 과정에서 이슈가 발생했다.
DTO 클래스를 만들고 값을 입력 받았는데 의도치 않게 계속해서 null값이 들어갔다.
변수명을 바꿔가면서 작성해보니 null값이 들어가지 않는 것을 확인했다.

## Jackson
Spring은 JSON 데이터를 매핑하기 위한 Message Converter로 Jackson을 사용한다.
Jackson에 대해 알게된 것 중 하나는 Jackson은 Getter 의 이름을 기반으로 Json Key 값을 만든다.
