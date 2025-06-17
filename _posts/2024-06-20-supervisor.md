---
layout: post
title: supervisor.md
categories: [supervisor]
description: supervisor
keywords: supervisor
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
```conf
[program:admin]
user=root
directory=/home/common-admin/app/
command=/home/common-admin/app/server.admin -f /home/common-admin/app/etc/admin_invitevisitor-location_test.yaml
autostart=true
autorestart=true
startsecs=1

[program:gateway]
user=root
directory=/home/common-gateway/app/
command=/home/common-gateway/app/server.gateway -f /home/common-gateway/app/etc/gateway-test.yaml
autostart=true
autorestart=true
startsecs=1

[program:yt-model]
user=root
directory=/home/common-model-server/app/
command=/home/common-model-server/app/server.model -mode test
autostart=true
autorestart=true
startsecs=1

[program:invitevisitor]
user=root
directory=/home/common-invitevisitor/app/
command=/home/common-invitevisitor/app/server.invitevisitor -f /home/common-invitevisitor/app/etc/invitevisitor-test.yaml
autostart=true
autorestart=true
startsecs=1

[program:etcd]
user=root
directory=/etc/etcd/etcd-v3.5.13-linux-amd64/
command=/etc/etcd/etcd-v3.5.13-linux-amd64/etcd
autostart=true
autorestart=true
startsecs=1

[program:pd]
user=root
directory=/home/pd-backend-Pd.WebApi-test/out
command=dotnet /home/pd-backend-Pd.WebApi-test/out/Pd.WebApi.dll -c appsettings.Test.json
autostart=true
autorestart=true
startsecs=1
stdout_logfile=/home/pd-backend-Pd.WebApi-test/out/Pd.WebApi.stdout.log
stderr_logfile=/home/pd-backend-Pd.WebApi-test/out/Pd.WebApi.stderr.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
stderr_logfile_maxbytes=10MB
stderr_logfile_backups=5

[program:jndtDataCollection]
user=root
directory=/home/jndt-backend-DataCollection-test/out
command=dotnet /home/jndt-backend-DataCollection-test/out/D.DataCollectionService.dll  -c appsettings.Test.json
autostart=true
autorestart=true
startsecs=1
stdout_logfile=/home/jndt-backend-DataCollection-test/out/D.DataCollectionService.stdout.log
stderr_logfile=/home/jndt-backend-DataCollection-test/out/D.DataCollectionService.stderr.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
stderr_logfile_maxbytes=10MB
stderr_logfile_backups=5

[program:jndtWebDataApi]
user=root
directory=/home/jndt-backend-Web.DataApi-test/out
command=dotnet /home/jndt-backend-Web.DataApi-test/out/YT.Web.DataApi.dll -c appsettings.Test.json
autostart=true
autorestart=true
startsecs=1
stdout_logfile=/home/jndt-backend-Web.DataApi-test/out/YT.Web.DataApi.stdout.log
stderr_logfile=/home/jndt-backend-Web.DataApi-test/out/YT.Web.DataApi.stderr.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
stderr_logfile_maxbytes=10MB
stderr_logfile_backups=5

[program:jndtWebApi]
user=root
directory=/home/jndt-backend-Web.API-test/out
command=dotnet /home/jndt-backend-Web.API-test/out/YT.Web.API.dll -c appsettings.Test.json
autostart=true
autorestart=true
startsecs=1
stdout_logfile=/home/jndt-backend-Web.API-test/out/YT.Web.API.stdout.log
stderr_logfile=/home/jndt-backend-Web.API-test/out/YT.Web.API.stderr.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
stderr_logfile_maxbytes=10MB
stderr_logfile_backups=5

[program:jndtWeb]
user=root
directory=/home/jndt-backend-Web-test/out
command=dotnet /home/jndt-backend-Web-test/out/YT.Web.dll -c appsettings.Test.json
autostart=true
autorestart=true
startsecs=1
stdout_logfile=/home/jndt-backend-Web-test/out/YT.Web.stdout.log
stderr_logfile=/home/jndt-backend-Web-test/out/YT.Web.stderr.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
stderr_logfile_maxbytes=10MB
stderr_logfile_backups=5
```
