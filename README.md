# nmap-ci
Running Nmap and other tools in a docker from Concourse CI.


## Usage

Currently experimenting with starting Nmap from a docker and creating a pipeline to run.

Don't forget to do login first.

### Run build

The run.sh performs a simple command:
fly -t ci execute -c ci/check-ssh-auth.yml

```
root@nordukali:~/projects/nmap-ci# sh run.sh
executing build 87 at http://10.80.0.194:8080/builds/87
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2801    0  2801    0     0   2801      0 --:--:-- --:--:-- --:--:--  547k
initializing
running ./nmap-ci/ci/test2.sh
+ pushd nmap-ci
/tmp/build/e55deab7/nmap-ci /tmp/build/e55deab7
+ echo SSH algorithms
SSH algorithms
+ nmap -A -p 22 --script ssh-auth-methods.nse 10.80.0.194

Starting Nmap 7.60 ( https://nmap.org ) at 2018-02-22 09:37 UTC
Nmap scan report for 10.80.0.194
Host is up (0.00078s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-auth-methods:
|   Supported authentication methods:
|_    publickey
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.8, Linux 3.2 - 4.8
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT     ADDRESS
1   0.06 ms 10.254.0.13
2   0.70 ms 10.80.0.194

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
+ popd
/tmp/build/e55deab7
succeeded
```

### Adding as a pipeline

The run2.sh performs a simple command:
fly -t ci set-pipeline -p nmap-ci -c ci/pipeline.yml


```
root@nordukali:~/projects/nmap-ci# sh run2.sh
resources:
  resource nmap-ci has been added:
    name: nmap-ci
    type: git
    source:
      branch: master
      uri: https://github.com/kramse/nmap-ci

jobs:
  job test-app has been added:
    name: test-app
    plan:
    - get: nmap-ci
      trigger: true
    - task: tests
      file: nmap-ci/build.yml

apply configuration? [yN]:
pipeline created!
you can view your pipeline here: http://10.80.0.194:8080/teams/main/pipelines/nmap-ci

the pipeline is currently paused. to unpause, either:
  - run the unpause-pipeline command
  - click play next to the pipeline in the web ui
```

and you can then unpause it:
```
root@nordukali:~/projects/nmap-ci# fly -t ci unpause-pipeline -p nmap-ci
unpaused 'nmap-ci'
```


# Problems observed with Concourse
