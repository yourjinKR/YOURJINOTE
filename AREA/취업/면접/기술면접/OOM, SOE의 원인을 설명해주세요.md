# OOM, SOE의 원인을 설명해주세요

StackOverflowError는 스택 공간이 꽉 찼을 때 발생함, 무한 재귀 호출이 주 원인입니다.

OutOfMemoryError는 힙 혹은 시스템 전체 메모리가 부족해서 GC가 더 이상 필요한 메모리를 확보 못할때  발생합니다.

StackOverflowError 방지 : 자기 자신을 참조하는 구조를 피하기, 
OutOfMemoryError 방지 : `try-with-resource`와 같은 구문을 사용하여 리소스 관리에 신경 쓰기



