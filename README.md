
Arista Roles for Ansible - Development Guidelines
=================================================

#### Table of Contents

1. [Executing Test Cases for an Arista Ansible Role] (#executing-test-cases-for-an-arista-ansible-role)
   * [Overview] (#overview)
   * [Details] (#details)
2. [Developing Arista Roles For Ansible] (#developing-arista-roles-for-ansible)
    * [Role Development Guidelines] (#role-development-guidelines)
    * [Role Test Development] (#role-test-development)
    * [Development for arista-ansible-role-test] (#development-for-arista-ansible-role-test)



Executing Test Cases for an Arista Ansible Role
-----------------------------------------------

#### Overview

To execute a role test suite:

- Update test/fixtures/hosts with the name(s) of your test devices.
- Update test/arista-ansible-role-test/group_vars/all.yml with the
  connection information for your devices.
- Execute `make tests` from the root of the role directory.

#### Details

This test framework should be used in a cloned copy of an Arista
ansible-eos-* Ansible role. The framework will ***not*** execute properly in an
ansible-galaxy installation of the role.

The arista-ansible-role-test framework is included as a subtree within
the Arista role. The framework is maintained in a separate repo located
at https://github.com/arista-eosplus/arista-ansible-role-test.

To use the test framework in your local environment, you will first need
to update test/fixtures/hosts and test/arista-ansible-role-test/group_vars/all.yml.
The hosts file should list your testing devices under the [test_hosts] 
section. The all.yml file should reflect the proper connection
parameters for your devices under the provider mapping.

Once the files have been updated for your local environment, execute
`make tests` from the root of the role directory to run the test suite against
the role.

To run a specific set of tests from the test suite, set the environment
variable `ANSIBLE_ROLE_TEST_CASES` to the name(s) of the file(s) under
test/testcases that you wish to execute (excluding the yml extension).
For example, if the testcases folder contains test files named first.yml,
second.yml, and third.yml, setting `ANSIBLE_ROLE_TEST_CASES=first,third`
would run only the tests in first.yml and third.yml.

The test framework executes the following steps when processing a test suite:
- The current state of each device is backed up in the /mnt/flash directory
  on the device using the `copy running-config <backup_file>` command.
- The current startup-config for each device is also backed up.
- Test cases are gathered from every file under test/testcases that matches
  the `ANSIBLE_ROLE_TEST_CASES` pattern, or all files if the variable is unset.
- Each test case is executed:
  - Setup for the test case is performed, if any exists.
  - The test case is executed against the role, verifying idempotency and
    any present or absent configuration that should exist on the device.
  - Test case teardown is performed, if any exists.
- The configuration for each device is restored from the backup file
  generated at test initialization.
- The startup-config for each device is restored from its backup location.
- The backup files are removed from each device, leaving the device in the
  state in which it was before the tests.

To prevent restoring device configuration after tests have run (to debug
a failing test case, for example), set the environment variable
`NO_ANSIBLE_ROLE_TEST_TEARDOWN` to True (or any value that would evaluate to true).
In this case, restoring the device configuration may be accomplished manually
by issuing the commands `configure replace <backup_file>` and
`copy <backup_of_startup> startup-config` on each device. A
message should have been printed in the test output indicating the file names
used for the backups, as well as how to restore the device and delete the backups.


Developing Arista roles for Ansible
-----------------------------------

#### Role development guidelines

##### Existing role development

To begin development on an existing Arista Ansible role, clone/fork
the role repository to your working environment, create a working 
branch, and proceed with development.

##### New Arista Ansible role development

To begin development on a new Arista role for Ansible, initialize a 
role directory using the `ansible-galaxy init` command.
  ```
  ansible-galaxy init ansible-eos-newrole
  ```

This will create a directory named ansible-eos-newrole with the following
directory structure:
  ```
    README.md
    .travis.yml
    defaults/
      main.yml
    files/
    handlers/
      main.yml
    meta/
      main.yml
    templates/
    tests/
      inventory
      test.yml
    vars/
      main.yml
  ```

Remove the .travis.yml file and the tests/ directory.

Create a filter_plugins/ directory in the root of your new role's path
and copy the filter_plugins/config_block.py source file from an existing
Arista role into your directory structure.

From an existing Arista role, copy the following files into the new
role's path, creating any missing directories as needed:

  - .gitignore
  - Makefile*
  - files/README.md
  - filter_plugins/config_block.py
  - handlers/main.yml
  - meta/main.yml*
  - test/fixtures/hosts

  ```
  Note: An asterisk (*) indicates file should be reviewed for changes specific
  to the new role, such as updating the role name.
  ```


*XXX File structure, formatting guidelines, and other info goes here*



#### Role test development

* Make sure the role's README.md file includes the **Developer Information**
  section, which points to this document under the
  <ansible-eos-role>/test/arista-ansible-role-test directory. This information
  can be copied from an existing role's README file.

* Create a testcases directory for the role:

    ```
    --roletest-- >> mkdir -p test/testcases/
    ```
    
* Import the arista-ansible-role-test repository into the role as a subtree.
  From the root of the role directory, issue the following commands:

  * git remote add role-test https[]()://github.com/arista-eosplus/arista-ansible-role-test.git  
  * git subtree add --prefix=test/arista-ansible-role-test --squash role-test master  

    ```
    NOTE: These commands must be issued from a clean repo branch without any
    pending changes or commits. The `git subtree add` command will generate
    a commit to add the external repo to the working repository.
    
    --roletest-- >> git remote add role-test https://github.com/arista-eosplus/arista-ansible-role-test.git  
    --roletest-- >> git subtree add --prefix=test/arista-ansible-role-test --squash role-test master  
    git fetch role-test master  
    warning: no common commits  
    remote: Counting objects: 59, done.  
    remote: Compressing objects: 100% (24/24), done.  
    remote: Total 59 (delta 14), reused 0 (delta 0), pack-reused 35  
    Unpacking objects: 100% (59/59), done.  
    From https://github.com/arista-eosplus/arista-ansible-role-test  
      * branch            master     -> FETCH_HEAD
      * [new branch]      master     -> role-test/master
    Added dir 'test/arista-ansible-role-test'  
    ```

* Add test cases for the role:

  Create a yml file for each group of test cases.
  Each group test file name should reflect the type of tests being run in
  that group. When a role contains several template modules, it is a good
  idea to have at least one test group for each template. Templates that
  perform multiple configuration changes may also be separated into several
  test group files.

  Each test group file should contain a defaults section and a testcases
  section. The defaults defines the module name reported by the test
  framework, and this should be the same as the filename without an extension.


*XXX Information specific to writing test cases and ensuring the test framework has been included as a submodule goes here*

#### Development for arista-ansible-role-test

Because the arista-ansible-role-test framework repository has been included
as a subtree, direct modification of the test framework files is possible.
If you need to make changes to the framework itself, please follow the steps
outlined below, to make the propagation of the changes to the main 
framework repo as smooth as possible.

For the purposes of the instructions below, `role repo` refers to the base 
repository of the role being worked on (e.g. ansible-eos-vxlan), and
`framework repo` refers to the arista-ansible-role-test repository that was
imported as a subtree, i.e. everything under the /test/arista-ansible-role-test
directory in the role repo.

* Always make sure you have the latest changes for the framework repo
  in your local repository by issuing the command at the root of your role repo.

      git subtree pull --prefix=test/arista-ansible-role-test --squash role-test master

* Please keep commits to files in the framework directory 
  (test/arista-ansible-role-test) separate from commits to the rest
  of the role repo. This helps keep commit messages specific
  to the framework repo itself.
* Changes to the framework files must be committed to the role repo
  before being pushed to the framework repo. (git commit the framework 
  changes as part of the role repo before pushing the changes to the
  framework repo) 
* To push the changes to the framework repo, enter the following command
  at the root of the role repo, where `<branch>` is the name of a branch on
  the framework repo where the changes will be pushed. This branch will be
  created if it does not exist.

      git subtree push --prefix=test/arista-ansible-role-test --squash role-test <branch>

* Make a pull request for the changes by visiting the [framework repo website]
  (https://github.com/arista-eosplus/arista-ansible-role-test.git). There
  you may create a new pull request for the branch you pushed the changes to.



License
-------

Copyright (c) 2016, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com
