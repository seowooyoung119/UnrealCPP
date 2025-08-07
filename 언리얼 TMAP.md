# 📌 TMap 완전 분석 (Unreal Engine 5 기준)

언리얼 엔진에서 제공하는 `TMap`은 키와 값으로 데이터를 저장하는 **해시 기반 컨테이너**입니다.  
C++ 표준 라이브러리의 `std::map`, `std::unordered_map`과 비슷하지만, **언리얼 엔진의 메모리 관리 및 직렬화 시스템과 통합**되도록 설계되었습니다.

---

## 🔷 목차
1. [TMap이란?](#1-tmap이란)
2. [기본 사용법](#2-기본-사용법)
3. [주요 메서드 정리](#3-주요-메서드-정리)
4. [반복자 (Iterator)](#4-반복자-iterator)
5. [TMap vs TArray 비교](#5-tmap-vs-tarray)
6. [직렬화 및 저장](#6-직렬화-및-저장)
7. [내부 구현 개요](#7-내부-구조-개요)
8. [사용 시 주의할 점](#8-사용-시-주의할-점)
9. [결론](#9-결론)
10. [참고 및 연습 예제](#📚-참고-자료--✅-추천-연습)

---

## 1. TMap이란?

- 언리얼 엔진의 템플릿 기반 **해시맵** 구조입니다.
- `Key -> Value` 형식으로 데이터를 저장합니다.
- **Key는 고유해야 하며 중복을 허용하지 않습니다.**
- 내부적으로 `TSet`을 기반으로 구성됩니다.

```cpp
TMap<FString, int32> PlayerScores;
2. 기본 사용법
▶ 선언 및 초기화
cpp
복사
편집
TMap<FString, int32> ScoreMap;
ScoreMap.Add("PlayerA", 100);
ScoreMap.Add("PlayerB", 150);
▶ 값 가져오기
cpp
복사
편집
int32* Score = ScoreMap.Find("PlayerA");
if (Score)
{
    UE_LOG(LogTemp, Log, TEXT("Score: %d"), *Score);
}
Find()는 포인터를 반환함.

키가 없으면 nullptr 반환.

▶ 값 덮어쓰기
cpp
복사
편집
ScoreMap.Add("PlayerA", 200); // 기존 값을 덮어씀
▶ Contains (키 존재 여부 확인)
cpp
복사
편집
if (ScoreMap.Contains("PlayerB"))
{
    UE_LOG(LogTemp, Log, TEXT("PlayerB exists"));
}
▶ Remove (항목 삭제)
cpp
복사
편집
ScoreMap.Remove("PlayerA");
3. 주요 메서드 정리
메서드	설명
Add(Key, Value)	키-값 쌍 추가 (덮어쓰기 포함)
Find(Key)	값 포인터 반환 (없으면 nullptr)
FindOrAdd(Key)	없으면 추가 후 포인터 반환
Contains(Key)	키 존재 여부 확인
Remove(Key)	특정 키 제거
Empty()	모든 데이터 초기화
Num()	저장된 쌍의 개수 반환
IsEmpty()	비어 있는지 확인

4. 반복자 (Iterator)
▶ Key-Value 반복
cpp
복사
편집
for (const TPair<FString, int32>& Pair : ScoreMap)
{
    UE_LOG(LogTemp, Log, TEXT("Player: %s, Score: %d"), *Pair.Key, Pair.Value);
}
TPair<KeyType, ValueType> 구조로 반복됨

.Key, .Value로 접근

5. TMap vs TArray
항목	TMap	TArray
구조	해시맵	배열
접근 방식	Key로 접근	Index로 접근
성능	빠른 검색	빠른 순차 처리
중복	Key 중복 불가	허용됨

6. 직렬화 및 저장
언리얼에서는 TMap도 UPROPERTY()로 선언하여 세이브/로드 및 블루프린트에서 사용 가능합니다.

cpp
복사
편집
UPROPERTY(BlueprintReadWrite)
TMap<FName, int32> ItemCounts;
직렬화도 자동으로 처리됩니다.

7. 내부 구조 개요
TMap은 내부적으로 TSet을 기반으로 구성되며,
TSet에는 TPair<KeyType, ValueType>이 저장됩니다.

cpp
복사
편집
TMapBase<KeyType, ValueType, SetAllocator, KeyFuncs>
TMap<KeyType, ValueType>는 위 클래스의 특수화 버전입니다.

다양한 정책을 플러그인 할 수 있도록 구성된 정책 기반 설계입니다.

8. 사용 시 주의할 점
Add()는 중복 키일 경우 값을 덮어씁니다.

Find()는 포인터를 반환하므로 널 체크는 필수입니다.

블루프린트에서 사용할 경우:

FName, FString, int32 등 블루프린트 지원 타입만 키로 사용 가능

9. 결론
TMap은 키 기반 데이터 관리에 최적화된 언리얼 전용 컨테이너입니다.

C++의 unordered_map과 유사하되, GC, 직렬화, UPROPERTY 연동 등에 적합합니다.

반복자와 포인터 반환 등 C++ 문법에 익숙하지 않다면 주의가 필요합니다.

📚 참고 자료 & ✅ 추천 연습
Unreal Engine Docs - TMap

TMap.h, TMapBase.h 헤더 소스 분석

관련 구조체: TPair, TSet, KeyFuncs

연습 예제
cpp
복사
편집
TMap<FString, float> ShoppingCart;

ShoppingCart.Add("Apple", 1.2f);
ShoppingCart.Add("Banana", 0.8f);
ShoppingCart.Add("Milk", 2.5f);

// 총합 계산
float Total = 0.0f;
for (const TPair<FString, float>& Item : ShoppingCart)
{
    Total += Item.Value;
}
UE_LOG(LogTemp, Log, TEXT("Total Price: %.2f"), Total);
🧠 요약
TMap<Key, Value>는 언리얼에서 사용되는 키-값 컨테이너

Find(), Add(), Remove() 등의 기본 메서드로 간단하게 사용 가능

반복자(TPair)를 이용해 키와 값 순회 가능

게임 저장/불러오기 등 다양한 곳에서 활용

yaml
복사
편집
