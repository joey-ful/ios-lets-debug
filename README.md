아래의 문제를 해결하기 위해서 참고할 자료는 WWDC 세션에서만 찾습니다.   
그 외에는 [LLDB 공식 문서](https://lldb.llvm.org/)까지만 허용합니다.  
다른 곳에서 자료를 찾는 것은 반칙입니다. 여러분의 양심에 맡깁니다.

# 참고할 WWDC 세션 힌트

* Debugging Tips and Tricks
* Debugging with Xcode 9
* Visual Debugging with Xcode
* LLDB: Beyond "po"

# 문제

## 1. ViewController.swift 파일의 23번째 줄에 브레이크 포인트를 설정하려면 입력해야 하는 LLDB 명령어는?

```shell=
br s -f ViewController.swift -l 23
```

## 2. changeTextColor라는 심볼에 브레이크 포인트를 설정하기 위해 입력해야 하는 LLDB 명령어는?
```shell=
br s -n changeTextColor
```

## 3. Breakpoint Navigator를 통해 titleLabel의 text가 "두 번째 뷰 컨트롤러!"인 경우에만 작동을 일시정지하고 titleLabel의 text를 출력하는 액션을 실행하도록 설정해보세요

### Breakpoint Navigator

1. Symbolic Breakpoint 생성

![image](https://user-images.githubusercontent.com/52592748/129889958-d034765c-f434-4397-a365-6eb0c1614303.png)


2. 세부 사항 입력
  - Symbol에 함수명 입력
  - Condition에 titleLabel.text == "두 번째 뷰 컨트롤러!" 조건 입력
  - Action에 Debugger Command 선택하고 "po titleLabel.text!" 로 출력

![image](https://user-images.githubusercontent.com/52592748/129889021-e4e9e829-0da3-4a47-b200-c6cf83e2e391.png)


3. @objc 메서드는 "Enable Breakpoint Location" 체크 해제

![image](https://user-images.githubusercontent.com/52592748/129889033-e3a5ccbf-f89b-4b89-af18-24f70733fe42.png)

### Command line으로는 다음과 같이 구현 가능

```bash
br s --name ViewController.viewDidLoad -c "titleLabel.text == \"두 번째 뷰 컨트롤러!\"" -C "po titleLabel.text!"
br di 1.2
```

- @objc 메서드는 disable 할 것
    - breakpoint번호.2 를 disable 하면 됨
- condition의 문자열은 `"` 로 감싸준다
- condition 내부에 `"` 가 존재하는 경우 escape 해주면 된다



## 4. 오류(Error) 혹은 익셉션(Exception)이 발생한 경우 프로세스의 동작을 멈추도록 하는 방법에 대해 알아봅시다

![image](https://user-images.githubusercontent.com/52592748/129889058-66f41d75-ceb5-4c81-81b8-9a4c0e03b2a1.png)

## 5. View Controller의 뷰 위에는 사용자 눈에 보이지 않는 뷰가 있습니다. 이 뷰의 오토레이아웃 제약을 확인해서 알려주세요

보이지 않는 뷰는 두 번째 View Controller의 정사각형 View
- 1:1 aspect ratio 제약이 걸려있다.

  <img src="https://user-images.githubusercontent.com/52592748/129889119-001b6a2f-7d7f-498f-8816-686d4f52690f.png" width="300"/><br>

1. viewDidLoad에 breakpoint 걸기

```shell=
br s -f ViewController.swift -l 14
```

2. 시뮬레이터에서 NEXT 눌러서 두 번째 ViewController로 이동하기
![image](https://user-images.githubusercontent.com/52592748/129889207-e34200c6-ae70-4cc0-be81-53d0ee546964.png)


3. 두 번째 VC에 어떤 뷰들이 있는지 확인해보기
```shell=
po self.view.subviews
```
- UIView는 subview 중 0번째 항목
![image](https://user-images.githubusercontent.com/52592748/129889189-8be419c6-e505-487b-b24a-5cc2c0db4499.png)

4. 제약 확인하기
```shell=
po self.view.subviews[0].constraints
```
- width와 height가 같다는 제약이 걸려있다 => 1:1 aspect ratio
![image](https://user-images.githubusercontent.com/52592748/129889239-9de9913a-e68c-460e-bb41-6b424e484261.png)


## 6. 디버그 모드로 실행중인 상태에서 사용자 눈에 보이지 않는 뷰의 색상을 분홍색으로 변겅해보세요

5번의 UIView의 색상을 바꿔야 한다

1. expression 명령어로 subview들 확인하기
```shell=
e self.view.subviews
```
- 아까 po로 출력한 결과와 비교해보면 UIView에 해당하는 것은 0번째 항목이다
![image](https://user-images.githubusercontent.com/52592748/129889256-1804e461-8027-4ac2-a099-51336489b2de.png)

2. 색상 바꾸기
```shell=
e $R18[0]!.backgroundColor = UIColor.systemPink
c
```
- 색상을 변경하고 continue해주면 실제로 시뮬레이터에서 색이 바뀌어있다
<img src="https://user-images.githubusercontent.com/52592748/129889300-c1f70774-3941-4809-bf2c-16c95e22b2d3.png" width="200"/>


## 7. 두 번째 뷰 컨트롤러의 뷰가 화면에 표시된 상태에서, 두 번째 뷰 컨트롤러 까지의 메모리 그래프를 캡쳐해보세요

![image](https://user-images.githubusercontent.com/52592748/129889347-708a9b4c-22e5-4a0d-b808-b385b9e13fd7.png)

## 8. LLDB의 특정 명령어의 별칭을 설정해줄 수 있는 명령어는 무엇일까요?

```shell=
command alias [별칭] [명령어]
```

## 9. LLDB의 v, po, p 명령어의 차이에 대해 알아봅시다
> 참고
> [Inspecting Variables with LLDB - Intermediate Debugging in iOS](https://www.youtube.com/watch?v=WwBUcof0lKw)
![image](https://user-images.githubusercontent.com/52592748/129889414-008222e0-a98f-4768-a832-2c30ab60c139.png)

셋 다 data를 출력하는 명령어

### v
frame variable
- 메서드를 실행하지 못 한다는 제약이 있다. 대신 안전하다
- data만 볼 수 있는 명령어
- built-in formatter로 출력한다
![image](https://user-images.githubusercontent.com/52592748/129889581-7f244ce6-9e30-47d2-a53f-f72844546cb7.png)

### p
- 메서드를 실행하는 것도 가능하고 데이터를 보는 것도 가능하다
- built-in formatter로 출력한다
![image](https://user-images.githubusercontent.com/52592748/129889619-751550d2-8ad7-40d1-b4a1-f8f8669ef106.png)

### po
- 디버거가 생성한 스코프에서 메서드를 실행하는 것이 가능하다
- self나 추론 등의 기능들을 모두 사용할 수 있다
- 객체의 debug description method를 호출할 수 있다
![image](https://user-images.githubusercontent.com/52592748/129889636-9200e371-e36c-43fd-b90b-3e858025f9a7.png)





