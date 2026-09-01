# 작업 지시 · 득점표 PDF 변환

## 목표
`docs/scorecard.html`(인쇄용 단독 HTML, 이미 완성됨)을 팀원 배포용 PDF로 변환한다.
**HTML 내용은 절대 수정하지 말 것.** 변환만 한다.

## 1. 변환 명령 (그대로 실행)

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --no-pdf-header-footer --virtual-time-budget=15000 --print-to-pdf="C:\Users\LG\orca\BellaRich\docs\scorecard.pdf" "file:///C:/Users/LG/orca/BellaRich/docs/scorecard.html"
```

- `--no-pdf-header-footer`가 인식되지 않으면 `--print-to-pdf-no-header`로 바꿔 재시도.
- 그래도 실패하면 `--headless=new`를 `--headless`로 바꿔 재시도.
- 웹폰트(Google Fonts) 로딩 때문에 `--virtual-time-budget`은 반드시 유지한다. 값을 줄이면 폰트가 깨진 채로 렌더된다.

## 2. 검증 (전부 통과해야 완료)

1. `docs/scorecard.pdf`가 존재하고 크기가 **100KB 이상**
2. `pdftotext -layout -enc UTF-8 docs/scorecard.pdf -` 결과에 아래 문자열이 **모두** 포함
   - `본선 진출 득점표`
   - `운영 인수인계 매뉴얼`
   - `천장 26.0`
   - `3차 점포 현장방문 보고서`
3. 페이지 수가 **6~14쪽** 범위
4. 텍스트가 `□□□` 같은 두부(tofu) 글자로 깨지지 않았는지 육안 확인

검증 실패 시 위 재시도 옵션을 순서대로 적용하고, 그래도 안 되면 시도한 명령과 오류 메시지를 그대로 기록한다. **성공한 척하지 말 것.**

## 3. 절대 금지

- `index.html`, `quiz-data.js`, `venn-diagram.png`, `README.md`, `docs/scorecard.html` 수정 금지
- 새 파일은 `docs/` 아래에만 생성
- `git commit` / `git push` 금지

## 4. 완료 보고 (필수)

작업이 끝나면 **반드시** 아래 명령을 실행해 메인 에이전트에게 보고한다. 파일만 만들고 끝내지 말 것.

```
"C:\Users\LG\AppData\Local\Programs\orca\resources\bin\orca.exe" orchestration send --to term_6c063f4c-3159-494e-b426-549b9fb83bce --subject "득점표 PDF 변환 완료" --body "페이지수 N쪽, 파일크기 NKB, 검증 4항목 통과/실패 여부, 사용한 최종 명령"
```

실패로 끝난 경우에도 같은 명령으로 `--subject "득점표 PDF 변환 실패"`와 원인을 보내야 한다.
