# Cluster administration

## Task management

We manage admin tasks using Gitlab issues in the ["HPC Admin Scripts" project](https://gitlab.com/bu_cnio/hpc/admin-scripts/-/boards).

### General workflow

0. Tasks are created normally and they start on the "Open" column.
0. Tasks are "refined" while on "Open"
    - description can be added
    - users are assigned
    - tasks can can be split into separate tasks (if these new tasks are independent)
    - ideally, no more task splitting happens after this point
0. Once the task has an initial asignee and the description is "clean" enough, it is moved to the "ToDo" column.
0. From here it can be moved to the "Doing" column by the assignee whenever work starts.
    - if the assignee is changed, the person making the change should move it back to the "ToDo" column.
0. Once the task is completed, it should be moved to the "Done" column.
