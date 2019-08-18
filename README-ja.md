# Dockerized Amazon ECR credential helper

## Supported tags and respective `Dockerfile` links

・latest ([versions/0.3/Dockerfile](https://github.com/pottava/dockerized-ecr-credential-helper/blob/master/versions/0.3/Dockerfile))  
・0.3 ([versions/0.3/Dockerfile](https://github.com/pottava/dockerized-ecr-credential-helper/blob/master/versions/0.3/Dockerfile))  
・beta ([versions/beta/Dockerfile](https://github.com/pottava/dockerized-ecr-credential-helper/blob/master/versions/beta/Dockerfile))  

## インストール

### 1. ヘルパーの挙動を確認します

* Case 1: EC2 インスタンスロールを使う場合

```sh
$ docker run --rm \
  -e REGISTRY=123457689012.dkr.ecr.us-east-1.amazonaws.com \
  pottava/amazon-ecr-credential-helper:0.3
```

* Case 2: 環境変数を使う場合

```sh
$ docker run --rm \
  -e REGISTRY=123457689012.dkr.ecr.us-east-1.amazonaws.com \
  -e AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY \
  pottava/amazon-ecr-credential-helper:0.3
```

* Case 3: クレデンシャルファイルを使う場合

```sh
$ docker run --rm \
  -e REGISTRY=123457689012.dkr.ecr.us-east-1.amazonaws.com \
  -v $HOME/.aws/credentials:/root/.aws/credentials \
  pottava/amazon-ecr-credential-helper:0.3
```

### 2. $PATH の通っているフォルダにスクリプトを設置

* Case 1: EC2 インスタンスロールを使う場合

```sh
$ sudo sh -c 'cat << EOF > /usr/bin/docker-credential-ecr-login
#!/bin/sh
SECRET=\$(docker run --rm \\
  -e METHOD=\$1 \\
  -e REGISTRY=\$(cat -) \\
  pottava/amazon-ecr-credential-helper:0.3)
echo \$SECRET | grep Secret
EOF'
$ sudo chmod +x /usr/bin/docker-credential-ecr-login
```

* Case 2: 環境変数を使う場合

```sh
$ sudo sh -c 'cat << EOF > /usr/bin/docker-credential-ecr-login
#!/bin/sh
SECRET=\$(docker run --rm \\
  -e METHOD=\$1 \\
  -e REGISTRY=\$(cat -) \\
  -e AWS_ACCESS_KEY_ID \\
  -e AWS_SECRET_ACCESS_KEY \\
  pottava/amazon-ecr-credential-helper:0.3)
echo \$SECRET | grep Secret
EOF'
$ sudo chmod +x /usr/bin/docker-credential-ecr-login
```

* Case 3: クレデンシャルファイルを使う場合

```sh
$ sudo sh -c 'cat << EOF > /usr/bin/docker-credential-ecr-login
#!/bin/sh
SECRET=\$(docker run --rm \\
  -e METHOD=\$1 \\
  -e REGISTRY=\$(cat -) \\
  -v $HOME/.aws/credentials:/root/.aws/credentials \\
  pottava/amazon-ecr-credential-helper:0.3)
echo \$SECRET | grep Secret
EOF'
$ sudo chmod +x /usr/bin/docker-credential-ecr-login
```

### 3. ~/.docker/config.json に以下の値をセット

```json
{
    "credsStore": "ecr-login"
}
```

## 使い方

環境変数を使う場合は、そのセット。

```console
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

次のコマンドは使う必要はありません: `eval "$(aws ecr get-login)"`.  
docker push や pull が通常通り利用できます。

* `docker push 123457689012.dkr.ecr.us-east-1.amazonaws.com/my-repo:tag`
* `docker pull 123457689012.dkr.ecr.us-east-1.amazonaws.com/my-repo:tag`
