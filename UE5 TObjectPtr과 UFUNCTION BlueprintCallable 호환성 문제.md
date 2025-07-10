# UE5 TObjectPtr과 UFUNCTION BlueprintCallable 호환성 문제

## 문제 상황

```cpp
// ❌ 컴파일 에러
UFUNCTION(BlueprintCallable)
void MyFunction(TObjectPtr<AActor> Actor);

// ✅ 현재 해결책
UFUNCTION(BlueprintCallable)
void MyFunction(AActor* Actor);
```

## 문제 원인

Unreal Engine 5에서 도입된 `TObjectPtr<T>`는 새로운 스마트 포인터 타입이지만, Blueprint 시스템과의 호환성 문제가 있습니다.

- **Blueprint Reflection 시스템**: 아직 `TObjectPtr<T>` 타입을 직접 인식하지 못함
- **UFUNCTION 파라미터**: 전통적인 원시 포인터(`AActor*`) 방식을 기대함
- **타입 변환**: Blueprint에서 C++ 함수 호출 시 타입 매칭 실패

## 해결 방법

### 1. 원시 포인터 사용 (권장)
```cpp
UFUNCTION(BlueprintCallable)
void MyFunction(AActor* Actor)
{
    // 함수 내부에서 안전하게 사용
    if (IsValid(Actor))
    {
        // 로직 구현
    }
}
```

### 2. 내부에서 TObjectPtr 변환
```cpp
UFUNCTION(BlueprintCallable)
void MyFunction(AActor* Actor)
{
    TObjectPtr<AActor> SafeActor = Actor;
    
    // TObjectPtr의 장점을 활용
    if (SafeActor)
    {
        SafeActor->DoSomething();
    }
}
```

### 3. 멤버 변수는 TObjectPtr 사용 가능
```cpp
UCLASS()
class MYGAME_API AMyActor : public AActor
{
    GENERATED_BODY()

public:
    // ✅ 멤버 변수로는 사용 가능
    UPROPERTY(BlueprintReadWrite, Category = "References")
    TObjectPtr<AActor> MyActor;
    
    // ❌ 함수 파라미터로는 사용 불가
    UFUNCTION(BlueprintCallable)
    void SetActor(AActor* NewActor) { MyActor = NewActor; }
};
```

## 추가 정보

### TObjectPtr의 장점
- **메모리 안전성**: 가비지 컬렉션과 더 나은 통합
- **성능 향상**: 참조 추적 최적화
- **디버깅 지원**: 더 나은 디버깅 정보 제공

### 현재 제한사항
- UFUNCTION 파라미터에서 직접 사용 불가
- Blueprint 노드 생성 시 타입 인식 문제
- 일부 Reflection 기능에서 제한적 지원

## 향후 전망

Epic Games는 향후 업데이트를 통해 Blueprint에서 `TObjectPtr`를 완전히 지원할 예정입니다. 현재는 UFUNCTION 파라미터에서 전통적인 포인터를 사용하고, 필요에 따라 함수 내부에서 `TObjectPtr`로 변환하는 방식을 권장합니다.

## 관련 참고사항

- UE5.0+ 버전에서 발생
- Blueprint와 C++ 간의 상호 운용성 문제
- 멤버 변수 선언 시에는 `TObjectPtr` 사용 권장
- 함
