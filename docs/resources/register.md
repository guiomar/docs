!!! note
    You will need a resource owner priviledge in order to register you own resource. Please contact brlife@iu.edu.

## Background

Brainlife provides a mechanism to make a computing resource, such as a Cloud compute system or a high-performance computing cluster, available to specific user, projects or user defined groups. By default, shared compute resources are available to all users where most of Brainlife apps are enabled. 

As Brainlife's default resources are available by all users, often users must wait on the queue for requested tasks to get executed. You can register your own compute resources for the following use cases.

1. You have access to your own HPC resource, and you'd like to use it to run Brainlife apps, for better performance, or better access control.
2. You are an app developer and you'd like to use your own resource to troubleshoot your apps on your own resource, for easier debugging.
3. You are an app developer and your app can only run on specialized resources (like Hadoop, Spark, etc..) that Brainlife's shared resources do not provide.

Currently, only Brainlife admin can share personal resources with other members. If you wish to share your resources, please contact [Brainlife Admin](mailto:brlife@iu.edu)

!!! note
	Resource owner decides which apps are allowed to run on their resource. If you register a resource and enable apps on it, only you can run those apps on that resource. If you are publishing your app, and you want all users to be able to execute your app, please contact [Brainlife Admin](mailto:brlife@iu.edu) to enable your app on Brainlife default resources.

!!! warning
	Although we do our best to limit access to your dataset on shared resources, we recommend registering your own resource for added security
	especially if you are planning to process sensitive data. We currently do not allow any datasets with PHI (protected health information).

## Configuring Resources

Before you can register your resource, you should make sure that your resource is ready to run brainlife Apps.

### ABCD Default Hooks

[ABCD Hooks](https://github.com/brainlife/abcd-spec){target=_blank} are used to start, stop and monitor apps on remote resources. Some app provides its own hooks, but many of them relies on default hooks that are installed on each resource. As a resource provider, you need to provide these default hooks and make them available by setting `$PATH`. If you are not sure how to write these scripts, you can install and use Brainlife's default ABCD hooks by doing following.

```
cd ~
git clone https://github.com/brainlife/abcd-spec
```

Then, add one of following to your ~/.bashrc

#### For PBS cluster

```
export PATH=~/abcd-spec/hooks/pbs:$PATH
```

#### For Slurm cluster (for most HPC systems)

```
export PATH=~/abcd-spec/hooks/slurm:$PATH
```

#### For direct execution - no batch submission manager (like.. VMs)

```
export PATH=~/abcd-spec/hooks/direct:$PATH
```

!!! note
    IU HPC resource, you can use brainlife's shared installation

    ```
    $ ~/.bashrc
    export PATH=/N/u/brlife/Carbonate/abcd-spec/hooks/slurm:$PATH
    ```

### Work/Scratch directory

In order to process data, you will need scratch / workdir to host your input/output files as well as any temporary files generated. Most HPC system provides access to scratch storage system, but for IU 
system, you will need to submit a request for [slate](https://kb.iu.edu/d/axms) and configure your brainlife resource to use it. You can then create a directory inside your scratch directory such as

```
mkdir /N/slate/<username>/brainlife-workdir
```

!!! note
    brainlife keeps data on scartch volume up to 3 months. Most likely, you will not be able to keep your data that long without running out of your storage space. Please purge unnsed data periodically 
    and monitor your storage usage (use `quota` command).
    
    You can run a command like this to search and purge unused workdirs (create a script and run this, or setup a cron job to run it automatically)

    ```
    find /N/slate/<username>/brainlife-workdir -maxdepth 2 -type d -ctime +$days -exec rm -v -rf {} \;
    ```

### Common Binaries

Brainlife expects certain binaries to be installed on all resources. Please make sure following commands are installed.

* jq (command line json parser commonly used by Brainlife apps to parse config.json)
* git (used to clone / update apps installed)
* singularity (user level container execution engine)
* node (some app uses node to generate job submission file)

!!! note
    For IU HPC resource, please feel free to use brlife's shared binary directory.

    ```
    $ ~/.bashrc
    export PATH=$PATH:/N/u/brlife/Carbonate/bin
    export PATH=$PATH:/N/u/brlife/Carbonate/app/node-v10.16.3-linux-x64/bin
    ```

For singularity, you can either install it on the system (`apt install singularity-container` with neurodebian, or `yum install epel-release singularity` for yum based systems), or for most HPC systems you can simply add `module load singularity` in your `~/.modules` file.

By default, singularity uses user's home directory to cache docker images (and /tmp to create a merged container image to run). If you have limited amount of home directory space, you will need to override it by setting SINGULARITY_CACHEDIR. 

```
export SINGULARITY_CACHEDIR=/N/slate/<username>/singularity-cachedir
```

* Please replace <username> with your username, and make sure specified directories exists.

!!! note
    singularity by default does not expose any mounted file systems inside the container. For IU HPC systems, for example, you will need to specify your slate work directory in your .bashrc (comma delimited)

    ```
    export SINGULARITY_BINDPATH=/N/slate/<username>/brainlife-workdir,/N/any/other/paths/you/want/to/mount
    ```

### Other ENV parameters

Depending on the app you are trying to run, some app may require additional ENV parameters. For example, brainlife/app-freesurfer requires you to provide your freesurfer license via `FREESURFER_LICENSE`.

```
export FREESURFER_LICENSE="hayashis@iu.edu 29511 *xxxxxxxxxxx xxxxxxxxxxx"
```

You can request your freesurfer license [here](https://surfer.nmr.mgh.harvard.edu/registration.html){target=_blank}

Some Apps do dynamic resource allocation. It *requests max number of tasks / memories / walltime based on configuration / input data. For these Apps uses a special ENV parameter `BRAINLIFE_MAXTASKS` to limit the number of
maximum number of tasks per threads. Please set this ENV if your resource support batch scheduler like SLURM.

```
#instruct dynamic Apps to limit number of tasks (#SLURM --ntasks-per-node=N)
export BRAINLIFE_MAXTASKS=24
```

## Registering Resources

To register your resource, go to the [resources](https://brainlife.io/resources){target=_blank} page, and click "Add New Account". A resource entry form should appear. Please populate the following fields.

* *Name* Enter the name of rhe resource
* *Hostname* The hostname of your compute resource (usually a login/submit host)
* *Username* Username used to ssh to this resource
* *Workdir* Directory used to stage and store generated datasets by apps. *You should not share the same directory with other resources*. Please make sure that the specified directory exits (mkdir if not).
* SSH Public Key: Copy the content of this key to your resource's ~/.ssh/authorized_keys. Please read [authorized_keys](https://www.ssh.com/ssh/authorized_keys/){target=_blank} for more detail.

You can leave the rest of the fields empty for now.

!!! warning
	IU HPC systems requires you to submit [ssh public key agreement form](https://hpceverywhere.iu.edu/agree){target=_blank} so that you can authenticate using your ssh public key.

Click OK. Once you are finished with copying ssh key and make sure the workdir exists, click "Test" button to see if Brainlife can access your resource. You should see a green checkbox if everything is good.

## Enabling Apps

Once you have registered and tested your resource, you can now enable apps to run on your resource.

Go back to the Brainlife's [resources](https://brainlife.io/resources){target=_blank} page, and click the resource you have created. Under the services section, enter the git org/repo name (such as like `brainlife/app-life`) for the app that you'd like to enable, and the score for each service. The higher the score is, the more likely the resource will be chosen to run your app (if there are multiple resources available). Brainlife gives higher score for resources that you own (not shared ones), you should leave it the default of 10 unless it's competing with other resource that you have access to. Click OK.

You can see which resource an app is configured to run, and which resource will be chosen when you submit it under App detail / Computing Resources section on Brainlife. [example](https://brainlife.io/app/58c56cf7e13a50849b258800){target=_blank}


