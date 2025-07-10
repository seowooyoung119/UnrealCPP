# TObjectPtr 가이드: 언리얼 엔진 5에서 안전한 객체 포인터 사용법

> 언리얼 엔진 5에서 `UObject*` 대신 `TObjectPtr<>`를 사용해야 하는 이유와 방법을 설명합니다.

## 📋 목차

- [개요](#개요)
- [기존 문제점](#기존-문제점)
- [TObjectPtr의 장점](#tobjectptr의-장점)
- [성능 고려사항](#성능-고려사항)
- [사용법 가이드](#사용법-가이드)
- [포인터 타입 비교](#포인터-타입-비교)
- [결론](#결론)

## 개요

언리얼 엔진 5는 `UObject*` 로우 포인터 대신 `TObjectPtr<>`를 사용하도록 강력히 권장합니다. 이는 단순한 문법적 변화가 아니라 **개발 경험, 안정성, 엔진 시스템 통합을 크게 향상**시키는 전략적 결정입니다.

### 핵심 특징
- **64비트 템플릿 기반** 포인터 시스템
- **에디터 빌드**에서 강력한 디버깅 기능 제공
- **배포용 빌드**에서는 로우 포인터로 자동 변환 (성능 영향 없음)
- UCLASS 및 USTRUCT 내 UObject 포인터 속성에 사용 권장

## 기존 문제점

### 🚨 로우 UObject* 포인터의 문제점

#### 1. 매달린 포인터 (Dangling Pointers)
```cpp
// 문제가 있는 코드
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
    UObject* MyObject; // UPROPERTY가 없으면 GC에 추적되지 않음
    
    void SomeFunction()
    {
        // MyObject가 이미 파괴되었을 수 있음
        MyObject->SomeMethod(); // 💥 충돌 가능성
    }
};
```

**문제점:**
- GC 시스템에 추적되지 않음
- 객체 파괴 시 자동으로 null 설정되지 않음
- 예측하기 어려운 충돌 발생

#### 2. 직렬화 문제
- UPROPERTY 없이는 자동 직렬화 안됨
- 저장/로드 시 참조 손실
- 수동 직렬화 코드 필요

#### 3. 에디터 불안정성
- 잦은 에디터 충돌
- 디버깅 어려움 (충돌 시점과 원인 분리)
- 메모리 관련 버그 추적 복잡

## TObjectPtr의 장점

### ✅ 주요 개선사항

#### 1. 향상된 접근 추적
```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
    UPROPERTY()
    TObjectPtr<UStaticMeshComponent> MyMesh; // 접근 추적 가능
    
    void SomeFunction()
    {
        if (MyMesh) // 안전한 null 검사
        {
            MyMesh->SetStaticMesh(SomeMesh); // 추적된 접근
        }
    }
};
```

#### 2. 자동 널링
- 객체 파괴 시 자동으로 null 설정
- 매달린 포인터 방지
- 신뢰할 수 있는 유효성 검사

#### 3. 지연 로딩 지원
- 필요할 때까지 객체 로딩 지연
- 메모리 사용량 최적화
- 에디터 시작 시간 단축

#### 4. 완벽한 엔진 통합
- 가비지 컬렉션 시스템과 통합
- 리플렉션 시스템 완전 지원
- 자동 직렬화 처리

## 성능 고려사항

### 🏗️ 에디터 빌드 vs 배포용 빌드

| 빌드 타입 | TObjectPtr 동작 | 성능 영향 |
|-----------|----------------|-----------|
| **에디터 빌드** | 전체 기능 (추적, 디버깅) | 미미한 오버헤드 |
| **배포용 빌드** | 로우 포인터로 변환 | 영향 없음 |

**핵심 포인트:**
- 개발 중에는 안전성과 디버깅 기능 제공
- 출시 제품에서는 성능 손실 없음
- 개발 효율성과 런타임 성능 모두 확보

## 사용법 가이드

### 🎯 TObjectPtr 사용 권장 사례

#### ✅ 사용해야 하는 경우

```cpp
// 1. UPROPERTY 멤버 변수
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    TObjectPtr<UStaticMeshComponent> MeshComponent;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TObjectPtr<UMaterial> MyMaterial;
};

// 2. 언리얼 컨테이너 클래스
UCLASS()
class AMyGameMode : public AGameModeBase
{
    GENERATED_BODY()
    
    UPROPERTY()
    TArray<TObjectPtr<APawn>> SpawnedPawns;
    
    UPROPERTY()
    TMap<FString, TObjectPtr<UTexture2D>> TextureMap;
};

// 3. 헤더 파일의 애셋 참조
UCLASS()
class AMyWeapon : public AActor
{
    GENERATED_BODY()
    
    UPROPERTY(EditDefaultsOnly)
    TObjectPtr<UStaticMesh> WeaponMesh;
    
    UPROPERTY(EditDefaultsOnly)
    TObjectPtr<USoundBase> FireSound;
};
```

#### ⚠️ 로우 포인터를 사용하는 경우

```cpp
// 1. 함수 지역 변수 (일시적 사용)
void AMyActor::ProcessActors()
{
    for (AActor* Actor : GetWorld()->GetAllActors()) // OK
    {
        if (Actor && Actor->IsValidLowLevel())
        {
            Actor->SomeFunction();
        }
    }
}

// 2. 함수 매개변수
void AMyActor::HandleActor(AActor* Actor) // OK
{
    if (Actor)
    {
        Actor->DoSomething();
    }
}

// 3. UFUNCTION 매개변수 (현재 제한사항)
UFUNCTION(BlueprintCallable)
void ProcessTarget(AActor* Target); // TObjectPtr 사용 불가
```

### 🚫 현재 제한사항

#### UFUNCTION 매개변수 문제
```cpp
// ❌ 컴파일 에러
UFUNCTION(BlueprintCallable)
void MyFunction(TObjectPtr<AActor> Actor);

// ✅ 현재 해결책
UFUNCTION(BlueprintCallable)
void MyFunction(AActor* Actor);
```

## 포인터 타입 비교

### 📊 언리얼 엔진 포인터 타입 매트릭스

| 포인터 타입 | 용도 | 소유권 | 자동 널링 | 주요 사용 사례 |
|-------------|------|--------|-----------|---------------|
| **TObjectPtr** | 주요 관리 포인터 | 강함 | ✅ | UPROPERTY 멤버, 컨테이너 |
| **UObject*** | 임시/지역 사용 | 없음 | ❌ | 함수 매개변수, 지역 변수 |
| **TWeakObjectPtr** | 소유하지 않는 참조 | 약함 | ✅ | 옵저버 패턴, 캐싱 |
| **TSoftObjectPtr** | 애셋 참조 | 없음 | N/A | 비동기 로딩, 대용량 애셋 |

### 📈 기능별 비교

| 기능 | 로우 포인터 | TObjectPtr |
|------|-------------|------------|
| **메모리 관리** | 수동 | 자동 (GC 통합) |
| **매달린 포인터** | 높은 위험 | 낮은 위험 |
| **에디터 안정성** | 충돌 위험 | 안정적 |
| **디버깅** | 어려움 | 쉬움 (접근 추적) |
| **직렬화** | 수동 처리 | 자동 처리 |
| **성능 (배포용)** | 기본 | 동일 (변환됨) |

## 관련 포인터 타입

### 🔧 다른 언리얼 포인터들

#### TWeakObjectPtr
```cpp
UCLASS()
class AObserver : public AActor
{
    GENERATED_BODY()
    
    // 약한 참조 - GC 방지하지 않음
    UPROPERTY()
    TWeakObjectPtr<AActor> ObservedActor;
    
    void CheckActor()
    {
        if (ObservedActor.IsValid())
        {
            ObservedActor->SomeFunction();
        }
    }
};
```

#### TSoftObjectPtr
```cpp
UCLASS()
class AAssetManager : public AActor
{
    GENERATED_BODY()
    
    // 애셋 경로 참조 - 지연 로딩
    UPROPERTY(EditAnywhere)
    TSoftObjectPtr<UTexture2D> LazyTexture;
    
    void LoadTexture()
    {
        if (LazyTexture.IsNull())
        {
            UTexture2D* LoadedTexture = LazyTexture.LoadSynchronous();
            // 사용...
        }
    }
};
```

#### ⚠️ 표준 C++ 스마트 포인터 주의사항
```cpp
// ❌ UObject에 사용 금지
std::shared_ptr<UMyObject> BadPointer; // GC, 리플렉션 기능 상실

// ✅ 비-UObject 타입에는 사용 가능
std::unique_ptr<MyCustomClass> GoodPointer; // OK
```

## 결론

### 🎯 모범 사례 요약

1. **UPROPERTY 멤버변수**에는 항상 `TObjectPtr` 사용
2. **언리얼 컨테이너**에서 UObject 저장 시 `TObjectPtr` 사용
3. **함수 지역변수**와 **매개변수**에는 로우 포인터 사용
4. **소유하지 않는 참조**에는 `TWeakObjectPtr` 고려
5. **애셋 지연 로딩**에는 `TSoftObjectPtr` 사용

### 🚀 개발 경험 향상

TObjectPtr 도입으로 얻을 수 있는 이점:
- ✅ 에디터 충돌 대폭 감소
- ✅ 디버깅 시간 단축
- ✅ 메모리 관련 버그 예방
- ✅ 코드 안정성 향상
- ✅ 개발 생산성 증대

**언리얼 엔진 5에서는 TObjectPtr가 새로운 표준입니다.** 
안전하고 효율적인 C++ 개발을 위해 적극적으로 활용하세요!

---

*이 가이드는 언리얼 엔진 5 기준으로 작성되었습니다. 엔진 버전에 따라 일부 내용이 다를 수 있습니다.*
