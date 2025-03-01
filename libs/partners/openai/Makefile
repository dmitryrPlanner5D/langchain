.PHONY: all format lint test tests integration_tests docker_tests help extended_tests

# Default target executed when no arguments are given to make.
all: help

# Define a variable for the test file path.
TEST_FILE ?= tests/unit_tests/

integration_test integration_tests: TEST_FILE=tests/integration_tests/

# unit tests are run with the --disable-socket flag to prevent network calls
# use tiktoken cache to enable token counting without socket (internet) access
test tests:
	mkdir -p tiktoken_cache
	@if [ ! -f tiktoken_cache/9b5ad71b2ce5302211f9c61530b329a4922fc6a4 ]; then \
		curl -o tiktoken_cache/9b5ad71b2ce5302211f9c61530b329a4922fc6a4 https://openaipublic.blob.core.windows.net/encodings/cl100k_base.tiktoken; \
	fi
	@if [ ! -f tiktoken_cache/fb374d419588a4632f3f557e76b4b70aebbca790 ]; then \
		curl -o tiktoken_cache/fb374d419588a4632f3f557e76b4b70aebbca790 https://openaipublic.blob.core.windows.net/encodings/o200k_base.tiktoken; \
	fi
	TIKTOKEN_CACHE_DIR=tiktoken_cache poetry run pytest --disable-socket --allow-unix-socket $(TEST_FILE)

integration_test integration_tests:
	poetry run pytest -n auto $(TEST_FILE)

test_watch:
	poetry run ptw --snapshot-update --now . -- -vv $(TEST_FILE)


######################
# LINTING AND FORMATTING
######################

# Define a variable for Python and notebook files.
PYTHON_FILES=.
MYPY_CACHE=.mypy_cache
lint format: PYTHON_FILES=.
lint_diff format_diff: PYTHON_FILES=$(shell git diff --relative=libs/partners/openai --name-only --diff-filter=d master | grep -E '\.py$$|\.ipynb$$')
lint_package: PYTHON_FILES=langchain_openai
lint_tests: PYTHON_FILES=tests
lint_tests: MYPY_CACHE=.mypy_cache_test

lint lint_diff lint_package lint_tests:
	[ "$(PYTHON_FILES)" = "" ] || poetry run ruff check $(PYTHON_FILES)
	[ "$(PYTHON_FILES)" = "" ] || poetry run ruff format $(PYTHON_FILES) --diff
	[ "$(PYTHON_FILES)" = "" ] || mkdir -p $(MYPY_CACHE) && poetry run mypy $(PYTHON_FILES) --cache-dir $(MYPY_CACHE)

format format_diff:
	[ "$(PYTHON_FILES)" = "" ] || poetry run ruff format $(PYTHON_FILES)
	[ "$(PYTHON_FILES)" = "" ] || poetry run ruff check --select I --fix $(PYTHON_FILES)

spell_check:
	poetry run codespell --toml pyproject.toml

spell_fix:
	poetry run codespell --toml pyproject.toml -w

check_imports: $(shell find langchain_openai -name '*.py')
	poetry run python ./scripts/check_imports.py $^

######################
# HELP
######################

help:
	@echo '----'
	@echo 'check_imports				- check imports'
	@echo 'format                       - run code formatters'
	@echo 'lint                         - run linters'
	@echo 'test                         - run unit tests'
	@echo 'tests                        - run unit tests'
	@echo 'test TEST_FILE=<test_file>   - run all tests in file'
