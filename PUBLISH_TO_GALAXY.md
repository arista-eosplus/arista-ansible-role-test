How to Publish Arista ansible-eos Roles
=======================================

***NOTE: This information is specific to the ansible-eos-\* roles only.
Any other Arista roles on Ansible Galaxy can be imported/updated in a 
similar manner, but should not be included in an update to the
ansible-eos-\* roles.***

## Generate a new release for each role

1. Set a new version string for each role by updating the VERSION file
in the role's root directory. Currently, we have been generating a new 
release for all ansible-eos-* roles simultaneously, regardless of whether
the role was updated or not. This maintains unity between the roles and
reduces confusion about versions when working with multiple roles.

2. Generate a new release in GitHub for each role
    * Select the `releases` tab from the role's main GitHub page
    * Click the `Draft a new release` button
    * Fill in the Tag Version and Release Title fields
      - Tag version: using the form v#.#.#, e.g. *v2.1.10*
      - Release title: \<role name\> role \<version tag\>, e.g.
      *ansible-eos-mlag role v2.1.10*
    * Write a release description if desired
    * Click the `Publish release` button to make it official

## Publish the new releases to Ansible Galaxy

1. Sign into [Ansible Galaxy](https://galaxy.ansible.com/) using your
GitHub credentials. *(You may have to grant permissions to Ansible Galaxy
the first time you log in)*

2. At the top left of the page next to your account name, select the
`MY ROLES` button. This will open a page where you can import each of
the roles from GitHub.
    * Click the import button on the far left of each role's row to
    bring in the new releases to Ansible Galaxy.
    * The status next to the import button should update to *Running*
    and then to *Succeeded*. If the import does not succeed, click the
    failure message to see the import log to determine the issue.

### *All Done!*
