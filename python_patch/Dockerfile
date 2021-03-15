#FROM u16c91nn7:dev1
FROM nvidia/cuda:9.1-cudnn7-devel-ubuntu16.04
MAINTAINER alvin
 
# account info (pwd is not necessary for this config)
ARG user=docker
#ARG pwd=1234
#this script will create dir to  /home/{$user}/${worddir} like /home/docker/git_repository
ARG workdir=git_repository
#some large package should copy to /home/${user}/$(local_package}
ARG local_package=utils_thisbuild
ARG github=cei_speaker_embedded
#vscode server 1.54.2
ARG vscommit=fd6f3bce6709b121a895d042d343d71f317d74e7

# init.
RUN apt-get update && apt-get install -y \
    apt-utils sudo vim zsh curl git make unzip \
    wget openssh-server \
    powerline fonts-powerline \
    # necessary ubuntu package for sudo add-apt-repository ppa:deadsnakes/ppa
    software-properties-common \
    # zsh by ssh issue : icons.zsh:168: character not in range
    language-pack-en \
    libsndfile1 

# udpate timezone
RUN apt-get update \
    &&  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata
    
RUN TZ=Asia/Taipei \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata 


# create account/ssh setup
RUN useradd -m ${user} && echo "${user}:${user}" | chpasswd && adduser ${user} sudo;\
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers;\
    chmod 777 /etc/ssh/sshd_config; echo 'GatewayPorts yes' >> /etc/ssh/sshd_config; chmod 644 /etc/ssh/sshd_config

USER ${user}

# oh-my-zsh setup
ARG omzthemesetup="POWERLEVEL9K_MODE=\"nerdfont-complete\"\n\
ZSH_THEME=\"powerlevel9k\/powerlevel9k\"\n\n\
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(pyenv virtualenv context dir vcs)\n\
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status root_indicator background_jobs history time)\n\
POWERLEVEL9K_VIRTUALENV_BACKGROUND=\"green\"\n\
POWERLEVEL9K_PYENV_PROMPT_ALWAYS_SHOW=true\n\
POWERLEVEL9K_PYENV_BACKGROUND=\"orange1\"\n\
POWERLEVEL9K_DIR_HOME_SUBFOLDER_FOREGROUND=\"white\"\n\
POWERLEVEL9K_PYTHON_ICON=\"\\U1F40D\"\n"


RUN cd ~/ ; mkdir .ssh ;\
    sudo mkdir /var/run/sshd ;\
    sudo sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd ;\
    sudo ssh-keygen -A ;\
    #sudo service ssh start ;\
    wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true ;\
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting ;\
    git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k ;\
    git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions ;\ 
    git clone https://github.com/davidparsson/zsh-pyenv-lazy.git ~/.oh-my-zsh/custom/plugins/pyenv-lazy ;\
    echo "source ~/.oh-my-zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc ;\
    sed -i -r "1s/^/export TERM=\"xterm-256color\"\n/" ~/.zshrc ;\ 
    sed -i -r "2s/^/LC_ALL=\"en_US.UTF-8\"\n/" ~/.zshrc ;\
    sed -i -r "s/^plugins=.*/plugins=(git zsh-autosuggestions virtualenv screen pyenv-lazy)/" ~/.zshrc ;\
    sed -i -r "s/^ZSH_THEM.*/${omzthemesetup}/" ~/.zshrc ;\
    wget https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/SourceCodePro.zip ;\
    unzip SourceCodePro.zip -d ~/.fonts ;\
    fc-cache -fv  ;\
    sudo chsh -s $(which zsh) ${user}

# python package setup
ARG cudapathsetup="alias python=\'python3\'\n\
alias pip=\'pip3\'\n\
alias watch1=\'watch -n 0.5\'\n\
#export PATH=\"/usr/local/cuda/bin:$PATH\"\n\
export PATH=\"/home/${user}/.local/bin:$PATH\"\n\
export LD_LIBRARY_PATH=\"/usr/local/cuda/lib64:$LD_LIBRARY_PATH\""

RUN export PATH="/home/${user}/.local/bin:$PATH";\
    sudo add-apt-repository ppa:deadsnakes/ppa -y 
RUN sudo apt-get update -y; \
    sudo apt-get autoremove python3.5 -y; \
    sudo apt-get install python3.6 -y;\
    sudo apt-get install python3-pip -y;\
    echo ${cudapathsetup} >> ~/.zshrc;\
    cd /usr/bin; sudo unlink python3; sudo ln -s python3.6 python3; \
    sudo unlink python; sudo ln -s python3.6 python; cd ~/ ;\
    mkdir ${workdir}; mkdir ${local_package};\
    cd ${workdir}
    
    
# copy local package
#ADD ./${local_package} /home/${user}/${local_package} 
#WORKDIR /${local_package}
COPY ${local_package} /home/${user}/${local_package}/
#COPY ${local_package}/torch-1.5.0+cu101-cp36-cp36m-linux_x86_64.whl /home/${user}/${local_package}/
#COPY ${local_package}/vscode-server-linux-x64.tar.gz /home/${user}/${local_package}/
#RUN sudo chown ${user}:${user} /home/${user}/${local_package}
#ADD ${local_package} /home/${user}/${local_package}    

# numpy=1.19.4, pytorch=1.5, torchvision=0.6
RUN python3 -m pip install --upgrade pip;\
    python3 -m pip install numpy==1.19.4;\
    #python3 -m pip install torch==1.5.0+cu101 torchvision==0.6.0+cu101 -f https://download.pytorch.org/whl/torch_stable.html;\
    python3 -m pip install /home/${user}/${local_package}/torch-1.5.0+cu101-cp36-cp36m-linux_x86_64.whl;\
    #python3 -m pip install torch==1.5.0;\ # BUG: torch.cuda.is_available() = False
    #python3 -m pip install torchtext==0.6;\
    # project related package   
    python3 -m pip install matplotlib==3.3.4;\
    python3 -m pip install visdom==0.1.8.9;\
    #python3 -m pip install umap==0.1.1;\ # bug on GE2E encoder_train
    python3 -m pip install umap-learn==0.3.10;\
    python3 -m pip install sklearn==0.0;\
    # project git clone
    git clone https://github.com/AlvinYC/${github}.git /home/${user}/${workdir}/${github};\
    ## speical solution in this project
    # (1) Fix issue: Fix issue: clip_grad_norm_ fails when parameters are on different device
    cp /home/${user}/${workdir}/${github}/python_patch/clip_grad.py /home/${user}/.local/lib/python3.6/site-packages/torch/nn/utils/clip_grad.py
    #visdom & 

# vscode server part
RUN curl -sSL "https://update.code.visualstudio.com/commit:${vscommit}/server-linux-x64/stable" -o /home/${user}/${local_package}/vscode-server-linux-x64.tar.gz;\
    mkdir -p ~/.vscode-server/bin/${vscommit};\
    tar zxvf /home/${user}/${local_package}/vscode-server-linux-x64.tar.gz -C ~/.vscode-server/bin/${vscommit} --strip 1;\
    touch ~/.vscode-server/bin/${vscommit}/0

ADD id_ras.pub /home/${user}/.ssh/authorized_keys

ENTRYPOINT sudo service ssh restart && zsh
