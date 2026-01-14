# CrewAI Flow 기반 SEO 콘텐츠 자동화 프로젝트

## 프로젝트 한 줄 요약

**Pydantic 기반 구조화 출력 + CrewAI Flow를 이용해 "SEO 친화 블로그 글 생성 → 검증 → 후처리"를 자동화한 LLM 파이프라인 프로젝트**

---

## 왜 이 프로젝트를 만들었는가 (문제 정의)

LLM을 이용해 글을 생성하면:

* 결과 포맷이 매번 흔들리고
* 후속 처리(검증, 재사용, 파이프라인화)가 어렵고
* 실제 서비스에 쓰기엔 신뢰성이 떨어진다

👉 **"생성 → 구조화 → 검증 → 다음 단계로 전달"** 되는 **프로덕션 지향 LLM 파이프라인**를 직접 구현해보기 위해 시작했다.

---

## 핵심 구현 포인트 (Kick 포인트)

### 1️⃣ LLM 출력물을 Pydantic 모델로 강제 구조화

```python
class BlogPost(BaseModel):
    title: str
    subtitle: str
    sections: List[str]
```

* `response_format=BlogPost` 사용
* LLM 응답을 **JSON 문자열이 아닌 Python 객체로 직접 수령**
* 파싱 실패 / 필드 누락을 **런타임에서 즉시 차단**

📌 *"LLM 결과는 텍스트"라는 전제를 깨고, 데이터 파이프라인의 한 단계로 취급"*

---

### 2️⃣ CrewAI Flow에서 상태(State) 기반 파이프라인 구성

```python
class SeoFlow(Flow):
    topic: str
    blog_post: BlogPost
    score: Score
```

* Flow state에 **Pydantic 객체 직접 저장**
* 단계별 결과를 JSON으로 재파싱하지 않고 **객체로 전달**
* Research → Writing → Evaluation 단계를 명확히 분리

📌 *Flow는 단순 실행 순서가 아니라 "데이터 흐름"으로 설계*

---

### 3️⃣ Task 입력 제약을 이해하고 명시적으로 변환

CrewAI Task는 입력으로 다음 타입만 허용:

```
str | int | float | bool | dict | list
```

그래서:

```python
# ❌ Pydantic 객체 직접 전달
"blog_post": BlogPost(...)

# ✅ 명시적으로 변환
"blog_post": blog_post.model_dump()
```

📌 *프레임워크의 추상화를 믿지 않고, 내부 제약을 코드로 통제*

---

### 4️⃣ `model_validate_json()` 오용 문제 해결

* `response_format=PydanticModel` 사용 시
* LLM 결과는 **이미 검증된 객체**

```python
# ❌ 잘못된 사용
BlogPost.model_validate_json(result)

# ✅ 정답
self.state.blog_post = result
```

📌 *LLM + Pydantic 연동 시 데이터 타입 흐름을 정확히 이해*

---

## 전체 아키텍처 흐름

```
[ Topic Input ]
      ↓
[ Research Agent ]
      ↓ (구조화 출력)
[ BlogPost (Pydantic) ]
      ↓
[ SEO Evaluation Agent ]
      ↓
[ Score (Pydantic) ]
```

---

## 기술 스택

* Python 3.13
* CrewAI (Flow 기반)
* Pydantic v2
* OpenAI LLM (`response_format` 활용)

---

## 이 프로젝트에서 증명하고 싶은 역량

* LLM 결과를 **"문장"이 아닌 "데이터"로 다루는 사고방식**
* 프레임워크 에러를 로그 수준이 아니라 **타입 흐름 관점에서 분석**
* PoC가 아닌 **확장 가능한 구조**로 설계
* 신입이지만 "왜 이렇게 써야 하는지"를 설명할 수 있는 구현

---

## 한 줄 정리

> **LLM을 잘 쓰는 사람이 아니라, LLM을 시스템 안에 넣을 수 있는 개발자가 되기 위한 프로젝트**
