# 네이밍 전략

- create
- viewList
- viewDetail
- update
- delete
# Create


# Read

## Read List

- [ ] 전달 데이터를 쿼리스트링과 바디를 적절히 사용했는가
- [ ] `Pageable` 사용 여부 고려

## Read Detail

- [ ] `PathVariable` or `RequestParams`
	- `PathVariable`은 url 주소를 깔끔하게 표현 가능
	- `RequestParams`은 다수의 상태를 전달하고 여러 상태들을 명확히 표현 가능
		- id, viewType…
