---
description: >-
  create-batch는 경희대 고정환 교수님이 개발 및 유지보수를 하고 계신 작업 제출 프로그램입니다. 이 프로그램은 기존 HTCondor
  자체를 사용한 작업 제출보다 편리하게 작업을 수행할 수 있습니다.
---

# create-batch를 이용한 배치 작업 수행

## 1. 프로그램 획득

create-batch는 github를 통해 다운로드 받으실 수 있습니다.

```bash
git clone https://github.com/cms-kr/hep-tools.git
```

다운로드된 스크립트 중 create-batch를 이용하시면 됩니다. $HOME/bin 디렉토리로 복사해두시면 매우 편리하게 사용이 가능합니다.

```bash
cp hep-tools/create-batch $HOME/bin
```

## 2. 사용방법 안내

create-batch는 최신 트렌드에 맞쳐 **cmsRun 작업**과 **일반 bash를 통한 스크립트 작업** 모두 **대응 가능**합니다. 다음 도움말을 참고하여 cmsRun인지 일반 스크립트인지에 따라 적절하게 사용하시기 바랍니다.

```bash
(base) [geonmo@ui10 hep-tools]$ ./create-batch  
./create-batch  : create pbs/condor jobs 
Usage: create-batch cmsRun --cfg CONFIG_cfg.py --jobName JOBNAME --fileList fileList.txt --maxFiles MAXFILE        

       create-batch bash --cfg YOUR_BASH_script.sh --jobName JOBNAME --nJobs NJOBS --transferFiles YOUR_OUTPUT_FILES  
Mandatory options :   
       --jobName  NAME                         Name of job   
       --fileList DATA_FILES            File list text file   
       --maxFiles N                     Maximum number of files per job   
       --nJobs    N                     Number of Job sections   
       --cfg      CONFIG_FILE_cfg.py    Configuration file  
Optional :   
       --queue QUEUE_NAME               Set the batch queue name   
       --secondFileList DATA_FILES      Secondary file list text file   
       -n                               Do not submit jobs to batch   
       --transferDest OUTPUT_LOCATION   OUTPUT DIRECTORY (/store will be assumed to SE)   
       -G                               Disable Grid certificate   
       --maxEvent N                     Maximum number of events per job (-1 by default)   
       --transferFiles                  Additional files to transfer   
       -T                               Automatically transfer new files by archiveing them   
       --customise CUSTOMISE_cfg.py     Configuration file for customization   
       --args                           general arguments   
       --firstRun N                     For MC: run number   
       -B                               Rebuild whole package before job starts  
Optional, condor-specific :   
       --blacklist HOST1,HOST2,...      Remove specific hosts   
       --whitelist HOST1,HOST2,...      Use specific hosts
```

\--whitelist 혹은 --blacklist 옵션을 통하여 특정 머신에서만 작업이 돌도록 지정하거나 해당 머신을 제외하고 작업을 수행하도록 지정할 수 있습니다. (--blacklist cms-t3-wn3001.sdfarm.kr)

## 참고문헌&#x20;

1. [https://github.com/cms-kr/hep-tools](https://github.com/cms-kr/hep-tools)
