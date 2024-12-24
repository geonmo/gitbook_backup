---
description: KISTI Tier3에 CRAB3를 사용할 때 주의사항에 관해 알려드립니다.
---

# CRAB3 사용 관련

## CRAB3 설정

CRAB3의 자세한 설정은 [CRAB3 페이지](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCRAB3Tutorial#Output_dataset_publication) 를 참고하시기 바랍니다. 여기서는 주요내용에 대해서만 다룹니다.

### CRAB3 결과를 KISTI에 저장하기

Tier-3에 결과파일을 저장하기 위해서는 아래 옵션을 사용합니다. 저장된 데이터셋은 /xrootd/store/user/\<CERN ID> 밑에서 확인하실 수 있습니다.

{% code title="crabConfig.py" %}
```python
config.General.transferOutputs = True
config.Site.storageSite = T3_KR_KISTI
# T2에 저장하려면
config.Site.storageSite = T2_KR_KISTI
```
{% endcode %}

{% hint style="danger" %}
T3\__KR\_KISTI에  저장된 데이터는 더이상 CRAB 작업을 수행할 수 없습니다. CRAB 활용이 필요하다면 해당 데이터셋을 반드시 T2\_KR\_KISTI로 저장하세요._
{% endhint %}

### 2차 작업이 필요한 경우

2차 작업이 필요할 경우 해당 데이터셋을 반드시 publish하여야 합니다. 그렇지 않으면 CRAB에서는 사용이 불가능합니다.

{% code title="crabConfig.py" %}
```python
config.General.transferOutputs = True
config.Site.storageSite = T2_KR_KISTI
config.Data.publication = True
config.Data.outputDatasetTag = '<DATASET's NAME>'
```
{% endcode %}

생성된 데이터셋을 대상으로 작업을 돌릴 경우에는 DBS Instance를 "global"에서 "phys03"으로 변경해야 합니다.

{% code title="crabConfig_2nd.py" %}
```python
config.Data.inputDataset = '<DATASET's NAME>/USER'
config.Data.inputDBS = 'phys03'
```
{% endcode %}
