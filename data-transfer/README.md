---
description: >-
  We will tell you how to transfer data files and how to request CMS dataset
  transfer.
---

# Data transfer and dataset request

We explained the gsiftp and xroot protocols in the description of the [cms-tier-3.md](../cms-tier-3.md "mention"). You can transfer the files to KISTI Tier-3 at any time.

## 1. Data Transfer

### When you know the exact location of the data file

If you know the exact location of the data file, you can process the data transfer very quickly and easily. You can transfer data to the following commands:

xrdcp root://\[source of data file]//\[Path of source data file \[Dest]

```bash
xrdcp root://eoscms.cern.ch//store/mc/SAM/GenericTTbar/AODSIM/CMSSW_9_2_6_91X_mcRun1_realistic_v2-v1/00000/CE860B10-5D76-E711-BCA8-FA163EAA761A.root .

## xrdcp는 -r (하위 디렉토리 포함) 기능이 지원됩니다.
xrdcp root://eoscms.cern.ch//store/mc/SAM/GenericTTbar/AODSIM/CMSSW_9_2_6_91X_mcRun1_realistic_v2-v1/00000 .

```

{% hint style="warning" %}
The receive and send destination must be a local directory or an XRootD server.
{% endhint %}

### You do not know the location of the data file

If you don't know the location of the data file, you can find the location of the data and transfer it, or use the CMS Any Data, Anytime, Anywhere (AAA) feature to transfer data regardless of the location of the data file.

#### Find Data Files&#x20;

It is most convenient to use the dasgoclient or DAS home page to locate the data file.

```bash
dasgoclient --query="site file=/store/mc/RunIIFall17NanoAODv5/QCD_HT1000to1500_TuneCP5_13TeV-madgraph-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano1June2019_102X_mc2017_realistic_v7-v1/60000/04323B9F-4B66-2D44-89DE-644E8C48246D.root"

```

> T1\_RU\_JINR\_Disk \
> T1\_US\_FNAL\_Buffer \
> T1\_US\_FNAL\_MSS \
> T2\_KR\_KISTI \
> **T2\_US\_Purdue** \
> T2\_US\_Wisconsin \
> T3\_KR\_KISTI

Now, we checked the name of the saved site and check the prefix of the site.

```
cat /cvmfs/cms.cern.ch/SITECONF/T2_US_Purdue/PhEDEx/storage.xml | grep "root://"
```

> ```
>   <lfn-to-pfn protocol="xrootd" destination-match=".*" path-match="/+store/(.*)" result="root://cmsxrootd.fnal.gov//store/$1"/>
>   <lfn-to-pfn protocol="root" destination-match=".*" path-match="/+store/(.*)" result="root://xrootd.rcac.purdue.edu//store/$1"/>
>   <pfn-to-lfn protocol="root" destination-match=".*" path-match="root://xrootd.rcac.purdue.edu//store/(.*)" result="/$1"/>
>
> ```

Above, xrootd protocol endpoint is&#x20;

> [root://xrootd.rcac.purdue.edu//store/$1](root://xrootd.rcac.purdue.edu/store/$1)

Let's transfer the file using the prefix.

```
xrdcp root://xrootd.rcac.purdue.edu//store/mc/RunIIFall17NanoAODv5/QCD_HT1000to1500_TuneCP5_13TeV-madgraph-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano1June2019_102X_mc2017_realistic_v7-v1/60000/04323B9F-4B66-2D44-89DE-644E8C48246D.root .
```

#### Using Data Auto-Discovery (CMS AAA)

CMS AAA functionality allows you to transfer files without browsing the site. Transfer files with a fixed prefix (root://cmsxrootd.fnal.gov).

```
xrdcp root://cmsxrootd.fnal.gov//store/mc/RunIIFall17NanoAODv5/QCD_HT1000to1500_TuneCP5_13TeV-madgraph-pythia8/NANOAODSIM/PU2017_12Apr2018_Nano1June2019_102X_mc2017_realistic_v7-v1/60000/04323B9F-4B66-2D44-89DE-644E8C48246D.root .
```

{% hint style="info" %}
Currently, CMS AAA does not offer additional features, such as searching for the fastest sites. Therefore, even if you want to transfer files from other sites in the country, there is a possibility of transferring files from Europe or the United States. If you have more files to send, let's search directly for data and put a prefix to transfer the file.
{% endhint %}

## 2. Request to send dataset&#x20;

#### (1) Rucio (For Public Dataset)

You must be assigned a space quota for T3\_KR\_KISTI to transfer data through the current lucio command. If you need a quota, please request by e-mail(mailto:cmst3-support@kisti.re.kr) with the following information.

1. CERN Computing ID
2. Requested quota capacity (up to 10TB applicable without additional proof)
3. Sites requesting quotas (T2\_KR\_KISTI or T3\_KR\_KISTI)

* Enable the rucio command with the following command: (We also provide `rucioenv` instead.)

```
###CMSSW 디렉토리 내에서 cmsenv를 하였을 경우 작동 안될 가능성 있음.
source /cvmfs/cms.cern.ch/cmsset_default.sh 
source /cvmfs/cms.cern.ch/rucio/setup.sh 
export RUCIO_ACCOUNT=<CERN ID>
voms-proxy-init --voms cms

### or below command
rucioenv 
```

* Proceed with the request to send the dataset with the command below.

```
### If you have enough quota,
rucio add-rule cms:<DATASET> 1 T3_KR_KISTI

## If you do not have enough quota,
rucio add-rule --ask-approval cms:<DATASET> 1 T3_KR_KISTI
```

* You can check your transfer request status using below command.

```
rucio list-rules --account geonmo
```

* Output Format
  * \[rucio ID as HexCode] \[User] cms:\[DataSet] \[Status/ OK, REPL,STUCK] \[Tier Center] \[COPIES]
    * OK : Completed files
    * REPL : Transferring files
    * STUCK : File being retried because transfer failed

> 20e8a18cb7e04a75b8d8106baf4ece01 geonmo cms:/gluinoGMSB\_M3000\_ctau10000p0\_TuneCP2\_13TeV\_pythia8/RunIISummer16NanoAODv7-PUMoriond17\_Nano02Apr2020\_102X\_mcRun2\_asymptotic\_v8-v1/NANOAODSIM OK\[9/0/0] T3\_KR\_KISTI 1

* 총 9개의 파일 중 9개 전송 완료 \[OK:9, REPL: 0, STUCK:0]

#### (2) xrdDownload.py (for User Private Dataset)

Dataset created and published by an individual user cannot be sent through rucio directly yet. As a result, we provide scripts that make it easy to transfer each file. Please follow the procedure below to transfer the data.

* [hep-tools](https://github.com/cms-kr/hep-tools) Git 저장소로부터 dataset2filelist.sh와 xrdDownload.py 를 본인의 PATH 경로로 다운로드 받습니다.
* dataset2filelist.sh 를 이용하여 다운로드 받으려는 데이터셋들의 개별 파일 목록을 작성합니다.

{% code title="input_dataset.txt" %}
```bash
/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/swertz-TopNanoAODv6-1-1_2018-0d1d4920f08f56d048ece029b873a2cc/USER
/TTTo2L2Nu_TuneCP5_13TeV-powheg-pythia8/palencia-TopNanoAODv6-1-1_2018-0d1d4920f08f56d048ece029b873a2cc/USER
/TTToHadronic_TuneCP5_13TeV-powheg-pythia8/swertz-TopNanoAODv6-1-1_2018-0d1d4920f08f56d048ece029b873a2cc/USER
```
{% endcode %}

{% tabs %}
{% tab title="Bash" %}
```bash
[geonmo@ui20 migration]$ dataset2filelist.sh input_dataset.txt datalist.txt
```

dataset2filelist.sh \<from dataset list file> \<to data list result file>
{% endtab %}
{% endtabs %}

{% code title="datalist.txt" %}
```bash
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_597.root   615580493  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_24.root   615219932  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_17.root   615423490  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_387.root   616238957  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_538.root   615412910  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_32.root   615054868  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_535.root   615491181  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_405.root   615219946  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_622.root   615852354  
/store/user/swertz/topNanoAOD/v6-1-1/2018/TTToSemiLeptonic_TuneCP5_13TeV-powheg-pythia8/TopNanoAODv6-1-1_2018/200610_101606/0000/tree_147.root   615512903 
```
{% endcode %}

* xrdDownload.py 스크립트에 해당 파일리스트를 이용하여 전송을 합니다.

{% tabs %}
{% tab title="Bash" %}
```bash
[geonmo@ui20 migration]$ xrdDownload.py -i datalist.txt -p 4
missing file                                                                                                                                                
missing file                                                                                                                                                
missing file
missing file
missing file
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0001/nano_1444.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0001/nano_1444.root
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0002/nano_2098.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0002/nano_2098.root
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_229.root
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_343.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_229.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_343.root

/store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_229.root is failed.        
Total: 1565 / Success: 1 / Fail: 0

/store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_343.root is failed.          
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0001/nano_1009.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0001/nano_1009.root
Total: 1565 / Success: 2 / Fail: 0

/store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0001/nano_1444.root is failed.         
Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_319.root
/store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0002/nano_2098.root is failed.         
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_319.root
Total: 1565 / Success: 3 / Fail: 0

Source:  root://cmsxrootd.fnal.gov//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_853.root
Destination:  root://cms-xrdr.private.lo:2094//xrd//store/group/lpccoffea/coffeabeans/NanoAODv6/nano_2016/MET/NanoTuples-2016_Run2016B-17Jul2018_ver2-v1/191210_034740/0000/nano_853.root
Total: 1565 / Success: 4 / Fail: 0


```

-i \<Data list file> : 데이터 파일 리스트를 지정합니다. 복사 과정 중에 이미 전송된 파일들의 경우 자동으로 빠지기 때문에 전체 리스트를 넣으면 됩니다.

-p \<num of parallel> : 동시에 전송할 파일 개수를 지정합니다. 4개가 적당하며 6개 이상부터는 후반에 에러가 발생할 확률이 높습니다. 네트워크 상태에 따라 선택하면 됩니다.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
xrdDownload.py는 기존 파일의 크기를 이용하여 상태를 확인합니다. 따라서, 일부만 전송되었거나 전송되지 않은 파일들만 전송을 시도하기 때문에 도중에 작업을 취소한 후에 다시 시작할 수 있습니다.
{% endhint %}

## 3. 전송 요청한 데이터셋 삭제 (쿼터 회복)

rucio 로 전송한 데이터셋은  rucio delete-rule 명령어로 삭제가 가능합니다. 단, 다른 사용자가 같은 데이터셋을 신청한 상태라면 데이터셋 자체는 삭제되지 않습니다. 하지만, 쿼터는 복구되므로 필요한 쿼터를 확보할 수 있습니다.

```bash
[geonmo@ui20 ~]$ rucio list-rules --account geonmo
ID                                ACCOUNT    SCOPE:NAME
                                    STATE[OK/REPL/STUCK]    RSE_EXPRESSION      COPIES  EXPIRES (UTC)    CREATED (UTC)
--------------------------------  ---------  ------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------  ----------------------  ----------------  --------  ---------------  -------------------
2de8f9060ac94da38b898d9fb267d16f  geonmo     cms:/StealthSHH_2t4b_mStop-1200_mSo-100_TuneCP2_13TeV-madgraphMLM-pythia8/RunIIAutumn18NanoAODv7-Nano02Apr2020_102X_upgrade2018_realistic_v21-v1
/NANOAODSIM                         OK[8/0/0]               T3_KR_KISTI              1                   2020-08-12 01:30:18

[geonmo@ui20 ~]$ rucio delete-rule 2de8f9060ac94da38b898d9fb267d16f
```

## 4. rucio 사용량 및 쿼터 확인

rucio를 호라용하실 때 사용량 및 쿼터 용량 확인이 필요하신 경우가 있습니다. 아래와 같이 확인하시기 바랍니다.

### (1) 쿼터 확인

```bash
[geonmo@ui20 ~]$ rucio list-account-limits geonmo                                                                                                        
/cvmfs/cms.cern.ch/rucio/x86_64/slc7/py2/current/lib/python2.7/site-packages/paramiko/transport.py:33: CryptographyDeprecationWarning: Python 2 is no longer
 supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.                              
  from cryptography.hazmat.backends import default_backend                                                                                                  
+-------------+-----------+                                                                                                                                  
| RSE         | LIMIT     |                                                                                                                                  
|-------------+-----------|                                                                                                                                  
| T2_KR_KISTI | 20.000 TB |
| T3_KR_KISTI | 20.000 TB |
+-------------+-----------+
+------------------+---------+
| RSE EXPRESSION   | LIMIT   |
|------------------+---------|
+------------------+---------+

```

### (2) 사용량 확인

```bash
[geonmo@ui20 ~]$ rucio list-rse-usage --show-accounts T3_KR_KISTI
USAGE:
------
  used: 16.726 TB
  rse: T3_KR_KISTI
  updated_at: 2021-09-08 02:20:32
  source: expired
  rse_id: 1983977ffc0e47ffb2b1c4d496a8297f
------
  used: 4.157 TB
  rse: T3_KR_KISTI
  updated_at: 2021-09-08 02:20:07
  source: obsolete
  rse_id: 1983977ffc0e47ffb2b1c4d496a8297f
------
  files: 78660
  used: 109.455 TB
  rse: T3_KR_KISTI
  updated_at: 2021-09-02 06:46:12
  source: rucio
  per account:
  ------
    account: sync_t3_kr_kisti       used: 64.192 TB                 percentage: 58.65                                                                      
  ------
    account: jlee                   used: 12.457 TB                 percentage: 11.38                                                                      
  ------
    account: geonmo                 used: 7.077 TB                  percentage: 6.47                                                                       
  ------
    account: heewon                 used: 4.831 TB                  percentage: 4.41                                                                       
  ------
    account: yhoonlee               used: 3.691 TB                  percentage: 3.37                                                                       
  ------
    account: jichoi                 used: 2.658 TB                  percentage: 2.43                                                                       
  ------
    account: soohwan                used: 2.204 TB                  percentage: 2.01                                                                       
  ------
    account: mchoi                  used: 1.995 TB                  percentage: 1.82                                                                       
  ------
    account: yjeong                 used: 1.507 TB                  percentage: 1.38                                                                       
  ------
    account: haoh                   used: 128.632 GB                percentage: 0.12                                                                       
  ------
    account: gylee                  used: 50.821 GB                 percentage: 0.05                                                                       
  ------
    account: dakang                 used: 10.063 GB                 percentage: 0.01                                                                       
  ------
  rse_id: 1983977ffc0e47ffb2b1c4d496a8297f
------
  used: 15.442 GB
  rse: T3_KR_KISTI
  updated_at: 2021-09-02 15:00:13
  source: unavailable
  rse_id: 1983977ffc0e47ffb2b1c4d496a8297f
------

```
