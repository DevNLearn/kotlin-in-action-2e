# 4챕터 질문 정리

## 질문 정리

---

1. 클래스 설계할 때 어떤 기준으로 생성자 vs 팩토리 메서드를 선택하시나요?
    - 질문 답변
        - 생성자 사용
            - 로직이 단순하고 외부에서 객체 생성의 별 제한이 없는 경우
        - 팩토리 메서드 사용
            - 로직이 복잡하거나 이름 있는 생성이 필요한 경우
            - 생성자를 외부에서 사용하지 못하게 할 때
2. 아래 코드는 가시성 컴파일 오류 왜 날까?
    - 질문 답변
        - 관련 코드
            
            ```kotlin
            internal open class TalkativeButton : Focusable{
              private fun yell() = ...
              protected fun whisper() = ...
            }
            
            fun TalkativeButton.giveSpeech(){ // 오류: public 멤버가 자신의 internal 수신 타입인 TalkativeButton을 노출
              yell() // private 접근 오류
              whisper() // protected 접근 오류
            }
            ```
            
        - 컴파일 오류를 없애려면
            - TalkativeButton 클래스를 public으로 변경
            - giveSpeech 확장 함수를 internal로 변경
3. value class 직렬화시 맹글링 발생 이유
    - 질문 답변
        - 컴파일 시 내부적으로 래핑을 제거하고 속성만 남기는데 게터 생성시 이름 충돌을 막기 위해 맹글링
4. 데이터 클래스는 불변 객체로 만드는 게 권장인데, 왜 `var`를 허용할까?
    - 질문 답변
        - 불변 사용을 권장하지만 유연성을 제공하는 개념인 것 같다.
        - 즉 권장사항일뿐 제약조건으로는 두지 않은 것