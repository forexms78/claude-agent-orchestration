---
name: db-guide
description: SQLAlchemy·Supabase DB 모델·쿼리 작성 규칙. DB 코드 작성 시 로드.
trigger: DB 모델·쿼리·마이그레이션 작업 시
---

# DB 작성 가이드 (SQLAlchemy + Supabase)

## 모델 설계 원칙

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import String, DateTime, func

class Base(DeclarativeBase):
    pass

class Resource(Base):
    __tablename__ = "resources"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(255), nullable=False)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
    updated_at: Mapped[datetime] = mapped_column(onupdate=func.now())
```

## 네이밍 규칙

| 항목 | 규칙 | 예시 |
|------|------|------|
| 테이블명 | 복수형 snake_case | `user_profiles` |
| 컬럼명 | snake_case | `created_at` |
| FK | `{테이블단수}_{id}` | `user_id` |
| 인덱스 | `ix_{테이블}_{컬럼}` | `ix_users_email` |

## 쿼리 작성 규칙
- ORM 사용 원칙. 날 SQL은 복잡한 집계·Full-text Search만 허용
- 관계 데이터는 `selectinload` / `joinedload` 명시적으로 지정
- 페이지네이션: `offset` + `limit` 필수. 대량 조회에 무제한 금지
- 트랜잭션: `async with session.begin():` 사용

## 성능 체크리스트
- [ ] 자주 조회하는 컬럼에 인덱스 추가
- [ ] N+1 쿼리 없음 (`selectinload` 확인)
- [ ] 불필요한 전체 행 로드 없음 (필요한 컬럼만 select)
- [ ] 대량 insert: `session.add_all()` 사용

## 마이그레이션 (Alembic)
- 스키마 변경 시 항상 Alembic 마이그레이션 생성
- 마이그레이션 파일 커밋 전 `downgrade` 함수 반드시 구현
- 프로덕션 `DROP COLUMN` 은 2단계로: nullable 전환 → 나중에 삭제

## Supabase 특이사항
- `.env` 에서 `SUPABASE_URL`, `SUPABASE_KEY` 로드
- RLS(Row Level Security) 활성화 된 테이블은 서비스 키 사용
- Realtime 구독이 필요한 테이블은 `supabase-py` 클라이언트 사용
