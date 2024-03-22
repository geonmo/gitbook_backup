---
description: 테스트를 위한 작업이 아니라 대용량 데이터를 처리하기 위한 배치 작업의 사용 방법을 배웁니다.
---

# 배치 작업 사용하기

## HTCondor에 관하여&#x20;

HTCondor는 우리 CMS Tier-3에서 제공하는 배치 시스템입니다. 해당 프로그램을 이용하여 작업 제출, 작업 결과 확인, 장비 상태 확인 등의 작업을 처리할 수 있습니다.

### HTCondor 사용시 알아둬야하는 부분

* /xrootd 및 /xrootd\_user 디렉토리는 WN에서 접근이 제한되므로 xrootd 프로토콜로 직접 접속을 하시기 바랍니다.
* KISTI 내부에서는 root://cms-xrdr.private.lo:2094 를 접두어로 사용하면 됩니다.
* 외부에서 KISTI 저장소에 접근을 하려면 root://cms-xrdr.sdfarm.kr:1094 를 사용하여야 합니다.&#x20;
  * 이 경로는 사용자 프록시 인증서가 꼭 필요합니다.  (voms-proxy-init 명령어 참고)
  * 실행 스크립트에서 인증서를 /tmp로 보내거나 작업디렉토리($\_CONDOR\_SCRATCH\_DIR)를 인증서 보관 디렉토리로 지정하여야 합니다.

## HTCondor 명령어 익혀보기

### condor\_status

condor\_status 명령어는 머신들의 상태를 확인하는 명령어입니다. 일반적으로 옵션 없이 사용되는 명령어를 자주 사용합니다.

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
작업이 실행되는 작업 슬롯은 요청된 작업의 특성에 따라 유동적으로 변하게 됩니다. KISTI CMS Tier-3에서는 각 머신당 3개의 유동 슬롯을 준비하였으며 각각의 슬롯은 2:1:1로 CPU 자원을 할당하였습니다. &#x20;
{% endhint %}

{% hint style="danger" %}
CMS 작업은 아무런 옵션이 없이 제출될 경우 \
CPU 1개, 메모리 2933MB를 사용하도록 설정됩니다. 이 자원 요구량기보다 더 크게 요청할 경우 자원 확보가 용이하지 않아 작업이 실행되지 않을 수 있으니 참고 바랍니다.
{% endhint %}

## condor\_q

condor\_q는 작업에 대한 정보를 확인할 수 있습니다. 제출된 작업의 자세한 정보를 위해서는 -l( -long) 옵션을 사용하여 확인이 가능합니다.

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

condor\_q  -better-analyze 명령어를 통해 작업이 요구 조건 확인할 수 있습니다. 작업이 정지 상태라면 정지 원인도 확인 가능합니다.

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

종료된 작업에 관한 정보를 확인하기 위한 명령입니다. 사용방법은 condor\_q 명령어와 동일합니다.

## condor\_submit

준비된 작업 제출 명세 파일(Job Description Submit File, .sub 혹은 .jds)을 이용하여 작업을 제출합니다. 기본적인 작업 제출 명세파일의 양식은 다음과 같습니다.

{% code title="filename : template.jds" %}
```bash
#### 작업목록에서 확인할이름을 지정합니다. 
JobBatchName = condorTestJobs 
#### executable은 실행할 프로그램을 입력합니다. 
# 바이너리 실행 파일을 입력하기도 하지만, 
# 일반적으로 실행파일의 환경설정을 위해 Bash 스크립트 파일을 넣습니다.
executable = test.sh
#### universe는 일반적인 경우는 vanilla를 설정합니다.
# 컨테이너를 실행하기 위한 container 유니버스, docker를 위한 docker 유니버스 등도 존재합니다.
# 각각의 유니버스들은 해당 프로그램 실행을 위한 환경설정을 지원합니다.
universe   = vanilla
#### arguments는 프로그램의 인자를 입력합니다.
# 여기서 사용된 $(Process)는 제출된 작업의 Process Id입니다.
# eg) test.sh 13
# FYI) JobId = $(Cluster).$(Process),  
arguments  = $(Process)

### 사용자의 환경변수를 WN에 전달하기 위한 옵션
# 하지만, 100% 전달되는 것은 아니기 때문에 확인이 꼭 필요.
getenv     = True

### 파일의 송수신 기능 활성화
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

JDS 파일을 이용하여 작업을 제출할 때는 아래 명령어로 실행합니다.

```bash
condor_submit template.jds
```

현재 KISTI GSDC Tier-3에는 condor\_submit 관련하여 일반적인 환경과 다른 사항이 몇가지 존재합니다. 통합팜 클러스터 활용을 위해 CMS 사용자들은 accounting\_group="group\_cms" 옵션이 반드시 포함된 JDS를 작성하여야 합니다. 하지만, 기본 bash 환경에서는 예약어로 설정된 명령어를 사용하기 때문에 위 내용을 생략할 수 있습니다. 하지만, bash 스크립트 등을 작성할 경우에는 해당 내용을 꼭 추가해주시기 바랍니다.

```bash
alias condor_submit='condor_submit -append accounting_group="group_cms"'
alias condor_submit6='condor_submit -append accounting_group="group_cms" -append "+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el6:latest\"" -append "+SingularityBindCVMFS=True" -append "+SingularityBind=\"/cvmfs,/cms,/share,/cms_scratch\""'
alias condor_submit7='condor_submit -append accounting_group="group_cms" -append "+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el7:latest\"" -append "+SingularityBindCVMFS=True" -append "+SingularityBind=\"/cvmfs,/cms,/share,/cms_scratch\""'
```

&#x20;CentOS6용 작업을 던질 때는 condor\_submit6를 7으로 던질 때는 condor\_submit  혹은 condor\_submit7명령어를 사용하면 됩니다.&#x20;

조금 내용을 수정하면 ubuntu 환경으로 작업 제출도 가능합니다.

