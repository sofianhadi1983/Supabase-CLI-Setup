# Database Migration Using GitHub Actions & Supabase CLI

## Step by step
1. Install Supabase CLI
    ``` 
    brew install supabase/tap/supabase
    ```
2. Create Supabase account and create new project
3. Create repository and inside this repo run following command
    ```
    supabase init
    ```
   this command will generate a folder called `supabase`
   ```
    your-repo/     # Root directory.
    |- supabase/   # Folder used to store supabase config and migration files.
    |- README.md   # Documentation for this project.
   ```
4. We need to link this local repository to our Supabase project that we've created.
    ```
    supabase login
    ```
   You can test if the login process successfull by typing following command:
    ```
    supabase projects list
    ```
    to show available projects. The output will show <em>Organization ID</em>, <em>Reference ID</em>, name, region, etc. <em>Reference ID</em> will be your <em>Project ID</em> and you need to pay attention on this. Since, Project ID will be required for our setup.
5. Link the Supabase CLI to desired Project ID.
    ```sh
    supabase link --project-ref {your_project_id}
    ```
    Then you will be asked to enter database password. This password should be shown when you create the project for the first time, but if you forgot, you can regenerate password through Supabase dashboard.
6. Create GitHub repository and push the supabase project to this repo
    ```sh
    git init
    git add .
    git commit -am "Initial commit"
    git remote add origin https://github.com/yourusername/yourrepo
    git branch -M main
    git push -u origin main

    # if error since remote repo has init files
    git branch --set-upstream-to=origin/main main
    git config pull.rebase true
    git pull
    git push
    ```
7. Start Supabase. Ensure you have Docker.
    ```sh
    supabase start
    ```
    you will see output like following:
    ```sh
    Started supabase local development setup.

            API URL : http://127.0.0.1:54321
        GraphQL URL : http://127.0.0.1:54321/graphql/v1
    S3 Storage URL  : http://127.0.0.1:54321/storage/v1/s3
            DB URL  : postgresql://postgres:postgres@127.0.0.1:54322/postgres
        Studio URL  : http://127.0.0.1:54323
        Inbucket URL: http://127.0.0.1:54324
        JWT secret  : super-secret-jwt-token-with-at-least-32-characters-long
            anon key: eyJhbGciOiJ2IUzI1NiI2sInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0
    service_role key: eyJhbGciOiJIUzI1NdiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImV4cCI6MTk4MzgxMjk5Nn0.EGIM96RAZx35lJzdJsyH-qQwv8Hdp7fsn3W0YpN81IU
    S3 Access Key   : 625729a08b95bf1b7fef351a663f3a23c
    S3 Secret Key   : 850181e4652dd023b7a98c58ae0d2d34bd487ee0cc3254aed6eda37307425907
        S3 Region   : local
    ```
    Now, you can access your Supabase dashboard using this link: `http://127.0.0.1:54323/`. This is connected to our remote Supabase project.

## Migration

### Local Migration
1. Create migration file
    ```sh
    supabase migration new new_employee
    ```
   This will generate blank sql file located in `supabase/migrations` folder.
2. Open this file and create query, for example:
    ```sql
    create table public.employees (
        id integer primary key generated always as identity,
        name text
    )
    ```
3. Apply the migration script to our local database
    ```sh
    supabase db reset
    ```
    the output:
    ```sh
    Resetting local database...
    Recreating database...
    Initialising schema...
    Seeding globals from roles.sql...
    Applying migration 20250321164102_new_employee.sql...
    WARN: no files matched pattern: supabase/seed.sql
    Restarting containers...
    Finished supabase db reset on branch main
    ```
    Basically, this command will recreate our database from scratch and applying our migration scripts under `migrations` folder.


### Remote Migration
We create two project in Supabase, for `Production` and `Staging`.
The following secret needs to be configured in your GitHub repository:
+ `SUPABASE_ACCESS_TOKEN` 
+ `PRODUCTION_DB_PASSWORD`
+ `PRODUCTION_PROJECT_ID`
+ `STAGING_DB_PASSWORD`
+ `STAGING_PROJECT_ID`

#### How to Get Your Supabase Access Token

To find your Supabase Access Token, follow these steps:

1. **Log in to your Supabase account** at [https://app.supabase.com](https://app.supabase.com)

2. **Navigate to Account Settings**:
   - Click on your profile icon in the bottom-left corner
   - Select "Account" from the menu

3. **Access API Settings**:
   - In the account settings page, click on "Access Tokens" in the left sidebar

4. **Generate a new token**:
   - Click on "Generate New Token"
   - Give your token a descriptive name (e.g., "GitHub Actions")
   - Set the appropriate permissions for your token
   - Click "Generate"

5. **Copy your token**:
   - The token will be displayed only once, so make sure to copy it immediately
   - Store it securely as it provides access to your Supabase resources

6. **Add the token to your GitHub repository secrets**:
   - Go to your GitHub repository
   - Navigate to "Settings" > "Secrets and variables" > "Actions"
   - Click "New repository secret"
   - Name: `SUPABASE_ACCESS_TOKEN`
   - Value: Paste the token you copied
   - Click "Add secret"

**Important**: Treat your access token like a password. Never commit it to your repository or share it publicly. If you suspect your token has been compromised, immediately revoke it in the Supabase dashboard and generate a new one.

#### How to Set Your Supabase Database Password

The database password is a sensitive piece of information that you need to set as a GitHub secret for your workflow. Here's how to handle it:

Currently, we're working with an existing Supabase project, so we should already have a database password. This is the password we set when we created the project.

1. **Retrieve your existing password**:
   - If you don't remember the password, you may need to reset it through the Supabase dashboard
   - Go to your project in the Supabase dashboard
   - Navigate to "Project Settings" > "Database"
   - You may be able to view or reset your database password here

2. **Add it to your GitHub repository secrets**:
   - Go to your GitHub repository
   - Navigate to "Settings" > "Secrets and variables" > "Actions"
   - Click "New repository secret"
   - Name: `SUPABASE_DB_PASSWORD`
   - Value: Your database password
   - Click "Add secret"

**Important**: Never commit your database password to your repository or share it publicly. If you suspect your password has been compromised, immediately change it in the Supabase dashboard.

#### How to Get Your Supabase Project ID

To find your Supabase Project ID, follow these steps:

1. **Log in to your Supabase account** at [https://app.supabase.com](https://app.supabase.com)

2. **Navigate to your project**:
   - After logging in, you'll see a list of your projects
   - Click on the project you want to use with GitHub Actions

3. **Access Project Settings**:
   - In the project dashboard, click on the gear icon (⚙️) in the left sidebar to access project settings
   - Select "General" if it's not already selected

4. **Find your Project ID**:
   - Look for "Reference ID" or "Project ID" in the settings page
   - It will be a string that looks something like: `abcdefghijklmnopqrst`

5. **Copy this value** and add it to your GitHub repository secrets:
   - Go to your GitHub repository
   - Navigate to "Settings" > "Secrets and variables" > "Actions"
   - Click "New repository secret"
   - Name: `PRODUCTION_PROJECT_ID`
   - Value: Paste the project ID you copied
   - Click "Add secret"


#### Setup CI/CD
1. **Create folder** `.github/workflows` in the root folder.
    ```sh
    mkdir -p .github
    mkdir -p .github/workflows
    ```
2. **Copy 3 files** `ci.yml`, `production.yaml`, `staging.yaml` from [https://github.com/supabase/supabase-action-example/blob/main/.github/workflows/](https://github.com/supabase/supabase-action-example/blob/main/.github/workflows/)
    ```sh
    cd .github/workflows
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/.github/workflows/ci.yaml
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/.github/workflows/production.yaml
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/.github/workflows/staging.yaml
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/.github/workflows/preview.yaml
    ```
3. **Create folder called `remotes`** and fetch `terraform` config  from [https://github.com/supabase/supabase-action-example/blob/main/supabase/remotes/](https://github.com/supabase/supabase-action-example/blob/main/supabase/remotes/)
    ```sh
    mkdir -p supabase/remotes
    cd supabase/remotes
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/supabase/remotes/preview.tf
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/supabase/remotes/production.tf
    curl -O https://raw.githubusercontent.com/supabase/supabase-action-example/refs/heads/main/supabase/remotes/provider.tf
    ```
## References

- [Supabase Doc](https://supabase.com/docs)