# 📈 Automated Repo Stats Tracker

A plug-and-play GitHub Action that automatically tracks your repository's traffic, stars, forks, and watchers, and updates a Markdown file every single hour. 

GitHub's built-in traffic insights only last for 14 days. This workflow ensures you always have a running log of your repository's growth without having to do any manual work.

## 🚀 How to use this in your own repository

It takes less than 2 minutes to set up. You don't need to install any external dependencies or write any code.

### Step 1: Create the Workflow File
1. In your repository, click **Add file** > **Create new file**.
2. Name the file exactly this: `.github/workflows/repo-stats.yml`

### Step 2: Paste the Code
Copy the code below and paste it into your new file, then commit the changes.

<details>
<summary><b>Click to expand the workflow code</b></summary>

```yaml
name: Track Repository Stats

on:
  schedule:
    - cron: '0 * * * *' # Runs at minute 0 past every hour
  workflow_dispatch:

permissions:
  contents: write 

jobs:
  update-stats:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Fetch Stats, Update File, and Commit
        run: |
          FOLDER="repo-stats"
          mkdir -p $FOLDER
          FILE="$FOLDER/stats.md"
          LAST_UPDATED=$(date -u +'%Y-%m-%d %H:%M:%S UTC')

          # Fetch Repo Info
          REPO_DATA=$(curl -s -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" [https://api.github.com/repos/$](https://api.github.com/repos/$){{ github.repository }})
          STARS=$(echo "$REPO_DATA" | jq -r '.stargazers_count // 0')
          FORKS=$(echo "$REPO_DATA" | jq -r '.forks_count // 0')
          WATCHERS=$(echo "$REPO_DATA" | jq -r '.subscribers_count // 0')

          # Fetch Traffic Views
          TRAFFIC_DATA=$(curl -s -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" [https://api.github.com/repos/$](https://api.github.com/repos/$){{ github.repository }}/traffic/views)
          VIEWS=$(echo "$TRAFFIC_DATA" | jq -r '.count // 0')
          UNIQUE_VIEWS=$(echo "$TRAFFIC_DATA" | jq -r '.uniques // 0')

          # Generate Markdown
          echo "# Repository Stats for ${{ github.repository }}" > "$FILE"
          echo "**⏱️ Last Updated:** $LAST_UPDATED" >> "$FILE"
          echo "" >> "$FILE"
          echo "- **⭐ Stars:** $STARS" >> "$FILE"
          echo "- **🍴 Forks:** $FORKS" >> "$FILE"
          echo "- **👀 Watchers:** $WATCHERS" >> "$FILE"
          echo "- **📈 Total Views (Last 14 days):** $VIEWS" >> "$FILE"
          echo "- **👤 Unique Visitors (Last 14 days):** $UNIQUE_VIEWS" >> "$FILE"

          # Commit and Push
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          
          git add "$FILE"
          git commit -m "docs: update repo stats - $LAST_UPDATED" || echo "No changes to commit"
          git push
