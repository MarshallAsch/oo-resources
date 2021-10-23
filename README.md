need to manually setup the initial wiki setting and copy that database into the container
need to set wiki url to localhost:3000


## Building the container

```bash
$ docker build --build-arg GIT_COMMIT=$(git rev-parse -q --verify HEAD) --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") -t judi:latest .
```


## Running the container

```bash
$ docker run --rm -it -p 3000:3000 -p 8888:8888 -v "$(pwd)/course:/course" judi:latest
```

This container is designed to be run in the foreground.
It will run a wiki.js server and a jupyter notebooks server in the background and provide the user with a bash shell

## Using the container


### Git
To configure git within the container this can be done manually by running the `git config` commands or by using the environment variables

`GIT_EMAIL='student@example.com'`, `GIT_NAME="My StudentName"`

These environment variables can be configured when you run the docker container

```bash
$ dockr run -it --rm -e GIT_EMAIL='student@example.com' -e GIT_NAME="My StudentName" judi:latest
```

### Wiki

The wiki can be found at http://localhost:3000

Username: admin@example.com
Password: password


### Jupyter Notebooks

The jupyter notebooks site can be found at http://localhost:8888

Password: password

When you go to the jupyter notebooks page there are two folders that will apear, `builtin` and `persistant`.
All of the notebooks that are in the `builtin` folder are part of the container, any changes made to those notebooks will not be saved between container recreations.
Each of the notebooks in the `persistant` folder will be saved to the `jupyter` folder inside the `course` volume; these will be saved.


## Customization of this container

### How to configure custom wiki content

Any built in content to be included in the wiki must be added manually.
Unfortuantly there is not currently an easy mechanism to automaticly load markdown files from a directory into the wiki. 

To Load custom content into the container the following proccess is suggested:

1. Start the container _with_ the volume mount `docker run -it --rm -v "$(pwd)/course:/course" judi:latest`
2. Go to http://localhost:3000 and create all of the desired wiki pages and configurations
3. Exit the container
4. Replace the `database.sqlite` file with the new one from `course/wiki/database.sqlite`
5. Rebuild the container using the docker build command

Alternativly if there is a large number of wiki pages to import that already exist as markdown files an alternative process can be used.

1. Create the folder `course/wiki/files`
2. Place all of your markdown files to be imported into that folder, the files should have the following frontmatter so they can be imported with the desired title, tags, and if it should be published or not. Subfolders can also be used
```
---
title: Untitled Page
description:
isPublished: 1
tags: coma, seperated, list
---
```
3. Start the container _with_ the volume mount `docker run -it --rm -v "$(pwd)/course:/course" judi:latest`
4. Go to http://localhost:3000 and navigate to Administration > Storage > Local File System
5. Enable local file storage, set the Path to `/course/wiki/files`
6. Scroll to the bottem of the page and run `Import Everything`, now all of the wiki pages should be imported
7. Exit the container
8. Replace the `database.sqlite` file with the new one from `course/wiki/database.sqlite`
9. Rebuild the container using the docker build command

### How to configure builtin jupyter notebooks

Adding built in jupyter notebooks to the container is simpler. 
Place all the desired files in the `builtinNotebooks` folder, then build the image.

### Extending this container with custom container

This image is designed to be used as a base image to be loaded with custom content for specific course deliveries. 

An example can be found in the `example` folder that will create an image with pre added jupyter notebooks and wiki configurations.
The steps to create the wiki configuration is the same as that for this container. 

## Software Licence

This project is licensed under the GPLv3 licence.
This is a strong copy left licence that requires that any derivative work is released under the same licence.
This was selected because the objective of this project is to provide a tool that can be used by others because it is something that is useful to us.
We believe that carrying that forward will be beneficial to the community.

#### TODO:

These are some of the tasks that can still be done to make it better

- automatically generate an ssh keypair to be used for git
- seed the wiki with some initial content
- specify a specific version for jupyter
- add bash completions for the main tools that have been installed
- add a MOTD to when the container starts up to give a little bit of usage information
- get all of the log messages for jupyter and the wiki to go to a log file
