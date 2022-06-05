# GitHub Actions 워크플로 파일 cheet sheats

## `on` 절

GitHub Actions이 어느 이벤트에서 실행할지를 선언하는 절이다.

### `paths-ignore`

배열로 파일 또는 디렉토리를 나열하여 빌드 시 포함시키지 않을 수 있다.

```yaml
name: (1_RELEASE) AR_BOOK_API
on:
  push:
    branches:
	  - 'releases/**'
	paths-ignore:
	  - '**.md'
	  - 'doc/**'
	  - '.gitignore'
	  - 'some-module/**'
```

