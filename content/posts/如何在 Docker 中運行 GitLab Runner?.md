---
title: "如何在 Docker 中運行 GitLab Runner?"
date: 2021-10-25T12:38:23+08:00
categories:
- "技術"
Tags: 
- "docker"
- "gitlab"
draft: false
---

## 前言
---

一般來說，我們可以在使用本地端安裝的 GitLab Runner 來作為專案 CI/CD 的機器。除此之外，GitLab 亦提供 GitLab Runner 的 [Docker images](https://docs.gitlab.com/runner/install/docker.html#docker-images) [[1](https://www.notion.so/Docker-GitLab-Runner-54d82b53020e4ea3887363992cc8cd63)]，讓我們在 Docker 的 container 中運行 GitLab Runner。

本篇文章將參考[官方文件](https://docs.gitlab.com/runner/install/docker.html) [[2](https://www.notion.so/Docker-GitLab-Runner-54d82b53020e4ea3887363992cc8cd63)] 來進行介紹，先從 docker 的安裝介紹，再逐步建立 Docker 容器，並運行 GitLab Runner，希望能幫自己做個紀錄，也能幫助正在尋求解答的人。
<!--more-->

## 安裝 Docker

使用套件管理系統安裝

- macOS
    
    ```bash
    $ brew casks install docker
    ```
    
- Linux (Ubuntu) [[3](https://www.notion.so/Docker-GitLab-Runner-54d82b53020e4ea3887363992cc8cd63)]
    - 如果你是第一次在電腦上安裝 Docker，需要先設定 Docker 的 repository
        1. 更新 `apt`，並同意 `apt` 在 HTTPS 上使用 repository
            
            ```bash
            $ sudo apt-get update
            $ sudo apt-get install \
                    ca-certificates \
                    curl \
                    gnupg \
                    lsb-release
            ```
            
        2. 添加 Docker 官方的 GPG key
            
            ```bash
            $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyring.gpg
            ```
            
        3. 透過以下的命令來設定 **stable** 的 repository。你也可以將 **stable** 改成 **nightly** 或是 **test** 的 repository，來使用不同版本的 repository
            
            ```bash
            $ echo \
                "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
                $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
            ```
            
    - 設定完 repository，接著來安裝 Docker 引擎
        1. 更新 `apt` package 的條目，以確保 package 皆在最新的條目上，並安裝 latest 版的 Docker 引擎 和 containerd
            
            ```bash
            $ sudo apt-get update
            $ sudo apt-get install docker-ce docker-ce-cli containerd.io
            ```
            
        2. 運行 hello-world 的映像檔(image)，確認是否安裝完成
            
            ```bash
            $ sudo docker run hello-world
            ```
            
    - 解除安裝 Docker 引擎
        1. 解除安裝 Docker Engine, CLI, Containerd packages
            
            ```bash
            $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
            ```
            
        2. 電腦上的 Images, Containers 和 Volumes 及設定檔並不會被自動移除，要刪除以上檔案需要自己手動移除他們
            
            ```bash
            $ sudo rm -rf /var/lib/docker
            $ sudo rm -rf /var/lib/containerd
            ```
            

## 在 container 中運行 GitLab Runner

我們可以透過 `docker run` 來一步完成建立 (`docker create`) 以及開啟 (`docker run`) 的步驟。在下面我將介紹兩種方法，來確保在 Docker container 中運行 `gitlab-runner` 時的設定檔不會在 container 重新運行時遺失。

- 選項一：使用本地端的系統 volume 掛載來啟動 Runner 容器
    
    ```bash
    $ docker run -d --name gitlab-runner --restart always \
            -v /srv/gitlab-runner/config:/etc/gitlab-runner \
            -v /var/run/docker.sock:/var/run/docker.sock \
            gitlab/gitlab-runner:latest
    ```
    
- 選項二：使用 Docker volumes 來啟動 Runner 容器
    1. 建立 Docker volume
        
        ```bash
        $ docker volume create gitlab-runner-config
        ```
        
    2. 使用步驟一建立的 volume 來啟動 GitLab Runner 容器
        
        ```bash
        $ docker run -d --name gitlab-runner --restart always \
                -v /var/run/docker.sock:/var/run/docker.sock \
                -v gitlab-runner-config:/etc/gitlab-runner \
                gitlab/gitlab-runner:latest
        ```
        

透過以上的兩個方法（二擇一），我們就可以成功的在 Docker 上建立並運行 `gitlab-runner` 了！

## 其他注意事項

- 更新 container
    
    當今天我們修改了 `config.toml` 的檔案後，若想更新 gitlab-runner 的 container，可以透過以下的指令來完成。
    
    ```bash
    $ docker restart gitlab-runner
    ```
    
    <aside>
    📢 `config.toml` 檔案通常可以在先前設定保存 `/etc/gitlab-runner` 運行資料的位置找到。以選項一保存於本地端的情形來說，我們可以在 `/srv/gitlab-runner/config/` 這個資料夾中找到它
    
    </aside>
    
- 讀取 GitLab Runner 的紀錄黨
    
    ```bash
    $ docker logs [container name]
    ```
    

## Reference

1. [https://docs.gitlab.com/runner/install/docker.html#docker-images](https://docs.gitlab.com/runner/install/docker.html#docker-images) 
2. [Run GitLab Runner in a container | GitLab](https://docs.gitlab.com/runner/install/docker.html)
3. [Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)
