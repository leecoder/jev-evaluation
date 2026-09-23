# Jev Evaluation Archive

TypeSafe **Jev 1.13.0** 평가에 필요한 자료만 분리해 보관한 독립 프로젝트입니다.

## 포함 범위

- `eval/`: Jev API 평가 진입점과 입력 검증·확률·캘리브레이션 모듈
- `data/`: 로컬에서만 준비하는 평가 입력 위치; 저작권 있는 JSONL은 공개하지 않음
- `results/`: 실행 manifest와 summary; raw 행별 응답은 로컬 보관만 함
- `metadata/`: 원본 URL, SHA-256, 포함·제외 범위와 집계 결과

원본 문제지 PDF와 OCR 텍스트는 공개 저장소에 포함하지 않습니다. 평가 입력 JSONL은
로컬 전용이며 `.gitignore`로 추적을 막습니다. 공식 URL과 SHA-256은 각 manifest에
남아 있으므로 권한과 이용 조건을 확인한 환경에서 입력을 별도로 준비할 수 있습니다.
따라서 이 저장소만으로는 원문 문항을 재현하지 않으며, 공개된 결과는 해당 로컬 입력에서
생성된 집계 결과입니다.

## 평가 결과

| 데이터셋 | 문항 수 | 정답 | 정확도 | NLL | Brier | ECE10 | 중앙 요청 시간 |
|---|---:|---:|---:|---:|---:|---:|---:|
| 2026 수능 국어 홀수형 텍스트 완전 subset | 48 | 25 | 52.08% | 1.141 | 0.572 | 0.149 | 0.743s |
| SAT #11 Reading & Writing | 62 | 53 | 85.48% | 0.428 | 0.227 | 0.039 | 0.649s |
| SAT #11 Math 객관식 | 28 | 13 | 46.43% | 1.239 | 0.692 | 0.178 | 0.661s |
| SAT #11 합계 | 90 | 66 | 73.33% | 0.680 | 0.371 | 0.074 | 0.653s |

수치는 저장된 `results/*/summary.json` 및 `metadata/*-manifest.json`에서 확인할
수 있습니다. SAT Math의 SPR(학생 제작 응답)과 텍스트로 복원할 수 없는 그림·도표·
그래프 의존 문항은 제외했습니다. 국어도 동일하게 시각 자료가 없는 문항만 포함했습니다.

## 평가 계약

각 행은 하나의 `decision` Choice 질문만 보냅니다.

```json
{
  "state": "문항 본문과 선지",
  "questions": {
    "decision": {
      "type": "choice",
      "instructions": "문항의 요구에 따라 가장 적절한 답을 고르시오.",
      "criteria": {"A": "선지 A", "B": "선지 B"}
    }
  }
}
```

`reference.target`와 정답표는 API 요청에 들어가지 않습니다. 러너는 모델 응답을
검증한 뒤에만 reference를 join하며, 요청 payload에 정답 관련 키가 들어가면 중단합니다.
확률 지표는 TypeSafe의 별도 `confidence`가 아니라 후보 분포에서 정답 후보의
확률을 사용합니다.

## 재실행

프로젝트 루트에서 실행한다고 가정합니다. Python 표준 라이브러리만으로
오프라인 검증과 평가 명령을 실행할 수 있습니다.

```sh
python3 -m eval \
  --data /path/to/local/csat-2026-korean-odd-text.jsonl \
  --output-dir results/csat-2026-korean-odd

python3 -m eval \
  --data /path/to/local/sat-practice-11-rw-text.jsonl \
  --output-dir results/sat-practice-11-rw

python3 -m eval \
  --data /path/to/local/sat-practice-11-math-mc-text.jsonl \
  --output-dir results/sat-practice-11-math
```

기본 모델은 `jev-1.13.0`이며, API endpoint는
`https://api.typesafe.ai/v1/systemone`입니다. 이미 완주한 결과 디렉터리에서
재실행하려면 새 출력 디렉터리를 사용하거나 기존 결과를 정리하십시오.

## 보안 및 데이터 주의

- 이전 실행 결과의 raw 응답에는 문항별 모델 출력이 포함될 수 있으므로
  `results/*/rows.jsonl`은 `.gitignore`로 제외했습니다. 공개 저장소에는
  summary와 manifest만 올립니다.
- 정답은 평가 후 정확도 계산에만 사용됩니다. 데이터셋의 `reference`는 모델 입력과
  별도 경로에 있습니다.
- 문제지 원문과 문항 텍스트는 제3자 저작권 보호 대상이며 이 저장소에는 포함하지 않습니다. [EBSi 공식 자료](https://www.ebsi.co.kr/ebs/xip/xipa/retrieveSCVMainInfo.ebs?irecord=202511133&targetCd=D300) 및 [College Board의 저작권·상표 사용 안내](https://privacy.collegeboard.org/copyright-trademark/request-instructions)를 확인하십시오.

## 참고 문서

- [TypeSafe API](https://docs.typesafe.ai/api.md)
- [TypeSafe Choice](https://docs.typesafe.ai/primitives/choice.md)
- [2026학년도 수능 공식 EBSi 페이지](https://www.ebsi.co.kr/ebs/xip/xipa/retrieveSCVMainInfo.ebs?irecord=202511133&targetCd=D300)
- [College Board SAT Practice Test #11](https://satsuite.collegeboard.org/media/pdf/sat-practice-test-11-digital.pdf)
