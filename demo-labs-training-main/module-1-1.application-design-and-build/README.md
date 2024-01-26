# Docker
### install docker
You can refers this link for latest version [docker on ubuntu](https://docs.docker.com/engine/install/ubuntu/)<br>
for development and testing environtment you can use convenience script
``` 
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

### Containerize an application
Download the image to local
```
docker pull ubuntu 
``` 

Running Container
```
docker run -it ubuntu
```

Rename container
```
docker rename old-name ct-ubuntu
```

Start Container
```
docker start ct-ubuntu
```

Execute command on container
```
docker exec -it ct-ubuntu bash
```

Backup container to image
```
docker commit -p ct-ubuntu ubuntu-nginx
```

Running the container on port 80
```
docker run -dit -p 80:80 --name ct-ubuntu-nginx ubuntu-nginx
``` 

Check container status
```
docker ps 
``` 

### Update the application
Execution shell on container
```
docker exec -it ct-ubuntu-nginx bash
```

Copy custom content to container
```
docker cp index.html ct-ubuntu-nginx:/var/www/html
``` 

### Build your own application
Create directory
```
mkdir my-image
cd my-image
mkdir apps
```

create index file
```
vim apps/index.html
```

add this
```
<title>Training Kubernetes</title>
<h1>Belajar Kubernetes Hari 1</h1>
```

Create Dockerfile
```
vim Dockerfile 
```

Build image
```
docker build -t image-nama .
``` 

Check the image
```
docker images 
```

Running a own image
```
docker run -d -p 8080:80 --name ct-apps-nama image-nama
``` 

### Share the application
Create account dockerhub, you can refers this link [Dockerhub](https://hub.docker.com/)

Login dockerhub account on docker
```
docker login 
```
Check info docker 
```
docker info 
``` 
Tag name image with repository name on dockerhub 
```
docker tag old-name-image repository/new-name-image
``` 
Push image to dockerhub 
```
docker push repository/new-name-image
``` 

### Share the application on Private Registry

Configure private registry if not https
```
vim /etc/docker/daemon.json
```

edit
```
{
    "insecure-registries":[
        "localhost:5000"
    ]
}
```

restart daemon
```
systemctl daemon-reload
```

Login private registry
```
docker login localhost:5000
```

Check info docker 
```
docker info 
``` 

Tag name image with repository name on private registry 
```
docker tag old-name-image localhost:5000/new-name-image
``` 
Push image to private registry 
```
docker push localhost:5000/new-name-image
``` 