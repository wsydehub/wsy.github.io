---
layout:     post
title:      "Unix下使用Socket通信"
subtitle:   ""
date:       2019-10-17 13:23:00
author:     "Dsyzxml"
header-img: "img/in-post/PostBackground/ab.jpeg"
catalog: true
tags:
    - C/C++
    - Socket
    - Linux
---

最近要实现一个简单的分布式系统,打算使用socket来实现节点间的通信.在学习了一下之后,做个笔记.

> **Server**

```
void *server(void *path)
{
    string *yamlPath = (string *)path;
    YAML::Node node = YAML::LoadFile(*(yamlPath));
    string ip = node["server"]["ip"].as<string>();
    int port = node["server"]["port"].as<int>();
    printf("server: %s:%d\n", ip.c_str(), port);

    int sock_fd, client_fd;
    socklen_t sin_size;
    struct sockaddr_in server_addr, client_addr;

    //create socket
    if ((sock_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("creation failed\n");
        exit(1);
    }

    //memory set zeros
    bzero(&server_addr, sizeof(server_addr));
    //set socket family, AF_INET means "internetwork: UDP, TCP, etc."
    server_addr.sin_family = AF_INET;
    //
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(port);

    //binding a file description with a socket
    if (::bind(sock_fd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) == -1)
    {
        perror("bind error\n");
        exit(1);
    }

    //open the socket to listen
    if (listen(sock_fd, 10) == -1)
    {
        perror("listen error\n");
        exit(1);
    }

    while (true)
    {
        sin_size = sizeof(struct sockaddr_in);
        //accept connection
        if ((client_fd = accept(sock_fd, (struct sockaddr *)&client_addr, &sin_size)) == -1)
        {
            perror("server accept error\n");
            continue;
        }

        cout << client_fd << endl;

        printf("recived a connection from %s:%d\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

        if (send(client_fd, "Hello, you are connected\n", 26, 0) == -1)
        {
            perror("send error\n");
        }
        close(sock_fd);
        exit(0);
    }

    return 0;
}
```

> **Client**

```
int client(void *yamlFile)
{
    printf("client start conneting......");
    int sock_fd;
    long reciveBytes;
    char buff[100];
    struct hostent *host;
    struct sockaddr_in server_addr, my_addr;

    YAML::Node node = YAML::LoadFile(*((string *)yamlFile));
    string name = node["server"]["name"].as<string>();
    int port = node["server"]["port"].as<int>();

    if ((host = gethostbyname(name.c_str())) == nullptr)
    {
        herror("gethostbyname error\n");
        exit(1);
    }

    if ((sock_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("socket creation error\n");
        exit(1);
    }

    bzero(&server_addr, sizeof(struct sockaddr_in));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(port);
    bcopy((char *)host->h_addr_list[0], (char *)&server_addr.sin_addr.s_addr, host->h_length);

    if (connect(sock_fd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) == -1)
    {
        perror("connect error\n");
        exit(1);
    }
    if (recv(sock_fd, buff, 100, 0) == -1)
    {
        perror("recv error\n");
        exit(1);
    }
    printf("Recived: %s", buff);

    return 0;
}
```


> **main**
```
int main(int argc, char **argv)
{

    string yamlPath = "test.yaml";

    pthread_t thread_id;

    pthread_create(&thread_id, 0, &server, &yamlPath);
    client(&yamlPath);

    pthread_exit(0);
    return 0;
}
```