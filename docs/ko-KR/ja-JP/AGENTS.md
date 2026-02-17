# AGENTS.md - ja-JP 문서 번역 작업 공간

## 읽어야 할 때

- `docs/ja-JP/**` 유지보수 시
- 일본어 번역 파이프라인(용어집/TM/프롬프트) 업데이트 시
- 일본어 번역 피드백이나 회귀 오류 처리 시

## 파이프라인 (docs-i18n)

- 원본 문서: `docs/**/*.md`
- 대상 문서: `docs/ja-JP/**/*.md`
- 용어집: `docs/.i18n/glossary.ja-JP.json`
- 번역 메모리: `docs/.i18n/ja-JP.tm.jsonl`
- 프롬프트 규칙: `scripts/docs-i18n/prompt.go`

일반적인 실행 방법:

```bash
# 대량 (문서 모드; 병렬 처리 가능)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# 단일 파일
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# 작은 패치 (세그먼트 모드; TM 사용; 병렬 처리 불가)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

주의사항:

- 전체 페이지 번역에는 `doc` 모드를 선호하세요. 작은 수정에는 `segment` 모드를 사용하세요.
- 매우 큰 파일이 타임아웃되는 경우, 타겟 편집을 하거나 페이지를 분할한 후 다시 실행하세요.
- 번역 후 점검: 코드 스팬/블록이 변경되지 않았는지, 링크/앵커가 변경되지 않았는지, 자리표시자가 유지되었는지 확인하세요.