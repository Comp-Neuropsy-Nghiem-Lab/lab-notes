## Intro
- 2 HPCs
	- Galvani cluster: 2080ti & A100 GPU
	- Ferranti cluster: H100 GPU
- 2 kinds of nodes
	- login nodes (✓: type & code, manage, sbatch & srum. ✕: conda, run)
	- compute nodes

## Login
### Login Process ([details](https://portal.mlcloud.uni-tuebingen.de/user-guide/accounts/account_access/))
Ask your PI or Group manager to submit a request
1. Receive a **welcome email** (including your **username**, like `pbb111`)
2. Login (a **two-day** period to deploy your key)
	1. Create your **SSH keys** (generate a pair of public and private keys)
	2. Upload your public key to the login nodes and place it correctly

> [!HINT] If you miss the period
> Send them a [support ticket](support@mlcloud.uni-tuebingen.de) with your public key attached, and they will deploy it for you.

*Note*:
Two clusters have two login nodes
```sh
ssh {Username}@{Clustername}-login.mlcloud.uni-tuebingen.de
# E.g., ssh pbb111@galvani-login.mlcloud.uni-tuebingen.de
# E.g., ssh pbb111@galvani-login2.mlcloud.uni-tuebingen.de
```


## Download
### Software download procedure
1. Hold personal `bin` (System has fitted this path name)
```sh
makdir -p ~/.local/bin
```
2. Download in the temporary folder `/tmp`
```sh
cd /tmp  # move to the temporary folder
wget {website link}  # download from this link
# E.g. wget https://github.com/sxyazi/yazi/releases/latest/download/yazi-x86_64-unknown-linux-musl.zip
unzip {download}
# E.g., unzip yazi-x86_64-unknown-linux-musl.zip
```
3. Move this to the personal `bin`
```sh
mv {file & foler} {folder}
# E.g. mv yazi-x86_64-unknown-linux-musl/yazi ~/.local/bin/
```

## Conda

### New Env building procedure
1. Configure
```sh
nano ~/.condarc
```
And then write these in `condarc`
```yaml
pkgs_dirs:
	- $WORK/conda/pkgs
channel:
	- conda-forge
```
Save: `Ctrl + O` -> `enter`
Quit: `Ctrl + X`

2. Shift to another compute node
```sh
srum --pty bash
# E.g., srun --pty --partition=2080-galvani bash
```
3. Create and activate conda env (MUST use path, not name)
```sh
conda create -p $WORK/.conda/NAME python=3.11
conda activate $WORK/.conda/NAME
```

> [!danger] Where to conda
> Only create and activate env in the **Compute Nodes** and **$WORK path**, not your home folder (because conda envs are usually big).


## VS Code
*Note*:
- SLURM job ✕: Only connect to **Login Nodes** through VS Code
- SLURM job ✓: Can connect to **Compute Nodes** through VS Code
After connecting to VS Code, you can write and run your code in the cluster through it. 

### Connecting procedure
1. Create SSH keys (**Login Nodes**)
```sh
ssh-keygen -b 521 -t ECDSA -C "$USER@$(hostname)"

# After running this, you will get:
# ~/.ssh/id_ecdsa
# ~/.ssh/id_ecdsa.pub
```

2. Choose a port (Pick a number between 7000 and 8000 randomly)
3. Write your port number in your [[#SLURM script]] (**Login Nodes**)
```bash
/usr/sbin/sshd -D -p {port number} -f /dev/null -h ${HOME}/.ssh/id_ecdsa
```
4. Change local config (**Local**)
```sh
nano ~/.ssh/config
```
Add this at the end of your local SSH config
```ssh
Host galvani-vs
  User {username}
  ProxyCommand ssh -A galvani "nc \$(squeue --me --name=vs --states=R -h -O NodeList) {port number}"
  StrictHostKeyChecking no
```
5. Submit job (**Login nodes**)
Submit SLURM script
```sh
sbatch $WORK/vscode/allocate-galvani-vs.sh
```
Check state
```sh
squeue --me
```
If you see your job state is 'R', meaning running, you succeed and can build the second tunnel from your local to compute nodes

6. Build the second tunnel (**Local**)
```sh
ssh galvani-vs
```
7. Use VS Code to connect
	1. Open VS Code
	2. Remote Explorer → SSH
	3. Click `galvani-vs`
So you can also use Terminal in VS Code

## SLURM script
Give an example:
```bash
#!/bin/bash
#SBATCH --job-name=vs                 # job name
#SBATCH --partition=2080-galvani.     # GPU/CPU partition
#SBATCH --time=01:00:00               # running time
#SBATCH --nodes=1                     # one node
#SBATCH --ntasks=1                    # one task
#SBATCH --mem=8G                      # memory
#SBATCH --output=%x-%j.out            # stdout log e.g. vs-2336123.out
#SBATCH --error=%x-%j.err             # stderr log e.g. vs-2336123.err

PORTNUMBER=7317
/usr/sbin/sshd -D -p $PORTNUMBER -f /dev/null -h ${HOME}/.ssh/id_ecdsa
```


## Command cheatsheet
| Cmd       | Meaning          | E.g.                                                                                                                 | Cmd      | Meaning            | E.g.                                           |
| --------- | ---------------- | -------------------------------------------------------------------------------------------------------------------- | -------- | ------------------ | ---------------------------------------------- |
| `echo`    | Print            | echo $USER  # print user folder path<br>echo $WORK  # print work folder path<br>echo $HOME  # print home folder path | `cd`     | Go to another path | cd ~  # go to HOME<br>cd ..  # go up one level |
| `ls`      | List files       | ls -a. # include hidden files<br>ls -lh  # Human-readable sizes                                                      | `mkdir`  | Create folder      | mkdir -p work/vscode  # create nested dirs     |
| `rm`      | Delete           | re -r old_run  # delete directory                                                                                    | `mv`     | Move/Rename        | mv old.py new.py  # rename                     |
| `sbatch`  | Submit batch job | sbatch run.sh                                                                                                        | `squeue` | Job queue          | squeue --me  # view only my jobs               |
| `scancel` | Cancel job       | scancel 2336123                                                                                                      | `sinfo`  | Partitions/nodes   |                                                |
| `top`     | CPU              | top  # CPU usage<br>htop  # Better CPU monitor                                                                       | `nvidia` | GPU                | nvidia-smi  # GPU status                       |
| `free`    | Memory           | free -h  # memory usage                                                                                              | `df`     | DIsk               | df -h  # disk usage                            |
