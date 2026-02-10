# Attribute Tree Structure for Images

이미지의 엔티티(객체), 속성(attributes), 관계(relationships)를 구조화된 JSON 형식으로 표현하기 위한 스키마.

## 목적

웹툰 컷(단일 이미지)을 다음 관점에서 의미론적으로 파싱하여 컷 이해, 객체 레벨 편집, PPL 삽입 판단 및 편집 지시 생성에 재사용 가능한 표준 포맷을 제공한다.

- 객체/텍스트 관점
- 관계(Relationship) 관점  
- 추론(Inference) 관점

---

## 설계 원칙

| 원칙 | 설명 |
|------|------|
| **객체 중심** | 이미지에서 "세는 것(countable)"은 모두 객체로 본다 |
| **텍스트 분리** | 말풍선, 내레이션, OCR 텍스트는 별도 배열에서 정규화 |
| **관계 분리** | 객체 간 상호작용은 엣지(edge)로 분리하여 관리 |
| **추론 분리** | 픽셀에 직접 표현되지 않은 추론은 별도 섹션에 기록 |
| **영어 키** | 고유명사가 아닌 한 모든 key는 영어 snake_case로 통일 |
| **확장성** | v1은 "단일 이미지" 기준. 다중 컷/시퀀스는 v2 예정 |

---

## 1. Root 구조

```
Root
├─ schema_version (string)              // e.g. "1.0"
├─ scene (Scene) [REQUIRED]
├─ objects (Object[]) [REQUIRED]
├─ environment_objects (EnvironmentObject[]) [OPTIONAL]
├─ text_elements (TextElement[]) [REQUIRED]
├─ relationships (Relationship[]) [OPTIONAL]
└─ implicit_information (ImplicitInfo) [OPTIONAL]
```

---

## 2. Scene - 장면 정보

이미지의 전체 배경 및 분위기를 정의한다.

### 2.1 구조

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `scene_type` | string (enum) | **O** | 장면의 유형 |
| `location` | string | | 구체적인 위치 설명 |
| `tone` | string (enum) | | 전체 분위기/톤 |
| `narrative_perspective` | string (enum) | | 서사 관점 |
| `context_summary` | string | | 장면 맥락 요약 |

### 2.2 권장 Enum 값

**scene_type**
- `interior` - 실내
- `exterior` - 실외
- `public_space` - 공공장소
- `cinema_entrance` - 영화관 입구
- `food_service_counter` - 음식점 카운터
- `unknown` - 불명

**tone**
- `warm_casual` - 따뜻한 캐주얼
- `comedic_tension` - 코믹한 긴장
- `serious` - 진지함
- `romantic` - 로맨틱
- `neutral` - 중립

**narrative_perspective**
- `interaction_focus` - 상호작용 중심
- `observer_reveal` - 관찰자 관점의 공개
- `observer_commentary` - 관찰자 해설
- `character_focus` - 캐릭터 중심

---

## 3. Object - 주요 객체

이미지에 포함된 개별 엔티티를 정의한다.


### 3.1 공통 필수 구조

모든 객체 타입에 공통으로 적용되는 필드.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `object_id` | string | **O** | 문서 내 유일한 ID |
| `object_name` | string | **O** | 물체가 사용자에게 인식되는 일반적인 이름 |
| `object_type` | string (enum) | **O** | 객체의 대분류 타입 |
| `sub_type` | string | | 세부 타입 (e.g., "cup", "chair", "phone") |
| `position` | string (enum) | | 프레임 내 위치 (3.6 참조) |
| `associated_with` | object_id | | 관련된 다른 객체 ID |

### 3.2 object_type 대분류

객체를 크게 3가지로 분류한다.

- `human` - 인물
- `animal` - 동물
- `object` - 사물 (모든 무생물 포함)

**사물(object)의 sub_type 예시:**
- `product` - 상품, PPL의 대상
- `tool` - 도구
- `drink` - 음료
- `food` - 음식
- `furniture` - 가구
- `signage` - 간판/표지
- `device` - 기기
- `vehicle` - 탈것
- `clothing_item` - 의류 아이템
- `accessory_item` - 액세서리 아이템
- `other` - 기타

### 3.3 인물(Human) 전용 속성

`object_type: "human"` 일 때 사용 가능한 선택적 필드.

| 필드 | 타입 | 설명 |
|------|------|------|
| `role` | string | 역할 (e.g., "customer", "staff", "observer") |
| `orientation` | string (enum) | 바라보는 방향 (3.7 참조) |
| `facial_visibility` | string (enum) | 얼굴 표현 여부 (3.8 참조) |
| `emotional_state` | string | 감정 상태 (e.g., "happy", "surprised", "neutral") |
| `gesture` | string | 제스처/포즈 (e.g., "pointing", "waving", "arms_crossed") |
| `action` | string | 현재 수행 중인 행동 |
| `clothing` | Clothing | 의류 정보 (3.9 참조) |
| `accessory` | string \| string[] | 착용 중인 액세서리 |
| `tool_used` | object_id | 사용 중인 도구의 object_id |
| `speech` | Speech | 발화 정보 (3.10 참조) |

### 3.4 동물(Animal) 전용 속성

`object_type: "animal"` 일 때 사용 가능한 선택적 필드.

| 필드 | 타입 | 설명 |
|------|------|------|
| `species` | string | 종 (e.g., "dog", "cat", "bird") |
| `role` | string | 역할 (e.g., "pet", "wild", "companion") |
| `orientation` | string (enum) | 바라보는 방향 (3.7 참조) |
| `emotional_state` | string | 감정 상태 (e.g., "playful", "alert", "resting") |
| `action` | string | 현재 수행 중인 행동 |
| `accessory` | string \| string[] | 착용 중인 액세서리 (목줄, 리본 등) |

### 3.5 사물(Object) 전용 속성

`object_type: "object"` 일 때 사용 가능한 선택적 필드.

| 필드 | 타입 | 설명 |
|------|------|------|
| `brand` | string | 브랜드명 |
| `branding_visible` | boolean | 브랜딩/로고 표시 여부 |
| `color` | string \| string[] | 주요 색상 |
| `state` | string | 상태 (e.g., "open", "closed", "damaged", "full", "empty") |
| `orientation` | string (enum) | 방향 (3.7 참조) |
| `function` | string | 기능/용도 |
| `text_on_object` | string | 객체에 표시된 텍스트 |

### 3.6 position Enum (공간 위치)

**깊이(Depth) × 좌우(Horizontal) 조합:**

| 전경(Foreground) | 중경(Midground) | 배경(Background) |
|------------------|-----------------|------------------|
| `left_foreground` | `left_midground` | `left_background` |
| `center_foreground` | `center_midground` | `center_background` |
| `right_foreground` | `right_midground` | `right_background` |

**일반 포지션:**
- `foreground` - 전경 (좌우 미지정)
- `midground` - 중경
- `background` - 배경
- `unknown` - 불명

### 3.7 orientation Enum (방향)

- `front` - 정면
- `back` - 뒤
- `left` - 좌향
- `right` - 우향
- `side` - 옆면
- `unknown` - 불명

### 3.8 facial_visibility Enum (얼굴 표현)

인물 전용. 얼굴의 가시성을 나타낸다.

- `visible` - 명확히 보임
- `partial` - 부분적으로 보임
- `hidden` - 보이지 않음
- `unknown` - 불명

### 3.9 Clothing - 의류 정보

인물 전용. 의류 속성을 정의한다. (선택 사항)

| 필드 | 타입 | 설명 |
|------|------|------|
| `uniform` | boolean | 제복 여부 |
| `outerwear_type` | string | 외투 종류 (e.g., "jacket", "coat") |
| `primary_color` | string | 주 색상 |
| `secondary_color` | string | 보조 색상 |

**색상 표기:**
- v1.0: 기본 색상명(영문) 또는 HEX 코드 모두 허용
- v1.1 권장: `color_hex` 표준화

### 3.10 Speech - 발화 정보

인물 전용. 발화할 때의 음성/텍스트 정보. (선택 사항)

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `text` | string | **O** | 발화 텍스트 |
| `language` | string | **O** | 언어 코드 (e.g., "ko", "ja", "en") |
| `speech_type` | string (enum) | **O** | 발화 유형 |
| `politeness_level` | string (enum) | | 존댓말 수준 |

**speech_type Enum:**
- `dialogue` - 대사
- `narration` - 내레이션
- `thought` - 생각/독백
- `sound_effect` - 의성어/의태어

**politeness_level Enum:**
- `informal` - 반말
- `casual_polite` - 편한 존댓말
- `formal_polite` - 정중한 존댓말
- `unknown` - 불명

---

## 4. EnvironmentObject - 환경 객체

배경에서 의미 있는 객체만 별도로 기록한다. (선택 사항)

배경은 무한정이므로 **"맥락을 규정하거나 편집 타겟이 되는 것만"** 환경 객체로 포함한다.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `object_id` | string | **O** | 문서 내 유일한 ID |
| `object_type` | string (enum) | **O** | 객체의 대분류 ("object" 고정) |
| `sub_type` | string | **O** | 세부 타입 (e.g., "furniture", "signage", "wallpaper") |
| `position` | string (enum) | | 프레임 내 위치 (3.6 참조) |
| `semantic_role` | string | | 의미론적 역할 (e.g., "location_identifier") |
| `interaction_role` | string | | 상호작용 역할 (e.g., "exchange_surface") |

**참고:** EnvironmentObject는 기본적으로 `object_type: "object"`로 고정되며, `sub_type`으로 구체적인 환경 객체 종류를 표현한다.

---

## 5. TextElement - 텍스트 요소

이미지 내 모든 텍스트를 정규화하여 검색, 학습, 인덱싱에 최적화한다.

**참고:** 말풍선의 내용은 `objects[].speech`에도 있을 수 있으나, 모든 텍스트를 한 번 더 정규화한다.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `text_id` | string | **O** | 문서 내 유일한 ID |
| `text` | string | **O** | 실제 텍스트 |
| `language` | string | **O** | 언어 코드 |
| `text_type` | string (enum) | **O** | 텍스트 유형 |
| `speaker_reference` | object_id \| string | | 발화자 객체 ID 참조 |
| `position` | string (enum) | | 텍스트의 위치 |
| `semantic_function` | string | | 의미론적 기능 |

**text_type Enum:**
- `dialogue` - 대사
- `narration` - 내레이션
- `thought` - 생각
- `sound_effect` - 의성어
- `sign_text` - 간판/표지 텍스트

**semantic_function 예시:**
- `setup` - 상황 설정
- `reveal` - 정보 공개
- `gratitude_response` - 감사 응답
- `service_greeting` - 서빙 인사
- `attention_call` - 주목 유도
- `inference` - 추론 유도

---

## 6. Relationship - 객체 간 관계

객체들 간의 상호작용 및 관계를 정의한다. (선택 사항)

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `source` | object_id | | 출발점 객체 (집단 관계 시 생략 가능) |
| `target` | object_id | | 대상 객체 |
| `relationship_type` | string (enum) | **O** | 관계의 유형 |
| `direction` | string | | 방향성 |
| `awareness` | string | | 인식 수준 |
| `emotional_inference` | string | | 감정 추론 |
| `group` | GroupRelationship | | 집단 관계 정보 |

**relationship_type Enum (예시):**
- `observing` - 관찰하는
- `being_observed` - 관찰받는
- `service_interaction` - 서비스 상호작용
- `handing` - 전달하는
- `receiving` - 받는
- `using_tool` - 도구 사용
- `speaking_to` - 말하는

### 6.1 GroupRelationship - 집단 관계

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `group_name` | string | **O** | 그룹 이름 (e.g., "moviegoers") |
| `members` | object_id[] | **O** | 그룹 멤버의 object_id 배열 |
| `collective_role` | string | | 집단의 역할 |
| `collective_emotional_state` | string | | 집단의 감정 상태 |

---

## 7. ImplicitInfo - 암묵적 정보

픽셀에 직접 표현되지 않으나 이미지에서 추론할 수 있는 정보. (선택 사항)

| 필드 | 타입 | 설명 |
|------|------|------|
| `social_context` | string | 사회적 맥락 |
| `power_dynamic` | string | 권력 관계/역학 |
| `humor_mechanism` | string | 유머 메커니즘 |
| `shared_experience` | string | 공유된 경험 |
| `ppl_potential` | string (enum) | PPL 삽입 가능성 |
| `ppl_reason` | string | PPL 삽입 이유 |

**ppl_potential Enum:**
- `high` - 높음
- `medium` - 중간
- `low` - 낮음
- `unknown` - 불명

---

## 8. MVP 검증 규칙

v1.0에서 "이미지 1개가 충분히 구조화되었는가"를 판단하는 최소 요구사항.

### 필수 필드

```
✓ schema_version
✓ scene.scene_type
✓ objects[] (최소 1개)
  - 각 object는 object_id, object_type 필수
  - object_type은 "human", "animal", "object" 중 하나
✓ text_elements[] (최소 1개 권장, 0개도 허용 가능)
```

### 데이터 일관성

- 모든 `object_id`, `text_id`는 문서 내 유일해야 함
- `object_type`은 반드시 "human", "animal", "object" 중 하나여야 함
- 각 object_type에 맞는 선택적 속성만 사용해야 함
  - human: role, orientation, facial_visibility, emotional_state, gesture, action, clothing, accessory, tool_used, speech
  - animal: species, role, orientation, emotional_state, action, accessory
  - object: brand, branding_visible, color, state, orientation, function, text_on_object
- `text_elements[].speaker_reference`가 `object_id`를 참조한다면, 반드시 `objects[]`에 존재해야 함
- `relationships[].source/target` 및 `GroupRelationship.members`는 반드시 존재하는 `object_id`만 참조
- `tool_used`는 objects[] 또는 environment_objects[]에 존재하는 object_id여야 함

---

## 9. 최소 JSON 스켈레톤

```json
{
  "schema_version": "1.0",
  "scene": {
    "scene_type": "interior",
    "location": "",
    "tone": "neutral",
    "narrative_perspective": "interaction_focus",
    "context_summary": ""
  },
  "objects": [],
  "environment_objects": [],
  "text_elements": [],
  "relationships": [],
  "implicit_information": {}
}
```

---

## 10. 사용 흐름

1. **이미지 분석** → 장면 유형, 분위기 파악
2. **객체 추출** → 세는 것 모두를 `objects[]`에 기록
   - `object_type`을 human/animal/object로 분류
   - 각 타입에 맞는 선택적 속성 추가
3. **텍스트 정규화** → 모든 텍스트를 `text_elements[]`에 수집
4. **관계 정의** → 객체 간 상호작용을 `relationships[]`로 표현
5. **추론 기록** → 픽셀에 없으나 추론되는 정보를 `implicit_information`에 기록
6. **유효성 검사** → MVP 검증 규칙 충족 확인

---

## 11. 버전 정보

| 버전 | 범위 | 상태 |
|------|------|------|
| v1.0 | 단일 이미지 | 현재 |
| v1.1 | + 색상 HEX 표준화 | 계획 |
| v2.0 | 다중 컷/시퀀스 | 계획 |
5. **추론 기록** → 픽셀에 없으나 추론되는 정보를 `implicit_information`에 기록
6. **유효성 검사** → MVP 검증 규칙 충족 확인

---

## 11. 버전 정보

| 버전 | 범위 | 상태 |
|------|------|------|
| v1.0 | 단일 이미지 | 현재 |
| v1.1 | + 색상 HEX 표준화 | 계획 |
| v2.0 | 다중 컷/시퀀스 | 계획 |
