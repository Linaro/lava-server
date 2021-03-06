.. index:: test definition repository

.. _test_repos:

Test definitions in version control
###################################

LAVA supports git and bzr version control for use with test definitions, and
this is the recommended way to host and use test definitions for LAVA. When a
repository is listed in a test definition, the entire repository is checked
out. This allows YAML files in the repository to reliably access scripts and
other files which are part of the repository, inside the test image.

.. code-block:: yaml

  - test:
     role:
     - server
     - client
     definitions:
     - repository: http://git.linaro.org/lava-team/lava-functional-tests.git
       from: git
       path: lava-test-shell/multi-node/multinode02.yaml
       name: multinode-intermediate

When this test starts, the entire repository will be available in the current
working directory of the test. Therefore, ``multinode/multinode02.yaml`` can
include instructions to execute ``multinode/get_ip.sh``.

Job definitions in version control
**********************************

It is normally recommended to also store your test job YAML files in the
repository. This helps others who may want to use your test definitions.::

  https://git.linaro.org/lava-team/refactoring.git/tree/panda-multinode.yaml

There are numerous test repositories in use daily in Linaro that may be good
examples for you, including:

* https://git.linaro.org/lava-team/lava-functional-tests.git
* https://git.linaro.org/qa/test-definitions.git

Using specific branch of a test definition repository
*****************************************************

If a public branch is specified as a parameter in the job submission YAML,
that branch of the repository will be used instead of the default 'master'
branch.

.. code-block:: yaml

 - test:
     timeout:
       minutes: 5
     definitions:
     - repository: https://git.linaro.org/lava-team/lava-functional-tests.git
       from: git
       path: lava-test-shell/android/get-adb-serial-hikey.yaml
       name: get-hikey-serial
       branch: stylesen

.. note:: Do not supply anything other than a branch name to this parameter,
          like tag or revision as this would create a clone in a 'detached
          HEAD' state.

Using specific revisions of a test definition
*********************************************

If a specific revision is specified as a parameter in the job submission YAML,
that revision of the repository will be used instead of HEAD.

.. code-block:: yaml

 - test:
    failure_retry: 3
    timeout:
      minutes: 10
    name: kvm-basic-singlenode
    definitions:
        - repository: git://git.linaro.org/lava-team/lava-functional-tests.git
          from: git
          path: lava-test-shell/smoke-tests-basic.yaml
          name: smoke-tests
        - repository: http://git.linaro.org/lava-team/lava-functional-tests.git
          from: git
          path: lava-test-shell/single-node/singlenode03.yaml
          name: singlenode-advanced
          revision: 441b61

Shallow clones in GIT
*********************

Some git repositories have a long history of commits and using a full clone
takes up a lot of space. When a test job involves multiple test repositories,
this can cause issues with adding the LAVA overlay to the test job. For
example, ramdisks could become too large or there could be insufficient
space in the partition used for the test shell or it could take longer than
desired to transfer the overlay to the device.

When ``git`` support is requested for a test shell definition, LAVA will
default to making a **shallow** clone using ``--depth=1``. The git history
will be truncated to the single most recent commit.

A full clone can be requested by passing the ``shallow`` parameter with a
value of ``False``. If the ``revision`` option is used, shallow clone
support will need to be turned off or the change to specified revision
will fail.

.. seealso:: https://git-scm.com/docs/git-clone

.. code-block:: yaml

 - test:
    failure_retry: 3
    timeout:
      minutes: 10
    name: kvm-basic-singlenode
    definitions:
        - repository: http://git.linaro.org/lava-team/lava-functional-tests.git
          from: git
          path: lava-test-shell/single-node/singlenode03.yaml
          name: singlenode-advanced
          shallow: False


Removing git history
********************

The size of the overlay can be an issue for jobs running on small devices.
By default, when cloning test definition from a git repository, LAVA will keep
the **.git** directory.
If needed, this directory can be removed from the overlay by setting
``history`` to **false**.

.. code-block:: yaml

 - test:
    failure_retry: 3
    timeout:
      minutes: 10
    name: kvm-basic-singlenode
    definitions:
        - repository: http://git.linaro.org/lava-team/lava-functional-tests.git
          from: git
          path: lava-test-shell/single-node/singlenode03.yaml
          name: singlenode-advanced
          history: False


Sharing the contents of test definitions
****************************************

A YAML test definition file can clone another repository by specifying the
address of the repository to clone

.. code-block:: yaml

  install:
      bzr-repos:
          - lp:lava-test
      git-repos:
          - git://git.linaro.org/people/davelong/lt_ti_lava.git

  run:
      steps:
          - cd lt_ti_lava
          - echo "now in the git cloned directory"

This allows a collection of LAVA test definitions to re-use other YAML custom
scripts without duplication. The tests inside the other repository will **not**
be executed.

Test repository for functional tests in LAVA
********************************************

LAVA regularly runs a set of test definitions to check for regressions and the
set is available for others to use as a template for their own tests::

* https://git.linaro.org/lava-team/lava-functional-tests.git
