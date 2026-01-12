# 🔐 Security Safe Service  
### 다이얼 금고 기반 네트워크 보안 관리 서비스

본 프로젝트는 **STM32 기반 아날로그 다이얼 금고 시스템**을 확장하여,  
**서버·데이터베이스·네트워크 통신·관리자 애플리케이션**을 결합한  
다중 금고 보안 관리 서비스 시스템입니다.

<img width="1764" height="1055" alt="image" src="https://github.com/user-attachments/assets/5157e410-179c-47ca-90d3-6843bd77dd75" />

기존 단일 금고 구조의 한계를 개선하고,  
Wi-Fi / Bluetooth 기반 인증과 서버 중심 보안 로직을 도입하여  
여러 개의 금고를 중앙에서 안전하게 관리할 수 있는 구조를 구현했습니다.

---

## 1. 프로젝트 개요

본 프로젝트는 이전 프로젝트인 **다이얼 금고 시스템**을 기반으로 하여  
금고 내부에서 처리되던 인증 로직을 **서버로 분산**하고,  
데이터베이스를 통해 **여러 금고를 통합 관리**할 수 있도록 확장한 보안 서비스입니다.

- 기존: 금고 단독 비밀번호 저장 및 비교
- 확장: 서버 기반 비밀번호 인증 + DB 로그 관리 + 관리자 시스템

---

## 2. 프로젝트 목표

- 기존 다이얼 금고 시스템의 기능적 한계 개선
- 서버 기반 비밀번호 인증 구조 설계
- 다중 금고 관리 가능한 데이터베이스 구축
- 보안 이벤트 감지 및 관리자 알림 시스템 구현
- 시스템 락(System Lock) 기반 보안 강화

---

## 3. Tech Stack
![C](https://img.shields.io/badge/C-00599C?style=flat&logo=c&logoColor=white)
![STM32CubeIDE](https://img.shields.io/badge/STM32CubeIDE-03234B?style=flat&logo=stmicroelectronics&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino%20Uno-00979D?style=flat&logo=arduino&logoColor=white)  
![Firmware](https://img.shields.io/badge/Firmware-0A9396?style=flat)
![TCP/IP Socket](https://img.shields.io/badge/TCP%2FIP%20Socket-005F73?style=flat)
![IoT](https://img.shields.io/badge/IoT-94D2BD?style=flat)

---

## 4. 주요 기능

### 1) 서버 기반 비밀번호 인증
- 금고에서 입력한 비밀번호를 서버로 전송
- 서버에서 DB에 저장된 비밀번호와 비교
- 인증 결과(success / failure)를 금고로 반환

### 2) 다중 금고 관리 (SafeDB)
- 금고 ID 기반 비밀번호 관리
- 로그인 실패 횟수 및 보안 상태 저장
- 모든 인증 시도 및 보안 이벤트 로그 저장

### 3) System Lock 보안 정책
- 동일 ID에서 비밀번호 5회 이상 연속 실패 시:
  - 자동 System Lock 활성화
  - 추가 인증 차단
- 관리자 애플리케이션을 통해서만 Lock 해제 가능

### 4) 관리자 시스템 및 알림
- 관리자 애플리케이션을 통한:
  - 시스템 Lock 해제
  - 비밀번호 변경
  - 로그 조회
- 보안 이벤트 발생 시 관리자 알림 제공

---

## 5. 시스템 구성

### 4-1. 금고 시스템 (STM32)
- 가변저항 기반 비밀번호 입력
- 버튼 입력 처리
- 서보모터 기반 잠금 / 해제 제어
- Wi-Fi 모듈(ESP-01)을 통한 서버 통신
- LCD 및 부저를 통한 상태 표시

### 4-2. 서버
- 금고로부터 인증 요청 수신
- DB 연동을 통한 비밀번호 검증
- 인증 결과 반환
- 로그 저장 및 보안 정책 처리

### 4-3. 데이터베이스 (SafeDB)
- 금고 ID / 비밀번호 정보 관리
- 로그인 실패 횟수 관리
- System Lock 상태 저장
- 보안 이벤트 로그 테이블 관리

### 4-4. 관리자 애플리케이션
- 금고 상태 모니터링
- System Lock 해제
- 비밀번호 변경
- 보안 이벤트 확인

---

## 6. 인증 및 통신 흐름

1. 사용자가 다이얼 금고에서 비밀번호 입력
2. 금고 → 서버로 인증 요청 전송  
   `[SSS_SQL] LOGIN@ID@PSWD`
3. 서버에서 DB 비밀번호 조회 및 비교
4. 인증 결과(success / failure) 생성
5. 서버 → 금고로 결과 전달
6. 결과에 따라:
   - 성공: 금고 잠금 해제
   - 실패: 실패 횟수 증가 / 로그 저장
7. 실패 횟수 초과 시 System Lock 발생

---

## 7. DB 관리 모듈 (DBmgr)

DB 관리를 위해 전용 모듈(`dbmgr.h`, `dbmgr.c`)을 구현했습니다.

### 주요 기능
- DB 초기화
- 신규 금고 등록
- 비밀번호 조회 및 비교
- 비밀번호 변경
- 로그인 결과 반환
- System Lock / Unlock 설정
- 인증 및 보안 이벤트 로그 저장
<img width="618" height="193" alt="image" src="https://github.com/user-attachments/assets/ebaafbc9-05df-4771-bb71-46b50cdbd980" />

---

## 8. 보안 이벤트 및 로그 처리

다음과 같은 보안 이벤트를 감지하고 기록합니다.

- 비밀번호 불일치 이벤트
- 로그인 실패 5회 초과
- System Lock 발생
- 볼 스위치(물리적 충격) 이벤트

이벤트 발생 시:
- 서버 로그 저장
- 금고 LCD 출력
- 부저 경고음 출력
- 관리자 시스템 알림

---

## 9. 담당 역할

- 기존 다이얼 금고 프로젝트 기반 시스템 확장 설계
- 금고–서버 간 인증 흐름 설계
- System Lock 보안 정책 로직 구현
- DB 기반 비밀번호 관리 구조 설계
- 보안 이벤트 처리 및 로그 흐름 설계

---

## 10. 프로젝트 결과 및 배운 점

- 단일 임베디드 시스템을 서버·DB 기반 구조로 확장하는 경험
- 보안 정책(System Lock) 설계 및 구현 경험
- 네트워크 기반 인증 흐름 이해
- 임베디드–서버–DB 연동 구조 설계 역량 향상
- 보안 이벤트 처리 및 관리자 시스템 설계 경험

---
