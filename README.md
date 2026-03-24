# Easy Crack

해당 프로그램을 분석하기 위해 주요코드들이 어디있는지 찾기 위해 우선 실행을 해보았고, Incorrect Password라는 문자열이 언급되는 부분을 중심으로 찾아보기 시작하면 될 것 같다.
![](https://velog.velcdn.com/images/jini135wii/post/0a37ae76-0206-49c7-80ed-1f0632bf3fda/image.png)


아마 저 일치하는 password를 찾는 것이 문제인 것 같고, 코드 내에서 password를 인증하는 함수가 있을것이니 해당 인증연산을 역산하면 될 것이다.

Ida를 이용해서 해당 문자열이 언급되는 부분을 확인해보니
 
![](https://velog.velcdn.com/images/jini135wii/post/d7e021d4-c322-46bd-bf70-af8597aa87ff/image.png)

생각보다 더 쉽게 끝낼 수 있을 것 같다.
Str2가 뭔지만 알면 되는데, 이는 5y라는 것을 정적분석을 통해 알수있다.
아래 조건을 만족하면 끝이므로 Str2가 뭔지만 알면 된다.
```
string[1] != 'a' || 
strncmp(&String[2], Str2, 2u) || 
strcmp(&String[4], aR3versing) || 
String[0] != 'E' )
```
password는 --

# Easy Keygen

시리얼번호가 `5B134977135E7D13`일 때의 이름을 찾으라고 한다.
이전 문제와 같이 문자열을 먼저 보고 이를 언급하는 함수를 찾아가보면 된다.
 ![](https://velog.velcdn.com/images/jini135wii/post/54e5e99f-4846-4aa8-bc93-b626f208791b/image.png)

해당 함수다.
이름으로부터 시리얼넘버를 계산하는데, 단순히 `v7(0x10, 0x20, 0x30)`을 번갈아 xor하는 것 같다. 
```
serial = "5B134977135E7D13"

serials = [int(serial[2*i:2*i+2], 16) for i in range(len(serial)//2)]
print(serials)
print("".join(chr(serials[i]^[0x10,0x20,0x30][i%3]) for i in range(len(serials))))
```

해당하는 name은 --

# Music Player

뮤직플레이어 프로그램에서 1분 듣기만 제공해주는데 해당 제약을 우회하는 방법을 찾는 것이 문제에서 요구하는 바이다.
결국 문자열이 어디서 접근되는지가 관건인데 여기서는 한글을 이용하기 때문에 IDA의 strings 기능에서 찾아주지를 않는다. 그래서 ‘1분’이라는 글자를 UTF-16LE 방식으로 적으면 ’`31 00 84 BD`’이기 때문에 이렇게 검색을 했고 2BAC 위치에 있다는 것을 발견했다. 해당 주소의 xref을 찾아보면 될 것 같다. 참고로 IDA 내에서 Alt+A를 누르면 아래와 같이 인코딩 방식을 바꿀 수 있다. 
![](https://velog.velcdn.com/images/jini135wii/post/1f495ef0-9d00-4686-ab3c-81469894bf3a/image.png)
60초가 지나면 저 창이 뜨게 되는 것이니 대충 생각해보면 `v29[0]`이라는 변수가 시간을 나타내고 해당 값이 `60000`을 넘으면 창이 띄워지는 방식인 것 같다.
![](https://velog.velcdn.com/images/jini135wii/post/5053bc8e-4cec-4f21-a924-0b62b68bebfd/image.png)
![](https://velog.velcdn.com/images/jini135wii/post/dfcc7279-23b1-42a2-ac47-05eef58e1d06/image.png)
그래서 우선 `0x7FFFFFFF` 전까지는 계속 실행할 수 있도록 해놨는데, 동적분석을 해보니 이 `v29[0]`에 들어있는 변수는 지나간 시간을 체크해주는 애인거같고, 1분을 기다려보면 아래와 같은 에러메시지가 뜨는 것을 보니 올바른 접근 방식은 아닌 것으로 보인다.
![](https://velog.velcdn.com/images/jini135wii/post/b546b2b4-e994-44d4-b820-e0e6ce48a495/image.png)
이 문제의 취지를 다시 생각해봐야겠다. 60초를 넘기면 나오는 코드들을 강제 실행시키는것이 아니라 음악이 60초를 딱 넘기게 되면 무언가가 될 것이다. 그래서 60초를 못넘기게 하는 조건들만 없애는 방식으로 하자. 우선 동적 분석을 해본 결과 시간이 60초를 넘었을 때 Runtime Error가 뜨게하는 코드는 아래 부분이였다.
![](https://velog.velcdn.com/images/jini135wii/post/69742700-02f1-4417-acdc-407702894172/image.png)
ai한테물어보니까 error code 380이 뜨게하는 것은 Max값을 넘어서 시간이 재생되었기 때문이라고 하고 스크롤바에서 그게 나오게되는것같다.
아무튼 저 예외처리를 하는 부분도 그냥 없애버리고 진행시켰는데, 그냥 노래만 재생되어 이게 아닌가 싶어 넘어가려했는데, 프로그램을 다시 확인해보니 이름이 바뀌어있었다.

# Replace

대충 문제 이름을 보면 어떤 부분을 패치하는 것 같다.
 ![](https://velog.velcdn.com/images/jini135wii/post/a21e6fcb-8578-42a0-8c37-166bfacd9760/image.png)

Correct라는 문자열이 나오기 위한 방법을 찾는 중에 Correct라는 결과를 내는 경로자체가 없다는 것을 알 수 있었다.
 ![](https://velog.velcdn.com/images/jini135wii/post/66c12c00-698c-4c74-9c89-8001e3acb050/image.png)

jmp를 단순히 없애면 Correct가 뜨긴 할텐데 그렇게하면 reversing.kr에 제출할 답 자체가 없다. 그러니 다른 방법을 찾아보도록 하겠다.
디스어셈블된 코드를 보면 입력값에 대해 특정한 연산을 가하는 것이 보이기는 하는데 어떤 부분을 어디부터 어디까지 패치해야 올바르게 패치를 한 것인지 감이 오질 않는다.
동적분석을 하면서 input이 어떻게 변하는지 확인해보면 input <- input + 2 + 601605c7 + 2 = input + 601605cb와 같이 변한다.
돌다보면 아래코드처럼 입력값 주소와 그 뒤에 있는 값을 0x90으로 바꾸는 코드가 있는데, 이 입력값이 잘못되면 권한침해로 프로세스가 끝난다. (eax = 입력값)
 ![](https://velog.velcdn.com/images/jini135wii/post/b934e4fe-8557-401f-ba81-2c38310cd1f9/image.png)

아마 jmp instruction이 2바이트이기 때문에 딱 그 위치에 집어넣으면 딱일 것 같다. 그러면 eax = 0x401071이 되게 하면 된다.
 답은 ---
프로그램을 패치하는 것인가 했는데 이 프로그램 자체가 데이터를 수정하는 것이기 때문에 내가 했던 고민이 전혀 상관없이 아주 명확한 답이 있는 문제였다.  

# ImagePrc

 ![](https://velog.velcdn.com/images/jini135wii/post/31865cc1-57f6-4d84-bb66-209823e029aa/image.png)

해당 함수에서 내가 한 싸인이랑 비교를 진행하는 것 같고 비교 대상이 되는 메모리를 그대로 복사해서 붙여넣으면 될 것 같다. 
windbg에서
```> .writemem [저장위치] [주소] [크기]```
를 입력하여 싸인을 파일로 뽑아내고, 이 포맷이 bmp 이미지 파일과 동일하여 확인해보니 특정 싸인이 나왔다.
같은 방식으로
```> .readmem [저장위치] [주소] [크기]```
을 입력하여 비교연산을 통과시킬 수 있다. 그래서 했는데 아무것도 변한게없다.

너무 짧아서 아니겠거니 했는데, 그냥 이미지 파일이 정답이였다.
