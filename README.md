# The-Workflow-File
2. The Workflow File (.github/workflows/daily-agent-sync.yml)​This runs every morning at 06:00 UTC (or manually anytime via the Run workflow button in GitHub's UI), installs the current Google GenAI SDK, executes your Python script, and commits the output to a staging branch so nothing hits main without your review.
