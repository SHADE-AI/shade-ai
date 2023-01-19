# DARPA SHADE Container Registry: GitLab Quickstart

A dedicated GitLab has been setup for DARPA SHADE. This GitLab allows users to store their different containers, or Docker images, in one place.

## Accessing the GitLab
Navigate to
```
https://darpa-gitlab.tacc.utexas.edu/
```
and login with your TACC username and password. If this is your first time accessing the GitLab and are not in a project, you will land on a page that says "Welcome to GitLab" and with a few different options. If you are part of one or more projects, they will be listed instead.

## Organizing the GitLab
There are a few ways to organize members and projects:
1) Group with subgroups and projects
2) Individual Projects
3) Single Project

There are probably other ways, but these seem to be the simplest/most straightforward way.

## 1) Groups, Subgroups, and Projects
GitLab utilizes the concept of a "Group." A Group allows you to create subgroups and projects directly tied to that group. Note that groups do not have their own container registries.

You could have a group called "World" and then create a project named "World Powers" which could store all of the images. This would look like
```
https://darpa-gitlab.tacc.utexas.edu/world/world-powers
```

You could also break it down further and create subgroups that represent different things, like "USA" and "Germany." These subgroups would then have their own projects where they store their own images. This would look like
```
https://darpa-gitlab.tacc.utexas.edu/world/usa/<project_name>
https://darpa-gitlab.tacc.utexas.edu/world/germany/<project_name>
```

## 2) Individual Projects
Instead of creating groups at all, you could create multiple projects and invite people to the projects directly. This could look like
```
https://darpa-gitlab.tacc.utexas.edu/<user>/usa
https://darpa-gitlab.tacc.utexas.edu/<user>/germany
```

## 3) Single Project
If the need to separate things out is not necessary, you could simply create a project called "Images" and have everyone push their images to this project. This project would be hosted at
```
https://darpa-gitlab.tacc.utexas.edu/<user>/<project_name>
```

## Creating a Project
If you need to setup a project to store your images, you will choose either 'Create a project' or 'New Project'. If you want to create a project tied to a group, just make sure you are on the groups homepage before clicking 'New Project.' The group homepage will be
```
https://darpa-gitlab.tacc.utexas.edu/<group_name>
```

### Create New Project
On the next screen, you will be given three options: 
1) Create blank project - as it says, this is a project with nothing pre-added
2) Create from template - will create a project pre-populated with different things depending on which template you choose
3) Import project - will migrate data from an external source like GitHub

Go ahead and choose the first option, 'Create blank project.' Since this GitLab is primarily for storing images, there's no need to pre-populate the repositories and take up unnecesarry storage. 

You can call the project whatever you want. Note that whatever you call it will be appended to the Project URL. For example, if you name the project `Bot Images`, it will be appended as `bot-images`.

## Accessing and Contributing to the Project Container Registry
When on the project page, yoou will see a navigation bar on the left hand side that has different options, such as `Project Information`, `Repository`, and so on. Hover over the option `Packages and registries` and select `Container Registry`. If the registry is empty, you will be presented with instructions on how to start contributing to the registry. You can copy paste these commands to get started right away. However, if other people have already contributed and this is your first time, or you just need a refresher, this is how you can contribute:

The first step is to login to Docker tied to the GitLab:
```
docker login darpa-gitlab.tacc.utexas.edu:5050
```

Now that you are authenticated to the registry, you can start adding images with the following commands:
```
docker build -t darpa-gitlab.tacc.utexas.edu:5050/<path_to_project> <path_of_Dockerfile>
docker push darpa-gitlab.tacc.utexas.edu:5050/<path_to_project>
```

If you are not sure of the path to the project, you can copy the URL of the container registry page and omit the ending `/container_registry`.
