# 간접 프롬프트 인젝션 공격 성공률(ASR) 평가

IndustryBench 문서에 숨긴 공격 지시를 오픈 모델 3종(Qwen2.5-7B-Instruct, Llama-3.1-8B-Instruct, Mistral-7B-Instruct-v0.3)이 얼마나 따르는지 측정하고, Gemma(`gemma-4-31B-it-FP8-dynamic`)로 채점했다.

- 공격 유형 1–8: document 공격
- 공격 유형 9–15: process 공격

## 최종 보고서

**`간접_프롬프트_인젝션_ASR_보고서_통합.pdf`** (공격 유형 1–15 통합, 8쪽)

## 폴더 구조

```
.
├── 간접_프롬프트_인젝션_ASR_보고서_통합.pdf   최종 보고서
├── README.md
│
├── results_document_attacks_12672/       공격 1–8: 모델 답변 + 채점 결과
│   ├── {qwen,llama,mistral}_{attack,clean}.jsonl
│   └── evaluation/
│       └── gemma_all_eval_document_attacks_12672.csv   ← 채점 결과 (분석 입력)
│
├── bipia/                                공격 9–15: 생성·병합·채점 코드와 결과
│   ├── 01_generate_responses.ipynb       답변 생성 (Colab)
│   ├── 02_data_merging.ipynb             9to12 + 13to15 → 9to15 병합
│   ├── 03_judge_v2.ipynb                 Gemma 채점 (judge_9to15.py 실행)
│   ├── run_hf_models.py / align_results.py / check_truncated.py / judge_9to15.py
│   ├── IndustryBench_Process_Attack_{9to12,13to15}.xlsx   입력 데이터
│   ├── results_process_attack_9to12/     생성 원본 (공격 9–12)
│   ├── results_process_attack_13to15/    생성 원본 (공격 13–15)
│   └── results_process_attack_9to15/     병합본 + evaluation/
│       └── evaluation/gemma_all_eval_process_attack_9to15.csv   ← 채점 결과 (분석 입력)
│
├── merge_and_analyze.py                  두 채점 CSV 합쳐 집계 → results_combined/
├── asr_analysis.ipynb                    집계·그림 노트북 → results_combined/
├── check_translation.py                  번역 점검 → results_combined/translation_check.xlsx
├── results_combined/                     통합 집계표(asr_combined.xlsx)와 그림(figures/)
```

## GitHub에서 내려받은 뒤

저장소 루트에서 두 ZIP 파일을 풀면 아래 재현 절차에 필요한 원래 폴더 구조가 만들어집니다.

```sh
unzip bipia.zip
unzip results_document_attacks_12672.zip
```

위에 적힌 최종 보고서 PDF와 `results_combined/` 폴더는 전달받은 제출 폴더에 포함되어 있지 않습니다.

## 재현 순서

1. **답변 생성**: `bipia/01_generate_responses.ipynb`를 Colab에서 실행한다. 공격 1–8의 답변은 `results_document_attacks_12672/`에 들어 있다.
2. **병합**: `bipia/02_data_merging.ipynb`를 실행해 `results_process_attack_9to15/`를 만든다.
3. **채점**: `bipia/03_judge_v2.ipynb`를 실행한다(`judge_9to15.py` 사용). 결과는 각 `evaluation/` 폴더에 저장된다.
4. **통합 집계**: 이 폴더에서 `python merge_and_analyze.py`를 실행하거나 `asr_analysis.ipynb`를 연다. 결과는 `results_combined/`에 저장된다.

`bipia/`의 노트북은 Colab(`/content/drive/MyDrive/bipia`)에서 실행했다. 4–5단계 스크립트는 이 폴더를 기준으로 한 상대 경로를 사용한다.

## 참고

- `evaluation/` 폴더마다 같은 채점 결과가 `gemma_judgments.jsonl`(원본 판정)과 `gemma_all_eval_*.csv`(분석용 표) 두 형식으로 들어 있다.
- 채점 설정(모델 revision, temperature, 입력 해시 등)은 `evaluation/evaluation_manifest.json`에, 생성 설정은 `input_manifest.json`에 기록되어 있다.
- 공격 1–8의 입력 파일 `IndustryBench_Document_Attacks_12672.xlsx`와 생성 코드는 이 폴더에 포함되어 있지 않다(서버 `/root/bipia`에서 실행).
