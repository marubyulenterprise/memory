# 💻 Developer (Lead Engineer) 개인 메모리

_Developer 에이전트만 읽고 쓰는 개인 노트. 학습·교훈·자주 쓰는 패턴이 누적됩니다._

## 학습 기록

- [2026-05-03] 요구된 '완전 자동화' 목표 달성을 위한 기술 아키텍처 다이어그램을 설계해야 합니다. (Input: 리서치 데이터/스크립트 $\rightarrow$ Process: API 연동 기반의 영상 생성 및 편집 $\rightarrow$ Output: 유튜브 업로드). 필요한 핵심 API와 데이터 파이프라인 구축 순서를 단계별로 제시하시오. → 산출물 sessions/2026-05-03T05-27/developer.md
- [2026-05-03] 전체 자동화 파이프라인 아키텍처를 4단계 모듈로 재설계하라. (1) 스크립트(JSON) 입력 $ightarrow$ (2) TTS 및 자막 파일 생성 $ightarrow$ (3) 영상 합성/편집(FFmpeg 기반) $ightarrow$ (4) YouTube Data API v3 업로드 및 스케줄링 로직을 포함하는 상세 기술 로드맵과 필요한 선행 모듈 코드를 정의하라. → 산출물 sessions/2026-05-03T05-44/developer.md
- [2026-05-04] Business에서 정의한 CTA(Call To Action)와 리드 확보 과정을 영상 파이프라인의 최종 모듈로 구현할 1차 프로토타입 코드를 작성해주세요. 이 코드는 다음 기능을 포함해야 합니다: (1) 'Blueprint' 언급을 위한 전용 비디오 세그먼트 삽입 로직, (2) 해당 구간에 맞는 다운로드 링크/QR 코드 오버레이를 자동 생성하고, (3) 최종 영상 업로드 시 설명란(Description Box)의 Funnel 구조화된 템플릿 코드를 포함하도록 설계해야 합니다. → 산출물 sessions/2026-05-04T12-3