# make 사용법

## Make Utility
#### make - GNU make utility to maintain groups of programs
* The purpose of the make utility is to determine automatically which pieces of a large program need to be recompiled, and issue the commands to recompile them.
#### GNU make는 보통 GNUmakefile, Makefile, makefile 중에서 하나가 있으면 그 파일을 읽게 된다. 하지만 일반적으로 Makefile을 추천하게 되는데, 그 이유는 우선 GNUmakefile은 기존의 make에서 인식을 못한다는 단점이 있고, makefile은 보통 소스 파일에 묻혀서 잘 안보이게 되기 때문이다.

## Makefile 구조
#### Makefile은 기본적으로 아래와 같이 목표(target), 의존 관계(dependency), 명령(command)의 세개로 이루어진 기분적인 규칙(rule)들이 계속적으로 나열되어 있다고 봐도 무방하다. make가 지능적으로 파일을 갱신하는 것도 모두 이 간단한 규칙에 의하기 때문이다.
```Text
target ... : dependency ...
                command
                ...
                ...
```
#### 여기서 목표(target) 부분은 명령(command)이 수행이 되어서 나온 결과 파일을 지정한다. 당연히 목적 파일(object file)이나 실행 파일이 될 것이다. 명령(command)부분에 정의된 명령들은 의존 관계(depenency)부분에 정의된 파일의 내용이 바뀌었거나, 목표 부분에 해당하는 파일이 없을 때 이곳에 정의된 것들이 차례대로 실행이 된다. 일반적으로 쉘에서 쓸 수 있는 모든 명령어들을 사용할 수가 있으며 bash에 기반한 쉘 스크립트도 지원한다.
* 참고로 목표 부분에는 결과 파일만 올 수 있는 것이 아니고, 보통 make clean 에서와 같이 간단한 레이블(label) 기능을 제공하기도 한다.
* **명령 부분은 꼭 TAB 글자로 시작해야 한다. 그냥 빈칸 등을 사용하면 make 실행 중에 에러가 난다. make가 명령어인지 아닌지를 TAB 가지고 구별하기 때문이다.**

## Makefile 예제
#### 예제 프로그램이 main.c read.c write.c로 구성되어 있고 모두 io.h라는 헤더 파일을 사용한다고 가정
* Makefile 사용하지 않을 경우1:
```Text
$ gcc -c main.c
$ gcc -c read.c
$ gcc -c write.c

$ gcc -o test main.o read.o write.o
```
* Makefile 사용할 경우1:
```Text
test : main.o read.o write.o
                gcc -o test main.o read.o write.o

main.o : io.h main.c 
                gcc -c main.c
read.o : io.h read.c
                gcc -c read.c
write.o: io.h write.c
                gcc -c write.c
```
* Makefile 사용하지 않을 경우2:
```Text
make.ps : make.dvi
                dvips make.dvi -o

make.dvi : make.tex
                latex make.tex 
```
* Makefile 사용할 경우2:
```Text
.SUFFIXES : .tex .dvi 

TEX = latex <- TEX 매크로를 재정의

PSFILE = make.ps 
DVIFILE = make.dvi

$(PSFILE) : $(DVIFILE)
                dvips $(DVIFILE) -o

make.ps : make.dvi 
make.dvi : make.tex
```

* Makefile에 macro를 같이 사용할 경우:
```Text
OBJECTS = main.o read.o write.o

test : $(OBJECTS)
                gcc -o test $(OBJECTS)

main.o : io.h main.c
                gcc -c main.c
read.o : io.h read.c
                gcc -c read.c
write.o: io.h write.c
                gcc -c write.c
```
> * 대입 시 = 연산자를 사용하고, macro를 사용하고자 할 때 $(..) 기호 안에 해당 macro를 넣어 사용해야 함
> * 매크로의 사용에서 ${..}, $(..), $..를 모두 사용할 수 있으나, 대부분의 책에서는 $(..) 사용 권장
    
* Makefile에 object file들을 삭제하는 label을 추가할 경우:
```Text
OBJECTS = main.o read.o write.o

test : $(OBJECTS)
                gcc -o test $(OBJECTS)

main.o : io.h main.c
                gcc -c main.c
read.o : io.h read.c
                gcc -c read.c
write.o: io.h write.c
                gcc -c write.c

clean :
                rm $(OBECTS)
```

## Predefined Macro
```Text
ASFLAGS = <- as 명령어의 옵션 세팅
AS = as
CFLAGS = <- gcc 의 옵션 세팅
CC = cc (= gcc)
CPPFLAGS = <- g++ 의 옵션
CXX = g++
LDLFAGS = <- ld 의 옵션 세팅
LD = ld
LFLAGS = <- lex 의 옵션 세팅
LEX = lex
YFLAGS = <- yacc 의 옵션 세팅
YACC = yacc
MAKE_COMMAND = make
```
>   * $make -p를 통해 predefined macro들을 확인해볼 것
* Makefile 내부에 predefined macro들을 같이 사용할 경우:
```Text
OBJECTS = main.o read.o write.o
SRCS = main.c read.c write.c <- 없어도 무방

CC = gcc <- gcc 로 세팅
CFLAGS = -g -c <- gcc 의 옵션에 -g 추가

TARGET = test <- 결과 파일을 test 라고 지정

$(TARGET) : $(OBJECTS)
$(CC) -o $(TARGET) $(OBJECTS)

clean : 
                rm -rf $(OBJECTS) $(TARGET) core 

main.o : io.h main.c <- (1)
read.o : io.h read.c
write.o: io.h write.c
```

## 확장자 규칙
* 확장자 규칙을 적용한 Makefile:
```Text
.SUFFIXES : .c .o 

OBJECTS = main.o read.o write.o
SRCS = main.c read.c write.c

CC = gcc 
CFLAGS = -g -c

TARGET = test

$(TARGET) : $(OBJECTS)
                $(CC) -o $(TARGET) $(OBJECTS)

clean : 
                rm -rf $(OBJECTS) $(TARGET) core 

main.o : io.h main.c 
read.o : io.h read.c
write.o: io.h write.c
```
>   * .SUFFIXES : .c .o 라고 했기 때문에 make 내부에서는 미리 정의된 .c(C 소스 파일)를 컴파일해서 .o(목적 파일)를 만들어 내는 루틴이 자동적으로 동작하게 되어 있음
>   * make 내부에서 기본적으로 서비스를 제공해 주는 확장자들의 리스트를 열거해 보면 아래와 같음
```Text
.out .a .ln .o .c .cc .C .p .f .F .r .y .l .s .S .mod .sym .def .h .info .dvi .tex .texinfo .texi .txinfo .w .ch .web .sh .elc .el
```
* .SUFFIXS의 동작 부분을 직접 구현한 Makefile:
```Text
.SUFFIXES : .c .o 

OBJECTS = main.o read.o write.o
SRCS = main.c read.c write.c

CC = gcc 
CFLAGS = -g -c 
INC = -I/home/raxis/include <- include 패스 추가

TARGET = test

$(TARGET) : $(OBJECTS)
                $(CC) -o $(TARGET) $(OBJECTS)

.c.o : <- 우리가 확장자 규칙을 구현
                $(CC) $(INC) $(CFLAGS) $<-

clean : 
                rm -rf $(OBJECTS) $(TARGET) core 

main.o : io.h main.c
read.o : io.h read.c
write.o : io.h write.c
```
>   *   Makefile 파일을 작성해 놓고, make 명령을 입력하게 되면 make는 Makefile의 내용을 살펴보다가 **첫 번째 목표 파일에 해당되는 것을 실행함**
>   *   따라서 위의 예제에서는 make test를 입력해도 make를 입력했을 때와 동일한 결과를 반환하며, 만약 clean에 해당하는 부분을 윗부분에 두게 되면 make는 항상 make clean을 수행함

## 내부 매크로
```Text
$* <- 확장자가 없는 현재의 목표 파일(Target)
$@ <- 현재의 목표 파일(Target)
$< <- 현재의 목표 파일(Target)보다 더 최근에 갱신된 파일 이름
$? <- 현재의 목표 파일(Target)보다 더 최근에 갱신된 파일이름
```
>   *  $<와 $?는 거의 같다고 봐도 무방함
```Text
main.o : main.c io.h
        gcc -c $*.c
```
>   *  $\*는 확장자가 없는 현재의 목표 파일이므로 $\*는 결국 main에 해당함

```Text
test : $(OBJS)
        gcc -o $@ $*.c
```
>   *  $\@는 현재의 목표 파일, 즉 test에 해당함

```Text
.c.o :
        gcc -c $< (또는 gcc -c $*.c)
```
>   *  **$<는 현재의 목표 파일보다 더 최근에 갱신된 파일 이름이라고 설명하였는데, .o 파일보다 더 최근에 갱신된 .c 파일은 자동적으로 컴파일이 수행됨(예를 들어, main.o를 만들고 난 다음에 main.c를 갱신하게 되면, main.c는 $<의 작용에 의해 새롭게 컴파일 수행됨)**

## Makefile Tip
#### 긴 명령어를 여러 라인으로 표시하기
```Text
# OBJS = shape.o rectangle.o circle.o line.o bezier.o
OBJS = shape.o \
rectangle.o \
circle.o \
line.o \
bezier.o 
```

#### 확장자 규칙 이용
* 아래에 열거된 파일들은 특별히 컴파일 또는 처리를 따로 정의하지 않아도 정해진 규칙에 의해 동작함
```Text
C 컴파일 (XX.c -> XX.o)
C++ 컴파일 (XX.cc 또는 XX.C -> XX.o)
Pascal 컴파일 (XX.p -> XX.o)
Fortran 컴파일 (XX.f 또는 XX.F -> XX.o)
Modula-2 컴파일 (XX.def -> XX.sym)
(XX.mod -> XX.o)
assembly 컴파일 (XX.s -> XX.o)
assembly 전처리 (XX.S -> XX.s)
single object file 의 링크 (XX.o -> XX)
Yacc 컴파일(?) (XX.y -> XX.o)
Lex 컴파일(?) (XX.l -> XX.o)
lint 라이브러리 생성 (XX.c -> XX.ln)
tex 파일 처리 (XX.tex -> XX.dvi)
texinfo 파일처리 (XX.texinfo 또는 XX.texi -> XX.dvi)
RCS 파일 처리 (RCS/XX,v -> XX)
SCCS 파일처리 (SCCS/XX.n -> XX)
```
* 파일들을 처리하기 위한 명령어가 정의된 macro 목록
```Text
AR = ar (Archive maintaining program)
AS = as (Assembler)
CC = cc (= gcc , C compiler)
CXX = g++ (C++ compiler)
CO = co (extracting file from RCS)
CPP = $(CC) -E (C preprocessor)
FC = f77 (Fortran compiler)
LEX = lex (LEX processor)
PC = pc (Pascal compiler)
YACC = yacc (YACC processor)
TEX = tex (TEX processor)
TEXI2DVI = texi2dvi (Texiinfo file processor)
WEAVE = weave (Web file processor)
RM = rm -f (remove file)
```
* Flag macro 목록
```
ARFLAGS = (ar achiver의 플래그) *
ASFLAGS = (as 어셈블러의 플래그)
CFLAGS = (C 컴파일러의 플래그) *
CXXFLAGS = (C++ 컴파일러의 플래그) *
COFLAGS = (co 유틸리티의 플래그)
CPPFLAGS = (C 전처리기의 플래그)
FFLAGS = (Fortran 컴파일러의 플래그)
LDFLAGS = (ld 링커의 플래그) *
LFLAGS = (lex 의 플래그) *
PFLAGS = (Pascal 컴파일러의 플래그)
YFLAGS = (yacc 의 플래그) *

* 표기된 것은 자주 사용하게 될 flag를 의미함
```
## 매크로 치환
* 필요에 의해 이미 정의된 매크로의 내용 일부만을 변경해야 할 경우 사용하며, $(MACRO_NAME:OLD=NEW)
```Text
# Micheal Jackson에서 Micheal Hookson으로 변경하고자 할 경우
MY_NAME = Michael Jackson
YOUR_NAME = $(NAME:Jack=Jook)
```
```Text
# 파일 확장자를 .o에서 .c로 변경하고자 할 경우
# 결과: SRCS = main.c read.c write.c
OBJS = main.o read.o write.o
SRCS = $(OBJS:.o=.c)
```
## 자동 의존 관계 생성
```Text
target : dependency
                command
                ...
```
* 일반적인 make 구조는 위와 같으며, 이때 command를 생략할 경우 단순히 target이 어느 파일에 의존하고 있는지를 나타냄
* 이러한 의존 정보를 작성할 경우, 파일이 많아질수록 작성이 힘들고 오류가 발생할 수 있음
* gccmakedep utility를 사용할 경우, 이러한 의존 정보를 자동으로 생성함
```Text
.SUFFIXES : .c .o 
CFLAGS = -O2 -g

OBJS = main.o read.o write.o 
SRCS = $(OBJS:.o=.c)

test : $(OBJS)
                $(CC) -o test $(OBJS)

dep :
                gccmakedep $(SRCS)
```
>   * make dep을 사용할 경우, gccmakedep에서 의존 정보를 자동으로 생성

## 다중 타켓
