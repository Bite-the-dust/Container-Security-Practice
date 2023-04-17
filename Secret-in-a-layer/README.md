# レイヤーに含まれる機密情報
コンテナイメージにアクセスできるユーザーはイメージに含まれる全てのファイルにアクセスできる。

下記のようなDockerfileを作成してコンテナをビルドすると、
2,3行目の`RUN`は別々のレイヤーとして保存される。
そのため2行目で保存した資格情報を含むファイルを、3行目で削除したとしても復元する事ができる。
```dockerfile
FROM alpine
RUN cat /dev/urandom | tr -dc "[:alnum:]" | fold -w 32 | head -1 > /flag.txt
RUN rm /flag.txt
```


1. 既にイメージが存在している場合を想定
```bash
parrot@parrot-virtualbox:~/Desktop
➤ docker images                                                                                                                                                                          15:40
REPOSITORY                                                   TAG                 IMAGE ID            CREATED             SIZE
flag                                                         latest              c44c0026e0e0        About an hour ago   7.04MB
```

2.  イメージをファイルに保存する
```bash
parrot@parrot-virtualbox:~/Desktop
➤ docker save flag > flag.tar
```

3. 保存したファイルをアンアーカイブする
```bash
parrot@parrot-virtualbox:~/D/flag
➤ tar xvf flag.tar 
-----省略-----

parrot@parrot-virtualbox:~/D/flag
➤ ls                                                                                                                                                                                     15:45
合計 12K
drwxr-xr-x 1 parrot parrot  154  4月 17 14:44 1cde3294bbc3fec873fb3d659e85b4fed4d0c04644cc83a70c486637ebac305b
drwxr-xr-x 1 parrot parrot   64  4月 17 14:44 b5fceebef2dae51c283e588577f6c2119e84c2d6c41e8cf077e647c02ebbc08a
-rw-r--r-- 1 parrot parrot 1.8K  4月 17 14:40 c44c0026e0e0b255037707322555c7b74cb1f0c4ce6612e1927a056ddce5ee75.json
drwxr-xr-x 1 parrot parrot   64  4月 17 14:55 f03ac96abf0521a358114f08a5ed536ff606e618cda0cdc4657f7d927089b0ae
-rw-r--r-- 1 parrot parrot  354  1月  1  1970 manifest.json
-rw-r--r-- 1 parrot parrot   87  1月  1  1970 repositories
```

3. c44c*.jsonはイメージの設定ファイルでありレイヤー情報が含まれていることが確認できる
```bash
parrot@parrot-virtualbox:~/D/flag
➤ cat c44c0026e0e0b255037707322555c7b74cb1f0c4ce6612e1927a056ddce5ee75.json | jq                                                                                                         15:48
{
  "architecture": "amd64",
  "config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh"
    ],
    "Image": "sha256:49151f6503369bfc35487e19c25269ef3b4d4817ba62890845dccae94cdc7aa1",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "container": "0a0862c2ef436e363e0c22f1cd0bf8732f88bfb3d7d79c0b85f63cda1d265c02",
  "container_config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh",
      "-c",
      "rm /flag.txt"
    ],
    "Image": "sha256:49151f6503369bfc35487e19c25269ef3b4d4817ba62890845dccae94cdc7aa1",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "created": "2023-04-17T05:40:51.558063801Z",
  "docker_version": "19.03.15",
  "history": [
    {
      "created": "2023-03-29T18:19:24.348438709Z",
      "created_by": "/bin/sh -c #(nop) ADD file:9a4f77dfaba7fd2aa78186e4ef0e7486ad55101cefc1fabbc1b385601bb38920 in / "
    },
    {
      "created": "2023-03-29T18:19:24.45578926Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/sh\"]",
      "empty_layer": true
    },
    {
      "created": "2023-04-17T05:40:50.064150902Z",
      "created_by": "/bin/sh -c cat /dev/urandom | tr -dc \"[:alnum:]\" | fold -w 32 | head -1 > /flag.txt"
    },
    {
      "created": "2023-04-17T05:40:51.558063801Z",
      "created_by": "/bin/sh -c rm /flag.txt"
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:f1417ff83b319fbdae6dd9cd6d8c9c88002dcd75ecf6ec201c8c6894681cf2b5",
      "sha256:a53fef7d9740367736dcd71c77ef287778064361d43ef4433ac00406c6c7be2a",
      "sha256:ab2e02d3a5cf412c723382a94ed5169fec1d35b921bc528eba26b3a0fd39cadd"
    ]
  }
}
```

4. manifest.jsonには各レイヤーのファイルシステムをアーカイブしたファイルのパスが記載される。
```bash
parrot@parrot-virtualbox:~/D/flag
➤ cat manifest.json | jq                                                                                                                                                                 15:48
[
  {
    "Config": "c44c0026e0e0b255037707322555c7b74cb1f0c4ce6612e1927a056ddce5ee75.json",
    "RepoTags": [
      "flag:latest"
    ],
    "Layers": [
      "1cde3294bbc3fec873fb3d659e85b4fed4d0c04644cc83a70c486637ebac305b/layer.tar",
      "f03ac96abf0521a358114f08a5ed536ff606e618cda0cdc4657f7d927089b0ae/layer.tar",
      "b5fceebef2dae51c283e588577f6c2119e84c2d6c41e8cf077e647c02ebbc08a/layer.tar"
    ]
  }
]
```

5. これらのアーカイブファイルから該当の機密情報が含まれるファイル(今回は`flag.txt`)を検索し抽出すると内容を取得する事ができる
```bash
parrot@parrot-virtualbox:~/D/flag
➤ find ./ -name layer.tar | xargs -I ARG tar xOvf ARG flag.txt 2>/dev/null                                                                                                               16:18
CyFkykk2oZJB2pL9dTRM8qbYBANLV5Ze
```



