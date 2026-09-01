# AGENTS.md — 벨라리치 진단테스트 저장소

## 이 저장소는 무엇인가
벨라리치(법인 자산관리 컨설팅) 고객이 **상담 전에 푸는 온라인 진단테스트지**의 배포용 스냅샷입니다.
`index.html` 단일 페이지에 마크업·CSS·JS가 전부 인라인되어 있고, `quiz-data.js`가 문항/채점/결과 카피를 담습니다.

## ⚠️ 에이전트가 지금 해야 할 일

**이 저장소를 열었다면, 다른 작업을 시작하기 전에 반드시 아래 지시서를 읽고 그대로 수행하십시오.**

👉 **[ANTIGRAVITY-작업지시서.md](./ANTIGRAVITY-작업지시서.md)**

요약: 코드 수정 없이 **내용(콘텐츠·채점로직·백엔드연동·버전이력·리스크)을 분석**하여
`docs/ANALYSIS-01~05.md` + `docs/HANDOFF.md` 를 한국어로 작성하는 작업입니다.
상세 요구사항·산출물 형식·근거 표기 규칙은 전부 지시서에 있습니다.

## 절대 규칙 (읽기 전용)
- `index.html`, `quiz-data.js`, `venn-diagram.png`, `README.md`, `.git` — **수정 금지**
- 새 파일은 `docs/` 아래에만 생성
- `git commit` / `git push` 하지 말 것 (사용자가 직접 판단)
- 모든 주장에 `파일명:줄번호` 근거를 달 것. 추측은 "추정:" 명시, 모르면 "확인 불가" 목록으로.

## 맥락이 더 필요할 때 (읽기 전용 참조)
이 저장소에는 서버 코드와 문서가 없습니다. 필요하면:
- `C:\Users\LG\Desktop\미니맥스\bellarich-test` — 원 작업 폴더. `apps-script.gs`(서버 전문), `HANDOFF.md`, 이메일 템플릿, PDF 샘플, 회의록, 질문지 엑셀
- `C:\Users\LG\.minimax\workspace\bellarich-v5|v6|v6.2|v6.5|v7|v8-backup` — 버전별 스냅샷 (diff 분석용)
