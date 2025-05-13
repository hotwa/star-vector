FROM nvidia/cuda:12.4.1-devel-ubuntu22.04
ENV DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]
# base tools
RUN <<EOT
#!/bin/bash
apt-get update
apt-get install -y libgl1-mesa-glx bash-completion wget curl htop jq vim bash libaio-dev build-essential openssh-server openssh-client python3 python3-pip python3-venv bzip2
apt-get install -y --no-install-recommends software-properties-common build-essential autotools-dev nfs-common pdsh cmake g++ gcc curl wget vim tmux emacs less unzip htop iftop iotop ca-certificates openssh-client openssh-server rsync iputils-ping net-tools sudo llvm-dev re2c
add-apt-repository ppa:git-core/ppa -y
apt-get install -y git libnuma-dev wget
pip install pipx
pipx install nvitop
pipx ensurepath 
. ~/.bashrc
mkdir ~/.ssh
printf "Host * \n    ForwardAgent yes\nHost *\n    StrictHostKeyChecking no" > ~/.ssh/config
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/^\(\s*\)GSSAPIAuthentication yes/\1GSSAPIAuthentication no/' /etc/ssh/ssh_config
sed -i "s/^#Port 22/Port ${SSH_PORT}/" /etc/ssh/sshd_config
sudo sed -i "s/#   Port 22/Port ${SSH_PORT}/" /etc/ssh/ssh_config
ssh-keygen -t rsa -b 4096 -f /root/.ssh/id_rsa -N "" <<< y
cat ~/.ssh/id_rsa.pub >> ~/.ssh/auth
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys2
chmod 600 /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys2
mkdir /var/run/sshd
echo "root:${ROOT_PASSWD}" | chpasswd
mkdir -p ~/.pip
wget -qO- https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh
bash /tmp/miniconda.sh -b -p /opt/conda 
rm /tmp/miniconda.sh 
ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh
echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc 
. /opt/conda/etc/profile.d/conda.sh 
conda init bash
conda config --set show_channel_urls true
cat <<EOF > ~/.condarc
channels:
  - conda-forge
  - bioconda
  - pytorch
  - pytorch-nightly
  - nvidia
  - defaults
show_channel_urls: true
EOF
EOT

RUN <<EOT
#!/bin/bash
git clone https://github.com/joanrod/star-vector.git /tmp/star-vector 
cd  /tmp/star-vector 
. /opt/conda/etc/profile.d/conda.sh
conda create -n starvector python=3.11.3 ninja cmake -y
conda run -n starvector pip install --upgrade pip  # enable PEP 660 support
conda run -n starvector pip install torch==2.5.1+cu124 torchvision==0.20.1+cu124 --index-url https://download.pytorch.org/whl/cu124
MAX_JOBS=12 pip install flash_attn==2.7.3 --no-build-isolation
conda run -n starvector pip install -e .
conda run -n starvector pip install jupyter deepspeed
rm -rf /tmp/star-vector
EOT

WORKDIR /workspace
EXPOSE 8888
CMD ["conda", "run", "-n", "starvector", "jupyter","notebook","--ip=0.0.0.0","--port=8888","--no-browser","--allow-root"]

