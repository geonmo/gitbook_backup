---
description: This page shows you how to submit and manage your jobs using HTCondor.
---

# Perform batch job using HTCondor

## Setup of CMSSW  Environment

In this example, you will run an analysis using the ZZ sample. You need to look at the data samples and see what kind of job environment you need to build.

### Checking the data file

Use one ZZ sample data file to view dataset information. The location of the file is as follows:

> /xrootd/store/mc/RunIIFall17NanoAODv4/ZZ\_TuneCP5\_13TeV-pythia8/NANOAODSIM/PU2017\_12Apr2018\_Nano14Dec2018\_102X\_mc2017\_realistic\_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root

The Logical File Name (LFN) address used by CMS must begin with "/store". All you have to do is subtract /xrootd from the top. The corresponding LFN is the same for any site.

Let's check the DataSet and Release information in the above file.

```bash
### 임의의 CMSSW 작업 공간에서 cmsenv를 수행한 후,
dasgoclient --query "dataset file=/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root"
```

> /ZZ\_TuneCP5\_13TeV-pythia8/RunIIFall17NanoAODv4-PU2017\_12Apr2018\_Nano14Dec2018\_102X\_mc2017\_realistic\_v6-v1/NANOAODSIM

```bash
 dasgoclient --query "release file=/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/E13A1753-58AD-2E4A-9D47-4BA07BB3FD64.root"
```

> \["CMSSW\_10\_2\_9"]

We checked the dataset and CMSSW version of the file. Since CMSSW is version 10\_2\_9, the workspace should be similar to or slightly higher than this version. We'll work on the 10\_2\_9 version here.

```bash
cmsrel CMSSW_10_2_9
```

> Waiting for release information to be obtained via [https://cmssdt.cern.ch/SDT/releases.map?release=CMSSW\_10\_2\_9\&architecture=slc7\_amd64\_gcc700\&scram=V2\_2\_9\_pre04\&releasetop=/cvmfs/cms.cern.ch/slc7\_amd64\_gcc700/cms/cmssw/CMSSW\_10\_2\_9](https://cmssdt.cern.ch/SDT/releases.map?release=CMSSW_10_2_9\&architecture=slc7_amd64_gcc700\&scram=V2_2_9_pre04\&releasetop=/cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_2_9) (timeout in 7s) WARNING: Developer's area is created for non-production architecture slc7\_amd64\_gcc700. **Production architecture for this release is slc6\_amd64\_gcc700.**

Referring to the above message, you need to setup SLC6 environment using singularity and use the SCRAM\_ARCH=slc6\_amd64\_gcc700 environment variable to create a workspace and navigate to the src directory.

```bash
ssh ui10
singularity shell -i -p /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el6:latest
SCRAM_ARCH=slc6_amd64_gcc700 cmsrel CMSSW_10_2_9
cd CMSSW_10_2_9
cd src
cmsenv
```

## Download the KISTIBatch Tutorial Program&#x20;

Download a simple CMSSW analysis example to perform the exercise using HTCondor.

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

Please refer to the GitHub page for a detailed description of each file.

The filestlist.txt file can be created using the `dasgoclient`. Please refer to the following command.

```
## 파일 리스트 추출
dasgoclient --query="file dataset=/ZZ_TuneCP5_13TeV-pythia8/RunIIFall17NanoAODv4-PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/NANOAODSIM" --limit 0 > filelist.txt
## Prefix 추가. 데이터 파일의 위치를 알고 있다면 prefix에 해당 센터의 xrootd 경로를 넣으시면 됩니다.
sed -i 's/^/root:\/\/cmsxrootd.fnal.gov\//g' filelist.txt
### T3_KR_KISTI의 경우,
## sed -i 's/^/root:\/\/cms-xrdr.private.lo:2094\/\/xrd/g' filelist.txt
```

### Description of the lab code

The lab code will be performed on each WN machine with an analysis code called getZMass.py. At this time, the analysis code receives the data file as the first argument.

```
./getZMass.py root://cms-xrdr.private.lo:2094//xrd/store/mc/RunIIFall17NanoAODv4/ZZ_TuneCP5_13TeV-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano14Dec2018_102X_mc2017_realistic_v6-v1/30000/8DC8188D-246C-6D41-A24A-78C0CBBC6F17.root
```

> (base) \[geonmo@ui10 KISTIBatch]$ ls -l \*.root \
> -rw-r--r--. 1 geonmo geonmo 3780 Sep 2 14:05 zcandmass.root

### Create HTCondor Job Submission Specification

To run a job on HTCondor, you must create a job submission file in accordance with HTCondor's grammar. We have enclosed the submit.jds file as an example of this submission.

> batch\_name = kisti batch test \
> executable = run.sh \
> universe = vanilla \
> getenv = True\
> arguments = $(DATAFile)&#x20;
>
> transfer\_input\_files = getZMass.py , /tmp/x509up\_u556800422, LFNTool.py\
> should\_transfer\_files = YES \
> when\_to\_transfer\_output = ON\_EXIT
>
> output = jo&#x62;_$(Process).out_ \
> _error = job_$(Process).err \
> log = condor.log
>
> transfer\__output\_files = zcandmass.root_ \
> _transfer\_output\_remaps = "zcandmass.root = zcandmass\__$(Process).root"\
> accounting\_group=group\_cms\
> +SingularityImage = "/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el6:latest" \
> +SingularityBind = "/cvmfs, /cms, /share"\
> queue DATAFile from filelist.txt

The above job is performed one by one by reading filelist.txt and substituting each line into the $(DATAFile) variable. Since there are three input files in total, a total of three jobs are submitted.

{% hint style="info" %}
Each time that job is submitted, $(Process) will increase by 1.
{% endhint %}

Run at WN.It is executed through the bash script and $(DATAFile) is entered as a argument. At this time, run.sh runs the configuration settings and ./getZMass.py for CMSSW execution.

The accounting\_group is required to run the job in the T3\_KR\_KISTI farm.

The +SingularityImageBind knob is used for environment configuration other than EL7. You can adjust the location of the image file to use other OSs such as EL6 or Ubuntu. If you are interested in image files, please check the images of other users in the `/cvmfs/singularity.opensciencegrid.org/opensciencegrid/` directory or the `/cvmfs/singularity.opensciencegrid.org/` directory.

## 참고문헌

1. KISTIBatch GitHub Page : [https://github.com/geonmo/KISTIBatch](https://github.com/geonmo/KISTIBatch)
2.
