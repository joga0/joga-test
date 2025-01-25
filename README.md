# Gitlab Test

## Table of Contents

1. [Task - 1 List all users](#task---1-list-all-users)
   - [Introduction](#introduction)
   - [Identifying Data Sources](#identifying-data-sources)
   - [Tool Selection](#tool-selection)
   - [Script Implementation](#script-implementation)
   - [Cron Job Setup](#create-a-cron-tab-entry)
   - [Citations and Prompts](#citations-and-prompts-used-in-google-search)

2. [Task - 2 Troubleshooting Web App](#task-2-troubleshooting-web-app)
   - [Overview](#overview)
   - [Potential Causes of Slowness](#potential-causes-of-slowness)
   - [Step-by-Step Troubleshooting Plan](#step-by-step-troubleshooting-plan)
   - [Conclusion](#conclusion)
   - [Citations and Prompts](#citations-and-prompts-used-in-google-search-1)

3. [Task - 3 Git Commit Graph Reconstruction](#task-3-git-commit-graph-reconstruction)
   - [Given Git Commit Graph](#given-git-commit-graph)
   - [Step-by-Step Git Commands](#step-by-step-git-commands)
   - [Citations and Prompts](#citations-and-prompts-used-in-google-search-2)

4. [Task - 4 Write a Git Tutorial](#task-4-write-a-git-tutorial)
   - [Introduction](#introduction-1)
   - [Why Use Branches?](#why-use-branches)
   - [Step-by-Step Implementation](#step-by-step-implementation)
   - [Best Practices](#some-best-practices-would-include)
   - [Citations and Prompts](#citations-and-prompts-used-in-google-search-3)

5. [Question - 5 Book/Blog/Course](#question-5)
   - [Brief Review](#brief-review)
   - [What I Liked](#what-i-liked)
   - [What I Didn’t Like](#what-i-didnt-like)
   - [Conclusion](#conclusion-1)

## Task - 1 List all users 

### Introduction
This document outlines the development and functionality of a Bash script designed to list user accounts from a Linux system and store the MD5 output into mentioned file and to create a Crontab.

### Identifying Data Sources

To ensure a comprehensive listing of all users, research was conducted on where user data is stored:

- **Local Users**: On Linux systems, user information is stored in the `/etc/passwd` file. This file is accessible to all users and contains user ID, group ID, home directory, and shell information.

### Tool Selection

For data extraction, the following tools were chosen:

- **`awk`**: A powerful text-processing tool in Unix/Linux, used to parse the `/etc/passwd` file to extract usernames and their corresponding home directories.

### Script Implementation

1. Below is the Bash script designed to perform the user listing:

- Save the below file to list_users.sh
```bash
#!/bin/bash
# Script to list all users and their home directories

USER_LIST_FILE="/var/log/user_list.txt"

# Get user list and store in file
awk -F: '{print $1 ":" $6}' /etc/passwd > "$USER_LIST_FILE"

# Print the list to standard output as well for direct use
cat "$USER_LIST_FILE"
```

2. Script to convert MD5 and monitor changes
- Save the below file to monitor_users.sh

```bash
#!/bin/bash
# Script to track changes in user home directories

USER_LIST_FILE="/var/log/user_list.txt"
USER_LIST_HASH_FILE="/var/log/current_users"
CHANGE_LOG_FILE="/var/log/user_changes"

# Ensure the user list exists by running the list script
if [ ! -f "$USER_LIST_FILE" ]; then
    /path/to/list_users.sh
fi

# Generate MD5 hash of the stored user list
CURRENT_HASH=$(md5sum "$USER_LIST_FILE" | cut -d ' ' -f1)

# Check if the hash file exists
if [ ! -f "$USER_LIST_HASH_FILE" ]; then
    echo "$CURRENT_HASH" > "$USER_LIST_HASH_FILE"
else
    OLD_HASH=$(cat "$USER_LIST_HASH_FILE")

    if [ "$CURRENT_HASH" != "$OLD_HASH" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') changes occurred" >> "$CHANGE_LOG_FILE"
        echo "$CURRENT_HASH" > "$USER_LIST_HASH_FILE"
    fi
fi
```
3. Create a cron tab entry
- run the below commands in order to setup crontab
```bash
$ chmod +x list_users.sh monitor_users.sh

$ 0 * * * * /path/to/list_users.sh && /path/to/monitor_users.sh
```
### Citations and prompts used in google search
1. [How can I list all user names and/or home directories?](https://unix.stackexchange.com/questions/409407/how-can-i-list-all-user-names-and-or-home-directories)

2. [Print all the usernames of users as well as their home directories](https://stackoverflow.com/questions/77554387/print-all-the-usernames-of-users-as-well-as-their-home-directories)

3. [Bash script in cron job - email when a file has been modified](https://serverfault.com/questions/793677/bash-script-in-cron-job-email-when-a-file-has-been-modified)

4. [Using Crontab to Detect Changes in File and Updating It](https://www.reddit.com/r/linuxquestions/comments/mc68zo/using_crontab_to_detect_changes_in_file_and/)

## Task-2 Troubleshooting Web App

### Overview

When a user reports that a page in our web application is taking a long time to load, it is essential to systematically investigate potential causes. Given the environment details, the application follows an MVC web framework and stores data in a relational database, all hosted on a single Linux machine with:

- **8GB RAM**
- **2 CPU cores**
- **SSD storage with ample free space**

This suggests that potential bottlenecks may arise from application, database, or infrastructure constraints.

### Potential Causes of Slowness

Several factors could contribute to slow page load times. These include:

#### 1. **Database Performance Issues**
- **Possible Causes:**
  - Inefficient SQL queries (e.g., missing indexes, full table scans).
  - High query execution times due to large datasets.
  - Locking or contention issues in the database.
- **Troubleshooting Steps:**
  - Enable query logging and analyze slow queries (e.g., MySQL's `slow_query_log` or PostgreSQL's `pg_stat_statements`).
  - Run `EXPLAIN` on slow queries to check indexing and execution plans.
  - Monitor database performance using tools like `top`, `iostat`, or database-specific monitoring tools.

#### 2. **Application Code Inefficiencies**
- **Possible Causes:**
  - Inefficient business logic in the controller layer.
  - Excessive API calls, loops, or complex operations.
  - Unoptimized code leading to increased memory and CPU usage.
- **Troubleshooting Steps:**
  - Profile application performance using tools such as New Relic, Xdebug (for PHP), or Ruby’s performance profiling tools.
  - Enable logging for slow function calls and application response times.
  - Review recent code changes for potential performance regressions.

#### 3. **Web Server Configuration Issues**
- **Possible Causes:**
  - Insufficient worker processes or threading limits.
  - Suboptimal caching settings.
  - Misconfigurations in web server settings (e.g., Apache, Nginx).
- **Troubleshooting Steps:**
  - Check server logs (`/var/log/nginx/access.log` or `/var/log/apache2/access.log`) for errors and response times.
  - Use benchmarking tools like `ab` (Apache Benchmark) or `wrk` to measure server capacity.
  - Review configuration settings related to keep-alive, gzip compression, and caching.


#### 4. **Resource Bottlenecks (CPU, Memory, Disk)**
- **Possible Causes:**
  - High CPU or memory consumption leading to resource starvation.
  - Background processes consuming resources.
  - Disk I/O bottlenecks despite SSD storage.
- **Troubleshooting Steps:**
  - Monitor system resource usage using `top`, `htop`, `free -m`, and `iostat`.
  - Check for memory leaks or high swap usage.
  - Evaluate long-running processes and cron jobs that might impact performance.

#### 5. **Lack of Caching Strategies**
- **Possible Causes:**
  - Missing database query caching (e.g., Redis, Memcached).
  - No static content caching at the web server level.
  - Application-layer caching not properly configured.
- **Troubleshooting Steps:**
  - Review the caching strategies in place at the database and application levels.
  - Enable caching mechanisms such as query caching and full-page caching where applicable.
  - Check HTTP headers for cache-control directives.

#### 6. **Network Latency or External Dependencies**
- **Possible Causes:**
  - Slow API calls to third-party services.
  - High latency in internal network communications.
  - Poorly configured DNS resolution.
- **Troubleshooting Steps:**
  - Test network latency using `ping`, `traceroute`, or `curl -w`.
  - Analyze third-party API response times and implement retries or fallbacks.
  - Check DNS resolution times using `dig` or `nslookup`.

### Step-by-Step Troubleshooting Plan

1. **Gather Initial Data:**
   - Use `top` or `htop` to monitor CPU and memory usage in real-time.
   - Check the web server and database logs for any recurring errors.
   - Load the page manually while monitoring resource usage.

2. **Analyze Application Performance:**
   - Enable debug logging to track slow functions and database queries.
   - Profile the application using built-in tools or third-party monitoring.

3. **Check Web Server Load:**
   - Use tools like `ab` (Apache Benchmark) to simulate concurrent requests.
   - Analyze response time and optimize configurations if needed.

4. **Examine Database Performance:**
   - Identify slow queries with profiling tools.
   - Optimize indexes and consider query caching.

5. **Optimize Resource Allocation:**
   - Tune server parameters such as PHP's `max_execution_time` or database connection pooling.

### Conclusion

By following the above approach, it is possible to systematically identify and resolve the root causes of the slow page load. Focusing on areas such as database optimization, application profiling, server resource tuning, and caching strategies will help improve overall application performance.

### Citations and Prompts Used in Google Search

1. [Common Causes of Website Slowness](https://www.cloudflare.com/learning/performance/why-is-my-website-slow/)
2. [How to Debug Web Application Performance Issues](https://stackify.com/debugging-performance-issues/)
3. [Database Performance Tuning Guide](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
4. [Optimizing Nginx Configuration for Performance](https://www.nginx.com/blog/tuning-nginx/)


## Task-3 Gitlab-Graph

### Git Commit Graph Reconstruction

The following sequence of Git commands will recreate the given commit graph from an empty directory.

## Given Commit Graph

## Step-by-Step Git Commands

### 1. Initialize a new Git repository

```bash
git init
```

### 2. Create and commit the first commit

```bash
echo "first commit" > file.txt
git add file.txt
git commit -m "first commit"
```
### 3. Create and commit the second commit
```bash
echo "second commit" >> file.txt
git commit -am "second commit"
```

### 4. Create and commit the third commit on main
```bash
echo "third commit" >> file.txt
git commit -am "third commit"
```
### 5. Create a new branch and commit an awesome feature
```bash
git checkout -b feature-branch HEAD~1
echo "awesome feature" > feature.txt
git add feature.txt
git commit -m "awesome feature"
```

### 6. Merge Feature-branch into main
```bash
git checkout main
git merge feature-branch --no-ff -m "Merge"
```

### 7. create and commit the fourth commit on main
```bash
echo "fourth commit" >> file.txt
git commit -am "fourth commit"
```

### 8. After running the above commands, running the following command should produce the expected graph:
```bash
git log --all --decorate --oneline --graph
```

### Citations and Prompts Used in Google Search

1. [Understanding Git Commit Graphs](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

2. [How to Create and Merge Git Branches](https://www.atlassian.com/git/tutorials/using-branches)

3. [Git Merge Strategies Explained](https://www.git-scm.com/docs/git-merge)

4. [Git Commit Best Practices](https://chris.beams.io/posts/git-commit/)


## Task-4 Write a Git Tutorial 

### Using Git to Implement a New Feature Without Affecting the Main Branch

### Introduction

When working on a project, making changes directly to the `main` branch can be risky. Mistakes can break the production code, disrupt team workflows, and make it harder to track different features or bug fixes. 
To avoid these issues, **Git branching** allows you to work on new features or changes in an isolated environment without affecting the stable code in `main`.
In this tutorial, we'll walk you through the **why** and **how** of implementing a new feature using Git, ensuring a safe and efficient workflow.


### Why Use Branches?

Here’s why you should use branches when developing new features:

- **Isolation:** Keeps your new feature separate from the stable code.
- **Collaboration:** Different team members can work on features independently.
- **Version Control:** Allows you to track changes easily and roll back if needed.
- **Clean History:** Keeps the `main` branch clean with only tested and approved changes.

#### Step 1: Cloning the Repository

Before making any changes, you need to have the latest version of the repository locally.

```bash
git clone <repository_url>
cd <repository_name>
```
#### Why?
Cloning the repository downloads all the project files and version history to your local machine.

#### Step 2: Creating a New Feature Branch

Once inside the project directory, create a new branch for your feature.
```bash
git checkout -b feature/new-awesome-feature
```
#### Why?

The checkout -b command creates a new branch and switches to it in one step. The naming convention feature/new-awesome-feature helps in organizing branches logically.

#### Step 3: Making changes and tracking them

Start working on your new feature by modifying or adding files. Once you've made some changes, check their status using the following command.
```bash
git status
```
#### Why?

This command shows which files have been modified, added, or deleted and helps you track your work before committing.

To stage the chnages use the following command
```bash
git add <filename>
```
If you want to stage all changes at once use the following command
```bash
git add .
```
#### why?

The git add command stages your changes, preparing them for the next commit.

#### Step 4 : Commiting your changes

Once the changes are staged, commit them with a meaningful message using the following command
```bash
git commit -m "message you wanted to give "
```
#### Why?

Commits act as save points, allowing you to track the progress of your feature development.

#### Step 5: Pushing the featureb branch to remote repository

After commiting, push your feature branch to the remote repository so others can collaborate or review your work.
```bash
git push origin feature/new-feature
```
#### Why?

Pushing uploads your local changes to the remote repository making them accessible to the team.

#### Step 6: Creating a pull request

After pushing your branch you can open a pull request on GitLab or GitHub. A pull request lets others review your code before merging it into the main branch.

Steps needed to create a pull request

1. Go to your repository on GitLab.
2. Navigate to merge requests.
3. select your feature/new-feature branch
4. Add reviwers and description of the feature.
5.click create merge request.

#### Step 7: Merging chnages to main branch

Once your feature has been reviewed and approved, it can be merged into the main branch.

To merge your changes manually using git

1. Switch to the main branch and give the following command
```bash
git checkout main
```

2. Merge the feature branch using following
```bash
git merge feature/new-feature --no-ff
```

3. Push the updated main branch to the remote repository using the following
```bash
git push origin main
```
#### Why?

The --no-ff flag enusres that the merge creates a seperate commit preserving the history of the feature.

#### Step 8: Cleaing up

After merging its a good practice to always delete the feature branch to keep the repository clean.
```bash
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```
#### Why?

Deleting branches after merging prevents clutter and keeps the repository organized.

### Some best practices would include:

1. use descriptive branch names like following a naming convention example "feature/feature-name or bugfix/issue-name".

2. commit frequently even the small ones like meaningful commits make it easier to track chnages.

3. Always pull the latest changes before starting a new branch run git pull to ensure you are working with the latest code.

4. Always conduct code reviews to maintain code quality.

### Citations and Prompts Used in Google Search

1. [Git Branching Explained](https://www.atlassian.com/git/tutorials/using-branches)

2. [Best Practices for Feature Branches](https://www.git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)

3. [GitLab Merge Requests Guide](https://docs.gitlab.com/ee/user/project/merge_requests/)

4. [Git Push and Merge Explained](https://www.git-tower.com/learn/git/commands/git-merge)

## Question-5

### Site Reliability Engineering - How Google Runs Production Systems

**Link to the book:**  
[Google SRE](https://sre.google/sre-book/table-of-contents/)

### Brief Review

The book provides a deep dive into how Google manages large-scale, reliable, and efficient software systems.

### What I Liked

- **Practical Insights:**  
  - Covers concepts like Service Level Objectives (SLOs), error budgets, incident management, automation, capacity planning, and monitoring.  
  - Focuses on reducing manual tasks through automation and optimizing deployment, monitoring, and scaling.

- **Comprehensive Coverage:**  
  - Discusses key areas such as risk management, scalability, and collaboration between development and operations.

- **Cultural Aspects:**  
  - Emphasizes a blameless culture, fostering collaboration, and encouraging teams to take ownership of system reliability.

### What I Didn’t Like

- The book is Google-centric, making it challenging to apply directly in smaller-scale environments with different infrastructures.

### Conclusion

This book is a valuable resource for system administrators and DevOps engineers looking to implement SRE principles with practical examples and automation strategies.

