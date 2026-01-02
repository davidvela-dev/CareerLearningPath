# Learning Path – Software Engineering
This repository documents my learning path throughout my studies and personal growth in Software Engineering.  
It includes documentation, exercises, and complete projects covering both academic foundations and practical software development areas.

Software Engineering offers a wide range of professional paths. During my studies, I often found myself uncertain about which area to specialize in.  
This repository is an attempt to organize that journey and provide a structured overview of the field.

The goal of this work is:
- To serve as a personal learning record and concepts organized from an experienced perspective
- To act as a guide for students entering the software field, helping them understand core concepts and explore different areas before choosing a specialization

<br>

## Repository Structure

### CAREER_FUNDAMENTALS/
Covers the main areas and concepts studied during the degree.  
These topics provide a broad perspective of software engineering and help identify potential specialization paths.

### CORE_PROGRAMMING/
Contains resources, tools, and techniques that are used across multiple programming languages and software areas.  
Although many of these topics are covered during the degree, they are sometimes scattered in topics.

### SPECIALIZATION/
This section is reserved for future in-depth work focused on my professional path.

<br>

## Repository Settings (Only for developing)

### Introduction
If you are interested in using this structure for personal use checkout the following settings:

We manage all of our resources from our local repository, but also we can handle documentation from **Google Drive**. In our repo all of the documents come from the same route **NOTES/**, wherever their location is. So we have a bidirectional transfer of the documentation from the local repository and drive, using two commands. Meanwhile in your local repository you can work with scripts or complete projects.

To accomplish this we use 2 applications **Rclone** to manage cloud files from the terminal and **Pandoc** to manage google documents in different formats. This permits us to make a transference between formats from our repository or our drive notes.

Our local project is the center of our system and here we have our private variables, the script with our commands and our secret variables with the specified routes to our root files.

### Requirements
- **IDE**: to work with your local project or repository (VSCode)
- **Bash:** the command interpreter for macOS or Linux (for windows you need to have git bash or WSL)
- **RClone:** to manage google drive documents
- **Pandoc:** to manage formats like markdown or docx

<hr>

### Project Setup

In order for the transference of documents to work, you need to have these two things:

- Your local project and your drive folder have exactly the same structure for subjects, topics, etc

- The documents you want to be transfered in your local repository need to be inside the folder with the same name (**NOTES** or the defined in **.env**)

<br>

**Local Project:**   
Documentation is managed in markdown format so your remote repository can be shared, every document is inside **NOTES/** while scripts and other resources are inside **PROJECTS/** or **EXERCICES/**, these folders will be ignored when they are defined in the .env. So they won't be loaded to google drive

```
ROOT/
    sync_notes.sh
    .gitignore
    .env
    .drive_sync/
    SUBJECT1/
        TOPIC1/
            NOTES/
                topic1.md
            PROJECTS/
                Backend/
            EXERCICES/
                exercice1.py
        TOPIC2/
            subtopic/
                NOTES/
                    example.md
    SUBJECT2/
```

**Google Drive:**   
The documents are loaded without the folder **NOTES/** since here you only manage documentation. The format is **.docx**, but in order to be transformed to **markdown** correctly you need to use the text format and headdings in google drive.

```
ROOT/
    SUBJECT1/
        TOPIC1/
            topic1.docx
        TOPIC2/
            subtopic/
                example.docx
    SUBJECT2/
```

<hr>

### Git bash Setup (for Windows)

Download git bash: https://git-scm.com/install/windows

Verify instalation:  
```git --version```

<hr>

### Rclone Setup
Download rclone: https://rclone.org/downloads/

Verify instalation:     
```rclone --version```

<br>

**In the terminal put the following commands:**

Setup drive account connection from terminal     
```rclone config```   
```new remote:             n```  
```name:                   gdrive```         
```type:                   drive```     
```client_id:              enter```  
```client_secret:          enter```  
```scope access:           1 full access```  
```service_account_file:   enter```  
```advanced config:        n```  
```web browser:            y```   


Authenticate google drive account   
``` shared drive:           n```    
``` keep gdrive remote:     y```    
``` quit config:            q``` 

Verify access to files  
```rclone ls gdrive```

<hr>

### Pandoc Setup
Download pandoc: https://github.com/jgm/pandoc/releases/tag/3.8.3

Verify instalation:  
```pandoc --version```

<hr> 

### Script Setup

Inside the root of our repository we need to create the files .env, and sync_notes.hs, the bash script. In our variables we will have the protected routes to our project root file in google drive and in the local repository.

```
ROOT/
    sync_notes.sh
    .gitignore
    .env
    .drive_sync/
```


**Commands**    
From our repository to google docs  
(.md → pandoc → .docx → rclone → Google Docs):  
- Windows: ```bash .\sync_notes.sh -DRIVE```
- MacOS or Linux: ```.\sync_notes.sh -DRIVE```

From google docs to our repository  
(Google Docs → rclone → .docx → Pandoc → .md):  
- Windows: ```bash .\sync_notes.sh -REPO```
- MacOS or Linux: ```.\sync_notes.sh -REPO```

**.env**  
```DRIVE_REMOTE=gdrive``` (name given in rclone configuration)   
```DRIVE_ROOT=ROOT/NOTES/CarreerLearningPath``` (root folder of your google drive notes)    
```PANDOC_PATH=C:/Users/Applications/Pandoc/pandoc.exe``` (path of pandoc executable)   
```NOTES_FOLDER=NOTES``` (folders that will have documents transference)     
```IGNORED_FOLDERS=EXERCISES,PROJECTS``` (folders that won't have transference)

**.gitignore**

The content in the .env is not dangerous, however it is good practice to hide your variables in this separate file, in this case the routes to apps or folders.      
The folder **.drive_sync/** is created on your first command, and here you have access to the files that have transference.

```.env```   
```.drive_sync/```   
```sync_notes.sh```


**sync_notes.sh**

```
    #!/bin/bash
    # This line is to self execute script in bash

    # README
    # For linux and macos give execution permission to the script:
    # chmod +x sync_notes.sh

    # LOAD .ENV VARIABLES
    if [ -f .env ]; then
        export $(grep -v '^#' .env | tr -d '\r' | xargs)
    else
        echo -e "\033[0;31m.env file not found\033[0m"
        exit 1
    fi

    # PARAMETERS
    DRIVE=false
    REPO=false

    while [[ "$#" -gt 0 ]]; do
        case $1 in
            -DRIVE) DRIVE=true ;;
            -REPO) REPO=true ;;
            *) echo "Unknown parameter: $1"; exit 1 ;;
        esac
        shift
    done

    if [ "$DRIVE" = "$REPO" ]; then
        echo -e "\033[0;31mUse ONLY one option: -DRIVE or -REPO\033[0m"
        exit 1
    fi

    # PATHS & CONFIG
    REPO_ROOT=$(pwd)
    SYNC_DIR="$REPO_ROOT/.drive_sync"
    PANDOC_CMD=${PANDOC_PATH:-pandoc}

    # RCLONE EXCLUDE FOLDERS
    RCLONE_EXCLUDES=()
    IFS=',' read -ra ADDR <<< "$IGNORED_FOLDERS"
    for folder in "${ADDR[@]}"; do
        RCLONE_EXCLUDES+=(--exclude "**/$folder/**")
    done

    # SYNC DIR PREPARATION
    rm -rf "$SYNC_DIR"
    mkdir -p "$SYNC_DIR"

    # --- GOOGLE DRIVE TO REPOSITORY ---
    if [ "$REPO" = true ]; then
        echo -e "\033[0;36mSyncing DRIVE to REPO\033[0m"

        if rclone sync "${DRIVE_REMOTE}:${DRIVE_ROOT}" "$SYNC_DIR" \
            --drive-export-formats docx \
            "${RCLONE_EXCLUDES[@]}"; then
            
            find "$SYNC_DIR" -name "*.docx" | while read -r docx_file; do
                relative=${docx_file#$SYNC_DIR/}
                filename=$(basename "$relative")
                dirname=$(dirname "$relative")
                target_dir="$REPO_ROOT/$dirname/$NOTES_FOLDER"
                target_md="$target_dir/${filename%.docx}.md"

                mkdir -p "$target_dir"
                echo "  Converting: $relative"
                $PANDOC_CMD "$docx_file" -t gfm -o "$target_md"
            done
            echo -e "\033[0;32mDRIVE to REPO completed successfully\033[0m"
        else
            echo -e "\033[0;31mError: Rclone failed. Check if DRIVE_ROOT exists in your Google Drive.\033[0m"
            exit 1
        fi
    fi

    # --- REPOSITORY TO GOOGLE DRIVE ---
    if [ "$DRIVE" = true ]; then
        echo -e "\033[0;36mSyncing REPO to DRIVE\033[0m"

        find "$REPO_ROOT" -type d -name "$NOTES_FOLDER" | while read -r notes_path; do
            ignore=false
            for folder in "${ADDR[@]}"; do
                if [[ "$notes_path" == *"/$folder/"* ]]; then ignore=true; break; fi
            done
            
            if [ "$ignore" = false ]; then
                relative_notes=${notes_path#$REPO_ROOT/}
                target_dir="$SYNC_DIR/${relative_notes%/$NOTES_FOLDER}"
                
                mkdir -p "$target_dir"
                cp "$notes_path"/*.md "$target_dir/" 2>/dev/null
            fi
        done

        find "$SYNC_DIR" -name "*.md" | while read -r md_file; do
            docx_file="${md_file%.md}.docx"
            echo "  Converting: ${md_file#$SYNC_DIR/}"
            $PANDOC_CMD "$md_file" -t docx -o "$docx_file"
            rm "$md_file"
        done

        if rclone sync "$SYNC_DIR" "${DRIVE_REMOTE}:${DRIVE_ROOT}" \
            --drive-import-formats docx \
            "${RCLONE_EXCLUDES[@]}"; then
            echo -e "\033[0;32mREPO to DRIVE completed successfully\033[0m"
        else
            echo -e "\033[0;31mError: Rclone failed during upload.\033[0m"
            exit 1
        fi
    fi
```

