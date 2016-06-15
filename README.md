
Arista Roles for Ansible Development Guidelines
===============================================


Executing Test Cases for an Arista Ansible Role
------------------

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

*XXX File structure, formatting guidelines, and other info goes here*

#### Role test development

*XXX Information specific to writing test cases and ensuring the test framework has been included as a submodule goes here*


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
