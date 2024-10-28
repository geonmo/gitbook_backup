---
description: 이 페이지에서는 HTCondor를 이용하여 작업을 제출하고 관리하는 방법을 알려드립니다.
---

# HTCondor를 이용한 배치 작업 수행

## CMSSW 작업환경 구축

이 예제에서는 ZZ 샘플을 이용하여 분석 작업을 실행할 예정입니다. 해당 데이터 샘플을 살펴보고 어떠한 작업 환경을 구축해야할지 확인을 해야 합니다.

### 데이터 파일 확인하기

ZZ샘플 데이터 파일 한개를 사용하여 데이터셋 정보를 확인합니다. 해당 파일의 위치는 다음과 같습니다.

> /xrootd/store/mc/RunIIFall17NanoAODv4/ZZ\_TuneCP5\_13TeV-pythia8/NANOAODSIM/PU2017\_12Apr2018\_Nano14Dec2018\_102X\_mc2017\_realistic\_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root

CMS에서 사용되는 LFN(Logical File Name) 주소는 반드시 "**/store**"로 시작됩니다. 위에서는 /xrootd만 빼면 됩니다. 해당 LFN은 어느 사이트에서도 동일합니다.

위 파일의 DataSet 및 Release 정보를 확인해봅시다.

```bash
### 임의의 CMSSW 작업 공간에서 cmsenv를 수행한 후,
dasgoclient --query "dataset file=/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root"
```

> /ZZ\_TuneCP5\_13TeV-pythia8/RunIIFall17NanoAODv4-PU2017\_12Apr2018\_Nano14Dec2018\_102X\_mc2017\_realistic\_v6-v1/NANOAODSIM

```bash
 dasgoclient --query "release file=/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root"
```

> \["CMSSW\_10\_2\_9"]

해당 파일의 데이터셋과 CMSSW 버전을 확인하였습니다. CMSSW의 버전이 10\_2\_9 이므로 작업 공간은 이 버전과 비슷하거나 살짝 높아야 합니다. 여기서는 10\_2\_9 버전으로 작업을 진행하겠습니다.

```bash
cmsrel CMSSW_10_2_9
```

> Waiting for release information to be obtained via [https://cmssdt.cern.ch/SDT/releases.map?release=CMSSW\_10\_2\_9\&architecture=slc7\_amd64\_gcc700\&scram=V2\_2\_9\_pre04\&releasetop=/cvmfs/cms.cern.ch/slc7\_amd64\_gcc700/cms/cmssw/CMSSW\_10\_2\_9](https://cmssdt.cern.ch/SDT/releases.map?release=CMSSW\_10\_2\_9\&architecture=slc7\_amd64\_gcc700\&scram=V2\_2\_9\_pre04\&releasetop=/cvmfs/cms.cern.ch/slc7\_amd64\_gcc700/cms/cmssw/CMSSW\_10\_2\_9) (timeout in 7s) WARNING: Developer's area is created for non-production architecture slc7\_amd64\_gcc700. **Production architecture for this release is slc6\_amd64\_gcc700.**

위 메시지를 참고하여 사용S중인 ui서버의  환경을 SLC6 로 변경합니다.  또한, SCRAM\_ARCH=slc6\_amd64\_gcc700 환경변수를 이용하여 작업공간을 생성하고 src 디렉토리로 이동합니다.

```bash
ssh ui10
singularity shell -i -p /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el6:latest
SCRAM_ARCH=slc6_amd64_gcc700 cmsrel CMSSW_10_2_9
cd CMSSW_10_2_9
cd src
cmsenv
```

## KISTIBatch 실습 프로그램 다운로드&#x20;

HTCondor를 이용한 실습을 수행하기 위하여 간단한 CMSSW 분석 작업 예제를 다운로드합니다.

```bash
git clone https://github.com/geonmo/KISTIBatch.git
```

> Cloning into 'KISTIBatch'... \
> remote: Enumerating objects: 72, done. \
> remote: Counting objects: 100% (72/72), done. \
> remote: Compressing objects: 100% (58/58), done. \
> remote: Total 87 (delta 39), reused 37 (delta 14), pack-reused 15 Receiving objects: 100% (87/87), 21.25 KiB | 109.00 KiB/s, done. Resolving deltas: 100% (41/41), done. \
> \
> (base) \[geonmo@ui20 src]$ ls KISTIBatch/ \
> LFNTool.py README.md filedialog.py filelist.txt getZMass.py gui.py run.sh setup.sh submit.jds test\_LFNTool.py

각 파일들의 세부 설명은 GitHub 페이지를 참고해주시기 바랍니다.&#x20;

filstlist.txt 파일은 dasgoclient를 이용하여 생성할 수 있습니다. 다음 명령어를 참고해주시기 바랍니다.

```
## 파일 리스트 추출
dasgoclient --query="file dataset=/ZZ_TuneCP5_13TeV-pythia8/RunIIFall17NanoAODv4-PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/NANOAODSIM" --limit 0 > filelist.txt
## Prefix 추가. 데이터 파일의 위치를 알고 있다면 prefix에 해당 센터의 xrootd 경로를 넣으시면 됩니다.
sed -i 's/^/root:\/\/cmsxrootd.fnal.gov\//g' filelist.txt
### T3_KR_KISTI의 경우,
## sed -i 's/^/root:\/\/cms-xrdr.private.lo:2094\/\/xrd/g' filelist.txt
```

### 실습 코드에 관한 설명

해당 실습 코드는 getZMass.py 라는 분석 코드를 각 WN 머신에서 수행하게 됩니다. 이 때, 해당 분석코드는 첫번째 인자로 데이터 파일을 입력 받습니다.

```
./getZMass.py root://cms-xrdr.private.lo:2094//xrd/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/8DC8188D-246C-6D41-A24A-78C0CBBC6F17.root
```

> (base) \[geonmo@ui10 KISTIBatch]$ ls -l \*.root \
> \-rw-r--r--. 1 geonmo geonmo 3780 Sep 2 14:05 zcandmass.root

### HTCondor 작업 제출 명세 작성

HTCondor에 작업을 동시(concurrent)에 실행시키려면 HTCondor 문법에 맞게 작업 제출 파일을 작성하여야 합니다. 해당 제출 명세의 예제로 submit.jds 파일을 동봉하였습니다.

> batch\_name = kisti batch test \
> executable = run.sh \
> universe = container
>
> container\_image =  /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el9:latest\
> getenv = True\
> arguments = $(DATAFile)&#x20;
>
> transfer\_input\_files = getZMass.py , /tmp/x509up\_u556800422, LFNTool.py\
> should\_transfer\_files = YES \
> when\_to\_transfer\_output = ON\_EXIT
>
> output = job_$(Process).out_ \
> _error = job_$(Process).err \
> log = condor.log
>
> transfer\__output\_files = zcandmass.root_ \
> _transfer\_output\_remaps = "zcandmass.root = zcandmass\__$(Process).root"\
> accounting\_group=group\_cms\
> \+SingularityBind = "/cvmfs,/etc/condor,/cms,/cms\_scratch,/var/lib/condor,/run/user"\
> queue DATAFile from filelist.txt

위 작업은 filelist.txt을 읽어 한줄씩 $(DATAFile) 변수에 대입하며 하나씩 실행됩니다. 총 입력 파일이 3개이므로 총 3개의 작업이 제출됩니다.&#x20;

{% hint style="info" %}
해당 작업이 제출될 때마다 $(Process)가 1씩 증가하게 됩니다.&#x20;
{% endhint %}

WN에서 run.sh 스크립트를 통해 실행이 되며 인자로 $(DATAFile)이 입력됩니다. 이 때, run.sh는 CMSSW 실행을 위한 환경 설정과 ./getZMass.py을 실행합니다.

accounting\_group은 통합팜에서 작업을 실행하기 위해 필요합니다.

container universe는 apptainer 컨테이너 런타임 활용을 위해서 사용됩니다. 사용되는 이미지는 .sif 포맷이나 일반 디렉토리(혹은  /cvmfs)를 지정할 수 있습니다.

## 참고문헌

1. KISTIBatch GitHub Page : [https://github.com/geonmo/KISTIBatch](https://github.com/geonmo/KISTIBatch)
2.
