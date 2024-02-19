---
description: 이 페이지에서는 KISTI-GSDC CMS Tier-3의 컴퓨팅 환경 및 구성에 대하여 알려드립니다.
---

# CMS Tier-3 컴퓨팅 환경 설명

## 컴퓨팅 자원 소개

![KISTI CMS Tier-3 구조도](.gitbook/assets/CMS\_Tier3\_구조도.png)

### 계산 자원

CMS Tier-3는 ALICE Tier-3인 KiAF 클러스터, 유전체사용자와클러스터 통합 환경으로 구성되어 있습니다. CMS Tier-3의 최저보장 논리CPU코어의 쿼터는  총 2,288개입니다. (물리코어는 절반인 1,144개)

&#x20;KIAF의 1696, BIO의 1160개와 합쳐 총 5,144개의 작업 슬롯을 가지고 있습니다.

CPU 코어들은 HTCondor 작업 관리자를 이용하여 자원을 배치하거나 회수합니다. 따라서, 작업을 제출하기 위해서는 HTCondor 사용방법을 알아야 합니다.

또한, 다른 그룹 사용자의 작업이 없을 경우에는 2,288개를 넘게 사용할 수 있습니다. 하지만, KIAF 사용자가 작업을 제출할 경우 KIAF 몫인 1696개의 코어를 자동으로 반납하며 반납된 작업 슬롯에서 실행되던 작업은 취소되어 대기열에 들어갑니다.

자세한 사용방법은 다음 페이지인 [HTCondor를 이용한 배치 작업 수행](usingbatchjob/htcondor.md) 참고해주시기 바랍니다.

### GPU 자원

KISTI-GSDC CMS Tier-3에는 머신러닝, GPGPU 기능 제공을 위해 GPU 서버를 1대 구비하고 있습니다.  이를 사용하시려면 해당 UI 서버로 직접 접속하셔서 사용을 권장드립니다.

해당 서버로 접속하기 위해 아래 명령어를 사용하십시오.

```bash
## 외부에서는 접근이 안됩니다. (ui 서버에서 접속하세요.)
ssh [CMS GPU Server]
```

#### 1. /cms (사용자 홈디렉토리 및 스크래치 공간)

/cms 공간은 사용자들의 홈디렉토리 및 임시 작업물 보관을 위한 스크래치 용으로 제공됩니다. 총 저장공간은 20TB로 NAS 형식으로 마운트되어 있습니다. 홈디렉토리와 스크래치 공간은 서로 같은 공간을 공유하여 사용합니다.&#x20;

스크래치를 너무 많이 사용하면 홈디렉토리 사용에도 지장을 주기 때문에 결과물의 크기를 미리 예측하여 문제가 발생하지 않도록 주의하여 사용하시기 바랍니다.

{% hint style="danger" %}
/cms 디렉토리는 모든 사용자가 2.5TB만 사용하실 수 있습니다. 추가 용량이 필요하시다면 아래 설명된 /cms\_scratch를 활용하시거나 스토리지로 바로 전송하도록 구성하여야 합니다.
{% endhint %}

#### 2. /cms\_scratch (배치 스크래치 공간)

공간 부족에 의하 추가로 10TB를 할당받은 공간입니다.&#x20;

{% hint style="danger" %}
해당 공간은 UI 및 WN에서 접근이 가능하지만 **파일 생성 후 7일**이 지나면 모든 파일이 삭제됩니다.
{% endhint %}

#### 3./xrootd (장기 보관 스토리지)

CMS Tier-3는  KCMS 연구자들에게 독점적인 1004TB 자원을 제공하고 있습니다. 이 중, 604TB는 SAN 스토리지이며, 400TB는 NAS 스토리지를 통하여 제공되고 있습니다.&#x20;

스토리지로 제공되는 서버는 총 5대이며 각각 10Gbps NIC로 연결되어 있습니다. 따라서,이론적인  최대 대역폭은 50Gbps입니다.

각 스토리지 서버는 XRootD 프로그램을 통하여 통합되어 있습니다. 따라서, xrootd 명령어를 통해 파일을 복사하거나 삭제할 때 성능이 가장 좋습니다.&#x20;

GSIFTP 기능도 제공하기 때문에 PhEDEx 및 gfal util을 이용한 SRM 프로토콜 사용도 가능합니다. 하지만, xrootd와 gsiftp를 사용할 때 URL이 다르므로 주의하여야 합니다.

| Protocol  |                    GSIFTP                    |              XROOT for Public             |             XROOT for Internal             |
| --------- | :------------------------------------------: | :---------------------------------------: | :----------------------------------------: |
| Prefix    |                   gsiftp://                  |                  root://                  |                   root://                  |
| Server    |               cms-se.sdfarm.kr               |             cms-xrdr.sdfarm.kr            |             cms-xrdr.private.lo            |
| Directory |                //xrootd/store                |                //xrd/store                |                 //xrd/store                |
| Port      |                     2811                     |                    1094                   |                    2094                    |
| Client    |        gfal util (gfal-copy, gfal-ls)        |                xrdfs, xrdcp               |                xrdfs, xrdcp                |
| Full URL  | gsiftp://cms-se.sdfarm.kr:2811//xrootd/store | root://cms-xrdr.sdfarm.kr:1094//xrd/store | root://cms-xrdr.private.lo:2094//xrd/store |



