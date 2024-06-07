# WINDOWS_SYSTEM

## 인라인 어셈블리와 MASM
### Inline assembly 
* .c 파일에서 사용할 수 있다. 
* **"Stack"** 에 인자를 넣고 함수로 이동
* 함수에서는 EAX 레지스터로 반환값을 전달한다.
* **32bit** 환경에서만 사용가능 (x86)
```c++
// inline_asm1.c
#include <stdio.h>

int Add(int a, int b)
{
	int c = a + b;

//	return c;	// eax <= c

	__asm
	{
		mov   eax, c   // return c
	}
}

int main()
{
	int n = 0; 

	n = Add(1, 2); // n <= eax

	/*
	Add(1, 2);

	__asm
	{
		mov  n, eax
	}
	*/


	printf("result : %d\n", n);
}
```
* 함수 반환 값 : **EAX 레지스터** 사용
* 함수 인자: **마지막** 인자부터 차례대로 **스택**에 넣고 함수로 이동
* 프로그램 실해시 운영체제에 의해 자동으로 스택이 할당 (Thread 당 한개)
* 인자 전달에 사용한 스택은 반드시 파괴되어야 한다!
```c++
// inline_asm2.c
#include <stdio.h>

int Add(int a, int b)
{
	return a + b;
}

int main()
{
	int n = 0;

	//n = Add(1, 2);

	__asm
	{
		push    2
		push    1
		call    Add 
		add     esp, 8 //이거 없으면 런타임에러

		mov     n, eax 
	}


	printf("result : %d\n", n);
}
```

### 에셈블리 빌드방법
* 어셈블리 파일은 관례적으로 ".ams" 확장자 사용
* 어셈블리에서는 세미콜론이 주석

#### 어셈블리 소스를 빌드하는 3가지 방법
* **command line에서 빌드**
  1. 개발자 명령 프롬프트 실행
  2. 소스파일이 있는 경로로 이동
  3. 아래 방법으로 빌드
     | 빌드 명령 | 빌드 결과 생성된 파일 |
     | ----------- | ----------- |
     | cl main.c **/c** | main.obj |
     | ml asm1.asm **/c** | Text |
     | link main.obj asm1.obj | main.exe|
     
     *"/c": 링크하지 말고 컴파일 하라는 옵션*
  
* **사용자 지정 빌드 명령 추가**



  .asm 확장자 인식을 못함. 속성 창에서 항목 형식 - 사용자 지정 빌드 도구
  | 명령줄 | ml asm1.asm /c |
  | ----------- | ----------- |
  | 출력 | asm.obj |

  
   *단점: 여러개의 .asm 파일 추가시 모두 설정 해주어야 함*


  
   *.asm 파일을 자동으로 인식하게 할 수 없을까? -> 3번째 방법*

* **빌드 종속성 추가**



  프로젝트 메뉴 - 빌드 종속성 - masm 체크



  .asm 속성에서 항목형식 - Microsoft Macro Assemler
 
 
```ams
; asm1.asm

.model flat

public _asm_main

.code
_asm_main:
	mov		eax, 100
	ret

end
```
```c
// main.c
#include <stdio.h>

int asm_main();

int main()
{
	int n = asm_main();

	printf("result : %d\n", n);
}
```
