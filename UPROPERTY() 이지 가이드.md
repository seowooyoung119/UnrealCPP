# UPROPERTY() 이지 가이드

언리얼 엔진에서 UPROPERTY() 매크로의 핵심 기능, 런타임 시스템 통합, 그리고 성능 최적화 방법을 완벽 정리했습니다.

## 📋 목차

- [개요](#개요)
- [핵심 기능](#핵심-기능)
- [주요 지정자](#주요-지정자)
- [런타임 시스템 통합](#런타임-시스템-통합)
- [성능 고려사항](#성능-고려사항)
- [사용 가이드라인](#사용-가이드라인)
- [FAQ](#faq)

## 🎯 개요

**UPROPERTY()는 단순한 C++ 매크로가 아닙니다.**

- **역할**: C++ 코드와 언리얼 엔진의 런타임 시스템을 연결하는 핵심 다리
- **목적**: 표준 C++ 변수를 언리얼 엔진이 관리할 수 있는 속성으로 변환
- **중요성**: 에디터 통합, 블루프린트 연동, 메모리 관리 등의 기반 기술

### 왜 필요한가?

```cpp
// ❌ 일반 C++ 포인터 - 엔진이 관리하지 않음
UMyActor* MyActor;

// ✅ UPROPERTY() - 엔진이 완전 관리
UPROPERTY()
UMyActor* MyActor;
```

## 🚀 핵심 기능

### 1. 리플렉션 시스템
- **C++ 한계 극복**: 런타임 타입 정보 제공
- **에디터 통합**: 디테일 패널에 속성 노출
- **블루프린트 연동**: 비주얼 스크립팅에서 접근 가능

### 2. 가비지 컬렉션
- **자동 메모리 관리**: UObject 참조 추적
- **댕글링 포인터 방지**: 객체 파괴 시 자동 null 처리
- **메모리 누수 방지**: 도달 가능한 객체 보호

### 3. 네트워크 리플리케이션
- **멀티플레이어 동기화**: 서버-클라이언트 간 상태 동기화
- **조건부 리플리케이션**: 필요할 때만 네트워크 전송
- **성능 최적화**: 불필요한 트래픽 최소화

### 4. 직렬화
- **자동 저장/로드**: 게임 상태 영구 보존
- **버전 관리**: 스키마 변경 자동 처리
- **설정 파일 연동**: .ini 파일과 자동 동기화

## 🛠️ 주요 지정자

### 에디터 관련
| 지정자 | 기능 | 사용 예시 |
|--------|------|-----------|
| `EditAnywhere` | 에디터에서 편집 가능 | 레벨 디자인 시 조정 필요한 값 |
| `VisibleAnywhere` | 에디터에서 읽기 전용 | 디버깅용 상태 표시 |
| `Category="그룹명"` | 속성 그룹화 | 관련 속성들 정리 |

### 블루프린트 관련
| 지정자 | 기능 | 사용 예시 |
|--------|------|-----------|
| `BlueprintReadWrite` | 블루프린트에서 읽기/쓰기 | 디자이너가 조정할 값 |
| `BlueprintReadOnly` | 블루프린트에서 읽기만 | 상태 정보 표시 |

### 네트워크 관련
| 지정자 | 기능 | 사용 예시 |
|--------|------|-----------|
| `Replicated` | 네트워크 동기화 | 플레이어 체력, 점수 |
| `ReplicatedUsing=함수명` | 동기화 + 콜백 | 특수 처리 필요한 값 |

### 직렬화 관련
| 지정자 | 기능 | 사용 예시 |
|--------|------|-----------|
| `SaveGame` | 세이브 파일에 저장 | 플레이어 진행 상황 |
| `Transient` | 저장하지 않음 | 임시 계산 결과 |
| `Config` | 설정 파일과 연동 | 게임 옵션 |

## 🔧 런타임 시스템 통합

### 시스템별 상호작용

| 시스템 | UPROPERTY() 역할 | 핵심 이점 |
|--------|-----------------|-----------|
| **리플렉션** | 런타임 메타데이터 생성 | 에디터 통합, 블루프린트 연동 |
| **가비지 컬렉션** | UObject 참조 추적 | 메모리 안전성, 자동 정리 |
| **네트워크** | 동기화 대상 표시 | 멀티플레이어 상태 관리 |
| **직렬화** | 저장/로드 대상 지정 | 게임 상태 영구 보존 |

### 실제 동작 흐름

```cpp
// 1. 컴파일 타임: UHT가 메타데이터 생성
UPROPERTY(EditAnywhere, BlueprintReadWrite, Replicated)
int32 PlayerHealth;

// 2. 런타임: 엔진이 자동 관리
// - 에디터에서 수정 가능
// - 블루프린트에서 접근 가능  
// - 네트워크로 자동 동기화
// - 세이브 파일에 자동 저장
```

## ⚡ 성능 고려사항

### 성능 영향 요소

#### ✅ 실제로는 미미한 오버헤드
- **UPROPERTY() 자체**: 거의 무시할 수 있는 수준
- **빈 UObject**: 약 1.6KB RAM 사용
- **객체 생성**: 수천 개 생성해도 피코초 단위 비용

#### ⚠️ 주의해야 할 상황
- **과도한 UObject 생성**: 수만 개 이상의 객체
- **복잡한 참조 그래프**: 깊은 중첩 구조
- **불필요한 리플리케이션**: 모든 속성을 네트워크 동기화
- **비효율적인 직렬화**: 임시 데이터까지 저장

### 성능 최적화 전략

#### 1. 가비지 컬렉션 최적화
```cpp
// ❌ 불필요한 UPROPERTY
UPROPERTY()
int32 TemporaryCalculation;  // 기본 타입에는 불필요

// ✅ 필요한 경우만 사용
UPROPERTY()
UMyActor* ImportantReference;  // UObject 참조에는 필수
```

#### 2. 네트워크 최적화
```cpp
// ❌ 모든 속성 리플리케이션
UPROPERTY(Replicated)
FVector DetailedPosition;

// ✅ 조건부 리플리케이션
UPROPERTY(Replicated)
FVector Position;
// GetLifetimeReplicatedProps에서 DOREPLIFETIME_CONDITION 사용
```

#### 3. 직렬화 최적화
```cpp
// ❌ 임시 데이터도 저장
UPROPERTY(SaveGame)
float FrameTime;

// ✅ 필요한 데이터만 저장
UPROPERTY(SaveGame)
int32 PlayerLevel;

UPROPERTY(Transient)  // 저장하지 않음
float CalculatedValue;
```

## 📖 사용 가이드라인

### 필수 사용 상황

#### 1. UObject 참조
```cpp
// 반드시 UPROPERTY() 필요
UPROPERTY()
UMyActor* MyActor;

UPROPERTY()
TArray<UMyActor*> MyActors;
```

#### 2. 에디터 노출
```cpp
// 디자이너가 조정할 값
UPROPERTY(EditAnywhere, Category="Settings")
float MovementSpeed = 100.0f;
```

#### 3. 블루프린트 연동
```cpp
// 블루프린트에서 접근할 값
UPROPERTY(BlueprintReadWrite, Category="Player")
int32 PlayerScore;
```

### 사용하지 않는 상황

#### 1. 기본 타입 (선택사항)
```cpp
// UPROPERTY() 없어도 됨
float LocalCalculation;
int32 TemporaryValue;
FVector WorkingVector;
```

#### 2. 단일 함수 범위
```cpp
void MyFunction() {
    // 함수 내에서만 사용하는 임시 포인터
    UMyActor* TempActor = GetSomeActor();
    // UPROPERTY() 불필요
}
```

### 모범 사례

#### 1. 적절한 지정자 조합
```cpp
// 완벽한 조합 예시
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Gameplay", 
          meta=(ClampMin=0.0f, ClampMax=100.0f))
float HealthPercentage = 100.0f;
```

#### 2. 캡슐화 유지
```cpp
// 비공개 멤버 + 접근 제어
private:
    UPROPERTY(EditAnywhere, meta=(AllowPrivateAccess="true"))
    float PrivateValue;

public:
    UFUNCTION(BlueprintCallable)
    float GetPrivateValue() const { return PrivateValue; }
```

#### 3. 네트워크 최적화
```cpp
// 조건부 리플리케이션 설정
void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    
    // 특정 조건에서만 리플리케이션
    DOREPLIFETIME_CONDITION(AMyActor, ImportantData, COND_OwnerOnly);
}
```

## ❓ FAQ

### Q: UPROPERTY() 없이 UObject 포인터를 사용하면?
**A**: 매우 위험합니다. 가비지 컬렉션되어 댕글링 포인터가 되거나, 메모리 누수가 발생할 수 있습니다.

### Q: 모든 멤버 변수에 UPROPERTY()를 붙여야 하나요?
**A**: 아닙니다. 기본 타입(int, float 등)이나 엔진 관리가 필요 없는 변수는 선택사항입니다.

### Q: 성능이 정말 문제가 되나요?
**A**: 일반적으로는 미미합니다. 문제가 되는 경우는 수만 개 이상의 객체나 과도한 리플리케이션 등 극단적인 상황입니다.

### Q: UE5에서 가비지 컬렉션이 바뀌었다던데?
**A**: 맞습니다. 이제 자동 널링이 보장되지 않으므로, 동적 객체의 경우 명시적 관리가 더 중요해졌습니다.

### Q: 블루프린트 노출 vs C++ 캡슐화, 어떻게 균형을 맞춰야 하나요?
**A**: 프로젝트 규모와 팀 구성에 따라 다릅니다. 빠른 반복이 필요하면 직접 노출, 견고한 아키텍처가 필요하면 getter/setter 사용을 권장합니다.

---

## 🎯 핵심 요약

**UPROPERTY()는 "필요할 때만, 올바르게" 사용하세요.**

- ✅ **UObject 참조**: 반드시 필요
- ✅ **에디터/블루프린트 연동**: 디자이너 워크플로우 개선
- ✅ **네트워크/직렬화**: 멀티플레이어와 세이브 시스템
- ❌ **과도한 사용**: 불필요한 오버헤드 발생
- ❌ **무분별한 노출**: 캡슐화 원칙 훼손

**성능은 걱정하지 마세요. 올바른 사용이 더 중요합니다.**
