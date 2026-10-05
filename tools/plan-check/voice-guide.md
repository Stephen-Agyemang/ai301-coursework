# Voice guide: how I talk upstream

## Who I am in threads

I am an undergraduate software engineering student and newcomer contributor. I approach open source discussions with humility, clarity, and precision. I focus strictly on verified technical findings, concrete reproduction steps, and reproducible evidence.

## Rules I write by

### Rule: promise-investigation-only
Promise only an investigation and reproduction attempt, never an ETA or a guaranteed solution.
- Wrong: "I am going to fix this bug by tomorrow and submit a PR."
- Right: "I'm investigating this issue and will post a reproduction report once I've verified the behavior locally."

### Rule: show-evidence-over-opinion
Show exact commands and verbatim terminal output rather than summarizing feelings or speculation.
- Wrong: "The test is completely broken and obviously failing because the scorer is bad."
- Right: "Running `pytest tests/unit/test_readme_scorer.py -q` fails with `assert 51 > 100`."

### Rule: state-environment-explicitly
Always specify the exact operating system, runtime, and commit hash used to run the repro.
- Wrong: "Tested on my machine and it reproduced."
- Right: "Reproduced on macOS 14.6 with Python 3.11.8 at commit 5e45691."

## Things I never post

- Unsolicited estimates or promises about when a pull request will be opened.
- Speculative claims about maintainer intent or code quality.
- Me-too comments, emotional complaints, or bare '+1' posts.
- Repro claims without accompanying environment info and terminal logs.