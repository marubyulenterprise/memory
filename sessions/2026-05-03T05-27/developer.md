# 💻 Developer — 요구된 '완전 자동화' 목표 달성을 위한 기술 아키텍처 다이어그램을 설계해야 합니다. (Input: 리서치 데이터/스크립트 $\rightarrow$ Process: API 연동 기반의 영상 생성 및 편집 $\rightarrow$ Output: 유튜브 업로드). 필요한 핵심 API와 데이터 파이프라인 구축 순서를 단계별로 제시하시오.

💻 Developer: 작업 시작합니다.

요청하신 '유튜브 쇼츠 완전 자동화 아키텍처'는 단순한 스크립팅을 넘어, **데이터 파이프라인(Data Pipeline)**과 **미디어 처리 워크플로우(Media Processing Workflow)**를 결합해야 가능한 영역입니다. 현재 저희의 목표는 이 워크플로우 전체를 자동화하는 것입니다.

아래에 핵심 API와 데이터 흐름을 포함한 3단계 구축 순서 및 아키텍처 다이어그램을 제시합니다. 이 시스템은 **JSON 형식으로 구조화된 스크립트**를 입력(Input)받아, 최종적으로 **업로드 가능한 MP4 파일과 YouTube 메타데이터**를 출력(Output)하는 것을 목표로 합니다.

---

## ⚙️ 자동화 파이프라인 아키텍처 다이어그램 (The Full Stack Pipeline)

```mermaid
graph TD
    A[Input: 구조화된 스크립트 JSON] --> B{Stage 1: 미디어 에셋 생성};
    B --> C[TTS API: 음성 오디오 파일(.mp3)];
    B --> D[Image/Video Gen API: 배경 영상 클립 및 이미지];
    C & D --> E{Stage 2: 비디오 컴포지팅 (영상 편집)};
    E --> F[FFmpeg Wrapper / Video SDK: 자막, 사운드믹싱, 트랜지션 적용];
    F --> G[Output: 최종 MP4 파일];
    G --> H{Stage 3: 배포 및 관리};
    H --> I[YouTube Data API v3: 업로드/스케줄링];
    I --> J[로그 데이터베이스: 성공/실패 기록, 분석 지표];
```

## 🚀 단계별 구축 순서 (Implementation Roadmap)

### Stage 1. 입력 및 미디어 에셋 생성 파이프라인
**목표:** 텍스트 스크립트를 비디오가 재생될 수 있는 원시 미디어 파일(Raw Assets)로 분해합니다.

1.  **[Input 구조화] 스크립트 → JSON 변환 (Writer/Researcher Output 활용):**
    *   스크립트는 단순한 글이 아닌, `{"scene_id": 1, "start_time": "0:05", "duration": 7, "text": "AI를 잘못 쓰면..."}` 와 같은 **구조화된 JSON 형식**이어야 합니다. (핵심!)
2.  **[Audio Generation] TTS API 연동:**
    *   **사용 기술:** Google Cloud Text-to-Speech API 또는 OpenAI Whisper/TTS 등 고품질 TTS 서비스.
    *   **작업 내용:** JSON의 `text` 필드를 반복문으로 읽어, 각 장면(Scene)별로 분리된 오디오 파일(.mp3)을 생성합니다. (오디오 길이는 해당 텍스트 길이와 정확히 일치해야 함).
3.  **[Visual Asset Fetching] 이미지/영상 클립 검색 API 연동:**
    *   **사용 기술:** Pexels/Pixabay 등 스톡 콘텐츠 API 또는 직접 제작한 에셋 풀(Asset Pool) 관리 시스템.
    *   **작업 내용:** 장면의 주제 키워드를 추출하여, 해당 키워드와 일치하는 배경 영상 클립(`B-roll`)을 자동으로 검색하고 다운로드합니다.

### Stage 2. 비디오 컴포지팅 (핵심 편집 자동화)
**목표:** 분리된 오디오, 영상, 자막을 하나의 완벽한 MP4 파일로 합칩니다. **가장 복잡하며 가장 높은 리소스가 필요한 단계입니다.**

1.  **[Synchronization & Timing] 시간 동기화 로직 구현:**
    *   오디오 파일의 길이를 기준으로 비디오 템플릿의 타임라인을 설계합니다. (예: 오디오가 5초면, 배경 영상 클립도 최소 5초 이상이어야 함).
2.  **[Video Rendering] FFmpeg Wrapper 구축:**
    *   **핵심 기술:** `FFmpeg` 라이브러리를 파이썬(`subprocess` 또는 전용 바인딩)에서 호출하여 제어합니다.
    *   **작업 내용 1 (비디오 레이어):** 배경 영상 클립들을 순서대로 자르고 붙여(Concatenate), 비디오 트랙을 만듭니다.
    *   **작업 내용 2 (오디오/자막 레이어):** 생성된 TTS 오디오를 메인 트랙으로 설정하고, 스크립트 JSON에 따라 **자동 타이밍 및 스타일이 적용된 자막(Subtitle Overlay)**을 영상 위에 합성합니다.
    *   **출력:** `final_video.mp4` 파일 획득.

### Stage 3. 배포 및 관리 파이프라인
**목표:** 최종 영상을 업로드하고, 운영 데이터를 수집하여 다음 콘텐츠 제작에 활용합니다.

1.  **[Metadata Generation] 메타데이터 자동 작성:**
    *   Writer가 제공한 스크립트의 핵심 키워드와 Researcher의 주제 구조(예: `ChatGPT vs Gemini`)를 기반으로 제목(`Title`), 상세 설명(`Description`), 해시태그(`Hashtag`) JSON을 완성합니다.
2.  **[Upload Execution] YouTube Data API v3 연동:**
    *   **필요 권한:** OAuth 2.0 Scope: `youtube.upload` 및 `youtube.force-ssl`.
    *   **작업 내용:** MP4 파일을 업로드하고, Stage 3에서 생성된 메타데이터를 동시에 전송하여 예약(Scheduling)합니다.
3.  **[Logging & Feedback] 데이터베이스 기록:**
    *   API 호출 성공/실패 여부, 업로드 URL, 사용된 API 비용 등을 DB에 기록하여 시스템 운영의 투명성을 확보하고 디버깅 자료로 활용합니다.

---

## 🛠️ 기술 스택 요약 (Technical Stack Summary)

| 영역 | 핵심 역할 | 추천 API / 라이브러리 | 비고 |
| :--- | :--- | :--- | :--- |
| **데이터 입력** | 구조화된 콘텐츠 전송 | Python JSON, Pydantic Schema | 가장 중요한 선행 조건. |
| **오디오 생성** | 텍스트 $\rightarrow$ 음성 파일 | Google Cloud TTS / ElevenLabs API | 고품질 음성이 필수적입니다. |
| **영상 처리** | 자막 합성, 클립 편집, 최종 MP4 출력 | FFmpeg (Python Wrapper) | 비전공자가 다루기 가장 어려운 부분이지만, 자동화의 핵심입니다. |
| **업로드/배포** | 유튜브 업로드 및 스케줄링 | YouTube Data API v3 | 반드시 OAuth 인증이 필요합니다. |

---
📊 평가: 진행중 — 아키텍처 설계는 완료되었으나, 실제 구현을 위한 각 단계별 코드 모듈(예: `TTS_generator.py`, `VideoComposer.py`) 작성이 남아있어 다음 스텝이 필요합니다.
📝 다음 단계: Stage 1의 가장 핵심적인 첫 모듈인 '구조화된 JSON 기반 TTS 오디오 파일 생성 및 관리' 파트를 구현하는 코드를 요청합니다.
