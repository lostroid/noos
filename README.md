### noos (no os baremetal base)  
C base super loop syatem  
Target board STM32H562ZI (NUCLEO-H563ZI)  
  
<img width="286" height="519" alt="image" src="https://github.com/user-attachments/assets/236be2e6-b912-41d6-aaed-9ef4e5f1d032" />  

목적   
Bare Metal의 방식과 Polling 기반으로 동작에 규격화 목적    
OS 기반 처럼 파일 실행 개념이 아니기에   
추상화개념 보단 절차적에 중점을 둔 프로젝트 입니다.  
이는 공장 생산라인의 컨베어밸트 속도 타이밍 중점으로 설계를 타나 냅니다  
  
주의사항   
- 모든 코드에는 Blocking Delay 사용을 금지한다. (쓸대없는 지연을 제거)  
- 모든 동작은 함수테이블 -> Swich Case 기반으로 수행을 분리 한다 (Scheduler 기능사용)  
- case 다음 step 을 넘기는 기준은 인터페이스 전송 후 넘긴다 (전송 시간동안 다른일 처리)  
- 모든 인터페이스는 Ring Buff 를 기반으로 통신 한다 (폴링 방식으로 순차적처리를 위해)  
- 기존 코드와 충돌을 방지 및 치환을 빠르게 하기 위해 네이밍 규칙이 존재합니다  
- Uart.c 경우 f_Uart1 같은 네이밍으로 시작 하며 이것은  
  추후 수정 전체 치환 작업,색인,키워드 검색시 유리한 정보를 제공합니다.  (뒤에서 자세히 설명)
- 모든 동작은 어디로 와서 어디로 가는가 데이터 흐름 기반 USB -> SPI 형태로  
   데이터 흐름 기반으로 네이밍을 작성 한다  
- 네이밍은 크게 대분류->소분류->기능->설명 헝태로 작성이 되어 있습니다.  
   Uart1SendAscii 말고 -> f_UART1_Send_Ascii(void) 형태로 _을 넣어 단어사이의 간격으로  
   빠른 분류를 인식 할수 있다.  
- 바로 수 us 응답이 필요 한 경우 인터럽트 루틴에서 처리 이외에 기본 인터럽트 카운터변수 만 사용  
- 인터럽트 처리 방식을 좀더 설명 하면 캐치테이블 개념이다 먼저 치고 들어갈 필요가 없음  
   순차적으로 처리 대다수 그렇게 빠른 응답성은 필요 없음.  
    
### Header File.  
헤더파일은 소스코드의 목차 입니다.  
소스코드에 함수는 헤더 파일에 다 작성 합니다.  
이는 헤더파일을 목차개념으로 활용하기 위한 내용입니다.  
(네이밍 규칙으로 이름이 충돌 할 일은 거이 없습니다.)  

외부연결 extern 도 헤더파일에 포함 하는 방식을 취합니다.   
절대 다른 소스코드에 extern 으로 끌어 쓰지 마십시오.   
extern 을 사용 하길 원하면 해당 대상 헤더를 포함 시키시오.  

### 활동링크 (해당 유튜브 영상은 전부 위와 같은 기반을 가지고 동작을 합니다)  
유튜브 : https://www.youtube.com/@lostroid  
reddit : https://www.reddit.com/r/LOSTROID  
NaverCafe : https://www.lostroid.com  

### BareMetal  
2가지 구조가 존재 합니다   
- HAL 기반으로 작성된 구조  
- 최저용량으로 동작시키기 위해 레지스터 직접제어 (M0+의 STM32C07 기반)  
  
### typedef.h  
목적: 데이터 크기 식별   
모든 변수에는 정해진 크기가 있습니다   
데이터를 처리할때 가장 중요한 부분 입니다    
기존 규칙 충돌을 방지하도록 다른 이름사용   
한 단어는 (float,double, char) 그대로 사용합니다    
```c
#ifndef H_TYPEDEF_H
#define H_TYPEDEF_H

#define D_ILP32_D       /// 32bit MCU
//#define D_LP64_D        /// 64bit MCU

#ifdef D_ILP32_D
    typedef unsigned char           tu8;
    typedef unsigned short          tu16;
    typedef unsigned long           tu32;
    typedef unsigned long long      tu64;

    typedef signed char             ts8;
    typedef short                   ts16;
    typedef long                    ts32;
    typedef long long               ts64;
#endif
#ifdef D_LP64_D
    typedef unsigned char           tu8;
    typedef unsigned short          tu16;
    typedef unsigned int            tu32;
    typedef unsigned long long      tu64;

    typedef signed char             ts8;
    typedef short                   ts16;
    typedef int                     ts32;
    typedef long long               ts64;
#endif

#define d_IDLE  0u
#define d_BUSY  1u
#define d_DONE  2u

#define d_OK    0u
#define d_PASS  0u
#define d_FAIL  1u

#define d_OFF   0u
#define d_ON    1u

#define d_NO    0u
#define d_YES   1u

#define d_NULL  0u
#define d_ERROR 0xFFFFFFFF

typedef enum {
    m_STATE_IDLE = 0,       //+ Idle: 대기 상태
    m_STATE_DONE,           //+ Done: 완료 상태
    m_STATE_BUSY            //+ Busy: 작업 중
} te_State;

typedef enum {
    m_RESULT_OK = 0,        //+ OK: 정상
    m_RESULT_PASS = 0,      //+ Pass: 통과
    m_RESULT_FAIL           //+ Fail: 실패
} te_Result;

typedef enum {
    m_ENABLE_OFF = 0,       //+ Off: 꺼짐
    m_ENABLE_ON             //+ On: 켜짐
} te_Enable;

typedef enum {
    m_YESNO_NO = 0,         //+ No: 아니오
    m_YESNO_YES             //+ Yes: 예
} te_YesNo;

typedef enum {
    m_RETURN_OK,            //+ Success: 성공
    m_RETURN_WAIT,          //+ Processing or Waiting: 처리 대기
    m_RETURN_ERR_CRC,       //+ CRC Error: CRC 오류
    m_RETURN_ERR_VALUE,     //+ Value Out of Range: 값 범위 오류
    m_RETURN_ERR_FRAME,     //+ Frame Error: 프레임 오류
    m_RETURN_ERR_TIMEOUT,   //+ Timeout: 시간 초과
    m_RETURN_ERR_SETTING,   //+ Configuration Error: 설정 오류
    m_RETURN_ERR_BUSY,      //+ Resource Busy: 자원 사용 중
    m_RETURN_ERR_OVERFLOW,  //+ Buffer Overflow: 버퍼 오버플로우
    m_RETURN_ERR_UNDERFLOW, //+ Buffer Underflow: 버퍼 언더플로우
    m_RETURN_ERR_NOT_INIT,  //+ Uninitialized: 초기화되지 않음
    m_RETURN_ERR_HW,        //+ Hardware Error: 하드웨어 오류
    m_RETURN_ERR_ARG,       //+ Invalid Argument: 잘못된 인자
    m_RETURN_ERR_LOCK,      //+ Failed to Lock Resource: 리소스 잠금 실패
    m_RETURN_ERR_UNKNOWN    //+ Unknown Error: 알 수 없는 오류
} te_Return;

#endif
```  
### 코딩 문법   
GIT의 모든 코드는 아래와 같은 규칙으로 작성이 됩니다.     
1. 코딩 스타일: Pascal_Snake_Case  
   보통 많이 쓰는 스타일은 classUartBuffConverLoop 형태를 많이 사용하는데  
   이는 하나 단어 로 인직 되어 제 시스템의 [대분류]_[소분류] 데이터 분류기법으로 작성 하는데 문제가 있다.  
   classUartBuffConverLoop == class_Uart_Buff_Conver_Loop /// 각 단어들이 _로 인해 피아 식별이 편하다.  
   소설 쓰듯 문장을 만드는게 아니고 하위 메뉴 개념으로 코드를 하기 때문에 많이 쓰는 방식은 적합하지 않다.  
3. 코딩 인코딩: UTF-8  
   주 언어가 영어이므로   
4. 기본 파일종류: 총 3가지 구성으로 됩니다.   
   xxx_type.h : 타입 enum, typedef, define 다른 타입과 종속 되지 않는 내용 선언  (호출 루프 방지목적)   
   xxx.h : 소스 코드의 보든 함수를 목차화 extern 은 다른 소스코드 금지 헤더에 선언, 순수 typdef 가 아닌 여러 type종류 혼합   
   xxx.c : 소스코드   
5. 네이밍  
   외부로 공개되는 함수나 변수는 다음 과 같은 구성으로 가집니다.   
   이는 데이터 종류, 데이터 출처, 데이터 처리/방향 을 식별 하기 위한 방법 입니다.   
   모든 코드느 데이터가 어디서 오고 어디로 가는 데이터 처리 흐름 기반이기 때문입니다.   
   [접두어]_[소스파일이름]_[기능]_[설명]  
   f_Uart1_Send_Ascii(tu8 *v_data);  /// USRT1으로 데이터를 Ascii 형태로 전송한다.  

  장점   
  (1). 치환이 쉽다 f_ 함수 s_구조체 와 같이 상위 구분과   
      소스파일 이름을 가지고 있기에 충돌 위험이 없다  
     프로그래밍을 하다면 새로운 구조 환경의 경우 변경 사항이 많거나  
      잘못되 의미로 수정해야 할 일 들이 많아 치환을 편하게 하기 위한 방법   
     드론만 만들고 개발 하면 어느정도 고착화 되지만    
       항공기 , 미사일, 자동차 형태로 새로운 분야시 반드시 의미와 기능 수정은 이루워진다.   
  (2). 데이터의 원천을 알 수 있다.   
     보통 인터페이스 기준으로 소스파일들이 작성되기 때문에  데이터 흐름을 알수 있다.  
     f_uart1_bypass_send( f_usb_recive() );   
     이는 USB 로 부터 온 데이터를 UART로 전송 한다는 주석이 필요 없이 이해 할수 있다.  
  (3) 데이터 유형을 빠르게 이해하기 쉽다   
     눈으로 읽기 많으로 Enum 인지 struct 인지 array 인지 판별이 가능 하므로 가독성이 있다.   
6. 모든 코드에는 접두어 사용 (아래 참조)   
```c
/******************************************************************************
* File:    ats_schedluer.c
* Author:  LOSTROID
* Created: 2024-11-10 생성날짜
*
* Description:
* This is the scheduler manager module.
*
* Revision History:
*   2025-05-21  MK  New project.
-------------------------------------------------------------------------------
01. a = array variable                      e.g. a_Data[]
02. b = bit fields                          e.g. b_Data
03. c = const                               e.g, c_Data
04. d = define                              e.g. d_Data
05. e = enum Type                           e.g. e_Data
06. f = Function                            e.g. f_Data
07. g = Source file static variable         e.g. gv_Data
08. i = Inline                              e.g. i_Data
09. l = Static variables inside functions   e.g. lv_Data
10. m = enum member                         e.g. m_Data
11. p = Pointer                             e.g. p_Data 
12. s = struct                              e.g. s_Data 
13. t = typedef                             e.g. t_Data 
14. u = Union                               e.g. u_Data 
15. v = variable                            e.g. v_Data
16. x = extern                              e.g. xv_data
******************************************************************************/
/*
e.g. 예시

x 접두사: 는 extern 변수를 의미.
xa_ : 전역 배열 변수.
xp_ : 전역 포인터 변수.
xpa_ : 전역 포인트 배열 변수.
xs_ : 전역 구조체를 변수.

g 접두사: 는 static 소스코드 글로벌 영역 을 의미.
ga_ : 소스코드 내 배열 변수.
gp_ : 소스코드 내 포인터 변수. (v는 생략가능)
gpa_ : 소스코드 내 포인터 배열 변수.

l 접두사: 함수내 static 변수를 의미.
lv_ : 함수내 비휘발성 변수.
la_ : 함수내 비휘발성 배열 변수.
lp_ : 함수내 비휘발성 포인트 변수.
lpa : 함수내 비휘발성 포인터 배열 변수.

f_ : 함수를 의미 (_첫글자 마다 대문자 표기).
s_ : 구조체 변수를 의미
ts_ : 구조체형 typedef (_첫글자 마다 대문자 표기)
te_ : enum typedef (_첫글자 마다 대문자 표기)
d_ : define 나머지 이름은 대문자로
c_ : 상수
b_ : 비트필드
*/
```
  
최대한 충돌을 피하기 위해 파일 이름의 함수 포함 시켜서 작성합니다.  
  
기본 파일 구성은 다음과 같습니다.   
test_type.h  (순수 Define 또는 타입만 선언)
```c

// 헤더 define 은 다음과 같인 처음과 끝에 헤더의 H로 선언 합니다.
#ifndef H_TEST_TYPE_H
#define H_TEST_TYPE_H

#define d_TEST_SETUP   0
typedef struct //이름은 생략
{
  uint32 v_data1;
  sint32 v_data2;
}ts_Test_Base;

typedef  enum
{
  m_TEST_BASE1,
  m_TEST_BASE2
}te_Test_Base;

typedef struct
{
  te_Test_Base e_test_Base;
  ts_Test_Base s_test_base;
}ts_Test_Ctrol;

#endif
```
test.h ( 다른소스 타입을 사용시에 선언)
```c
#ifndef H_TSET_H
#define H_TEST_H

#include "camera_type.h" /// 또는 Camera.h 함수 사용시 
#include "test_type.h"

typedef struct
{
  ts_Camera_base s_camera_base;
  ts_Test_bsase s_test_base;
}ts_Test_Moulde;

#endif
```

test_type.c
```c
#include "test.h"

uint32 xv_test_flag;          /// extren 변수
uint32 xa_test_array[29];     /// extren 배열 변수

static uint32 gv_test_timer;  /// 소스파일 범위 변수
static uint32 gv_test_state;  /// 소스파일 범위 변수

static sint32* gp_test_point;         /// 소스파일 범위 포인터 변수
static sint32* gpa_test_array[20];    /// 소스파일 범위 포인터 배열 변수

void f_test_int(void)
{
  static uint32 lv_time;  /// 함수 내부는 파일명 접두사를 넣을 필요 없음.
  uint32 v_buff;
{
void f_test_module(void)
{
}

```

코딩 문법룰 예시 (정확한 사용은 코드를 보세요)
1. #ifdef 같은 헤더 중복은 H_XXXXX_H  앞 뒤로 H 포함 하여 대문자로 작성합니다.  
2. #define 접두어 d소문자 d_XXXXX 나머진 대문자로 사용  
3. const 상수 접두어 c소문자 c_XXXXX  
4. v_value 변수는 상황에 맞게 접두어 사용 소문자로만 사용  
5. typdef struct 및 함수는 f_Aaaa_Aaaa or f_AaaaAaaㅁ 형태로 Pascal_Snake_Case 사용
6. 

xbot_camera.c 인 경우 소스코드 이름을 내부 접두어에 포함 작성합니다.  
이는 서로다른 소스파일의 중복 함수를 피하기 위한 조치입니다.  
f_Xbot_Camera_xxx;   
v_xbot_camera_xxx;
static 변수 파일 이름을 포함 하지 않습니다.
이는 함수의 소스파일 이름을 바로 파악 및 이름 충돌을 피함.  


--- 이하 작성중---
### 동작 단위 정의   
#Module    
- Camera   
- LCD  
  
#JOB  
  - 동작에 대한 테이블 변수로 존재 (함수는 없음)  
  - JOB_CAMERA_COLOR : 컬러로 동작 테이블  
  - JOB_CAMERA_BlackWhite : 흑백모드 테이블  
    ```c  
    void (*gap_JOB_Camera_Color[]) (void) = {  
        f_task_camera_enable,  
        f_task_camera_disable,  
        f_task_camera_init,  
        f_task_camera_setup  
    }
    ```    
#TASK  
  - 한가지 동작을 정이  
  - 창문열기 동작.  
    case1. 모터를 정방향으로 가속.  
    case2. 모터 현재 위치 파악.  
    case3. 목표 확인.  
  - 창문닫기 동작.
  
  각 부분을 세분화 하여 Switch 문의 Case로 구분  
  동작을 깔끔하게 보기 위해 함수를 호출 방식으로 권장  
  Case 값을 step으로 지칭  
  ```c  
  Switch(step)  
  {  
      case 0:  
          f_camera_work_en_l();   
          break;   
      case 1:  
          f_camera_work_en_h();  
          break;  
      default:   
          break;   
  }  
  ```
#WORK  
  - 단순 동작 수행 합니다.  
  - f_WindowOpen_Motor_Work_ON() 창문열기 모터 ON  
  - f_WindowsOpen_Motor_Work_Off() 창문열기 모터 OFF  
  - f_WindowsOpen_Moter_Work_Postion_Check() 창문위치 확인


주석
```c
///============================================================ 구분줄
/*  함수 기능 설명
---------------------------------------------------------------
주의 사항
+ 인자 설명
+ retrun 설명
---------------------------------------------------------------*/
```
| 기호    | 의미 / 활용          | 예시                        |
| ----- | ---------------- | ------------------------- |
| `###` | 중요 / 주의          | `// ### NOTE: 초기화 순서 중요`  |
| `***` | 강력한 경고 / 반드시 확인  | `/* *** FIXME: 버그 있음 */`  |
| `+`   | 추가/개선 / 기능 추가    | `// + TODO: 로그 기능 개선`     |
| `-`   | 제거/삭제 / 주석 처리 대상 | `// - OBSOLETE: 사용 안 함`   |
| `>`   | 설명 / 상세 정보       | `// > 참고: 동기화 필요`         |
| `<`   | 이전 내용 참조         | `// < 참고: 이전 버전`          |
| `!`   | 주의 / 위험          | `// ! WARNING: 메모리 누수 가능` |
| `?`   | 의문 / 확인 필요       | `// ? TODO: 테스트 필요`       |

| 키워드         | 의미       | 예시                     |
| ----------- | -------- | ---------------------- |
| `TODO:`     | 구현 예정    | `// TODO: 예외 처리 추가`    |
| `FIXME:`    | 버그/수정 필요 | `/* FIXME: 계산식 오류 */`  |
| `NOTE:`     | 참고 사항    | `// NOTE: 성능 중요`       |
| `HACK:`     | 임시방편     | `/* HACK: 임시 처리 */`    |
| `BUG:`      | 알려진 버그   | `// BUG: 범위 체크 필요`     |
| `OPTIMIZE:` | 성능 개선 필요 | `// OPTIMIZE: 루프 최적화`  |
| `REVIEW:`   | 코드 리뷰 필요 | `// REVIEW: API 구조 확인` |

  
    
