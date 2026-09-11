# 실행 명령 고정. 세션은 이 타겟만 사용한다. (쓰기 X)
# 각 타겟의 실제 명령을 스택에 맞게 채운다. 세션이 실행 방법을 추측하지 못하게 하는 것이 목적.

.PHONY: setup dev test test-slice test-e2e lint

setup:            ## clone 직후 1회. 훅 연결 (project/ 가 레포 하위 폴더여도 동작)
	git config core.hooksPath "$$(git rev-parse --show-prefix).githooks"
	chmod +x .githooks/*

dev:              ## 백엔드 + 프론트 개발 서버
	@echo "TODO: e.g. ./gradlew bootRun & (cd front && npm run dev)"; exit 1

test:             ## 단위 + 계약 테스트 전체
	@echo "TODO: e.g. ./gradlew test && (cd front && npm test)"; exit 1

test-slice:       ## 자기 슬라이스 범위만. SLICE=order-cancel make test-slice
	@test -n "$(SLICE)" || { echo "SLICE 를 지정하세요"; exit 1; }
	@echo "TODO: e.g. ./gradlew test --tests '*feature.$(SLICE)*'"; exit 1

test-e2e:         ## 슬라이스 완료 시에만
	@echo "TODO: e.g. npx playwright test"; exit 1

lint:             ## 슬라이스 간 import 차단 검사
	@echo "TODO: e.g. ./gradlew archTest && (cd front && npx eslint .)"; exit 1
