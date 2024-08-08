Great! Now that the script works for a single repository, I'll modify it to work at the organization level for all repositories. The script will fetch commits from all repositories in your GitHub organization for the year 2024 and include the repository name, commit SHA, commit date, committer's name, and committer's email.

Here’s the updated script:

```python
import requests
import csv
import time

# Replace these with your own values
GITHUB_TOKEN = 'your_personal_access_token'
ORG_NAME = 'your_organization_name'

headers = {
    'Authorization': f'token {GITHUB_TOKEN}'
}

def check_rate_limit():
    rate_limit_url = 'https://api.github.com/rate_limit'
    response = requests.get(rate_limit_url, headers=headers)
    data = response.json()
    remaining = data['rate']['remaining']
    reset_time = data['rate']['reset']
    return remaining, reset_time

# GitHub API endpoint to list all repositories in an organization
url = f'https://api.github.com/orgs/{ORG_NAME}/repos'

repositories = []
page = 1

# Fetch all repositories
while True:
    remaining, reset_time = check_rate_limit()
    if remaining == 0:
        wait_time = reset_time - time.time() + 1
        print(f"Rate limit exceeded. Waiting for {wait_time} seconds.")
        time.sleep(wait_time)
        
    response = requests.get(url, headers=headers, params={'per_page': 100, 'page': page})
    if response.status_code != 200:
        print(f"Error fetching repositories: {response.status_code}")
        break
    data = response.json()
    if not data:
        break
    repositories.extend(data)
    page += 1

# Define the CSV file path
csv_file_path = 'c:\\sre\\github_commits_2024.csv'

# Define the CSV headers
csv_headers = ['Repository Name', 'Commit SHA', 'Commit Date', 'Committer Name', 'Committer Email']

# Write data to CSV
with open(csv_file_path, mode='w', newline='', encoding='utf-8') as csv_file:
    writer = csv.DictWriter(csv_file, fieldnames=csv_headers)
    writer.writeheader()

    for repo in repositories:
        repo_name = repo['name']
        commits_url = f'https://api.github.com/repos/{ORG_NAME}/{repo_name}/commits'
        
        # Fetch commits made in 2024
        params = {
            'since': '2024-01-01T00:00:00Z',
            'until': '2024-12-31T23:59:59Z',
            'per_page': 100
        }
        
        commit_page = 1
        while True:
            remaining, reset_time = check_rate_limit()
            if remaining == 0:
                wait_time = reset_time - time.time() + 1
                print(f"Rate limit exceeded. Waiting for {wait_time} seconds.")
                time.sleep(wait_time)

            commits_response = requests.get(commits_url, headers=headers, params={**params, 'page': commit_page})
            if commits_response.status_code == 403:
                print(f"Error fetching commits for {repo_name}: {commits_response.status_code}. Skipping repository.")
                break
            elif commits_response.status_code != 200:
                print(f"Error fetching commits for {repo_name}: {commits_response.status_code}")
                break
            commits = commits_response.json()
            if not commits:
                break
            
            for commit in commits:
                writer.writerow({
                    'Repository Name': repo_name,
                    'Commit SHA': commit['sha'],
                    'Commit Date': commit['commit']['committer']['date'],
                    'Committer Name': commit['commit']['committer']['name'],
                    'Committer Email': commit['commit']['committer']['email']
                })
            
            commit_page += 1

print(f"Report generated: {csv_file_path}")
```

### Script Overview:
1. **Repository Fetching**: It retrieves all repositories under the specified organization.
2. **Commit Fetching**: For each repository, it fetches commits made in the year 2024.
3. **CSV Output**: The commit data (repository name, commit SHA, commit date, committer's name, and committer's email) is written to a CSV file.

### Error Handling:
- **Rate Limiting**: The script checks the rate limit before making API requests and pauses if the limit is reached.
- **403 Errors**: If a repository cannot be accessed due to a 403 error, the script logs the error and skips that repository.

### CSV Headers:
The output CSV will include:
- Repository Name
- Commit SHA
- Commit Date
- Committer Name
- Committer Email

### Running the Script:
- **GITHUB_TOKEN**: Replace with your personal access token.
- **ORG_NAME**: Replace with your GitHub organization name.
- **File Path**: The CSV file is saved to `c:\\sre\\github_commits_2024.csv`. Adjust the path if necessary.

This script will generate a complete report of all commits made in 2024 across all repositories in your organization.

Let me know if you have any questions or need further adjustments!Here’s the updated script that includes the committer's name and email along with the commit SHA, commit date, and repository name:

```python
import requests
import csv

# Replace these with your own values
GITHUB_TOKEN = 'your_personal_access_token'
REPO_NAME = 'your_repository_name'
ORG_NAME = 'your_organization_name'

headers = {
    'Authorization': f'token {GITHUB_TOKEN}'
}

# Define the API endpoint for the specific repository
commits_url = f'https://api.github.com/repos/{ORG_NAME}/{REPO_NAME}/commits'

# Define the CSV file path
csv_file_path = 'c:\\sre\\github_commits_2024.csv'

# Define the CSV headers
csv_headers = ['Repository Name', 'Commit SHA', 'Commit Date', 'Committer Name', 'Committer Email']

# Write data to CSV
with open(csv_file_path, mode='w', newline='', encoding='utf-8') as csv_file:
    writer = csv.DictWriter(csv_file, fieldnames=csv_headers)
    writer.writeheader()

    # Fetch commits made in 2024
    params = {
        'since': '2024-01-01T00:00:00Z',
        'until': '2024-12-31T23:59:59Z',
        'per_page': 100
    }
    
    commit_page = 1
    while True:
        commits_response = requests.get(commits_url, headers=headers, params={**params, 'page': commit_page})
        if commits_response.status_code == 403:
            print(f"Error fetching commits for {REPO_NAME}: {commits_response.status_code}.")
            break
        elif commits_response.status_code != 200:
            print(f"Error fetching commits for {REPO_NAME}: {commits_response.status_code}")
            break
        commits = commits_response.json()
        if not commits:
            break
        
        for commit in commits:
            writer.writerow({
                'Repository Name': REPO_NAME,
                'Commit SHA': commit['sha'],
                'Commit Date': commit['commit']['committer']['date'],
                'Committer Name': commit['commit']['committer']['name'],
                'Committer Email': commit['commit']['committer']['email']
            })
        
        commit_page += 1

print(f"Report generated: {csv_file_path}")
```

### Explanation of Added Fields:
- **Committer Name**: Added `commit['commit']['committer']['name']`.
- **Committer Email**: Added `commit['commit']['committer']['email']`.

### What This Script Does:
- It fetches commits from the specified repository for the year 2024.
- It extracts the commit SHA, commit date, committer's name, and committer's email.
- The extracted data is written to a CSV file.

### CSV Headers:
The CSV will now have the following columns:
- Repository Name
- Commit SHA
- Commit Date
- Committer Name
- Committer Email

Let me know if this covers everything you need!
