# Guidelines for collaboration when writing code

Here are some guidelines for writing and managing code. You should be familiar
with all the concepts in all the text and links in this page (except for the
examples for code structure, which are on a need-to-know basis, and the source
pages from the extracted paragraphs that you will find pasted inline).

## Using version control (git)

We use git for version control. If you're not familiar with git, [this is a good place to start](https://www.freecodecamp.org/news/what-is-git-and-how-to-use-it-c341b049ae61/), and [this is a fantastic follow-up](http://think-like-a-git.net/). 

There are some good interactive resources to learn [git in general](https://gitexercises.fracz.com/), and [git branching in particular](https://learngitbranching.js.org/).

## Where to put your code

We have a [gitlab group for the Unit](https://gitlab.com/bu_cnio/) for hosting and managing our code.

To gain access, you simply need to create a gitlab user, go to the [group page](https://gitlab.com/bu_cnio/), and request access.

## How to organise and manage your code

In order to facilitate collaboration when more than one person are working on a project, we follow some rules for repository structure and git-related operations (branches, commits, merge requests, etc.).

### Project structure

The structure of the repository will be defined by the project owner, ideally following well-established guidelines (see examples for [Snakemake](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html), [Python](https://docs.python-guide.org/writing/structure/), and [R](https://richpauloo.github.io/2018-10-17-How-to-keep-your-R-projects-organized/).

### git workflow

For git, we follow the [GitHub flow model](https://guides.github.com/introduction/flow/). It's a simple and concise model that (at the time of writing) perfectly suits our requirements.

#### Commits

Some more details on how we do commits and merge them into master are available [here](https://innerjoiner.com/guide/git-team-workflow-cheatsheet/).

#### Merge requests

Start [here](https://medium.com/@fagnerbrack/one-pull-request-one-concern-e84a27dfe9f1) to understand why merge requests, similarly to commits, need to be "atomic".

Here are two paragraphs extracted from other sites with some additional details on merge requests:

----

Adapted from [https://alexsav.io/git-collaboration-guidelines.html]()

The Pull or Merge Requests (PR) are used to share with your co-developers the code changes you did
in your branch and ask them to review it. Here we always do a PR from your branch to the ‘master’
branch. In a PR, your colleagues will have a view of the differences between the branches and the
commits. You must also add a title and a description. The title should be sufficient to understand
what is being changed. In the description you should: - make a useful description, - describe what
was changed in the pull request, - explain why this PR exists, - make it clear how it does what it
sets out to do. E.g: Does it change a column in the database? How is this being done? What happens
to the old data? - you may want to use screenshots to demonstrate what has changed if there is a
GUI involved in the project.

**Single Responsibility Principle**: The pull request should do only 1 thing.

**Pull request size**: It should be small. The pull request must have a maximum of approximately 250 lines of change.

**Feature breaking**: Whenever it’s possible break pull requests into smaller ones.

----

Adapted from [https://yalantis.com/blog/code-review-via-gitlab-merge-requests-code-review-must/]()

To minimize the time spent on reviewing each merge request, you need to have a strategy for code review.

In our humble opinion, a good developer is not just someone who follows a programming workflow and writes high-quality code. A good developer knows how to deliver code for review and make the whole code review process effortless for the reviewer.

Keep in mind that a good merge request should solve a specific task. We suggest not including more than one feature in one merge request. It would be much better if you created several merge requests instead.

As a reviewer, don’t hesitate to pull the source branch and test incoming code by yourself, especially if the merge request contains plenty of changes. Build the project and check that everything works as expected.

Also, an important detail in our code review checklist is deleting branches when they’re no longer needed. The responsibility for deleting branches after they’re fully merged lies with the reviewer.

----

