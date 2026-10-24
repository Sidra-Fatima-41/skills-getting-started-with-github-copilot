# Project Guidelines

## Architecture

- This is a small FastAPI application for the Mergington High School activities exercise. The backend entry point is [`src/app.py`](../src/app.py), and the ASGI application is exposed as `src.app:app`.
- Activity and participant data live in the module-level `activities` dictionary in [`src/app.py`](../src/app.py). It is intentionally in memory, so changes are lost when the server restarts; do not introduce persistence unless the task explicitly requires it.
- `GET /` redirects to the static frontend at `/static/index.html`. The frontend files are [`src/static/index.html`](../src/static/index.html), [`src/static/app.js`](../src/static/app.js), and [`src/static/styles.css`](../src/static/styles.css).
- The API owns activity validation and registration state. The frontend uses `fetch` for `/activities` and the activity signup endpoints, then reloads the activity list after mutations so cards and participant lists stay synchronized.
- The GitHub Actions files under [`.github/workflows`](../.github/workflows) run the staged GitHub Skills exercise. The Markdown files under [`.github/steps`](../.github/steps) are learner-facing exercise content; preserve their workflow integration when changing application code.

## Build and Test

Install dependencies from the repository root:

```bash
pip install -r requirements.txt
```

Run the development server from the repository root:

```bash
python -m uvicorn src.app:app --reload --reload-include 'src/static/*'
```

This matches the launch configuration in [`.vscode/launch.json`](../.vscode/launch.json). The API is available at `http://localhost:8000`, with interactive documentation at `/docs` and `/redoc`.

The repository is configured for pytest in [`pytest.ini`](../pytest.ini). Run the full suite with:

```bash
pytest
```

Run one test file or one test function with:

```bash
pytest tests/test_app.py
pytest tests/test_app.py::test_name
```

There is currently no checked-in test suite, build script, or lint configuration. If tests are added, keep them in a top-level `tests/` directory and include pytest in the project dependencies.

## Conventions

- Keep API route behavior and error responses explicit: unknown activities return `404`, duplicate signups return `400`, and unregistering a nonparticipant returns `404`, using FastAPI `HTTPException`.
- Activity names are the API resource identifiers and may contain spaces. Client-side URLs must use `encodeURIComponent` when inserting activity names or email addresses into endpoint paths or query strings.
- Keep the activity record shape consistent: each activity has `description`, `schedule`, `max_participants`, and a `participants` list of email strings. The API currently returns this dictionary directly rather than using Pydantic response models.
- Frontend mutations should update the UI by calling `fetchActivities()` after a successful signup or unregister operation. Use DOM text APIs for participant email values and preserve the existing success/error message behavior.
- Follow the existing simple, functional style: route handlers are plain functions, frontend behavior is initialized from `DOMContentLoaded`, and static assets are served by FastAPI rather than a separate frontend build system.

## MCP

- Playwright MCP is the most relevant optional integration for this project because the application includes a browser UI and signup flows. Prefer configuring it through the developer's VS Code MCP settings so credentials and local server choices are not committed to the repository.
