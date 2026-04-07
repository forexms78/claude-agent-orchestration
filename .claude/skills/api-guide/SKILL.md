---
name: api-guide
description: FastAPI 엔드포인트 설계·작성 규칙. API 코드 작성 시 로드.
trigger: FastAPI 라우터·엔드포인트 작성 시
---

# API 작성 가이드 (FastAPI)

## 엔드포인트 설계 원칙

| 항목 | 규칙 |
|------|------|
| HTTP 메서드 | GET(조회) / POST(생성) / PUT(전체 수정) / PATCH(부분 수정) / DELETE(삭제) |
| 상태 코드 | 200(OK) / 201(Created) / 204(No Content) / 400(Bad Request) / 404(Not Found) / 422(Validation) / 500(Server Error) |
| URL 구조 | `/resource/{id}` (복수형, 소문자, 하이픈) |
| 버전 | prefix: `/api/v1/` |

## 라우터 구조

```python
# routers/resource.py
router = APIRouter(prefix="/resource", tags=["resource"])

@router.get("/", response_model=list[ResourceResponse])
async def list_resources(...):
    ...

@router.get("/{id}", response_model=ResourceResponse)
async def get_resource(id: int, ...):
    ...
```

## Pydantic 모델 규칙
- Request: `ResourceCreate`, `ResourceUpdate` (PATCH용은 Optional 필드)
- Response: `ResourceResponse` (항상 id 포함)
- 민감 데이터(password 등) Response 모델에서 exclude
- `model_config = ConfigDict(from_attributes=True)` 사용

## 성능 체크리스트
- [ ] N+1 쿼리 없음 (관계 데이터는 `selectinload` / `joinedload`)
- [ ] 불필요한 `await` 없음
- [ ] 대량 조회에 페이지네이션 (`limit` + `offset`)
- [ ] 응답 캐싱 필요 여부 검토 (5~30분)

## 보안 체크리스트
- [ ] 인증/인가 의존성 주입 (`Depends(get_current_user)`)
- [ ] 사용자 입력 직접 SQL 조합 금지 (SQLAlchemy ORM 사용)
- [ ] 에러 응답에 내부 스택 트레이스 노출 금지
- [ ] 민감 데이터 로그 출력 금지

## 에러 처리

```python
from fastapi import HTTPException

raise HTTPException(status_code=404, detail="Resource not found")
```

전역 예외 핸들러는 `main.py`에 등록. 라우터 내 try/except 남발 금지.
