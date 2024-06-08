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





## MASM 기본 문법
### 어셈블리 기본 코드
#### 함수 이름
* VC 에서 C언어로 foo 함수를 만들면
* visual studio 컴파일러는 컴파일시에 함수 이름 앞에 "_" 를 추가함 -> _foo
* c 표준이 아니라 컴파일러마다 다르게 동작.
* asm_main 함수를 만들거나 호출 할때 함수이름
  | C언어 | asm_main() |
  |------- | ---------|
  |인라인 어셈블리 | asm_main()|
  |어셈블리 파일 | **_asm_main()** |

``` c++
  // main.c
#include <stdio.h>

int asm_main();

//void foo();

int main()
{
	//foo();

	int n = asm_main(); // call _asm_main	

	printf("result : %d\n", n);
}

```
#### MASM Directive
* 대소문자를 구별하지 않음
* 파일 끝에 반드시 "end"가 있어야 함
* .model directive (memory model, language-type, stack option)(16,32bit에서만 사용)
  | | 32bit | 16bit |
  |----| ----| -----|
  |memory model | FLAT | TINY, SMALL, COMPACT, MEDIUM. LARGE, HUGE, FLAT|
  |language-type| C, STDCALL| C, BISCI, FORTRAN, PASCAL, SYSCALL, STDCALL|
  | Stack-option| NOT USED | NEARSTACK, FARSTACK|
  
* 1. .model flat
  2. .model flat, C
  3. .model flat, stdcall
 #### PE Format
 ![image](https://github.com/jiin212/WINDOWS_SYSTEM/assets/49266230/a0fad917-4f75-4fb9-894b-9cafef88f44a)
#### 함수를 만드는 방법
* "함수이름:"
* ret 명령의 함수 종료
* eax 레지스터에 넣은 값이 함수 반환값
* 함수를 다른 파일(C언어)에서 호출하기 위해서는 Public 지시어를 사용해서 선언.
```asm
; asm1.asm

.model flat

public _asm_main

;.data
; 전역변수 만드는 자리.

.code   ; .text 

_asm_main:
	mov		eax, 100
	ret

end

```
* 이런 표기법도 가능
```asm
; asm2.asm

.model flat

public _asm_main

_DATA SEGMENT
; 전역변수 만드는 자리.
_DATA ENDS

_TEXT SEGMENT

_asm_main proc
	mov		eax, 100
	ret
_asm_main endp

_TEXT ends

end
```


#### C 소스코드를 어셈블리 파일로 생성하는 방법
* Developer Command Prompt 실행 후, 소스가 있는 디렉터리로 이동
* cl 소스이름.c/FAs /c
* "소스이름.asm" 파일 생성
* 

### Data Segment

#### 전역변수 만들기
![image](https://github.com/jiin212/WINDOWS_SYSTEM/assets/49266230/1fb8f4ee-91d3-40c8-ad58-a9a84e11e4b9)


 ```c
// call.c
int add()
{
	return 300;
}
void __fastcall foo(int a, int b, int c, int d)
{

}
int main()
{
	foo(1, 2, 3, 4);
	add();
}

```
```asm
; asm1.asm
.model flat

public _asm_main

.code

_asm_main:	
	mov		eax, 100

	;  다른 함수 호출
	;mov	ebx, POS_A
	;push	POS_A
	;jmp		_add
;POS_A:
 
	call	_add

	ret	


_add:
	mov		eax, 300
	ret

	;pop		ebx
	;jmp		ebx


end

```
