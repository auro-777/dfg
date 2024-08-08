Here’s the updated script that includes the committer's name and email along with the commit SHA, commit date, and repository name:

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
