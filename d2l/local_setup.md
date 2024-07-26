# steps for local setup

local location: 
```bash
(d2l) wei@wei-pc:~/d2l-zh/pytorch$ pwd
/home/wei/d2l-zh/pytorch
```

```bash
  902  sh ~/Downloads/Miniconda3-latest-Linux-x86_64.sh -b # initial install
  903  sh ~/Downloads/Miniconda3-latest-Linux-x86_64.sh -u # update
  904  ~/miniconda3/bin/conda init
  906  conda create --name d2l python=3.9 -y
  907  conda activate d2l # conda deactivate
  909  sudo apt install nvidia-cuda-toolkit
  910  nvcc --version
  911  pip install torch==1.12.0
  912  pip install torchvision==0.13.0
  913  pip install d2l==0.17.6


  914  mkdir d2l-zh && cd d2l-zh
  915  curl https://zh-v2.d2l.ai/d2l-zh-2.0.0.zip -o d2l-zh.zip
  916  unzip d2l-zh.zip && rm d2l-zh.zip
  917  cd pytorch
  918  jupyter notebook
```

