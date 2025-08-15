 commento_ai_week_4
===========================
멀티 분석 (LangChain + GPT-4o)

아이돌/연예 사진을 넣으면 인물 식별, 촬영 장소/시기 추정, 이벤트/목적 요약, EXIF-GPS 역지오코딩, 웹 검색(선택), PDF 리포트 생성, Streamlit 대시보드까지 한 번에 수행하는 노트북입니다.<br>
본 README는 복사→붙여넣기만 하면 바로 쓸 수 있도록 구성했습니다.<br>

> 기준 파일: test1_gpt_4o.ipynb
> 권장 파이썬: Python 3.10+

핵심 기능<br>
	•	EXIF 기반 분석: GPS/촬영시각 추출 → 위/경도 변환 → Google Maps 역지오코딩으로 장소명 도출<br>
	•	멀티모달 요약: OpenAI GPT-4o / 4o-mini로 이미지 장면 분석, 인물/장소/시간/이벤트/목적을 bullet로 출력<br>
	•	EventCard 생성: EventCard 스키마에 맞춘 구조적 결과(타이틀/유형/발생시점/장소/근거/출처 등)<br>
	•	웹 검색 보강(옵션): Tavily(LangChain tool) 또는 OpenAI Responses API의 web_search_preview로 단서 확인<br>
	•	PDF 리포트: 이미지(좌) + 분석결과(우)를 페이지 단위로 정리해 PDF 생성(한글 폰트 지원)<br>
	•	Streamlit UI: 여러 장 업로드 → 진행/결과를 챗 형식으로 확인<br>

디렉터리 구조(권장)
```bash
.
├─ test1_gpt_4o.ipynb
├─ data/                 # 분석할 이미지 파일(.jpg/.png/...) 넣는 폴더
├─ outputs/              # PDF/JSON 결과가 저장될 폴더
└─ fonts/                # (선택) 한글 OTF/TTF 폰트
```

----------------------------
빠른 시작(노트북 실행)<br>
	1.	test1_gpt_4o.ipynb를 Jupyter에서 열고, 위에서 아래로 순서대로 실행<br>
	2.	첫 설치 셀 실행 후, API 키 셀에서 키가 들어갔는지 확인<br>
	3.	마지막 부분의 PDF 생성 셀에서 다음과 같이 실행됩니다.<br>

 pdf_path, results = build_pdf_from_data("data", "outputs/report.pdf")
print("PDF 생성:", pdf_path)

•	data/ 폴더의 모든 이미지가 순회되며 분석됩니다.<br>
•	outputs/report.pdf와 outputs/results.json(페이지별 원시 결과)이 생성됩니다.<br>

----------------------------

코드 개요(핵심 함수)
	•	has_gps_metadata(image_path) -> bool<br>
	•	extract_gps(image_path) -> (lat, lon) / _extract_exif_datetime(image_path) -> datetime|None<br>
	•	reverse_geocode(lat, lon, gmaps_api_key) -> str|None<br>
	•	explain_place_with_gpt(place_name, lat, lon, openai_api_key) -> str<br>
	•	analyze_image_with_gpt(image_path, openai_api_key, ...) -> str<br>
	•	_infer_event_card(...) -> EventCard (Tavily 검색을 내부적으로 사용 가능)<br>
	•	process_image(image_path, gmaps_api_key, openai_api_key) -> dict (엔드투엔드)<br>
	•	build_pdf_from_data(data_dir="data", out_pdf="outputs/report.pdf", ...) -> (pdf_path, results)<br>

```python
 {
  "method": "VISION|VISION+SEARCH",
  "event_title": "string|null",
  "event_type": "string|null",
  "occurred_time_utc": "string|null",
  "occurred_time_confidence": 0.0,
  "location_name": "string|null",
  "latitude": 0.0,
  "longitude": 0.0,
  "why_summary": "string|null",
  "what_happened": "string|null",
  "who_involved": "string|null",
  "key_evidence": ["..."],
  "sources": [{"url":"...","title":"..."}],
  "notes": "string|null"
}
```
실제 코드는 pydantic 모델(SourceItem, EventCard)을 사용합니다.

----------------------------

웹 검색 보강(참고)

노트북에는 OpenAI Responses API의 web_search_preview를 활용해 EventCard를 VISION → VISION+SEARCH로 승격하는 섹션이 포함되어 있습니다.<br>
해당 블록은 내부에서 try/except로 감싸져 있어, 보조 헬퍼(예: _build_search_seed, _parse_iso_utc_loose)가 없거나 키가 없으면 자동으로 생략됩니다.<br>
필요 시 직접 헬퍼를 추가해 활용하세요.

⸻-----------------------

PDF 리포트<br>
	•	한글 폰트를 지정하지 않으면 Helvetica로 대체됩니다.<br>
	•	한글이 깨질 경우 KR_FONT_PATH를 OTF/TTF 경로로 지정하세요.<br>
	•	결과는 outputs/report.pdf와 outputs/results.json으로 저장됩니다.<br>

⸻⸻-----------------------

권한 & 비용<br>
	•	이미지 분석/텍스트 생성은 OpenAI API 요금이 발생합니다.<br>
	•	역지오코딩은 Google Maps API 요금 정책을 따릅니다.<br>
	•	Tavily 검색은 별도 키/과금 정책을 확인하세요.<br>

⸻⸻-----------------------

트러블슈팅<br>
	•	googlemaps 키 오류: GMAPS_API_KEY 누락 시 역지오코딩만 건너뛰고 계속 진행합니다.<br>
	•	Tavily 키 없음: EventCard는 생성되지만, 웹 검색 보강은 생략됩니다.<br>
	•	폰트 경로 오류: PDF에서 한글이 네모로 보이면 KR_FONT_PATH를 올바른 OTF/TTF로 지정하세요.<br>
	•	OpenAI 인증 실패: OPENAI_API_KEY 재확인 및 네트워크/프록시 환경 확인.<br>
	•	Streamlit에서 함수 미정의: format_chat_style이 없으면 위의 간이 구현을 복사해 pipeline.py에 추가하세요.<br>

⸻⸻-----------------------

라이선스/주의<br>
	•	얼굴인식·신원추정은 민감한 개인정보에 해당될 수 있습니다. 반드시 합법적 목적·합법적 데이터로만 사용하세요.<br>
	•	공개 배포 시 API 키를 코드/깃에 커밋하지 마세요. .env/환경변수를 사용하세요.<br>

⸻⸻-----------------------

빠른 체크리스트<br>
	•	pip install ...로 의존성 설치 완료<br>
	•	OPENAI_API_KEY 설정<br>
	•	(선택) GMAPS_API_KEY, TAVILY_API_KEY, KR_FONT_PATH 설정<br>
	•	data/에 이미지 넣기<br>
	•	노트북 실행 또는 streamlit run app.py<br>
	•	outputs/report.pdf 생성 확인 ✅<br>
