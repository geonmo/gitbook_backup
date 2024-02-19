---
description: >-
  최근들어 빅데이터 분석 시스템을 이용한 CMS 데이터 분석을 활용하시는 사용자분들이 늘어나고 있습니다. 이에 따라, 해당 환경 구축 방법에
  대해 알려드립니다.
---

# 빅데이터 분석 환경 구축하기

## Apptainer를 이용한 PyTorch 사용하기

pytorch 사용을 위한 NGC 컨테이너 사용을 추천드립니다.

```
## 컨테이너 이미지 다운로드 (/tmp 20GB 이상 필요)
apptainer pull docker://nvcr.io/nvidia/pytorch:23.11-py3
apptainer shell --nv pytorch_23.11-py3.sif
```

## Apptainer를 이용한 TensorFlow 사용하기

현재 제공되는 GPU 서버에서는 tensorflow를 바로 실행하실 수 없습니다. Singularity 가상 환경을 추천드립니다.

```
## CMS GPU 서버에서 실행하세요.
apptainer pull docker://nvcr.io/nvidia/tensorflow:23.11-tf2-py3
apptainer shell --nv tensorflow_23.11-tf2-py3.sif
```

HTCondor를 이용한 작업 제출은 다음과 같습니다.

```
# Job description file for condor job cat_jpsi_ftl_full
jobbatchname = GPU test
executable = gpu_test.sh
universe   = container
container_image = tensorflow_23.11-tf2-py3.sif
arguments  = $(Process)
log = condor.log

getenv     = True
should_transfer_files = YES
when_to_transfer_output = ON_EXIT

output = job_$(Process).out
error = job_$(Process).err

#+SingularityBindCVMFS = True

request_cpus=1
request_GPUs = 1

queue 1
```

## 주피터 노트북 환경

주피터 노트북은 CMS GPU 서버에서 써보시는 것을 추천드립니다. (P100 그래픽카드 활용 가능)

&#x20;본인 컴퓨터에서 8888 -> localhost:8889 로 지정하시고, UI 서버에서 8889번 포트를 GPU서버의 localhost:8888번으로 연결하신 후,  jupyter의 포트 번호를 8888로 여시면 됩니다.&#x20;

MobaXterm 사용자는 다음 링크를 참조하여 본인 컴퓨터측 설정을 완료하시기 바랍니다. 비슷한 방식으로 putty에서 설정이 가능합니다.  ([https://developyo.tistory.com/84](https://developyo.tistory.com/84))

이후 아래 설정대로 진행하시면 됩니다.

```
ssh -Y cms-gpu01
### ssh local tunneling을 통해 간접 접속이 가능합니다.
# 본인의 데스크탑에서 접속을 하시려면 2중 포트 포워딩을 하세요.
# ### ui -> [CMS GPU 서버 FQDN]
# ex) ssh -4 -L 8889:localhost:8888 $USER@[CMS GPU 서버FQDN]
```

### VirtualEnv 을 이용한 Python 환경 구축

VirtualEnv 를 통해 Python 설치 환경을 일반 사용자로서 설치가 가능하도록 합니다.

```
cd $HOME
### PyROOT 실행을 위해서는 python2 필요. ROOT 사용 안하면 python3 권장.
virtualenv -p python2 my_workspace
```

### 작업 공간에서 Python 패키지 설치

대부분의 경우 해당 패키지들을 pip로 설치하게 됩니다.

```
cd my_workspace
source bin/activate
mkdir $HOME/pip_cache
TMPDIR=$PWD pip install --cache-dir=$HOME/pip_cache --build $PWD/build jupyter pandas seaborn torch sklearn  metakernel zmq
```

### &#x20;Jupyter 실행&#x20;

```
### 나중에 다시 실행하기 위해서,
source my_workspace/bin/activate
jupyter notebook
## 혹은 포트 개방 후 본인 컴퓨터의 브라우저에서 지정한 포트로 연결하여 사용 가능
jupyter notebook --no-browser --ip 0.0.0.0 --port XXXX
```

### local 컴퓨터에서 접속&#x20;

제대로 접속이 되었다면 본인 컴퓨터의 브라우저에서 localhost:8888로 접속이 가능합니다.

![Token을 물어봅니다.](<.gitbook/assets/image (5).png>)

토큰 정보는 jupyter를 실행할 때 표시되므로 해당 내용을 복사&붙여넣기로 접속이 가능합니다.
