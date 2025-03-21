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
    ```
    supabase link --project-ref {your_project_id}
    ```
    Then you will be asked to enter database password. This password should be shown when you create the project for the first time, but if you forgot, you can regenerate password through Supabase dashboard.
6. Create GitHub repository and push the supabase project to this repo
    ```
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
    ```
    supabase start
    ```
    you will see output like following:
    ```
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
    ```
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
    ```
    supabase db reset
    ```
    the output:
    ```
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
1. Create GitHub Actions > Simple workflow

### Installing

In order to use this makefile you will need to make sure that the following
dependencies are installed on your system:
  - GNU make
  - Pandoc
  - LuaLaTeX
  - DejaVu Sans fonts

### Folder structure

Here's a folder structure for a Pandoc document:

```
my-document/     # Root directory.
|- build/        # Folder used to store builded (output) files.
|- src/          # Markdowns files; one for each chapter.
|- images/       # Images folder.
|- metadata.yml  # Metadata content (title, author...).
|- Makefile      # Makefile used for building our documents.
```

### Setup generic data

Edit the *metadata.yml* file to set configuration data:

```yml
---
title: My document title
author: Ralph Huwiler
rights:  Creative Commons Attribution 4.0 International
language: en-US
tags: [document, my-document, etc]
abstract: |
  Your summary text.
---
```

You can find the list of all available keys on [this
page](http://pandoc.org/MANUAL.html#extension-yaml_metadata_block).

### Creating chapters

Creating a new chapter is as simple as creating a new markdown file in the
*src/* folder; you'll end up with something like this:

```
src/01-introduction.md
src/02-installation.md
src/03-usage.md
src/04-references.md
```

Pandoc and Make will join them automatically ordered by name; that's why the
numeric prefixes are being used.

All you need to specify for each chapter at least one title:

```md
# Introduction

This is the first paragraph of the introduction chapter.

## First

This is the first subsection.

## Second

This is the second subsection.
```

Each title (*#*) will represent a chapter, while each subtitle (*##*) will
represent a chapter's section. You can use as many levels of sections as
markdown supports.

#### Links between chapters

Anchor links can be used to link chapters within the document:

```md
// src/01-introduction.md
# Introduction

For more information, check the [Usage] chapter.

// src/02-installation.md
# Usage

...
```

If you want to rename the reference, use this syntax:

```md
For more information, check [this](#usage) chapter.
```

Anchor names should be downcased, and spaces, colons, semicolons... should be
replaced with hyphens. Instead of `Chapter title: A new era`, you have:
`#chapter-title-a-new-era`.

#### Links between sections

It's the same as anchor links:

```md
# Introduction

## First

For more information, check the [Second] section.

## Second

...
```

Or, with al alternative name:

```md
For more information, check [this](#second) section.
```

### Inserting objects

Text. That's cool. What about images and tables?

#### Insert an image

Use Markdown syntax to insert an image with a caption:

```md
![A cool seagull.](images/seagull.png)
```

Pandoc will automatically convert the image into a figure (image + caption).

If you want to resize the image, you may use this syntax, available in Pandoc
1.16:

```md
![A cool seagull.](images/seagull.png){ width=50% height=50% }
```

Also, to reference an image, use LaTeX labels:

```md
Please, admire the gloriousnes of Figure \ref{seagull_image}.

![A cool seagull.\label{seagull_image}](images/seagull.png)
```

#### Insert a table

Use markdown table, and use the `Table: <Your table description>` syntax to add
a caption:

```md
| Index | Name |
| ----- | ---- |
| 0     | AAA  |
| 1     | BBB  |
| ...   | ...  |

Table: This is an example table.
```

If you want to reference a table, use LaTeX labels:

```md
Please, check Table /ref{example_table}.

| Index | Name |
| ----- | ---- |
| 0     | AAA  |
| 1     | BBB  |
| ...   | ...  |

Table: This is an example table.\label{example_table}
```

#### Insert an equation

Wrap a LaTeX math equation between `$` delimiters for inline (tiny) formulas:

```md
This, $\mu = \sum_{i=0}^{N} \frac{x_i}{N}$, the mean equation, ...
```

Pandoc will transform them automatically into images using online services.

If you want to center the equation instead of inlining it, use double `$$`
delimiters:

```md
$$\mu = \sum_{i=0}^{N} \frac{x_i}{N}$$
```

[Here](https://www.codecogs.com/latex/eqneditor.php)'s an online equation
editor.

### Output

This template uses *Makefile* to automatize the building process. Instead of
using the *pandoc cli util*, we're going to use some *make* commands.

#### Export to PDF

Use this command:

```sh
make pdf
```

The generated file will be placed in *build/pdf*.

Please, note that PDF file generation requires some extra dependencies (~ 800
MB):

```sh
sudo apt-get install texlive-latex-base texlive-fonts-recommended texlive-latex-extra 
```

#### Export to EPUB

Use this command:

```sh
make epub
```

The generated file will be placed in *build/epub*.

#### Export to HTML

Use this command:

```sh
make html
```

The generated file(s) will be placed in *build/html*.

## References

- [Pandoc](http://pandoc.org/)
- [Pandoc Manual](http://pandoc.org/MANUAL.html)
- [Wikipedia: Markdown](http://wikipedia.org/wiki/Markdown)