## TockOwl

![build status](https://img.shields.io/github/actions/workflow/status/asonnino/hotstuff/rust.yml?style=flat-square&logo=GitHub&logoColor=white&link=https%3A%2F%2Fgithub.com%2Fasonnino%2Fhotstuff%2Factions)
[![golang](https://img.shields.io/badge/golang-1.21.1-blue?style=flat-square&logo=golang)](https://golang.google.cn/doc/go1.21)
[![python](https://img.shields.io/badge/python-3.10-blue?style=flat-square&logo=python&logoColor=white)](https://www.python.org/downloads/release/python-31012/)
[![license](https://img.shields.io/badge/license-Apache-blue.svg?style=flat-square)](LICENSE)

This repo provides a minimal implementation of the TockOwl protocol. The codebase has been designed to be small, efficient, and easy to benchmark and modify. It has not been designed to run in production but uses real cryptography (kyber), networking(native), and storage (nutsdb).

Say something about TockOwl...

## Quick Start

TockOwl is written in Golang, but all benchmarking scripts are written in Python and run with Fabric. To deploy and benchmark a testbed of 4 nodes on your local machine, clone the repo and install the python dependencies:

```shell
git clone https://github.com/kaly20021110/TockOwl.git
cd TockOwl/benchmark
pip install -r requirements.txt
```

You also need to install tmux (which runs all nodes and clients in the background).
Finally, run a local benchmark using fabric:

```shell
fab local
```

This command may take a long time the first time you run it (compiling golang code in release mode may be slow) and you can customize a number of benchmark parameters in fabfile.py. When the benchmark terminates, it displays a summary of the execution similarly to the one below.

- [TockOwl]
```
Starting local benchmark
Setting up testbed...
Running TockOwl
0 byzantine nodes
tx_size 256 byte, batch_size 512, rate 5000 tx/s
DDOS attack False
Waiting for the nodes to synchronize...
Running benchmark (30 sec)...
Parsing logs...

-----------------------------------------
 SUMMARY:
-----------------------------------------
 + CONFIG:
 Protocol: TockOwl 
 DDOS attack: False 
 Committee size: 4 nodes
 Input rate: 5,000 tx/s
 Transaction size: 256 B
 Batch size: 512 tx/Batch
 Faults: 0 nodes
 Execution time: 25 s

 + RESULTS:
 Consensus TPS: 20,236 tx/s
 Consensus latency: 76 ms

 End-to-end TPS: 20,323 tx/s
 End-to-end latency: 147 ms
 The epoch count can not commit block: 55
 The all epoch counts : 315
 The all epoch count commit block: 973
-----------------------------------------
```

## Aliyun Benchmarks
The following steps will explain that how to run benchmarks on Alibaba cloud across multiple data centers (WAN).

**1. Set up your Aliyun credentials**

Set up your Aliyun credentials to enable programmatic access to your account from your local machine. These credentials will authorize your machine to create, delete, and edit instances on your Aliyun account programmatically. First of all, [find your 'access key id' and 'secret access key'](https://help.aliyun.com/document_detail/268244.html). Then, create a file `~/.aliyun/access.json` with the following content:

```json
{
    "AccessKey ID": "your accessKey ID",
    "AccessKey Secret": "your accessKey Secret"
}
```

**2.  Add your SSH public key to your Aliyun account**

You must now [add your SSH public key to your Aliyun account](https://www.alibabacloud.com/help/en/yunxiao/user-guide/configure-ssh-key). This operation is manual and needs to be repeated for each Aliyun region that you plan to use. Upon importing your key, Aliyun requires you to choose a 'name' for your key; ensure you set the same name on all Aliyun regions. This SSH key will be used by the python scripts to execute commands and upload/download files to your Aliyun instances. If you don't have an SSH key, you can create one using [ssh-keygen](https://www.ssh.com/ssh/keygen/):

```
ssh-keygen -f ~/.ssh/Aliyun
```

**3. Configure the testbed**

The file [settings.json](https://github.com/kaly20021110/TockOwl/blob/main/benchmark/settings.json) located in [TockOwl/benchmark](https://github.com/kaly20021110/TockOwl/tree/main/benchmark) contains all the configuration parameters of the testbed to deploy. Its content looks as follows:

```json
{
    "key": {
        "name": "TockOwl",
        "path": "/root/.ssh/id_rsa",
        "accesskey": "/root/.aliyun/access.json"
    },
    "ports": {
        "consensus": 8000,
        "mempool":7000
    },
    "instances": {
        "type": "ecs.g6e.xlarge",
        "regions": [
            "eu-central-1",
            "ap-northeast-2",
            "ap-southeast-1",
            "us-east-1"
        ]
    }
}
```

The first block (`key`) contains information regarding your SSH key and Access Key:

```json
"key": {
    "name": "TockOwl",
    "path": "/root/.ssh/id_rsa",
    "accesskey": "/root/.aliyun/access.json"
}
```

The second block (`ports`) specifies the TCP ports to use:

```json
"ports": {
    "consensus": 8000,
    "mempool": 7000
}
```

The the last block (`instances`) specifies the[Aliyun Instance Type](https://help.aliyun.com/zh/ecs/user-guide/general-purpose-instance-families)and the [Aliyun regions](https://help.aliyun.com/zh/ecs/product-overview/regions-and-zones) to use:

```json
"instances": {
    "type": "ecs.g6e.xlarge",
    "regions": [
        "eu-central-1",
        "ap-northeast-2",
        "ap-southeast-1",
        "us-east-1"
    ]
}
```

**4. Create a testbed**

The Aliyun instances are orchestrated with [Fabric](http://www.fabfile.org/) from the file [fabfile.py](https://github.com/kaly20021110/TockOwl/blob/main/benchmark/fabfile.py) (located in [TockOwl/benchmark](https://github.com/kaly20021110/TockOwl/tree/main/benchmark)) you can list all possible commands as follows:

The command `fab create` creates new Aliyun instances; open [fabfile.py](https://github.com/kaly20021110/TockOwl/blob/main/benchmark/fabfile.py) and locate the `create` task:

```python
@task
def create(ctx, nodes=2):
    ...
```

The parameter `nodes` determines how many instances to create in *each* Aliyun region. That is, if you specified 4 Aliyun regions as in the example of step 3, setting `nodes=2` will creates a total of 8 machines:

```shell
$ fab create

Creating 8 instances |██████████████████████████████| 100.0% 
Waiting for all instances to boot...
Successfully created 8 new instances
```

You can then install goland on the remote instances with `fab install`:

```shell
$ fab install

Installing rust and cloning the repo...
Initialized testbed of 8 nodes
```

Next,you should upload the executable file

```shell
$ fab uploadexec
```

**5. Run a benchmark**

```shell
$ fab remote
```

## License
This software is licensed as [Apache 2.0](https://github.com/kaly20021110/TockOwl/blob/main/LICENSE)