---
description: We are informing you about the submission and utilization of batch job.
---

# How to use batch Jobs

## About HTCondor

HTCondor is a batch system of our CMS Tier-3. You can use HTCondor command for submitting jobs, checking job results, and checking server conditions.

### Import Infomation for HTCondor

* You can not access /xrootd or /xrootd\_user directory on WN. You should use xrootd protocol directly.
* You can use a private network on KISTI server. Its prefix is root://cms-xrdr.private.lo:2094&#x20;
* On the outside, you need to public address to access KISTI storage. Its address is root://cms-xrdr.sdfarm.kr:1094/\~\~.
  * This path requires User proxy certificates which is signed for VOCMS (Please see voms-proxy-init command)
  * You need to copy your proxy CA to WN's /tmp directory or set up a working directory ($\_CONDOR\_SCRATCH\_DIR) as a certificate directory.

## Important HTCondor Command

### condor\_status

condor\_status command is used to check machine status. Generally, no option command was used.

```
[geonmo@ui20 geonmo]$ condor_status                                                                                                                         
Name                             OpSys      Arch   State     Activity LoadAv Mem     ActvtyTime                                                             
                                                                                                                                                            
slot1@cms-gpu01.sdfarm.kr        LINUX      X86_64 Unclaimed Idle      0.000 386684  2+21:24:51                                                             
slot1@cms-t3-wn3001.sdfarm.kr    LINUX      X86_64 Unclaimed Idle      0.000 193349  0+16:29:10                                                             
slot2@cms-t3-wn3001.sdfarm.kr    LINUX      X86_64 Unclaimed Idle      0.000  87842  2+18:49:40                                                             
slot2_1@cms-t3-wn3001.sdfarm.kr  LINUX      X86_64 Claimed   Busy      0.010   2944  0+14:22:49                                                             
slot2_9@cms-t3-wn3001.sdfarm.kr  LINUX      X86_64 Claimed   Busy      1.000   2944  2+19:09:24                                                             
slot2_10@cms-t3-wn3001.sdfarm.kr LINUX      X86_64 Claimed   Busy      1.000   2944  2+19:06:09       
```

{% hint style="info" %}
The job slot can be divided dynamically depending on the characteristics of the requested job. KISTI CMS Tier-3 prepared three dynamic slots for each machine, and each slot allocated CPU resources 2:1:1.
{% endhint %}

{% hint style="danger" %}
For CMS jobs without any resource option, the resource was requested as 1 core, 2933MB RAM. If you want more resources for slots, you can request to modify resource options. However, slot matching will be difficult for big size jobs. (Longer wait time)
{% endhint %}

## condor\_q

condor\_q can check information about the job. For more detailed information on the submitted job, you can check it using the -l( -long) option.

```bash
[geonmo@ui20 geonmo]$ condor_q


-- Schedd: ui10.sdfarm.kr : <134.75.124.121:9618?... @ 05/21/20 10:59:50
OWNER  BATCH_NAME                                                   SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS
jhchoi RequestTest_VBFHToWWToLNuQQ_M350__2016__10000               5/16 15:46      _      1      _      _      1 3671768.0
jhchoi RequestTest_VBFHToWWToLNuQQ_M450__2016__10000               5/16 15:46      _      1      _      _      1 3671777.0
jhchoi RequestTest_VBFHToWWToLNuQQ_M175__2016__10000               5/16 15:46      _      1      _      _      1 3671780.0

[geonmo@ui20 geonmo]$ condor_q -l 3671768.0 | grep Request
RequestCpus = 1
RequestDisk = DiskUsage
RequestMemory = 2930
Requirements = ((HasSingularity == true)) && (TARGET.Arch == "X86_64") && (TARGET.OpSys == "LINUX") && (TARGET.Disk >= RequestDisk) && (TARGET.Memory >= RequestMemory) && ((TARGET.FileSystemDomain == MY.FileSystemDomain) || (TARGET.HasFileTransfer))

```

If you want to know job resource request information, you use condor\_q  -better-analyze command. If the job is held, you can check hold reason.

```bash
[geonmo@ui20 geonmo]$ condor_q -better-analyze 3671768.0 


-- Schedd: ui10.sdfarm.kr : <134.75.124.121:9618?...
The Requirements expression for job 3671768.000 is

    ((HasSingularity == true)) && (TARGET.Arch == "X86_64") && (TARGET.OpSys == "LINUX") && (TARGET.Disk >= RequestDisk) &&
    (TARGET.Memory >= RequestMemory) && ((TARGET.FileSystemDomain == MY.FileSystemDomain) || (TARGET.HasFileTransfer))

Job 3671768.000 defines the following attributes:

    DiskUsage = 2
    FileSystemDomain = "sdfarm.kr"
    RequestDisk = DiskUsage
    RequestMemory = 2930

The Requirements expression for job 3671768.000 reduces to these conditions:

         Slots
Step    Matched  Condition
-----  --------  ---------
[0]         211  HasSingularity == true
[1]         211  TARGET.Arch == "X86_64"
[3]         211  TARGET.OpSys == "LINUX"
[5]         211  TARGET.Disk >= RequestDisk
[7]         211  TARGET.Memory >= RequestMemory
[9]         211  TARGET.FileSystemDomain == MY.FileSystemDomain


3671768.000:  Job is running.

Last successful match: Wed May 20 21:41:45 2020


3671768.000:  Run analysis summary ignoring user priority.  Of 88 machines,
      0 are rejected by your job's requirements
      0 reject your job because of their own requirements
      0 match and are already running your jobs
      0 match but are serving other users
     88 are able to run your job



-- Schedd: ui20.sdfarm.kr : <134.75.124.127:9618?...

```

## condor\_history

It is a command to check information about the finished jobs. The method of use is the same as the condor\_q command.

## condor\_submit

Submit your job using the Job Description Submit File (.sub or .jds). The basic job submission statement form is as follows.

{% code title="filename : template.jds" %}
```bash
#### Job Batch name / Your jobs will be displayed as this name.
JobBatchName = condor_status_check
#### Executable main program file
# Generally, a binary file was used. 
# However, you can choose a bash script to setup envirionment and run. 
executable = test.sh
#### Most cases, the "vanilla" universe is selected for a normal job. 
# Java universe for java application, docker universe for docker application.
# Each universe provides the application's environment and extra ClassADs.
universe   = vanilla
#### Argument for application.
# eg) test.sh 13
# $(Process) means job's process ID
# FYI) JobId = $(Cluster).$(Process),  
arguments  = $(Process)

### Sync OS environments.
# However, it can not provide perfectly. Please, check WN's env setting.
getenv     = True

### Enable the feature to send and receive files
# 이 기능이 꺼졌다면 executable 파일을 WN로 전송하지 않습니다.
# 또한, 실행 결과 파일도 가져오지 않습니다. 
# 작업 결과 파일을 가져오지 않고 공유 디렉토리에서 실행할 경우에는 NO로 설정합니다.
should_transfer_files = YES
### 위 키워드와 같이 사용합니다.
when_to_transfer_output = ON_EXIT

### 요구조건 설정
# 여기서는 Hostname과 동일한 Machine을 사용하도록 설정하였습니다.
# 특별한 조건이 필요한 머신만 특정지어 사용할 때 사용됩니다.
requirements = (Machine =?= "$(Hostname)")

### CMS에서만 사용되는 별도의 태그들 (선택사항)
# Tag는 프로그램의 이름, JobType은 MC, Analysis 중 설정합니다.
+Tag = "condor_check v1.22"
+JobType = "Analysis"

### 실행시 표준 출력 및 에러 저장 파일
output = job_$(Hostname).out
error = job_$(Hostname).err
### 작업 제출 때의 로그, submit 머신의 로그라고 볼 수 있습니다.
log = job.log

### 송신할 입력 파일 및 수신할 결과 파일 이름 지정
transfer_input_files = input_sandbox.tar.gz 
transfer_output_files = result.root
### 결과파일의 이름이 작업마다 겹칠 경우 다른 이름으로 저장하도록 지정하여야 합니다.
# 여기서는 result.root 파일들을 Hostname 변수를 추가하여 저장합니다.
transfer_output_remaps = "result.root = result_$(Hostname).root"

### 자원 요구량 설정
#request_Cpus=1
#request_GPUs =0 
#request_memory=2933
#request_disk = 1

### 이메일 알람 설정
#notification = Error
#notify_user = cmst3-support@kisti.re.kr

### Group Account 정보


#queue 13

queue 1 Hostname from test.txt
```
{% endcode %}

When submitting a job using the JDS file, run it with the command below.

```bash
condor_submit template.jds
```

Currently, there are several differences between KISTI GSDC Tier-3 and the general environment regarding the `condor_submit`. In order to utilize the integrated farm cluster, CMS users must create a JDS with the accounting\_group="group\_cms". We set default bash environment for this to alias **condor\_submit**. However, if you write a bash script directly, please add the contents.

```bash
alias condor_submit='condor_submit -append accounting_group="group_cms"'
alias condor_submit6='condor_submit -append accounting_group="group_cms" -append "+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el6:latest\"" -append "+SingularityBindCVMFS=True" -append "+SingularityBind=\"/cvmfs,/cms,/share,/cms_scratch\""'
alias condor_submit7='condor_submit -append accounting_group="group_cms" -append "+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el7:latest\"" -append "+SingularityBindCVMFS=True" -append "+SingularityBind=\"/cvmfs,/cms,/share,/cms_scratch\""'
```

If you modify the contents a little, you can submit the job to the ubuntu linux environment.

