+++
title = "Updating Goyo Theme"
description = "Learn how to update the Goyo theme to the latest version."
weight = 2
sort_by = "weight"

[extra]
+++

Keeping your Goyo theme up to date ensures you have the latest features, improvements, and bug fixes. This guide covers different methods for updating the theme.

## Manual Update Methods

### If you cloned the theme

If you installed Goyo by cloning the repository directly, you can update it with:

```bash
cd themes/goyo
git pull origin main
```

This will fetch and merge the latest changes from the main branch.

### If you added the theme as a submodule

If you installed Goyo as a git submodule, update it with:

```bash
git submodule update --remote themes/goyo
```

Or for a more comprehensive update of all submodules:

```bash
git submodule sync
git submodule update --remote
```

After updating the submodule, commit the changes to your repository:

```bash
git add themes/goyo
git commit -m "Update Goyo theme to latest version"
git push
```

## Automated Updates with GitHub Actions

For projects hosted on GitHub, you can automate theme updates using GitHub Actions. This will create periodic pull requests whenever a new version of Goyo is available.

### Step 1: Create the Workflow File

Create a file at `.github/workflows/update-goyo-theme.yml` in your documentation repository:

```yaml
name: Update Goyo Theme

on:
  schedule:
    # Run every Monday at 9:00 AM UTC
    - cron: '0 9 * * 1'
  workflow_dispatch: # Allow manual trigger

jobs:
  update-theme:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: true
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Update Goyo submodule
        id: update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          
          # Get current commit hash
          cd themes/goyo
          OLD_COMMIT=$(git rev-parse HEAD)
          cd ../..
          
          # Update submodule to latest
          git submodule update --remote themes/goyo
          
          # Get new commit hash
          cd themes/goyo
          NEW_COMMIT=$(git rev-parse HEAD)
          cd ../..
          
          # Check if there are changes
          if [ "$OLD_COMMIT" != "$NEW_COMMIT" ]; then
            echo "updated=true" >> $GITHUB_OUTPUT
            echo "old_commit=$OLD_COMMIT" >> $GITHUB_OUTPUT
            echo "new_commit=$NEW_COMMIT" >> $GITHUB_OUTPUT
          else
            echo "updated=false" >> $GITHUB_OUTPUT
          fi
      
      - name: Create Pull Request
        if: steps.update.outputs.updated == 'true'
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "Update Goyo theme to latest version"
          title: "Update Goyo theme"
          body: |
            This PR updates the Goyo theme to the latest version.
            
            **Changes:** ${{ steps.update.outputs.old_commit }} → ${{ steps.update.outputs.new_commit }}
            
            Please review the [Goyo changelog](https://github.com/hahwul/goyo/releases) for details on what's new.
            
            ---
            *This PR was automatically created by the Update Goyo Theme workflow.*
          branch: update-goyo-theme
          delete-branch: true
          labels: dependencies, documentation
```

### Step 2: Customize the Schedule

The workflow is set to run every Monday at 9:00 AM UTC by default. You can customize the schedule by modifying the `cron` expression:

- **Daily**: `'0 9 * * *'` - Runs at 9:00 AM UTC every day
- **Weekly (Monday)**: `'0 9 * * 1'` - Runs at 9:00 AM UTC every Monday
- **Monthly**: `'0 9 1 * *'` - Runs at 9:00 AM UTC on the 1st of each month

You can also manually trigger the workflow anytime from the Actions tab in your repository.

### Step 3: Enable Actions

1. Go to your repository on GitHub
2. Navigate to the **Actions** tab
3. If Actions are disabled, click "I understand my workflows, go ahead and enable them"

### Step 4: Monitor Updates

Once configured, the workflow will:
- Automatically check for Goyo theme updates on the scheduled time
- Create a pull request if updates are available
- Include commit hash information in the PR description

You can review the changes in the pull request and merge when ready.

## Checking for Updates Manually

To check if updates are available without pulling them:

```bash
# For cloned repositories
cd themes/goyo
git fetch origin
git log HEAD..origin/main --oneline

# For submodules
git submodule update --remote --dry-run themes/goyo
```

## Troubleshooting

### Merge Conflicts

If you've made local modifications to the theme and encounter conflicts during updates:

1. **For cloned repositories**: Resolve conflicts manually or reset to the latest version:
   ```bash
   cd themes/goyo
   git stash  # Save local changes
   git pull origin main
   git stash pop  # Reapply local changes
   ```

2. **For submodules**: Consider using a fork instead of direct modifications

### Permission Issues

If the GitHub Actions workflow fails with permission errors, ensure:
- The workflow has `contents: write` and `pull-requests: write` permissions
- Your repository settings allow Actions to create pull requests (Settings → Actions → General → Workflow permissions)

## Best Practices

- **Test updates locally** before deploying to production
- **Review the changelog** to understand what changed
- **Keep your theme as a submodule** for easier updates
- **Don't modify theme files directly** - use configuration options or custom CSS instead
- **Monitor the workflow** to catch update failures early
