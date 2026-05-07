# Live Coding: A Mini Data Pipeline with Real Libraries

## Goal
Build a small, end-to-end Python project that uses **real libraries** — `requests`, `pandas`, `matplotlib`, `pytest`, plus the standard `datetime` and `logging` — to fetch data, transform it, save it, and visualize it. We'll finish by exposing it through a tiny **Flask** app to feel the difference between *library* and *framework*.

---

## Part A: Setup & Dependencies

## Step 1: Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate
```

Create `requirements.txt`:

```
requests==2.32.3
pandas==2.2.2
matplotlib==3.9.0
pytest==8.2.0
flask==3.0.3
```

Install:

```bash
pip install -r requirements.txt
```

> **Discussion:** Why pin versions? What happens to your project in 2 years if you don't?

---

## Part B: Fetching Data with `requests`

## Step 2: Build a small client around the GitHub public API

Create `github_client.py`:

```python
import requests


class GitHubClient:
    BASE_URL = "https://api.github.com"

    def __init__(self, timeout=10):
        self._session = requests.Session()
        self._timeout = timeout

    def get_user(self, username):
        url = f"{self.BASE_URL}/users/{username}"
        response = self._session.get(url, timeout=self._timeout)
        response.raise_for_status()
        return response.json()

    def list_repos(self, username):
        url = f"{self.BASE_URL}/users/{username}/repos"
        response = self._session.get(url, timeout=self._timeout, params={"per_page": 100})
        response.raise_for_status()
        return response.json()
```

Try it from a Python REPL:

```python
from github_client import GitHubClient

gh = GitHubClient()
user = gh.get_user("torvalds")
print(user["name"], "-", user["followers"], "followers")
```

### Output:
```
Linus Torvalds - 198432 followers
```

> **Discussion:** Where is the *library* in this code? Where is *your code*? Who calls whom? (Hint: your `GitHubClient` calls `requests.Session.get`. That's a library.)

---

## Part C: Reshaping Data with `pandas`

## Step 3: Turn an API response into a DataFrame

Create `repo_stats.py`:

```python
import pandas as pd
from github_client import GitHubClient


def repos_to_dataframe(repos):
    rows = [
        {
            "name": r["name"],
            "stars": r["stargazers_count"],
            "forks": r["forks_count"],
            "language": r["language"] or "unknown",
            "pushed_at": r["pushed_at"],
        }
        for r in repos
    ]
    return pd.DataFrame(rows)


def top_repos(df, n=5):
    return df.sort_values("stars", ascending=False).head(n)


def language_breakdown(df):
    return df.groupby("language").agg(
        repos=("name", "count"),
        total_stars=("stars", "sum"),
    ).sort_values("total_stars", ascending=False)
```

Try it:

```python
gh = GitHubClient()
repos = gh.list_repos("torvalds")
df = repos_to_dataframe(repos)

print(top_repos(df))
print()
print(language_breakdown(df))
```

### Output (truncated):
```
        name   stars  forks  language             pushed_at
0      linux  178432  53210         C  2026-05-07T10:21:00Z
3  subsurface    2310    690      C++  2026-05-04T07:14:00Z
...

           repos  total_stars
language
C              4       180120
C++            2         3210
Python         1          840
```

> **Discussion:** Notice we never wrote a `for` loop to compute totals. `groupby().agg()` is the *vectorized* pandas way.

---

## Part D: Plotting with `matplotlib`

## Step 4: Save a chart to disk

Add to `repo_stats.py`:

```python
import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt


def save_top_repos_chart(df, path, n=5):
    top = top_repos(df, n)
    fig, ax = plt.subplots(figsize=(8, 4.5))
    ax.barh(top["name"], top["stars"])
    ax.invert_yaxis()
    ax.set_title(f"Top {n} repos by stars")
    ax.set_xlabel("Stars")
    fig.tight_layout()
    fig.savefig(path, dpi=120)
    plt.close(fig)
```

Run it:

```python
df = repos_to_dataframe(GitHubClient().list_repos("torvalds"))
save_top_repos_chart(df, "top_repos.png")
print("saved top_repos.png")
```

> **Discussion:** Why call `plt.close(fig)`? (Hint: matplotlib leaks figures if you don't.)

---

## Part E: Standard Library — `datetime` + `logging`

## Step 5: Add lightweight logging and date parsing

Create `report.py`:

```python
import datetime
import logging

import pandas as pd

from github_client import GitHubClient
from repo_stats import repos_to_dataframe, save_top_repos_chart


logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
log = logging.getLogger("report")


def days_since_last_push(df, now=None):
    now = now or datetime.datetime.now(datetime.timezone.utc)
    pushed = pd.to_datetime(df["pushed_at"], utc=True)
    return (now - pushed).dt.days


def build_report(username):
    log.info("fetching repos for %s", username)
    gh = GitHubClient()
    df = repos_to_dataframe(gh.list_repos(username))

    df["days_since_push"] = days_since_last_push(df)

    log.info("found %d repos", len(df))
    save_top_repos_chart(df, f"{username}_top.png")
    df.to_csv(f"{username}_repos.csv", index=False)
    return df


if __name__ == "__main__":
    df = build_report("torvalds")
    print(df.head())
```

> **Discussion:** `datetime`, `logging` and `pandas` are doing very different jobs but compose cleanly because they all follow the *library* model — they don't fight over control flow.

---

## Part F: Testing with `pytest`

## Step 6: Test pure logic without hitting the network

Create `test_repo_stats.py`:

```python
import pandas as pd
from repo_stats import repos_to_dataframe, top_repos, language_breakdown


def make_repos():
    return [
        {"name": "alpha", "stargazers_count": 10, "forks_count": 2, "language": "Python", "pushed_at": "2026-01-01T00:00:00Z"},
        {"name": "beta",  "stargazers_count": 50, "forks_count": 8, "language": "Python", "pushed_at": "2026-02-01T00:00:00Z"},
        {"name": "gamma", "stargazers_count": 30, "forks_count": 5, "language": "Go",     "pushed_at": "2026-03-01T00:00:00Z"},
    ]


def test_repos_to_dataframe_has_expected_columns():
    df = repos_to_dataframe(make_repos())
    assert set(df.columns) >= {"name", "stars", "forks", "language", "pushed_at"}


def test_top_repos_returns_highest_first():
    df = repos_to_dataframe(make_repos())
    top = top_repos(df, n=2)
    assert list(top["name"]) == ["beta", "gamma"]


def test_language_breakdown_aggregates_stars():
    df = repos_to_dataframe(make_repos())
    breakdown = language_breakdown(df)
    assert breakdown.loc["Python", "total_stars"] == 60
    assert breakdown.loc["Go", "total_stars"] == 30
```

Run:

```bash
pytest -v
```

### Output:
```
test_repo_stats.py::test_repos_to_dataframe_has_expected_columns PASSED
test_repo_stats.py::test_top_repos_returns_highest_first         PASSED
test_repo_stats.py::test_language_breakdown_aggregates_stars     PASSED
```

> **Discussion:** Notice we never made a real HTTP request. We tested the *transformations* — the part of the code that has most of the bugs.

## Step 7: Mock the network

Add to `test_repo_stats.py`:

```python
from unittest.mock import patch
from github_client import GitHubClient


@patch("github_client.requests.Session.get")
def test_get_user_calls_correct_url(mock_get):
    mock_get.return_value.json.return_value = {"login": "alice", "id": 1}
    mock_get.return_value.raise_for_status.return_value = None

    gh = GitHubClient()
    user = gh.get_user("alice")

    assert user["login"] == "alice"
    mock_get.assert_called_once()
    args, kwargs = mock_get.call_args
    assert args[0].endswith("/users/alice")
```

> **Discussion:** What would happen if we ran this test without the mock and GitHub's API was down? Tests should be **deterministic**.

---

## Part G: Touching a Framework — Flask

## Step 8: Expose the report through a tiny web app

Create `app.py`:

```python
from flask import Flask, jsonify
from repo_stats import repos_to_dataframe
from github_client import GitHubClient


app = Flask(__name__)
gh = GitHubClient()


@app.route("/users/<username>/top")
def top(username):
    df = repos_to_dataframe(gh.list_repos(username))
    df = df.sort_values("stars", ascending=False).head(5)
    return jsonify(df.to_dict(orient="records"))


@app.route("/health")
def health():
    return {"status": "ok"}


if __name__ == "__main__":
    app.run(debug=True)
```

Run:

```bash
python app.py
```

Visit `http://127.0.0.1:5000/users/torvalds/top`.

> **Discussion:** Spot the **inversion of control**. You never call `top()`. Flask does — when an HTTP request matches the route. Same code, totally different control flow than the script in Step 5.

---

## Comparison: library vs. framework in *the same project*

| Aspect | `requests`, `pandas`, `matplotlib` | Flask |
|---|---|---|
| Who calls whom? | You call them | It calls you |
| Flow of control | Top-down script | Event-driven (HTTP requests) |
| What you write | Glue code | Handlers / views |
| Replaceability | Easy to swap | Migration project |
| Where the loop lives | In your `__main__` | Inside Flask |

---

## Discussion Questions

1. Why is `requests` a library while Flask is a framework, even though they're both about HTTP?
2. We use `pandas` to transform data and then plug the result into a `flask` route. Could you imagine *replacing* pandas with `numpy` or raw Python here? What would change?
3. The `GitHubClient` class wraps `requests.Session`. What design pattern is that? (Hint: it appeared in lectures 8 and 10.)
4. The `@app.route("/users/<username>/top")` decorator is **registering** the function with Flask. Which pattern from lecture 10 does that resemble?
5. Why is mocking `requests.Session.get` more useful than letting the test hit the real GitHub API?
6. If you wanted to swap Flask for FastAPI, how much of this code would change? How much of it (the `pandas`/`requests` parts) would *not*?
7. What is the right place to put a `try/except` in this pipeline? Around the network call? Around the file write? Both?
