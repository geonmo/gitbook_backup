---
description: /xrootd 스토리지에 저장된 데이터에 접근하는 방법에 대하여 설명드립니다.
---

# 스토리지 접근 방법

## 데이터 사용량 확인

Google Sheet를 이용하여 사용자들의 데이터 사용량 확인이 가능합니다. 아래 링크를 확인하시기 바랍니다.

[구글 시트 모니터링 페이지](https://docs.google.com/spreadsheets/d/1mk335AFIkW7X4Pod8pecfTw0m1-S\_NTjNkxFzHEwAL0/edit#gid=1404507412)

## 데이터 쓰기 방법

UI서버들의 /xrootd 공간을 확인하시면 데이터 파일들의 경로와 위치 정보를 아실 수 있습니다. 일반적으로 사용자는 /xrootd공간 이후 어느 공간이든 사용이 가능하지만 다음과 같은 구조로 데이터들을 저장하고 있습니다.

#### xrdcp 명령어 사용하기

xrdcp는 xrootd 프로토콜을 이용한 데이터 전송 프로그램입니다. 사용방법은 scp와 매우 유사하지만 대상이 xrootd 프로토콜이어야 합니다. 테스트를 위해 CMS Tier-3 XRootD 디렉토리에 저장된 파일을 이용합니다. /xrootd 디렉토리는 각 UI 서버들에 마운트되어 있습니다.

```bash
[geonmo@ui20 geonmo]$ ls /xrootd/store/user/geonmo/dummy.dat 
/xrootd/store/user/geonmo/dummy.dat
[geonmo@ui20 geonmo]$ xrdcp -f root://cms-xrdr.private.lo:2094//xrd/store/user/geonmo/dummy.dat /dev/null
[100MB/100MB][100%][==================================================][100MB/s]  
```

#### xrdMigration.py 사용하기&#x20;

xrdMigration.py는 /cms/ldap_home,_ /cms/scratch, /cms\_scratch에 저장된 데이터 파일들을 /xrootd/store/user/\<USERID>로 옮기기 위해 개발되었습니다. 해당 /cms 디렉토리들의 경로를 지정하면 동일한 경로명으로 /xrootd 디렉토리로 옮겨줍니다.

{% tabs %}
{% tab title="옵션 설명" %}
```bash
[geonmo@ui20 ~]$ xrdMigration.py -d -c geonmo /cms/ldap_home/geonmo/migration_test/
Number of files is 1

xrdcp -p /cms/ldap_home/geonmo/migration_test/subdir root://cms-xrdr.private.lo:2094//xrd/store/user/geonmo/migration_test/subdir
xrdcp: /cms/ldap_home/geonmo/migration_test/subdir is a directory.
```

\-d                     : 복사 후 checksum 값 비교후 정상이면 기존 파일 삭제\
\-c \<CERN ID> : CERN ID를 기입하여 실행 속도 향상. -c 옵션이 없을 경우 CERN CRIC에 질의를 하여 사용자의 인증서 정보를 이용하여 CERN ID 정보를 가져오기 때문에 약 1분 가량의 지연시간이 발생합니다.
{% endtab %}
{% endtabs %}

xrdMigration.py 스크립트를 사용하면 새로 생성된 디렉토리의 캐쉬 정보를 리셋을 합니다. 따라서, 디렉토리를 생성하였으나 실제로 없는 디렉토리라면서 접근이 안되는 현상이 사라집니다. 되도록 xrdMigration.py를 사용하길 추천드립니다.

{% hint style="danger" %}
/xrootd 디렉토리에서는 절대로 mv 명령어를 사용하시면 안됩니다. 현재 시스템 상으로 mv를 하면 파일이 사라질 가능성이 존재합니다. \
(최근 파일의 경우 더더욱 확률이 높습니다.)
{% endhint %}

#### 데이터 접근 방식&#x20;

WN에서 스토리지 데이터를 접근하는 방법에 대해서 기술합니다.&#x20;

#### (1) CMS T3\_KR\_KISTI 데이터 접근

{% tabs %}
{% tab title="ROOT" %}
```
## XRootD 프로토콜로 접근
root -l root://cms-xrdr.private.lo:2094//xrd/store/user/<CERNID>/~~.root
or
datafile = TFile::Open("root://cms-xrdr.private.lo:2094//xrd/store/user/~~");
```
{% endtab %}

{% tab title="PyROOT" %}
```
## In pyROOT,
datafile = TFile.Open("root://cms-xrdr.private.lo:2094//xrd/store/user/~~")
```
{% endtab %}

{% tab title="CMSSW" %}
```
## T3 데이터의 경우, LFN 및XRootD PFN 접근 둘다 가능합니다.
process.source = cms.Source("PoolSource",                            
            fileNames = cms.untracked.vstring(
            '/store/user/~~~.root',
            'root://cms-xrdr.private.lo:2094//xrd/store/user/~~.root',
    )
)
```
{% endtab %}
{% endtabs %}

#### (2) CMS T2\_KR\_KISTI 데이터 접근

{% tabs %}
{% tab title="ROOT" %}
```
## XRootD 프로토콜로 접근
root -l root://cms-t2-se01.sdfarm.kr:1096//store/user/<CERNID>/~~.root
or
datafile = TFile::Open("root://cms-t2-se01.sdfarm.kr:1096//store/user/~~");
```
{% endtab %}

{% tab title="PyROOT" %}
```
## In pyROOT,
datafile = TFile.Open("root://cms-t2-se01.sdfarm.kr:1096//store/user/~~")
```
{% endtab %}

{% tab title="CMSSW" %}
```
## T2 데이터의 경우, XRootD PFN 접근 가능합니다.
process.source = cms.Source("PoolSource",                            
            fileNames = cms.untracked.vstring(
            'root://cms-t2-se01.sdfarm.kr:1096//store/user/~~.root',
    )
)
```
{% endtab %}
{% endtabs %}

## 디렉토리 정보&#x20;

#### /xrootd/store/mc

* 위 공간은 CMS DataSet 중 MC 샘플들이 저장되는 공간입니다.

#### /xrootd/store/data

* 위 공간은 CMS Dataset 중 Real Data가 저장되는 공간입니다.

#### /xrootd/store/himc

* 위 공간은 HeavyIon MC 샘플이 저장됩니다.

{% hint style="danger" %}
위 공간은 PhEDEx를 이용하여 데이터셋을 신청하여 파일을 전송하실 수 있습니다. 그외에 일반 유저가 해당 디렉토리에 파일을 쓰거나 삭제하지 않도록 주의해주시기 바랍니다.

다시 한번 말씀드리지만, XRootD 시스템의 기능의 한계로 인하여 일반 유저가 해당 공간에 대해 쓰기 권한을 가지고 있습니다. 해당 공간에 대해서 삭제 기능을 수행하지 않도록 주의를 기울여주시기 바랍니다.&#x20;
{% endhint %}

#### /xrootd/store/user

* 위 공간은 개별 사용자들의 데이터 샘플을 장기 보관하는 공간입니다.&#x20;
* 각 사용자들은 해당 공간에 데이터를 집어넣기 위해 gfal-copy, xrdcp 명령어를 사용하거나 UI 서버에서 /xrootd\_user/\[ID]/xrootd 디렉토리를 이용하여 데이터를 복사할 수 있습니다.&#x20;

{% hint style="warning" %}
/xrootd\_user 공간을 사용하실 때에는 mv 같은 암시적인 삭제 기능을 수행하지 않도록 주의해주시기 바랍니다. 해당 공간에서 mv를 실행하면 확률적으로 파일이 손실됩니다. 따라서, 꼭 cp로 복사한 후 복사파일들을 확인한 후 rm으로 지워주시기 바랍니다.
{% endhint %}

#### /xrootd/store/group

위 공간은 KCMS 연구자들이 생산한 공용 데이터들을 저장하고 있습니다.&#x20;
