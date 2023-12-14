# 공유 메모리 함수 (Shared Memory Functions)

## 공유 메모리(Shared Memory)란? 
공유 메모리(Shared memory)는 컴퓨터 환경에서 여러 프로그램이 동시에 접근할 수 있는 메모리이다. 과다한 복사를 피하거나 해당 프로그램 간 통신을 위해 고안되었다. 환경에 따라 프로그램은 하나의 프로세서에서나 여러 개의 프로세서에서 실행할 수 있다. (예를 들어 여러 개의 스레드 간에) 하나의 프로그램 안에서 통신을 위해 메모리를 사용하는 일은 일반적으로 공유 메모리로 부르지 않는다. [wikipedia](https://ko.wikipedia.org/wiki/%EA%B3%B5%EC%9C%A0_%EB%A9%94%EB%AA%A8%EB%A6%AC)

멀티 프로세스 환경에서, 프로세스 간 통신(Inter-Process Communication, IPC)을 위한 방법은 여러 가지가 존재하며 그중 하나가 바로 공유 메모리 방법이다.

공유 메모리 방법은 IPC 방법 중에서도 가장 빠른 수행속도를 보여주는데, 하나의 메모리를 여러 프로세스에서 공유해서 접근하기 때문에 데이터 복사 등과 같은 불필요한 오버헤드가 발생하지 않기 때문이다.

그러나 하나의 메모리를 여러 프로세스에서 접근하면 모든 병렬 환경에서 그렇듯 동시성 문제가 발생하기에 Semaphore, Mutex와 같은 도구를 활용하는데, 여기서는 공유 메모리를 활용하기 위한 함수에 대해서만 다루도록 하겠다.

-----------------------------------------
## - 공유 메모리와 관련된 함수들
```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);             : 공유메모리 get
int shmctl(int shmid, int cmd, struct shmid_ds *buf);       : 공유메모리 control
----------------------------------------------------------
#include <sys/types.h>
#include <sys/shm.h>

void *shmat(int shmid, const void *shmaddr, int shmflg);    : 공유메모리 attach
int shmdt(const void *shmaddr);                             : 공유메모리 detach
```

## - 공유 메모리와 관련된 구조체
```c
<sys/shm.h> 에 정의되어 있다.

struct shmid_ds {
    struct ipc_perm shm_perm;    /* 소유권과 권한 */
    size_t          shm_segsz;   /* 메모리 공간의 크기 (bytes) */
    time_t          shm_atime;   /* 마지막 attach 시간 */
    time_t          shm_dtime;   /* 마지막 detach 시간 */
    time_t          shm_ctime;   /* Creation time/ shmctl()에 의해
                                    마지막으로 변경된 시간 */
    pid_t           shm_cpid;    /* 생성 프로세스의 PID */
    pid_t           shm_lpid;    /* 마지막으로 shmat(2)/shmdt(2) 한
                                    프로세스의 PID */
    shmatt_t        shm_nattch;  /* 현재 attach 한 프로세스의 수 */
    ...
};

struct ipc_perm {
    key_t          __key;       /* Key supplied to shmget(2) */
    uid_t          uid;         /* Effective UID of owner */
    gid_t          gid;         /* Effective GID of owner */
    uid_t          cuid;        /* Effective UID of creator */
    gid_t          cgid;        /* Effective GID of creator */
    unsigned short mode;        /* Permissions + SHM_DEST and
                                    SHM_LOCKED flags */
    unsigned short __seq;       /* Sequence number */
};
```

-----------------------------------------
## 1. _**`shmget()`**_ - 공유 메모리를 생성하거나 접근하기 위한 함수

```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);
```

### 1.1. 매개 변수
1. key_t *key*
    
    공유 메모리에 접근 및 할당하기 위한 **고유한 key** 값이며 커널에서 관리된다.

2. size_t *size*
    
    새로 할당 할 공유 메모리의 최소 **크기(byte)**이며 이미 할당된 메모리에 접근할 때는 0으로 입력받는다.

3. int *shmflg*
    
    함수 동작 관련 플래그, **IPC_CREAT**, **IPC_EXCL** 로 구성된다.

    - IPC_CREAT :  새로운 공유 메모리를 할당한다. 만약 이 플래그가 사용되지 않는다면, `shmget()` 함수는 *key* 와 연관된 공유 메모리를 찾고 사용자가 메모리에 접근할 수 있는 권한이 있는지 확인한다.
    - IPC_EXCL  : IPC_CREAT 와 함께 사용되며, 만약 공유 메모리가 이미 할당되어있다면 실패를 반환한다.

### 1.2. 반환값
   - 실패 : 실패하는 경우, -1을 반환하며 errno 를 설정한다.
   - 성공 : *key* 에 해당하는 공유 메모리의 id 값(int)

### 1.3. Description
첫 번째 인자로 전달되는 고유한 *key* 값으로 공유메모리를 얻고 공유메모리의 id 를 리턴한다.

*key* 값이 지시하는 메모리 공간에 이미 공유 메모리 세그먼트가 생성되어 있다면 그 공간의 id 를 반환하고, 없다면 새로운 공간을 생성하여 id 를 반환한다.
 
새로운 공유 메모리의 할당은 *key* 값이 지시하는 메모리 공간이 이미 할당되어 있지 않고 *shmflg* 가 IPC_CREAT인 경우에 *size* 만큼 이루어진다.

*shmflg* 는 [IPC_CREAT, IPC_EXCL] 두 가지로 사용 가능하며, *key* 로 접근 가능한 공유 메모리가 이미 존재한다면, `shmget()`함수는 실패하며 -1 을 반환하고 errno 를 셋팅한다.

-----------------------------------------
## 2. _**`shmat()`**_ - 공유 메모리를 프로세스에서 사용할 수 있게 하는 함수
```c
#include <sys/types.h>
#include <sys/shm.h>

void *shmat(int shmid, const void *shmaddr, int shmflg);
```

### 2.1. 매개 변수
   1. int *shmid*
       
       `shmget()` 함수를 이용해서 얻은 id

   2. const void * *shmaddr*
       
       공유 메모리가 붙을 주소(address)를 명시

       만약 NULL 인 경우, 시스템은 적절한(사용하지 않은) 주소를 붙인다. _일반적으로 사용한다._

       ~~만약 NULL 이 아닌 경우, *shmflg* 가 SHM_RND 일 때 SHMLBA 의 가장 가까운 배수의 주소를 반환(?)~~

   3. int *shmflg*
       
       bit-mask argument로, 공유 메모리에 대한 프로세스의 접근 권한을 설정
       
       - SHM_RDONLY : 공유 메모리를 읽기 전용으로(READONLY) 붙인다. 이 플래그가 없으면 읽기/쓰기 모드로 붙는다. 쓰기 전용은 없다.

### 2.2. 반환값
   1. 실패 : (void *) -1 을 반환하며, errno 를 설정한다.
   2. 성공 : 붙은 공유 메모리의 주소를 (void *)형으로 반환한다. 사용할 때는 형변환을 통해 사용.

### 2.3. Description
`shmget()` 함수를 이용해서 얻은 *shmid* 에 해당하는 공유 메모리를 프로세스의 메모리 공간에 붙인다.(attach)

생성된 공유 메모리의 id를 이용해서 프로세스가 공유 메모리를 사용 가능하도록 붙인다.

프로세스가 종료될 때 자동으로 공유 메모리는 detach 된다.

-----------------------------------------
## 3. _**`shmctl()`**_ - 공유 메모리를 제어하기 위한 함수
```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

### 3.1. 매개 변수
   1. *shmid*

        `shmget()` 함수를 이용해서 얻은 id

   2. *cmd*

        공유 메모리를 제어하기 위한 command.
        - IPC_STAT : *shmid* 가 가리키는 [`shmid_ds`](#--공유-메모리와-관련된-구조체) 구조체의 정보를 buf 로 복사하여 가져온다.
        - IPC_SET  : 공유 메모리의 정보 중 아래 멤버들을 수정하고, shm_ctime 을 업데이트 한다. 즉, 공유 메모리에 대한 사용자 소유자 및 그룹, 권한 변경을 하기 위해 사용된다.
          - shm_perm.uid
          - shm_perm.gid
          - shm_perm.mode
        - IPC_RMID : 공유 메모리 공간을 삭제했다고 표시하기 위해 사용된다. 그러나 바로 삭제되는 것은 아니고, 공유 메모리에 attach 했던 모든 프로세스가 detach 하는 순간 (예를 들면, shm_nattch 가 0 인 경우) 에 삭제된다. 그렇기 때문에 공유 메모리 공간이 최종적으로 삭제되는지 확인할 필요가 있다.
        - SHM_LOCK : 공유 메모리 공간이 스왑되는 것을 방지할 수 있도록 lock 한다. Semaphore, Mutex 의 동작과는 다르다.
        - SHM_UNLOCK : 공유 메모리 공간이 스왑되는 것을 허용하도록 unlock 한다.

   3. *buf*

        [`shmid_ds`](#--공유-메모리와-관련된-구조체) 구조체 포인터. 


### 3.2. 반환값
   1. 실패 : -1 을 반환하며, errno 를 설정한다.
   2. 성공 : Linux-specific 한 동작들을 제외하고, 성공 시 0 을 반환한다.

### 3.3. Description
두 번째 인자로 전달되는 cmd(command) 를 통해, 첫 번째 인자인 shmid 가 가리키는 공유 메모리를 제어한다.

공유 메모리의 정보 구조체 `shmid_ds` 를 직접 제어함으로써, 해당 공유 메모리의 소유자, 그룹, 모드를 변경하거나, 공유 메모리를 삭제하거나, 공유 메모리의 스왑 잠금을 설정 및 해제할 수 있다.

-----------------------------------------
## 4. _**`shmdt()`**_ - 공유 메모리를 프로세스에서 분리하기 위한 함수
```c
#include <sys/types.h>
#include <sys/shm.h>

int shmdt(const void *shmaddr);
```
### 4.1. 매개 변수
   1. *shmaddr*

        `shmat()` 함수에 의해 반환되는 메모리의 주소

### 4.2. 반환값
   1. 실패 : -1 을 반환하며, errno 를 설정한다.
   2. 성공 : 0 을 반환한다.

### 4.3. Description
프로세스가 더 이상 공유 메모리를 사용할 필요가 없을 경우 프로세스와 공유 메모리를 분리하기 위해 사용한다.

프로세스의 주소 공간에서 *shmaddr*에 의해 지정된 주소에 위치한 공유 메모리 세그먼트를 분리한다.

분리할 세그먼트는 `shmat()` 함수에 의해 반환되는 값과 동일한 *shmaddr* 이여야 한다.

이 함수는 현재 호출한 프로세스와 공유 메모리를 분리시킬 뿐, 공유 메모리의 내용을 삭제하지는 않는다. 공유 메모리를 삭제하거나 제어하려면 `shmctl()` 와 같은 함수를 이용해야 한다.

## 참고 자료
- [랄라라 - [공유 메모리 (shared memory)]](https://unabated.tistory.com/entry/%EA%B3%B5%EC%9C%A0-%EB%A9%94%EB%AA%A8%EB%A6%AC-shared-memory)
- [Mint&Latte - [공유메모리 생성 및 관리 :: shmget(), shmat(), shmdt(), shmctl()]](https://mintnlatte.tistory.com/27)
- [REAKWON - [[리눅스] 공유메모리 (SHARED MEMORY) 개념과 예제(SHMGET, SHMAT, SHMDT, SHMCTL)]](https://reakwon.tistory.com/96)
- Linux manual page
  - [shmget](https://man7.org/linux/man-pages/man2/shmget.2.html)
  - [shmat](https://man7.org/linux/man-pages/man2/shmat.2.html)
  - [shmctl](https://man7.org/linux/man-pages/man2/shmctl.2.html)
  - [shmdt](https://man7.org/linux/man-pages/man2/shmdt.2.html)
