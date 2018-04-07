Jenkins Sandbox Process
-----------------------

Hyperledger Jenkins Sandbox provides Jenkins Job testing/experimentation environment
that can be used before pushing job templates to the production
`Jenkins <https://jenkins.hyperledger.org>`__. It is configured similar to the Hyperledger
`ci-management <http://github.com/hyperledger/ci-management>`__
production instance; however, it cannot publish artifacts or vote in Gerrit.
Be aware that this is a test environment, and as such there are a limited allotment
of minions to test on before pushing code to the Hyperledger repos.

Keep the following points in mind prior to beginning work
on Hyperledger Jenkins Sandbox environment:

-  Jobs are automatically deleted from Jenkins Sandbox every SATURDAY
-  Committers can login and configure Jenkins jobs in the SANDBOX
   directly
-  Jenkins Sandbox jobs CANNOT upload build images to Docker Hub
-  Jenkins Sandbox jobs CANNOT vote on Gerrit
-  Jenkins nodes are configured using Hyperledger openstack VMs and you
   can not access these VMs directly.

Before you proceed further, ensure you have a Linux Foundation ID (LFID), which is
required to access Gerrit & Jenkins. The documentation on requesting
a Linux Foundation account is available here
`lf-account <http://hyperledger-fabric.readthedocs.io/en/latest/Gerrit/lf-account.html>`__.
Also, to get an access to Sandbox environment please send an email to
helpdesk@hyperledger.org (LF helpdesk team)

Follow below steps to work on the Jenkins Sandbox:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Step 1:**

To download **ci-management**, execute the following command to clone the **ci-managment** repository.

``git clone ssh://<LFID>@gerrit.hyperledger.org:29418/ci-management && scp -p -P 29418 <LFID>@gerrit.hyperledger.org:hooks/commit-msg ci-management/.git/hooks/``

**Step 2:**
After cloning we need to make sure the submodules are also synced:

Make sure to sync global-jjb submodule using:
`git submodule update --init`

**Step 3:**

Once you successfully clone the repository, the next step is to install JJB
(Jenkins Job Builder) in order to do experiment with the Jenkins jobs.

Execute the following commands to install JJB on your machine:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    cd ci-management
    sudo apt-get install python-virtualenv
    virtualenv hyp
    source hyp/bin/activate
    pip install jenkins-job-builder
    jenkins-jobs --version
    jenkins-jobs test --recursive jjb/

**Step 3:**

Make a copy of the example JJB config file (in the builder/ directory)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Backup the jenkins.ini.example to jenkins.ini

``cp jenkins.ini.example jenkins.ini``

After copying the jenkins.ini.example, modify ``jenkins.ini`` with your
**Jenkins LFID username**, **API token** and **Hyperledger JENKINS
SANDBOX URL**

::

    [job_builder]
    ignore_cache=True
    keep_descriptions=False
    include_path=.:scripts:~/git/
    recursive=True

    [jenkins]
    user=rameshthoomu <Provide your Jenkins Sandbox username>
    password= <Refer below steps to get API token>
    url=https://jenkins.hyperledger.org/sandbox
    This is deprecated, use job_builder section instead
    ignore_cache=True
    query_plugins_info=false

How to retrieve API token?
~~~~~~~~~~~~~~~~~~~~~~~~~~

Login to the `Jenkins
Sandbox <https://jenkins.hyperledger.org/sandbox/>`__, go to your user
page by clicking on your username. Click **Configure** and then click
**Show API Token**.

To work on existing jobs or create new jobs, navigate to the ``/jjb``
directory where you will find all job templates for the project. Follow
the below commands to test, update or delete jobs in your SANDBOX
environment.

**Step 4:**

To Test a Job
~~~~~~~~~~~~~

After you modify or create jobs in the above environment, it is good
practice to test the job in sandbox environment before you submit this
job to production CI environment.

``jenkins-jobs --conf jenkins.ini test jjb/ <job-name>``

**Example:**
``jenkins-jobs --conf jenkins.ini test jjb/ fabric-verify-x86_64``

If the job you’d like to test is a template with variables in its name,
it must be manually expanded before use. For example, the commonly used
template ``fabric-verify-{arch}`` might expand to
``fabric-verify-x86-64``.

A successful test will output the XML description of the Jenkins job
described by the specified JJB job name.

Execute the following command to pipe-out to a directory:

``jenkins-jobs --conf jenkins.ini test jjb/ <job-name> -o <directoryname>``

The output directory will contain files with the XML configurations.

**Step 5:**

To Update a job
~~~~~~~~~~~~~~~

Ensure you’ve configured your ``jenkins.ini`` and verified it by
outputting valid XML descriptions of Jenkins jobs. Upon successful
verification, execute the following command to update the job to the
Jenkins SANDBOX.

``jenkins-jobs --conf jenkins.ini update jjb/ <job-name>``

**Example:**
``jenkins-jobs --conf jenkins.ini update jjb/ fabric-verify-x86_64``

**Step 6:**

Trigger jobs from Jenkins Sandbox:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you push the Jenkins job configuration to the Hyperledger Sandbox
environment, run the job from Jenkins Sandbox webUI. Follow the below
process to trigger the build:

Step 1: Login into the `Jenkins Sandbox
WebUI <https://jenkins.hyperledger.org/sandbox/>`__

Step 2: Click on the **job** which you want to trigger, then click
**Build with Parameters**, and finally click **Build**.

Step 3: Verify the **Build Executor Status** bar and make sure that the
build is triggered on the available executor. In Sandbox you may not see
all platforms build executors and you don’t find many like in production
CI environment.

Once the job is triggered, click on the build number to view the job
details and the console output.

**Step 7:**

Modify an Existing Job
~~~~~~~~~~~~~~~~~~~~~~

In the Hyperledger Jenkins Sandbox, you can directly edit or modify the
job configuration by selecting the job name and clicking on the
**Configure** button. Then, click the **Apply** and **Save** buttons to
save the job. However, it is recommended to simply modify the job in
your terminal and then follow the previously described steps in **To
Test a Job** and **To Update a Job** to perform your modifications.

**Step 8:**

To Delete a Job:
~~~~~~~~~~~~~~~~

Execute the following command to Delete a job from Sandbox:

``jenkins-jobs --conf jenkins.ini delete jjb/ <job-name>``

**Example:**
``jenkins-jobs --conf jenkins.ini delete jjb/ fabric-verify-x86_64``

The above command would delete the ``fabric-verify-x86-64`` job.
